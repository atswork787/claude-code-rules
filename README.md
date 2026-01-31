# Everything Claude Code

[![Stars](https://img.shields.io/github/stars/affaan-m/everything-claude-code?style=flat)](https://github.com/affaan-m/everything-claude-code/stargazers)
[![License](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)
![Shell](https://img.shields.io/badge/-Shell-4EAA25?logo=gnu-bash&logoColor=white)
![TypeScript](https://img.shields.io/badge/-TypeScript-3178C6?logo=typescript&logoColor=white)
![Markdown](https://img.shields.io/badge/-Markdown-000000?logo=markdown&logoColor=white)

**Anthropicハッカソン優勝者からのClaude Code設定の完全なコレクション。**

実際の製品を構築する10ヶ月以上の集中的な日常使用で進化した、本番環境対応のエージェント、スキル、フック、コマンド、ルール、MCP設定。

---

## ガイド

このリポジトリは生のコードのみです。ガイドがすべてを説明します。

<table>
<tr>
<td width="50%">
<a href="https://x.com/affaanmustafa/status/2012378465664745795">
<img src="https://github.com/user-attachments/assets/1a471488-59cc-425b-8345-5245c7efbcef" alt="The Shorthand Guide to Everything Claude Code" />
</a>
</td>
<td width="50%">
<a href="https://x.com/affaanmustafa/status/2014040193557471352">
<img src="https://github.com/user-attachments/assets/c9ca43bc-b149-427f-b551-af6840c368f0" alt="The Longform Guide to Everything Claude Code" />
</a>
</td>
</tr>
<tr>
<td align="center"><b>簡易ガイド</b><br/>セットアップ、基礎、哲学。<b>これをまず読んでください。</b></td>
<td align="center"><b>詳細ガイド</b><br/>トークン最適化、メモリ永続化、評価、並列化。</td>
</tr>
</table>

| トピック | 学べること |
|-------|-------------------|
| トークン最適化 | モデル選択、システムプロンプトのスリム化、バックグラウンドプロセス |
| メモリ永続化 | セッション間でコンテキストを自動的に保存/ロードするフック |
| 継続的学習 | セッションからパターンを自動抽出して再利用可能なスキルに |
| 検証ループ | チェックポイント vs 継続的評価、採点タイプ、pass@kメトリクス |
| 並列化 | Gitワークツリー、カスケード方式、インスタンスをスケールするタイミング |
| サブエージェントオーケストレーション | コンテキスト問題、反復取得パターン |

---

## クロスプラットフォーム対応

このプラグインは、**Windows、macOS、Linux**を完全にサポートしています。すべてのフックとスクリプトは、最大の互換性のためにNode.jsで書き直されています。

### パッケージマネージャー検出

プラグインは、優先するパッケージマネージャー（npm、pnpm、yarn、またはbun）を以下の優先順位で自動検出します:

1. **環境変数**: `CLAUDE_PACKAGE_MANAGER`
2. **プロジェクト設定**: `.claude/package-manager.json`
3. **package.json**: `packageManager`フィールド
4. **ロックファイル**: package-lock.json、yarn.lock、pnpm-lock.yaml、またはbun.lockbからの検出
5. **グローバル設定**: `~/.claude/package-manager.json`
6. **フォールバック**: 最初に利用可能なパッケージマネージャー

優先するパッケージマネージャーを設定するには:

```bash
# 環境変数経由
export CLAUDE_PACKAGE_MANAGER=pnpm

# グローバル設定経由
node scripts/setup-package-manager.js --global pnpm

# プロジェクト設定経由
node scripts/setup-package-manager.js --project bun

# 現在の設定を検出
node scripts/setup-package-manager.js --detect
```

またはClaude Codeで`/setup-pm`コマンドを使用してください。

---

## 中身

このリポジトリは**Claude Codeプラグイン**です - 直接インストールするか、コンポーネントを手動でコピーしてください。

```
everything-claude-code/
|-- .claude-plugin/   # プラグインとマーケットプレイスマニフェスト
|   |-- plugin.json         # プラグインメタデータとコンポーネントパス
|   |-- marketplace.json    # /plugin marketplace add用のマーケットプレイスカタログ
|
|-- agents/           # 委任用の特化したサブエージェント
|   |-- planner.md           # 機能実装計画
|   |-- architect.md         # システム設計決定
|   |-- tdd-guide.md         # テスト駆動開発
|   |-- code-reviewer.md     # 品質とセキュリティレビュー
|   |-- security-reviewer.md # 脆弱性分析
|   |-- build-error-resolver.md
|   |-- e2e-runner.md        # Playwright E2Eテスト
|   |-- refactor-cleaner.md  # デッドコードクリーンアップ
|   |-- doc-updater.md       # ドキュメント同期
|
|-- skills/           # ワークフロー定義とドメイン知識
|   |-- coding-standards/           # 言語ベストプラクティス
|   |-- backend-patterns/           # API、データベース、キャッシングパターン
|   |-- frontend-patterns/          # React、Next.jsパターン
|   |-- continuous-learning/        # セッションからパターンを自動抽出（詳細ガイド）
|   |-- strategic-compact/          # 手動コンパクション提案（詳細ガイド）
|   |-- tdd-workflow/               # TDD方法論
|   |-- security-review/            # セキュリティチェックリスト
|   |-- eval-harness/               # 検証ループ評価（詳細ガイド）
|   |-- verification-loop/          # 継続的検証（詳細ガイド）
|
|-- commands/         # クイック実行用のスラッシュコマンド
|   |-- tdd.md              # /tdd - テスト駆動開発
|   |-- plan.md             # /plan - 実装計画
|   |-- e2e.md              # /e2e - E2Eテスト生成
|   |-- code-review.md      # /code-review - 品質レビュー
|   |-- build-fix.md        # /build-fix - ビルドエラー修正
|   |-- refactor-clean.md   # /refactor-clean - デッドコード削除
|   |-- learn.md            # /learn - セッション中にパターンを抽出（詳細ガイド）
|   |-- checkpoint.md       # /checkpoint - 検証状態を保存（詳細ガイド）
|   |-- verify.md           # /verify - 検証ループを実行（詳細ガイド）
|   |-- setup-pm.md         # /setup-pm - パッケージマネージャーを設定（新規）
|
|-- rules/            # 常に従うガイドライン（~/.claude/rules/にコピー）
|   |-- security.md         # 必須のセキュリティチェック
|   |-- coding-style.md     # イミュータビリティ、ファイル構成
|   |-- testing.md          # TDD、80%カバレッジ要件
|   |-- git-workflow.md     # コミット形式、PRプロセス
|   |-- agents.md           # サブエージェントへの委任タイミング
|   |-- performance.md      # モデル選択、コンテキスト管理
|
|-- hooks/            # トリガーベースの自動化
|   |-- hooks.json                # すべてのフック設定（PreToolUse、PostToolUse、Stopなど）
|   |-- memory-persistence/       # セッションライフサイクルフック（詳細ガイド）
|   |-- strategic-compact/        # コンパクション提案（詳細ガイド）
|
|-- scripts/          # クロスプラットフォームNode.jsスクリプト（新規）
|   |-- lib/                     # 共有ユーティリティ
|   |   |-- utils.js             # クロスプラットフォームファイル/パス/システムユーティリティ
|   |   |-- package-manager.js   # パッケージマネージャー検出と選択
|   |-- hooks/                   # フック実装
|   |   |-- session-start.js     # セッション開始時にコンテキストをロード
|   |   |-- session-end.js       # セッション終了時に状態を保存
|   |   |-- pre-compact.js       # コンパクション前の状態保存
|   |   |-- suggest-compact.js   # 戦略的コンパクション提案
|   |   |-- evaluate-session.js  # セッションからパターンを抽出
|   |-- setup-package-manager.js # インタラクティブPMセットアップ
|
|-- tests/            # テストスイート（新規）
|   |-- lib/                     # ライブラリテスト
|   |-- hooks/                   # フックテスト
|   |-- run-all.js               # すべてのテストを実行
|
|-- contexts/         # 動的システムプロンプト注入コンテキスト（詳細ガイド）
|   |-- dev.md              # 開発モードコンテキスト
|   |-- review.md           # コードレビューモードコンテキスト
|   |-- research.md         # 研究/探索モードコンテキスト
|
|-- examples/         # 例の設定とセッション
|   |-- CLAUDE.md           # 例のプロジェクトレベル設定
|   |-- user-CLAUDE.md      # 例のユーザーレベル設定
|
|-- mcp-configs/      # MCPサーバー設定
|   |-- mcp-servers.json    # GitHub、Supabase、Vercel、Railwayなど
|
|-- marketplace.json  # セルフホスト型マーケットプレイス設定（/plugin marketplace add用）
```

---

## インストール

### オプション1: プラグインとしてインストール（推奨）

このリポジトリを使用する最も簡単な方法 - Claude Codeプラグインとしてインストール:

```bash
# このリポジトリをマーケットプレイスとして追加
/plugin marketplace add affaan-m/everything-claude-code

# プラグインをインストール
/plugin install everything-claude-code@everything-claude-code
```

または、`~/.claude/settings.json`に直接追加:

```json
{
  "extraKnownMarketplaces": {
    "everything-claude-code": {
      "source": {
        "source": "github",
        "repo": "affaan-m/everything-claude-code"
      }
    }
  },
  "enabledPlugins": {
    "everything-claude-code@everything-claude-code": true
  }
}
```

これにより、すべてのコマンド、エージェント、スキル、フックへの即座のアクセスが可能になります。

---

### オプション2: 手動インストール

インストールするものを手動で制御したい場合:

```bash
# リポジトリをクローン
git clone https://github.com/affaan-m/everything-claude-code.git

# エージェントをClaude設定にコピー
cp everything-claude-code/agents/*.md ~/.claude/agents/

# ルールをコピー
cp everything-claude-code/rules/*.md ~/.claude/rules/

# コマンドをコピー
cp everything-claude-code/commands/*.md ~/.claude/commands/

# スキルをコピー
cp -r everything-claude-code/skills/* ~/.claude/skills/
```

#### settings.jsonにフックを追加

`hooks/hooks.json`からフックを`~/.claude/settings.json`にコピーします。

#### MCPを設定

`mcp-configs/mcp-servers.json`から必要なMCPサーバーを`~/.claude.json`にコピーします。

**重要:** `YOUR_*_HERE`プレースホルダーを実際のAPIキーに置き換えてください。

---

## 主要概念

### エージェント

サブエージェントは、限られた範囲で委任されたタスクを処理します。例:

```markdown
---
name: code-reviewer
description: 品質、セキュリティ、保守性のためにコードをレビュー
tools: ["Read", "Grep", "Glob", "Bash"]
model: opus
---

あなたはシニアコードレビュアーです...
```

### スキル

スキルは、コマンドまたはエージェントによって呼び出されるワークフロー定義です:

```markdown
# TDDワークフロー

1. まずインターフェースを定義
2. 失敗するテストを書く（RED）
3. 最小限のコードを実装（GREEN）
4. リファクタリング（IMPROVE）
5. 80%以上のカバレッジを確認
```

### フック

フックは、ツールイベントで発火します。例 - console.logについて警告:

```json
{
  "matcher": "tool == \"Edit\" && tool_input.file_path matches \"\\\\.(ts|tsx|js|jsx)$\"",
  "hooks": [{
    "type": "command",
    "command": "#!/bin/bash\ngrep -n 'console\\.log' \"$file_path\" && echo '[Hook] console.logを削除' >&2"
  }]
}
```

### ルール

ルールは常に従うガイドラインです。モジュール化して保持:

```
~/.claude/rules/
  security.md      # ハードコードされた秘密情報なし
  coding-style.md  # イミュータビリティ、ファイル制限
  testing.md       # TDD、カバレッジ要件
```

---

## テストの実行

プラグインには包括的なテストスイートが含まれています:

```bash
# すべてのテストを実行
node tests/run-all.js

# 個別のテストファイルを実行
node tests/lib/utils.test.js
node tests/lib/package-manager.test.js
node tests/hooks/hooks.test.js
```

---

## 貢献

**貢献を歓迎し、奨励します。**

このリポジトリは、コミュニティリソースであることを意図しています。以下をお持ちの場合:
- 有用なエージェントまたはスキル
- 巧妙なフック
- より良いMCP設定
- 改善されたルール

ぜひ貢献してください！ガイドラインについては[CONTRIBUTING.md](CONTRIBUTING.md)を参照してください。

### 貢献のアイデア

- 言語固有のスキル（Python、Go、Rustパターン）
- フレームワーク固有の設定（Django、Rails、Laravel）
- DevOpsエージェント（Kubernetes、Terraform、AWS）
- テスト戦略（異なるフレームワーク）
- ドメイン固有の知識（ML、データエンジニアリング、モバイル）

---

## 背景

実験的なロールアウト以来、Claude Codeを使用しています。2025年9月のAnthropic x Forum Venturesハッカソンで[@DRodriguezFX](https://x.com/DRodriguezFX)と共に[zenith.chat](https://zenith.chat)を構築して優勝しました - 完全にClaude Codeを使用。

これらの設定は、複数の本番アプリケーションで実戦テストされています。

---

## 重要な注意事項

### コンテキストウィンドウ管理

**重要:** すべてのMCPを一度に有効にしないでください。有効にするツールが多すぎると、200kのコンテキストウィンドウが70kに縮小する可能性があります。

経験則:
- 20-30個のMCPを設定
- プロジェクトごとに10個未満を有効に保つ
- 80個未満のツールをアクティブに

プロジェクト設定で`disabledMcpServers`を使用して、未使用のものを無効にします。

### カスタマイズ

これらの設定は私のワークフローに適していますが、以下を行うべきです:
1. 共感できるものから始める
2. スタックに合わせて変更
3. 使用しないものを削除
4. 独自のパターンを追加

---

## スター履歴

[![Star History Chart](https://api.star-history.com/svg?repos=affaan-m/everything-claude-code&type=Date)](https://star-history.com/#affaan-m/everything-claude-code&Date)

---

## リンク

- **簡易ガイド（ここから始める）:** [The Shorthand Guide to Everything Claude Code](https://x.com/affaanmustafa/status/2012378465664745795)
- **詳細ガイド（上級）:** [The Longform Guide to Everything Claude Code](https://x.com/affaanmustafa/status/2014040193557471352)
- **フォロー:** [@affaanmustafa](https://x.com/affaanmustafa)
- **zenith.chat:** [zenith.chat](https://zenith.chat)

---

## ライセンス

MIT - 自由に使用、必要に応じて変更、可能であれば貢献してください。

---

**役立つ場合はこのリポジトリにスターを付けてください。両方のガイドを読んでください。素晴らしいものを構築してください。**
