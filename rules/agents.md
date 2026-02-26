# エージェントオーケストレーション

## 利用可能なエージェント

`~/.claude/agents/`（Windowsの場合: `%USERPROFILE%\.claude\agents\`）に配置:

| エージェント | 目的 | 使用する場合 |
| ------- | --------- | ----------- |
| planner | 実装計画 | 複雑な機能を実装する時、リファクタリングする時 |
| architect | システム設計 | アーキテクチャを決定する時 |
| tdd-guide | テスト駆動開発 | 新機能を追加する時、バグを修正する時 |
| code-reviewer | コードレビュー | コードを変更した後 |
| security-reviewer | セキュリティ分析 | コミットする前 |
| build-error-resolver | ビルドエラー修正 | ビルドが失敗した時 |
| e2e-runner | E2Eテスト | 重要なユーザーフローをテストする時 |
| refactor-cleaner | デッドコードクリーンアップ | コードをメンテナンスする時 |
| doc-updater | ドキュメント | ドキュメントを更新する時 |
| database-reviewer | DB最適化・RLSレビュー | SQLクエリ、スキーマ設計、DBパフォーマンス問題発生時 |

## 自動エージェント起動

以下の条件ではユーザーの指示を待たずにエージェントを起動:

1. 複雑な機能リクエスト - **planner**エージェントを使用
2. コード変更後 - **code-reviewer**エージェントを使用
3. バグ修正または新機能 - **tdd-guide**エージェントを使用
4. アーキテクチャの決定 - **architect**エージェントを使用

## エージェント優先順位

タスクタイプに応じて適切なエージェント順序を選択:

| タスクタイプ | エージェント順序 |
| ------------ | ---------------- |
| 新機能追加 | planner → architect → tdd-guide → code-reviewer → security-reviewer |
| バグ修正 | tdd-guide → code-reviewer → security-reviewer |
| リファクタリング | planner → refactor-cleaner → code-reviewer |
| セキュリティ関連変更 | security-reviewer → planner → tdd-guide → code-reviewer → security-reviewer |
| DBスキーマ変更 | database-reviewer → security-reviewer |

## エージェント依存関係

| エージェント | 必須の前提 | 推奨の前提 | 出力形式 |
| ------------ | ---------- | ---------- | -------- |
| planner | - | - | 実装計画（Markdown） |
| architect | planner | - | 設計ドキュメント（Markdown） |
| tdd-guide | planner | - | テストコード + 実装コード |
| code-reviewer | コード変更 | tdd-guide | レビューコメント（テキスト） |
| security-reviewer | コード変更 | code-reviewer | 脆弱性レポート（テキスト） |
| build-error-resolver | build失敗 | - | 修正コード |
| e2e-runner | - | code-reviewer | テスト結果（Pass/Fail + ログ） |
| refactor-cleaner | - | planner | 修正コード |
| doc-updater | - | code-reviewer | 更新ドキュメント（Markdown） |
| database-reviewer | - | security-reviewer | レビューレポート（テキスト） |

## エラー時の対応

| エラー種類 | 対応 | リトライ | スキップ可否 |
| ---------- | ---- | -------- | ------------ |
| タイムアウト | リトライ後、ユーザーに確認 | 最大2回 | 不可 |
| 前提条件エラー | 前提エージェントを先に実行 | - | 不可 |
| 出力形式エラー | リトライ後、テキスト形式で続行 | 最大1回 | 可 |
| エージェント不在 | ユーザーに報告して代替案を提示 | - | 可 |
| リソース不足 | 処理を分割して再実行 | - | 不可 |

## 並列タスク実行

独立した操作には常に並列タスク実行を使用:

```markdown
# 良い: 並列実行（Taskツールを同時に複数呼び出す）
Task(subagent_type="security-reviewer", prompt="auth.tsのセキュリティ分析")
Task(subagent_type="code-reviewer", prompt="キャッシュシステムのパフォーマンスレビュー")
Task(subagent_type="code-reviewer", prompt="utils.tsの型チェック")

# 悪い: 不必要な順次実行
Task(...) -> 完了を待つ -> Task(...) -> 完了を待つ -> Task(...)
```

### 並列実行の制限

同時に起動するエージェント数: 最大3つ

適用ルール:

- 独立したタスクがある場合のみ並列化する
- 依存関係がある場合は順次実行する
- 必要なエージェントが1つなら1つだけ起動する

## 多角的分析

複雑な問題には、役割を分担した複数のサブエージェントを使用:

| サブエージェント | 責務 |
| --------------- | ---- |
| 事実レビュワー | 技術的な正確性と事実関係を検証する |
| シニアエンジニア | 設計パターンとベストプラクティスを評価する |
| セキュリティエキスパート | 脆弱性とセキュリティリスクを特定する |
| 一貫性レビュワー | コードスタイルと命名規則の統一性を確認する |
| 冗長性チェッカー | 重複コードと不要な複雑さを検出する |
