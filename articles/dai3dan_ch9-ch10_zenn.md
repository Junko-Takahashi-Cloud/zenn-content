---
title: "スポーツボウリング場構想・第三弾⑨⑩ Claudeが止まった。Copilotへバトンタッチ、そして走り始める"
emoji: "🎳"
type: "tech" # tech: 技術記事 / idea: アイデア記事
topics: ["個人開発", "AI活用", "キャリアチェンジ", "生成AI"]
published: true
---
Claudeが制限にかかっても、開発は止めたくありませんでした。そこでCopilotに実装をつなぎました。

Copilotは最初から第三弾全体の主担当ではなく、実装補助として想定していました。しかしClaudeが止まる場面で、実際の実装を進める比重が大きくなっていきました。

スタッフログイン、予約一覧、レーン、チェックイン、チェックアウト、教室、決済、API、UI。AIに実装を任せる速度を実感する一方で、どこまで任せるかを管理する難しさも、ここから大きくなっていきました。

Copilotによる実装は速いものでした。スタッフログイン、レーン管理、当日予約、チェックイン、チェックアウト、教室参加者、出席、決済といった機能が、具体的な形になっていきました。

実装案には、/staff/login、/lanes/today、/reservations/today、/reservations/checkin、/reservations/checkout、/class/participants、/class/attend、/payments/today、/payments/confirm といったAPIも含まれていました。

AIに実装を任せると、短い時間でここまで進む。その発見があった一方で、「作れること」と「今作るべきこと」は違う、という問題も見えてきました。

## 次回予告
次の章では、実装が進む中で感じた違和感について書きます。

---
スポーツボウリング場構想・第三弾開発記録は、まだ続きます。🎳

📝 業務のご相談・お問い合わせ → [ホームページ](https://junko-takahashi-cloud.github.io/personal-site/) / [ココナラ](https://coconala.com/users/6008491)