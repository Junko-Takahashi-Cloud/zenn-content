---
title: "スポーツボウリング場構想・第三弾⑭⑮ 第三弾に戻る。そして、AIに任せても最後の判断は人間"
emoji: "🎳"
type: "tech" # tech: 技術記事 / idea: アイデア記事
topics: ["個人開発", "AI活用", "キャリアチェンジ", "生成AI"]
published: false
---

第三弾は、第一弾・第二弾を捨てて作り直すものではありません。既存システムを活かし、その上に店舗運営の機能を追加する。この方針が、確認済みのモデル定義にも表れています。

class_group は新設せず、支払いの参照には既存の class_courses.course_id を使います。ClassSession は置き換えず lane_pair を追加して拡張し、既存の LaneSet は維持したまま、新規の lanes を lane_set_id で既存の lane_sets につなぎます。ClassAttendance も既存の class_sessions.class_session_id を参照します。

lane_pair はペア番号ではなく先頭レーン番号として保持します。たとえば 1 は1・2番、3 は3・4番を表します。

ここで確認できたのは、「作り直す」のではなく「拡張する」ことの重要性でした。

AIは設計案を出し、コードを書き、データベースやAPIを考え、エラー修正も支援できます。複数のAIを使えば、別の視点からレビューもできます。

それでも最後に残るのは、「このコードを、このシステムに入れてよいのか」という判断でした。

一つのAIの案を正解として扱うのではなく、複数の意見を比較し、既存コードとの整合、既存仕様との互換性、新規追加か既存拡張かを確認する。そして、採用する設計を決める。

コードを書くことと、システム全体を設計することは同じではない。第三弾では、そのことを繰り返し確かめました。

## 次回予告

最終章では、第三弾を作って分かったことをまとめます。

---

スポーツボウリング場構想・第三弾開発記録は、まだ続きます。🎳

📝 業務のご相談・お問い合わせ → [ホームページ](https://junko-takahashi-cloud.github.io/personal-site/?utm_source=zenn&utm_medium=article&utm_campaign=daisandan14-15) / [ココナラ](https://coconala.com/users/6008491?utm_source=zenn&utm_medium=article&utm_campaign=daisandan14-15)