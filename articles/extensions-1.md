---
title: "未経験スタックだけで作る店舗メンテナンス管理システム ― 第三弾・第四弾との双方向API連携まで"
emoji: "🔧"
type: "tech"
topics: ["fastapi", "docker", "aws", "postgresql", "個人開発"]
published: false
---

## この記事でわかること

- FastAPI + PostgreSQL + Docker + AWS(EC2)という新スタックを、ゼロからどう組み合わせて動かしたか
- 既存システム(SQLite構成)との連携を、疎結合を保ったままAPIキー認証で安全に行う設計
- Docker/AWSの初学者がつまずきやすいポイントとその解決策

## 技術スタック

| レイヤー | 採用技術 | 選定理由 |
|---|---|---|
| API | FastAPI | 既存プロジェクトで使い慣れているため維持 |
| DB | PostgreSQL 16 | 実務でよく使われるRDBMSの経験を積むため |
| ORM/マイグレーション | SQLAlchemy + Alembic | モデル変更を安全に追跡するため |
| コンテナ | Docker Compose(db + web の2サービス) | ローカルと本番の環境差をなくすため |
| インフラ | AWS EC2(t3.micro)+ Docker Compose | まずはシンプルな構成で「動かす」ことを優先 |
| 認証 | APIキー(X-API-Keyヘッダー) | 書き込み系エンドポイントのみ保護する軽量認証 |

あえてKubernetesやTerraformは今回は採用していません。学習範囲を絞り、一つずつ確実に理解することを優先した結果です。

## テーブル設計

新設したのは2テーブルのみです。

```
rental_items(貸出用品)
  item_type / size_or_weight / status / acquired_at

maintenance_log(整備記録・共通)
  target_type / target_id / maintenance_type / maintenance_category
  / status / repair_method / vendor_name / performed_by / cost
  / action_date / note
```

maintenance_logを「対象種別(target_type)+対象ID(target_id)」で汎用化したのがポイントです。これにより、拡張①の中の設備(lanes)にも、外部システム(第四弾の会員ギア)にも、同じテーブルで整備記録を残せる設計になっています。

rental_itemsの削除は物理削除ではなく論理削除(statusを「廃棄」に変更)。廃棄履歴を追えるようにするための設計判断です。

## 開発ステップ

- Step1: Docker ComposeでFastAPI+PostgreSQLをローカル構築
- Step2: SQLAlchemy+Alembicでモデル/マイグレーション作成
- Step3: API実装(rental_items・maintenance_logのCRUD)
- Step4: Dockerイメージの本番相当化(--reload除去、.env分離)
- Step5: AWSデプロイ(EC2+Docker Compose)

## つまずきポイントと解決策

技術的な詰まりどころは、後から見返せるように「エラー→原因→解決」の形で残しています。

### ① .pemファイルでSSH接続できない

- 原因:Windows上の.pemファイルの権限が緩すぎる
- 解決:`icacls <ファイル> /inheritance:r` で継承を切り、`/grant:r "<自分のユーザー>:R"` で自分のみ読み取り権限を付与

### ② .envを変更してもコンテナに反映されない

- 原因:Docker Composeはupだけでは環境変数の再読み込みをしない場合がある
- 解決:`docker compose up -d --force-recreate` でコンテナを作り直す

### ③ PowerShellでコンテナ内の環境変数を確認すると値がおかしい

- 原因:`sh -c "echo $VAR"` のようにダブルクォートを使うと、PowerShell側で先に変数展開されてしまう
- 解決:シングルクォートを使う(`sh -c 'echo $VAR'`)

### ④ Swagger UIのPATCHで意図しない項目が上書きされる

- 原因:サンプル雛形の一部だけ書き換えて送信すると、残りの項目がサンプル値のまま送られる
- 解決:PATCH送信前にリクエストボディを一度全消去し、変更したい項目だけを書く

## 第三弾との連携設計

```
[第三弾: 店舗管理システム]
   │ レーンが「故障中」に変化した時のみ
   ▼ (X-API-Keyヘッダー付きPOST)
[拡張①: maintenance_log API]
```

- 書き込み系(POST/PATCH/DELETE)のみAPIキー認証必須、GET系は認証なし ─ 用途に応じてセキュリティレベルを分けた
- 第三弾側はwas_brokenフラグで、同一の故障状態からの重複登録を防止
- 拡張①側が落ちていても第三弾の処理は止めない(例外を握りつぶして本処理を継続する設計)

## 第四弾との連携設計(双方向)

```
[第四弾: 会員マイギア管理]  ⇄  [拡張①: メンテナンス管理]
  会員が整備依頼を出す →              → maintenance_log登録
  整備完了情報を反映  ←              ← 完了コールバック
```

第四弾は第三弾と同じSQLite+FastAPI構成でしたが、load_dotenv()が未導入だったため.envが読み込まれず、最初は接続に失敗しました。main.py冒頭への追加とrequirements.txtへのpython-dotenv/requests追記で解決しています。

なお第四弾側には元々、リマインド計算専用の独自MaintenanceLogモデルが存在しており、拡張①の汎用maintenance_logとは別物です。名前が同じでも役割が違うため、混同しないよう注意しながら実装しました。

## まとめ

Docker・AWSともに未経験からのスタートでしたが、EC2への本番デプロイ、そして既存システムとの双方向API連携まで実機で確認できました。特に「エラー→原因→解決」を都度言語化したことで、次に似た問題に当たったときの対応速度が上がると感じています。

スポーツボウリング場構想・拡張①の開発記録は、これで一区切りです。🎳