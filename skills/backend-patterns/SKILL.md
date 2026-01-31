---
name: backend-patterns
description: Node.js、Express、Next.js APIルートのためのバックエンドアーキテクチャパターン、API設計、データベース最適化、サーバーサイドベストプラクティス。
---

# バックエンド開発パターン

スケーラブルなサーバーサイドアプリケーションのためのバックエンドアーキテクチャパターンとベストプラクティス。

## API設計パターン

### RESTful API構造

```typescript
// ✅ リソースベースのURL
GET    /api/markets                 # リソースをリスト
GET    /api/markets/:id             # 単一リソースを取得
POST   /api/markets                 # リソースを作成
PUT    /api/markets/:id             # リソースを置換
PATCH  /api/markets/:id             # リソースを更新
DELETE /api/markets/:id             # リソースを削除

// ✅ フィルタリング、ソート、ページネーションのためのクエリパラメータ
GET /api/markets?status=active&sort=volume&limit=20&offset=0
```

### リポジトリパターン

```typescript
// データアクセスロジックを抽象化
interface MarketRepository {
  findAll(filters?: MarketFilters): Promise<Market[]>
  findById(id: string): Promise<Market | null>
  create(data: CreateMarketDto): Promise<Market>
  update(id: string, data: UpdateMarketDto): Promise<Market>
  delete(id: string): Promise<void>
}

class SupabaseMarketRepository implements MarketRepository {
  async findAll(filters?: MarketFilters): Promise<Market[]> {
    let query = supabase.from('markets').select('*')

    if (filters?.status) {
      query = query.eq('status', filters.status)
    }

    if (filters?.limit) {
      query = query.limit(filters.limit)
    }

    const { data, error } = await query

    if (error) throw new Error(error.message)
    return data
  }

  // その他のメソッド...
}
```

### サービス層パターン

```typescript
// ビジネスロジックをデータアクセスから分離
class MarketService {
  constructor(private marketRepo: MarketRepository) {}

  async searchMarkets(query: string, limit: number = 10): Promise<Market[]> {
    // ビジネスロジック
    const embedding = await generateEmbedding(query)
    const results = await this.vectorSearch(embedding, limit)

    // 完全なデータを取得
    const markets = await this.marketRepo.findByIds(results.map(r => r.id))

    // 類似性でソート
    return markets.sort((a, b) => {
      const scoreA = results.find(r => r.id === a.id)?.score || 0
      const scoreB = results.find(r => r.id === b.id)?.score || 0
      return scoreA - scoreB
    })
  }

  private async vectorSearch(embedding: number[], limit: number) {
    // ベクトル検索実装
  }
}
```

### ミドルウェアパターン

```typescript
// リクエスト/レスポンス処理パイプライン
export function withAuth(handler: NextApiHandler): NextApiHandler {
  return async (req, res) => {
    const token = req.headers.authorization?.replace('Bearer ', '')

    if (!token) {
      return res.status(401).json({ error: '認証されていません' })
    }

    try {
      const user = await verifyToken(token)
      req.user = user
      return handler(req, res)
    } catch (error) {
      return res.status(401).json({ error: '無効なトークン' })
    }
  }
}

// 使用法
export default withAuth(async (req, res) => {
  // ハンドラーはreq.userにアクセス可能
})
```

## データベースパターン

### クエリ最適化

```typescript
// ✅ 良い: 必要な列のみを選択
const { data } = await supabase
  .from('markets')
  .select('id, name, status, volume')
  .eq('status', 'active')
  .order('volume', { ascending: false })
  .limit(10)

// ❌ 悪い: すべてを選択
const { data } = await supabase
  .from('markets')
  .select('*')
```

### N+1クエリの防止

```typescript
// ❌ 悪い: N+1クエリ問題
const markets = await getMarkets()
for (const market of markets) {
  market.creator = await getUser(market.creator_id)  // Nクエリ
}

// ✅ 良い: バッチフェッチ
const markets = await getMarkets()
const creatorIds = markets.map(m => m.creator_id)
const creators = await getUsers(creatorIds)  // 1クエリ
const creatorMap = new Map(creators.map(c => [c.id, c]))

markets.forEach(market => {
  market.creator = creatorMap.get(market.creator_id)
})
```

## キャッシング戦略

### Redisキャッシング層

```typescript
class CachedMarketRepository implements MarketRepository {
  constructor(
    private baseRepo: MarketRepository,
    private redis: RedisClient
  ) {}

  async findById(id: string): Promise<Market | null> {
    // まずキャッシュをチェック
    const cached = await this.redis.get(`market:${id}`)

    if (cached) {
      return JSON.parse(cached)
    }

    // キャッシュミス - データベースから取得
    const market = await this.baseRepo.findById(id)

    if (market) {
      // 5分間キャッシュ
      await this.redis.setex(`market:${id}`, 300, JSON.stringify(market))
    }

    return market
  }

  async invalidateCache(id: string): Promise<void> {
    await this.redis.del(`market:${id}`)
  }
}
```

## エラー処理パターン

### 集中エラーハンドラー

```typescript
class ApiError extends Error {
  constructor(
    public statusCode: number,
    public message: string,
    public isOperational = true
  ) {
    super(message)
    Object.setPrototypeOf(this, ApiError.prototype)
  }
}

export function errorHandler(error: unknown, req: Request): Response {
  if (error instanceof ApiError) {
    return NextResponse.json({
      success: false,
      error: error.message
    }, { status: error.statusCode })
  }

  if (error instanceof z.ZodError) {
    return NextResponse.json({
      success: false,
      error: '検証に失敗しました',
      details: error.errors
    }, { status: 400 })
  }

  // 予期しないエラーをログ
  console.error('予期しないエラー:', error)

  return NextResponse.json({
    success: false,
    error: '内部サーバーエラー'
  }, { status: 500 })
}
```

## 認証と認可

### JWTトークン検証

```typescript
import jwt from 'jsonwebtoken'

interface JWTPayload {
  userId: string
  email: string
  role: 'admin' | 'user'
}

export function verifyToken(token: string): JWTPayload {
  try {
    const payload = jwt.verify(token, process.env.JWT_SECRET!) as JWTPayload
    return payload
  } catch (error) {
    throw new ApiError(401, '無効なトークン')
  }
}

export async function requireAuth(request: Request) {
  const token = request.headers.get('authorization')?.replace('Bearer ', '')

  if (!token) {
    throw new ApiError(401, '認証トークンがありません')
  }

  return verifyToken(token)
}

// APIルートでの使用
export async function GET(request: Request) {
  const user = await requireAuth(request)

  const data = await getDataForUser(user.userId)

  return NextResponse.json({ success: true, data })
}
```

## ロギングと監視

### 構造化ロギング

```typescript
interface LogContext {
  userId?: string
  requestId?: string
  method?: string
  path?: string
  [key: string]: unknown
}

class Logger {
  log(level: 'info' | 'warn' | 'error', message: string, context?: LogContext) {
    const entry = {
      timestamp: new Date().toISOString(),
      level,
      message,
      ...context
    }

    console.log(JSON.stringify(entry))
  }

  info(message: string, context?: LogContext) {
    this.log('info', message, context)
  }

  warn(message: string, context?: LogContext) {
    this.log('warn', message, context)
  }

  error(message: string, error: Error, context?: LogContext) {
    this.log('error', message, {
      ...context,
      error: error.message,
      stack: error.stack
    })
  }
}

const logger = new Logger()

// 使用法
export async function GET(request: Request) {
  const requestId = crypto.randomUUID()

  logger.info('マーケットを取得中', {
    requestId,
    method: 'GET',
    path: '/api/markets'
  })

  try {
    const markets = await fetchMarkets()
    return NextResponse.json({ success: true, data: markets })
  } catch (error) {
    logger.error('マーケットの取得に失敗', error as Error, { requestId })
    return NextResponse.json({ error: '内部エラー' }, { status: 500 })
  }
}
```

**覚えておく**: バックエンドパターンは、スケーラブルで保守可能なサーバーサイドアプリケーションを可能にします。複雑さのレベルに合ったパターンを選択してください。
