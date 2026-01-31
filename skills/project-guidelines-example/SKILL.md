# プロジェクトガイドラインスキル（例）

これはプロジェクト固有のスキルの例です。自分のプロジェクトのテンプレートとして使用してください。

実際の本番アプリケーションに基づいています: [Zenith](https://zenith.chat) - AI駆動のカスタマーディスカバリープラットフォーム。

---

## 使用する場合

設計された特定のプロジェクトで作業する際にこのスキルを参照してください。プロジェクトスキルには以下が含まれます:
- アーキテクチャ概要
- ファイル構造
- コードパターン
- テスト要件
- デプロイワークフロー

---

## アーキテクチャ概要

**技術スタック:**
- **フロントエンド**: Next.js 15（App Router）、TypeScript、React
- **バックエンド**: FastAPI（Python）、Pydanticモデル
- **データベース**: Supabase（PostgreSQL）
- **AI**: ツール呼び出しと構造化出力を備えたClaude API
- **デプロイ**: Google Cloud Run
- **テスト**: Playwright（E2E）、pytest（バックエンド）、React Testing Library

**サービス:**
```
┌─────────────────────────────────────────────────────────────┐
│                         フロントエンド                       │
│  Next.js 15 + TypeScript + TailwindCSS                     │
│  デプロイ: Vercel / Cloud Run                              │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                         バックエンド                         │
│  FastAPI + Python 3.11 + Pydantic                          │
│  デプロイ: Cloud Run                                       │
└─────────────────────────────────────────────────────────────┘
                              │
              ┌───────────────┼───────────────┐
              ▼               ▼               ▼
        ┌──────────┐   ┌──────────┐   ┌──────────┐
        │ Supabase │   │  Claude  │   │  Redis   │
        │ Database │   │   API    │   │  Cache   │
        └──────────┘   └──────────┘   └──────────┘
```

---

## コードパターン

### APIレスポンス形式（FastAPI）

```python
from pydantic import BaseModel
from typing import Generic, TypeVar, Optional

T = TypeVar('T')

class ApiResponse(BaseModel, Generic[T]):
    success: bool
    data: Optional[T] = None
    error: Optional[str] = None

    @classmethod
    def ok(cls, data: T) -> "ApiResponse[T]":
        return cls(success=True, data=data)

    @classmethod
    def fail(cls, error: str) -> "ApiResponse[T]":
        return cls(success=False, error=error)
```

### Claude AI統合（構造化出力）

```python
from anthropic import Anthropic
from pydantic import BaseModel

class AnalysisResult(BaseModel):
    summary: str
    key_points: list[str]
    confidence: float

async def analyze_with_claude(content: str) -> AnalysisResult:
    client = Anthropic()

    response = client.messages.create(
        model="claude-sonnet-4-5-20250514",
        max_tokens=1024,
        messages=[{"role": "user", "content": content}],
        tools=[{
            "name": "provide_analysis",
            "description": "構造化された分析を提供",
            "input_schema": AnalysisResult.model_json_schema()
        }],
        tool_choice={"type": "tool", "name": "provide_analysis"}
    )

    # ツール使用結果を抽出
    tool_use = next(
        block for block in response.content
        if block.type == "tool_use"
    )

    return AnalysisResult(**tool_use.input)
```

---

## テスト要件

### バックエンド（pytest）

```bash
# すべてのテストを実行
poetry run pytest tests/

# カバレッジ付きで実行
poetry run pytest tests/ --cov=. --cov-report=html

# 特定のテストファイルを実行
poetry run pytest tests/test_auth.py -v
```

### フロントエンド（React Testing Library）

```bash
# テストを実行
npm run test

# カバレッジ付きで実行
npm run test -- --coverage

# E2Eテストを実行
npm run test:e2e
```

---

## デプロイワークフロー

### デプロイ前チェックリスト

- [ ] すべてのテストがローカルで合格
- [ ] `npm run build`が成功（フロントエンド）
- [ ] `poetry run pytest`が合格（バックエンド）
- [ ] ハードコードされた秘密情報なし
- [ ] 環境変数が文書化されている
- [ ] データベースマイグレーションの準備完了

---

## 重要なルール

1. **絵文字なし** コード、コメント、ドキュメントに
2. **イミュータビリティ** - オブジェクトや配列を決してミューテートしない
3. **TDD** - 実装前にテストを書く
4. **80%カバレッジ** 最低
5. **多数の小さなファイル** - 通常200-400行、最大800行
6. **console.logなし** 本番コードに
7. **適切なエラー処理** try/catchで
8. **入力検証** Pydantic/Zodで

---

## 関連スキル

- `coding-standards.md` - 一般的なコーディングベストプラクティス
- `backend-patterns.md` - APIとデータベースパターン
- `frontend-patterns.md` - ReactとNext.jsパターン
- `tdd-workflow/` - テスト駆動開発方法論
