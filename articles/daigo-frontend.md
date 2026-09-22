---
title: "【React+Recharts】データ可視化のために1プロジェクトだけReactを選んだ話 ― 第五弾フロントエンド実装記"
emoji: "📊"
type: "tech"
topics: ["react", "recharts", "fastapi", "python", "個人開発"]
published: false
---

## この記事でわかること

- Streamlit統一ではなく、あえて1プロジェクトだけReactを採用した判断基準
- ダミーデータ先行→本物のAPI接続、という開発の進め方
- 会員向け・スタッフ向け2画面の構成と、実際に使ったAPIエンドポイント
- 実機CSVデータを取り込むための投球者自動特定ロジック

## なぜここだけReactなのか

これまでのプロジェクト(第一弾・第二弾)はStreamlitでフロントエンドを構築してきました。今回、全プロジェクトをStreamlitに統一するのではなく、第五弾のみReactに挑戦する方針を採りました。

| 観点 | 判断 |
|---|---|
| 統合のしやすさ(建物へ即導入) | Streamlit統一の方が有利。今回は優先しない |
| 転職市場での需要 | React経験の方が評価されやすい |
| プロジェクトの性質 | データ可視化が中心 → Rechartsとの相性が良いReactが適する |

「統合を犠牲にしてでもReact経験を積む」という、割り切った投資判断です。

## 技術スタック

| レイヤー | 採用技術 |
|---|---|
| ビルドツール | Vite |
| フレームワーク | React |
| グラフ描画 | Recharts |
| HTTPクライアント | axios(本番接続時) |
| 実行環境 | Node.js v24.21.0 |
| プロジェクト構成 | バックエンド(bowling-lane-scoring-system)とは別リポジトリ(bowling-lane-scoring-frontend) |

## 開発の進め方:ダミーデータ先行

いきなり本物のAPIに接続せず、まずダミーデータで画面を完成させる進め方を取りました。

- Step1: ダミーデータの型を、実際のAPIレスポンス形式に合わせて作成
- Step2: ダミーデータで画面(レイアウト・グラフ・表)を完成させる
- Step3: CORS設定 + axios呼び出しに差し替えて本物のAPIに接続

ダミーデータの形を最初から本番のレスポンス形式に揃えておいたことで、Step3では「呼び出し部分を差し替えるだけ」で済みました。ここが個人的に一番効いたポイントです。

## 会員向け画面

画面構成:サマリーカード4つ + 指標カード5つ + フレーム内訳の円グラフ(Recharts) + ギア別成績表 + 直近5ゲームリスト

使用API:

```
GET /api/v1/dashboard/member/{user_id}
GET /api/v1/dashboard/member/{user_id}/gear-performance
GET /api/v1/dashboard/member/{user_id}/loss-factors
```

本物のAPIと接続する際は、app/main.pyに以下を追加しています。

```python
app.add_middleware(
    CORSMiddleware,
    allow_origins=["http://localhost:5173"],
    ...
)
```

テスト会員(member_code=M001)・テストゲーム1件(スコア162、ストライク5・スペア1・オープン3)を実データとして投入し、画面表示がAPIレスポンスと完全一致することを確認しました。認証は暫定でStaffトークンを流用し、会員本人ログインは次の段階に回しています。

## スタッフ向け画面

バックエンド側に既に実装済みだった3エンドポイントを活用しました。

```
GET /api/v1/dashboard/lanes/performance       … レーン別成績
GET /api/v1/dashboard/conditions/performance  … オイルパターン別成績
GET /api/v1/dashboard/center/overview         … センター全体サマリー
```

conditions/performanceのロジックが少し複雑で、オイルパターンをレーン番号+適用日で逆引きしています(applied_date <= session.start_timeとなる直近のレコードを採用)。これをStaffDashboard.jsxから呼び出し、Rechartsの棒グラフでセンターサマリー・レーン別・コンディション別を表示しています。

## 会員本人ログイン

```
POST /api/v1/users/login
  body: { identifier, pin_code }
  → 成功時トークンをlocalStorageに保存
```

ログアウトボタンでlocalStorageのトークンを削除する、シンプルな構成です。

## 実機データ取込:CSV設計とマッチングロジック

実機からの生データを受け取る口として、汎用CSVフォーマットを新規設計しました。

```
1行 = 1投
列: lane_number, player_slot, game_number, frame_number,
    shot_number, pins_knocked, remaining_pin_numbers, timestamp
```

エンドポイントはPOST /api/v1/scores/import-csv。ポイントは投球者の自動特定です。

```
レーン番号 + タイムスタンプ
  → BowlingSessionを検索
  → SessionPlayerを逆引き
  → マッチしなければunmatchedリストに追加
```

タイムスタンプとレーン番号という、実機側が確実に持っているであろう2つの情報だけで投球者を特定できるようにしたことで、実機側の実装をシンプルに保てる設計になっています。マッチしなかった行は個別にリストアップされるので、取込ミスの発見にもつながります。

## まとめ

ダミーデータ先行の進め方が、本番API接続時のスムーズさにつながったのが今回の一番の学びでした。会員向け・スタッフ向け・ログイン・CSV取込まで、第五弾のフロントエンドはこれで一通り完了です。

スポーツボウリング場構想・第五弾フロントエンド編は、これで一区切りです。🎳
