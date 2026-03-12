---
description: 優先するパッケージマネージャー（npm/pnpm/yarn/bun）を設定
---

# パッケージマネージャー設定

このプロジェクトまたはグローバルで優先するパッケージマネージャーを設定します。

## 前提条件

- `node --version` で Node.js が動作することを確認してください
- 使用するパッケージマネージャーがインストール済みであることを確認してください

## 使用法

> **注意**: 以下のコマンドは `scripts/setup-package-manager.js` が存在する場合にのみ使用できます。スクリプトが存在しない場合はこのセクションを読み飛ばし、「設定ファイル」または「環境変数」セクションを参照してください。

```bash
# 現在のパッケージマネージャーを検出
node scripts/setup-package-manager.js --detect

# グローバル設定を設定
node scripts/setup-package-manager.js --global pnpm

# プロジェクト設定を設定
node scripts/setup-package-manager.js --project bun

# 利用可能なパッケージマネージャーをリスト
node scripts/setup-package-manager.js --list
```

## 検出優先順位

使用するパッケージマネージャーを決定する際、以下の順序でチェックされます:

1. **環境変数**: `CLAUDE_PACKAGE_MANAGER`
2. **プロジェクト設定**: `.claude/package-manager.json`
3. **package.json**: `packageManager`フィールド
4. **ロックファイル**: package-lock.json、yarn.lock、pnpm-lock.yaml、またはbun.lock（bun v1.0以前はbun.lockb）の存在
5. **グローバル設定**: `~/.claude/package-manager.json`
6. **フォールバック**: 最初に利用可能なパッケージマネージャー（pnpm > bun > yarn > npm）

## 設定ファイル

### グローバル設定

```json
// ~/.claude/package-manager.json
{
  "packageManager": "pnpm"
}
```

### プロジェクト設定

```json
// .claude/package-manager.json
{
  "packageManager": "bun"
}
```

### package.json

```json
{
  "packageManager": "pnpm@8.6.0"
}
```

## 環境変数

`CLAUDE_PACKAGE_MANAGER`を設定して、他のすべての検出方法をオーバーライド:

```bash
# Windows（PowerShell）
$env:CLAUDE_PACKAGE_MANAGER = "pnpm"

# macOS/Linux
export CLAUDE_PACKAGE_MANAGER=pnpm
```

## 検出を実行

> **注意**: `scripts/setup-package-manager.js` が存在しない場合は読み飛ばしてください。

現在のパッケージマネージャー検出結果を表示するには、実行:

```bash
node scripts/setup-package-manager.js --detect
```

## 設定のリセット

設定を削除してデフォルトに戻すには、対応する設定ファイルを削除します。

### プロジェクト設定のリセット

```bash
rm .claude/package-manager.json
```

### グローバル設定のリセット

```bash
# macOS/Linux
rm ~/.claude/package-manager.json

# Windows（PowerShell）
Remove-Item "$env:USERPROFILE\.claude\package-manager.json"
```

### 環境変数のリセット

```bash
# macOS/Linux
unset CLAUDE_PACKAGE_MANAGER

# Windows（PowerShell）
Remove-Item Env:CLAUDE_PACKAGE_MANAGER
```
