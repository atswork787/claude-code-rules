# 検証ループスキル

Claude Codeセッションのための包括的な検証システム。

## 使用する場合

このスキルを呼び出す:
- 機能または重要なコード変更を完了した後
- PRを作成する前
- 品質ゲートが通過することを確認したい時
- リファクタリング後

## 検証フェーズ

### フェーズ1: ビルド検証
```bash
# プロジェクトがビルドされるかチェック
npm run build 2>&1 | tail -20
# または
pnpm build 2>&1 | tail -20
```

ビルドが失敗したら、続行前に停止して修正。

### フェーズ2: 型チェック
```bash
# TypeScriptプロジェクト
npx tsc --noEmit 2>&1 | head -30

# Pythonプロジェクト
pyright . 2>&1 | head -30
```

すべての型エラーを報告。続行前に重要なものを修正。

### フェーズ3: Lintチェック
```bash
# JavaScript/TypeScript
npm run lint 2>&1 | head -30

# Python
ruff check . 2>&1 | head -30
```

### フェーズ4: テストスイート
```bash
# カバレッジ付きでテストを実行
npm run test -- --coverage 2>&1 | tail -50

# カバレッジしきい値をチェック
# 目標: 最低80%
```

報告:
- 総テスト数: X
- 合格: X
- 失敗: X
- カバレッジ: X%

### フェーズ5: セキュリティスキャン
```bash
# 秘密情報をチェック
grep -rn "sk-" --include="*.ts" --include="*.js" . 2>/dev/null | head -10
grep -rn "api_key" --include="*.ts" --include="*.js" . 2>/dev/null | head -10

# console.logをチェック
grep -rn "console.log" --include="*.ts" --include="*.tsx" src/ 2>/dev/null | head -10
```

### フェーズ6: 差分レビュー
```bash
# 何が変更されたかを表示
git diff --stat
git diff HEAD~1 --name-only
```

各変更されたファイルをレビュー:
- 意図しない変更
- 欠落しているエラー処理
- 潜在的なエッジケース

## 出力形式

すべてのフェーズを実行した後、検証レポートを作成:

```
検証レポート
==================

ビルド:     [PASS/FAIL]
型:         [PASS/FAIL] (Xエラー)
Lint:       [PASS/FAIL] (X警告)
テスト:     [PASS/FAIL] (X/Y合格、Z%カバレッジ)
セキュリティ: [PASS/FAIL] (X問題)
差分:       [Xファイル変更]

全体:       [準備完了/未準備] PR用

修正すべき問題:
1. ...
2. ...
```

## 継続モード

長いセッションの場合、15分ごとまたは大きな変更後に検証を実行:

```markdown
メンタルチェックポイントを設定:
- 各関数を完了した後
- コンポーネントを完了した後
- 次のタスクに移る前

実行: /verify
```

## フックとの統合

このスキルはPostToolUseフックを補完しますが、より深い検証を提供します。
フックは問題を即座にキャッチします。このスキルは包括的なレビューを提供します。
