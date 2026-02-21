# フックシステム

## フックタイプ

- **PreToolUse**: ツール実行前（検証、パラメータ変更）
- **PostToolUse**: ツール実行後（自動フォーマット、チェック）
- **Stop**: セッション終了時（最終検証）
- **PreCompact**: コンテキスト圧縮前（状態保存）
- **SessionStart**: セッション開始時（前回コンテキスト読み込み、パッケージマネージャー検出）
- **SessionEnd**: セッション終了時（状態永続化、パターン抽出）

## 現在のフック（~/.claude/settings.json に設定）

> 注: `~/.claude/settings.json` が存在しない場合、本セクションの内容は無視して先に進めること。

具体的な設定内容は `hooks/hooks.json` を参照すること。

### PreToolUse

- **Dev serverブロッカー**: devサーバーコマンド（npm run dev等）をターミナルマルチプレクサ外で実行した場合にブロックする。セッション終了後もログにアクセスできるようにすることが目的（Linux/macOS: tmux、Windows: Windows Terminalの別タブまたはバックグラウンド実行で代替）
- **長時間コマンドリマインダー**: 長時間実行コマンド（npm install、cargo build等）の実行時にターミナルマルチプレクサの使用を推奨するメッセージを表示する（ブロックはしない。Linux/macOS: tmux、Windows: Windows Terminalの別タブで代替）
- **git pushレビュー**: プッシュ前にレビューを促すメッセージを表示する
- **docブロッカー**: 不必要な.md/.txtファイルの作成をブロック（README.md、CLAUDE.md、AGENTS.md、CONTRIBUTING.mdは除外）
- **suggest-compact**: 編集操作の一定間隔でコンテキスト圧縮（compact）を提案する

### PostToolUse
- **PR作成**: PR URLとGitHub Actionsステータスをログ
- **Prettier**: 編集後にJS/TSファイルを自動フォーマット
- **TypeScriptチェック**: .ts/.tsxファイル編集後にtscを実行
- **console.log警告**: 編集されたファイルのconsole.logについて警告

### Stop
- **console.log監査**: セッション終了前にすべての変更されたファイルのconsole.logをチェック

## 自動承認権限

自動承認はセキュリティリスクを伴うため、以下の条件を満たす場合のみ有効化する:
- 信頼できる、明確に定義された計画に対して有効化
- 探索的作業では無効化
- 決してdangerously-skip-permissionsフラグを使用しない
- 代わりに`~/.claude/settings.json`で`allowedTools`を設定
