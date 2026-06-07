# Cloudflare Primitives Lab | Fund Admin-in-a-Box for Alternative Investments

A hands-on lab demonstrating all major Cloudflare Developer Platform primitives through the lens of private markets and alternative investments workflows.

Each lab is a self-contained page with a real-world analog drawn from LP portals, middle-office operations, and investment advisory.

---

## Lab Index

| # | Primitive | Lab Page | Use Case | Real-World Analog |
|---|-----------|----------|----------|-------------------|
| 1 | **R2** | `/lab/r2-report-vault` | GP report vault | Document repository |
| 2 | **KV** | `/lab/kv-market-assumptions` | Market assumptions cache | App config / cache |
| 3 | **Durable Objects** | `/lab/durable-portfolio-room` | Live portfolio session | Stateful workflow / session |
| 4 | **D1** | `/lab/d1-lp-commitments` | LP commitments database | Small SQL app database |
| 5 | **Queues** | `/lab/queues-capital-call` | Capital call processing | Async middle-office pipeline |
| 6 | **Workers AI** | `/lab/ai-memo-writer` | Investment memo writer | AI analyst / copilot |
| 7 | **Vectorize** | `/lab/vectorize-lpa-search` | LPA clause search | Semantic document retrieval |

---

## Labs

### 1. R2 — GP Report Vault

**Page:** `/lab/r2-report-vault`

**Atomic test:** Can your web page upload a document into R2 and fetch it back?

R2 is Cloudflare's S3-compatible object storage. The Workers R2 binding API lets a Worker read and write objects through a bucket binding.

**What the page does**

Upload a file with fund metadata:

```
Fund:    Blackstone Growth VII
Quarter: Q1 2026
File:    fake-quarterly-report.pdf
```

Stored in R2 at:

```
gp-reports/blackstone-growth-vii/2026-Q1/fake-quarterly-report.pdf
```

Then list uploaded reports and click to retrieve any one.

**Real-world relevance**

- LP portal document vault
- Capital call notices
- Distribution notices
- K-1s
- Quarterly GP letters
- Due diligence packets

---

### 2. KV — Market Assumptions Cache

**Page:** `/lab/kv-market-assumptions`

**Atomic test:** Can you save and retrieve a JSON config object from KV?

KV is Cloudflare's globally distributed key-value store, optimized for low-latency reads at scale.

**What the page does**

Create and edit a house assumptions object:

```json
{
  "riskFreeRate": 4.25,
  "privateCreditSpread": 6.75,
  "defaultRate": 2.1,
  "recoveryRate": 55,
  "illiquidityPremium": 2.0
}
```

Saved under the key `market-assumptions:base-case`, then retrieved instantly.

**Real-world relevance**

- Base-case and stress-case house views
- Advisor dashboard defaults
- Model toggles
- Feature flags
- Demo config

---

### 3. Durable Objects — Live Portfolio Room

**Page:** `/lab/durable-portfolio-room`

**Atomic test:** Can one unique object maintain mutable portfolio state across multiple requests?

Durable Objects combine compute with storage and provide a unique instance for coordinated, stateful work — making them the right primitive for sessions and workflows rather than stateless APIs.

**What the page does**

Create a portfolio session:

```
Portfolio:     HNW-Client-123
Starting cash: $1,000,000
```

Then interact via buttons:

- Buy SPY — $100,000
- Buy BND — $200,000
- Add Private Credit Fund — $250,000
- Rebalance
- Reset

The Durable Object holds and mutates session state across all interactions.

**Real-world relevance**

- Advisor proposal room
- Real-time portfolio construction
- Household rebalancing session
- Multi-step investment workflow
- Order staging

---

### 4. D1 — LP Commitments Database

**Page:** `/lab/d1-lp-commitments`

**Atomic test:** Can you insert, query, and update structured private-markets data with SQL?

D1 is Cloudflare's serverless SQLite database. Workers query it through the D1 binding using prepared statements.

**Schema**

```sql
CREATE TABLE commitments (
  id                INTEGER PRIMARY KEY AUTOINCREMENT,
  lp_name           TEXT    NOT NULL,
  fund_name         TEXT    NOT NULL,
  commitment_amount INTEGER NOT NULL,
  called_amount     INTEGER NOT NULL DEFAULT 0,
  vintage_year      INTEGER NOT NULL
);
```

**What the page does**

Add rows such as:

```
LP:         Suresh Family Office
Fund:       Apollo Private Credit Fund III
Commitment: $500,000
Called:     $125,000
Vintage:    2026
```

Automatically calculates and displays:

```
Unfunded commitment = commitment_amount − called_amount
```

**Real-world relevance**

The simplest private-markets data model that still feels real: LP → Fund → Commitment → Called capital → Unfunded → Vintage.

---

### 5. Queues — Capital Call Processing

**Page:** `/lab/queues-capital-call`

**Atomic test:** Can your Worker put a message on a queue and have a consumer process it?

Cloudflare Queues deliver messages with guaranteed delivery, offloading async work from the request path.

**What the page does**

Submit a capital call notice:

```json
{
  "fund":     "Carlyle Credit Opportunities II",
  "lp":       "Suresh Family Office",
  "callAmount": 75000,
  "dueDate":  "2026-07-15",
  "noticeId": "CC-2026-001"
}
```

The page immediately returns: **Queued for processing.**

A queue consumer later writes the status to KV:

```
capital-call:CC-2026-001:status = processed
```

**Real-world relevance**

This mirrors a real middle-office pipeline:

1. Capital call received
2. Validate notice
3. Check LP commitment
4. Notify advisor
5. Store document
6. Update commitment schedule

---

### 6. Workers AI — Investment Memo Writer

**Page:** `/lab/ai-memo-writer`

**Atomic test:** Can your Astro API route call a Cloudflare-hosted LLM and return a useful memo?

Workers AI invokes models running on Cloudflare's GPU network directly from a Worker via `env.AI.run(...)`. The current catalog includes text-generation models such as Qwen3 and QwQ.

**What the page does**

Input form:

```
Asset class:   Private Credit
Strategy:      Senior secured direct lending
Target return: SOFR + 650 bps
Risk:          Covenant-lite structures, refinancing risk
Client type:   HNW taxable investor
```

Generates a 5-bullet investment memo covering:

1. Strategy overview
2. Return drivers
3. Key risks
4. Suitability considerations
5. Questions for diligence

**Real-world relevance**

- Advisor education
- Private markets diligence
- Investment committee memo drafts
- Client-friendly explanations
- Suitability narratives

---

### 7. Vectorize | LPA Clause Search

**Page:** `/lab/vectorize-lpa-search`

**Atomic test:** Can you embed text, store vectors, and retrieve semantically similar clauses?

Vectorize is Cloudflare's vector database. Its Workers binding supports inserting, upserting, and querying vectors. This lab pairs Vectorize with Workers AI embeddings (Qwen3 Embedding) for a full semantic search pipeline.

**What the page does**

Seeds 10 representative LPA clauses:

1. Key person event
2. GP removal for cause
3. No-fault divorce
4. Recycling provision
5. Co-investment rights
6. Advisory committee consent
7. Most favored nation
8. Transfer restrictions
9. Defaulting LP remedies
10. Valuation policy

Example query:

> *Can the LP remove the GP if senior partners leave?*

Returns the semantically closest matching clauses.

**Real-world relevance**

- LPA search
- Side letter search
- Subscription document search
- PPM clause extraction
- GP report semantic search

---

## Mental Model

```
Cloudflare Primitive   →   Mini Lab                →   Real-World Analog
─────────────────────────────────────────────────────────────────────────
R2                         GP report vault             Document repository
KV                         Market assumptions          App config / cache
Durable Objects            Live portfolio room         Stateful workflow / session
D1                         LP commitments              Small SQL app database
Queues                     Capital call processing     Async middle-office pipeline
Workers AI                 Memo writer                 AI analyst / copilot
Vectorize                  LPA clause search           Semantic document retrieval
```

---

*Built on Astro + Cloudflare Pages. Each lab is an isolated, atomic test of a single platform primitive.*
