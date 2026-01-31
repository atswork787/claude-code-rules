---
name: continuous-learning
description: Claude Codeセッションから再利用可能なパターンを自動的に抽出し、将来の使用のために学習済みスキルとして保存します。
---

# 継続的学習スキル

終了時にClaude Codeセッションを自動的に評価し、学習済みスキルとして保存できる再利用可能なパターンを抽出します。

## 動作方法

このスキルは各セッションの終わりに**Stopフック**として実行されます:

1. **セッション評価**: セッションに十分なメッセージがあるかチェック（デフォルト: 10以上）
2. **パターン検出**: セッションから抽出可能なパターンを識別
3. **スキル抽出**: 有用なパターンを`~/.claude/skills/learned/`に保存

## 設定

`config.json`を編集してカスタマイズ:

```json
{
  "min_session_length": 10,
  "extraction_threshold": "medium",
  "auto_approve": false,
  "learned_skills_path": "~/.claude/skills/learned/",
  "patterns_to_detect": [
    "error_resolution",
    "user_corrections",
    "workarounds",
    "debugging_techniques",
    "project_specific"
  ],
  "ignore_patterns": [
    "simple_typos",
    "one_time_fixes",
    "external_api_issues"
  ]
}
```

## パターンタイプ

| パターン | 説明 |
|---------|-------------|
| `error_resolution` | 特定のエラーがどのように解決されたか |
| `user_corrections` | ユーザー修正からのパターン |
| `workarounds` | フレームワーク/ライブラリの癖への解決策 |
| `debugging_techniques` | 効果的なデバッグアプローチ |
| `project_specific` | プロジェクト固有の規約 |

## フックセットアップ

`~/.claude/settings.json`に追加:

```json
{
  "hooks": {
    "Stop": [{
      "matcher": "*",
      "hooks": [{
        "type": "command",
        "command": "~/.claude/skills/continuous-learning/evaluate-session.sh"
      }]
    }]
  }
}
```

## なぜStopフック？

- **軽量**: セッション終了時に一度だけ実行
- **非ブロッキング**: すべてのメッセージにレイテンシを追加しない
- **完全なコンテキスト**: 完全なセッショントランスクリプトにアクセス可能

## 関連

- [The Longform Guide](https://x.com/affaanmustafa/status/2014040193557471352) - 継続的学習に関するセクション
- `/learn`コマンド - セッション中のパターン抽出の手動実行
