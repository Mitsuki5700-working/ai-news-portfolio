# AI活用ニュース自動収集システム

AI関連の最新ニュースを定期的に自動収集・要約し、GitHub上にデータベース化して蓄積するシステム。
副業（業務効率化系）のポートフォリオとして開発。

## 概要

- RSS + Claude Web検索のハイブリッドで、AI関連ニュースを定期収集
- Claude APIで日本語要約・カテゴリタグ付け
- `data/articles.json` に記録を蓄積し、重複を排除しながらREADMEの一覧を自動生成

## アーキテクチャ（段階移行方針）

まずはローカルで動くPythonスクリプト + GitHub Actionsのスケジュール実行でv1を完成させ、
その後AWS Lambda + EventBridgeによるサーバーレス構成に移行する予定。

```
v1: GitHub Actions (cron) → Python → RSS/Claude Web検索 → Claude要約 → data/articles.json → README自動生成
v2: EventBridge → Lambda → (同様の処理) → S3/DynamoDB等
```

## 開発ステータス

現在セットアップ中。

## ライセンス

未定
