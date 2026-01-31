---
name: security-review
description: 認証の追加、ユーザー入力の処理、秘密情報の操作、APIエンドポイントの作成、または支払い/機密機能の実装時にこのスキルを使用します。包括的なセキュリティチェックリストとパターンを提供します。
---

# セキュリティレビュースキル

このスキルは、すべてのコードがセキュリティベストプラクティスに従い、潜在的な脆弱性を識別することを保証します。

## アクティブ化する場合

- 認証または認可の実装
- ユーザー入力またはファイルアップロードの処理
- 新しいAPIエンドポイントの作成
- 秘密情報または認証情報の操作
- 支払い機能の実装
- 機密データの保存または送信
- サードパーティAPIの統合

## セキュリティチェックリスト

### 1. 秘密情報管理

#### ❌ 決してこれをしない
```typescript
const apiKey = "sk-proj-xxxxx"  // ハードコードされた秘密情報
const dbPassword = "password123" // ソースコード内
```

#### ✅ 常にこれをする
```typescript
const apiKey = process.env.OPENAI_API_KEY
const dbUrl = process.env.DATABASE_URL

// 秘密情報が存在することを確認
if (!apiKey) {
  throw new Error('OPENAI_API_KEYが設定されていません')
}
```

#### 検証ステップ
- [ ] ハードコードされたAPIキー、トークン、またはパスワードなし
- [ ] すべての秘密情報が環境変数に
- [ ] `.env.local`が.gitignoreに
- [ ] git履歴に秘密情報なし
- [ ] 本番の秘密情報がホスティングプラットフォーム（Vercel、Railway）に

### 2. 入力検証

#### 常にユーザー入力を検証
```typescript
import { z } from 'zod'

// 検証スキーマを定義
const CreateUserSchema = z.object({
  email: z.string().email(),
  name: z.string().min(1).max(100),
  age: z.number().int().min(0).max(150)
})

// 処理前に検証
export async function createUser(input: unknown) {
  try {
    const validated = CreateUserSchema.parse(input)
    return await db.users.create(validated)
  } catch (error) {
    if (error instanceof z.ZodError) {
      return { success: false, errors: error.errors }
    }
    throw error
  }
}
```

#### 検証ステップ
- [ ] すべてのユーザー入力がスキーマで検証されている
- [ ] ファイルアップロードが制限されている（サイズ、タイプ、拡張子）
- [ ] クエリで直接ユーザー入力を使用していない
- [ ] ホワイトリスト検証（ブラックリストではない）
- [ ] エラーメッセージが機密情報を漏洩しない

### 3. SQLインジェクション防止

#### ❌ 決してSQLを連結しない
```typescript
// 危険 - SQLインジェクションの脆弱性
const query = `SELECT * FROM users WHERE email = '${userEmail}'`
await db.query(query)
```

#### ✅ 常にパラメータ化されたクエリを使用
```typescript
// 安全 - パラメータ化されたクエリ
const { data } = await supabase
  .from('users')
  .select('*')
  .eq('email', userEmail)

// または生のSQLで
await db.query(
  'SELECT * FROM users WHERE email = $1',
  [userEmail]
)
```

#### 検証ステップ
- [ ] すべてのデータベースクエリがパラメータ化されたクエリを使用
- [ ] SQLで文字列連結なし
- [ ] ORM/クエリビルダーが正しく使用されている
- [ ] Supabaseクエリが適切にサニタイズされている

### 4. 認証と認可

#### JWTトークン処理
```typescript
// ❌ 間違い: localStorage（XSSに脆弱）
localStorage.setItem('token', token)

// ✅ 正しい: httpOnly cookies
res.setHeader('Set-Cookie',
  `token=${token}; HttpOnly; Secure; SameSite=Strict; Max-Age=3600`)
```

#### 認可チェック
```typescript
export async function deleteUser(userId: string, requesterId: string) {
  // 常に最初に認可を確認
  const requester = await db.users.findUnique({
    where: { id: requesterId }
  })

  if (requester.role !== 'admin') {
    return NextResponse.json(
      { error: '認可されていません' },
      { status: 403 }
    )
  }

  // 削除を続行
  await db.users.delete({ where: { id: userId } })
}
```

#### 行レベルセキュリティ（Supabase）
```sql
-- すべてのテーブルでRLSを有効化
ALTER TABLE users ENABLE ROW LEVEL SECURITY;

-- ユーザーは自分のデータのみ表示可能
CREATE POLICY "Users view own data"
  ON users FOR SELECT
  USING (auth.uid() = id);

-- ユーザーは自分のデータのみ更新可能
CREATE POLICY "Users update own data"
  ON users FOR UPDATE
  USING (auth.uid() = id);
```

#### 検証ステップ
- [ ] トークンがhttpOnly cookieに保存されている（localStorageではない）
- [ ] 機密操作前の認可チェック
- [ ] SupabaseでRow Level Securityが有効
- [ ] ロールベースのアクセス制御が実装されている
- [ ] セッション管理が安全

### 5. XSS防止

#### HTMLをサニタイズ
```typescript
import DOMPurify from 'isomorphic-dompurify'

// 常にユーザー提供のHTMLをサニタイズ
function renderUserContent(html: string) {
  const clean = DOMPurify.sanitize(html, {
    ALLOWED_TAGS: ['b', 'i', 'em', 'strong', 'p'],
    ALLOWED_ATTR: []
  })
  return <div dangerouslySetInnerHTML={{ __html: clean }} />
}
```

#### 検証ステップ
- [ ] ユーザー提供のHTMLがサニタイズされている
- [ ] CSPヘッダーが設定されている
- [ ] 未検証の動的コンテンツレンダリングなし
- [ ] Reactの組み込みXSS保護が使用されている

### 6. レート制限

#### APIレート制限
```typescript
import rateLimit from 'express-rate-limit'

const limiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15分
  max: 100, // ウィンドウあたり100リクエスト
  message: 'リクエストが多すぎます'
})

// ルートに適用
app.use('/api/', limiter)
```

#### 検証ステップ
- [ ] すべてのAPIエンドポイントでレート制限
- [ ] 高コストな操作でより厳しい制限
- [ ] IPベースのレート制限
- [ ] ユーザーベースのレート制限（認証済み）

## デプロイ前セキュリティチェックリスト

すべての本番デプロイ前に:

- [ ] **秘密情報**: ハードコードされた秘密情報なし、すべて環境変数に
- [ ] **入力検証**: すべてのユーザー入力が検証されている
- [ ] **SQLインジェクション**: すべてのクエリがパラメータ化されている
- [ ] **XSS**: ユーザーコンテンツがサニタイズされている
- [ ] **CSRF**: 保護が有効
- [ ] **認証**: 適切なトークン処理
- [ ] **認可**: ロールチェックが実施されている
- [ ] **レート制限**: すべてのエンドポイントで有効
- [ ] **HTTPS**: 本番で強制
- [ ] **セキュリティヘッダー**: CSP、X-Frame-Optionsが設定されている
- [ ] **エラー処理**: エラーに機密データなし
- [ ] **ログ記録**: 機密データがログに記録されていない
- [ ] **依存関係**: 最新、脆弱性なし
- [ ] **行レベルセキュリティ**: Supabaseで有効
- [ ] **CORS**: 適切に設定されている
- [ ] **ファイルアップロード**: 検証されている（サイズ、タイプ）

## リソース

- [OWASP Top 10](https://owasp.org/www-project-top-ten/)
- [Next.jsセキュリティ](https://nextjs.org/docs/security)
- [Supabaseセキュリティ](https://supabase.com/docs/guides/auth)
- [Webセキュリティアカデミー](https://portswigger.net/web-security)

---

**覚えておく**: セキュリティはオプションではありません。1つの脆弱性がプラットフォーム全体を危険にさらす可能性があります。疑わしい場合は、慎重な側に誤ってください。
