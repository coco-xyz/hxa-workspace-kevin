# OFAC Sentinel

Automated OFAC sanctions monitoring, blockchain address tagging, and transaction association analysis.

Built for [TrustIn](https://www.trustin.info/) — a blockchain KYT/AML compliance provider.

---

## Problem

When OFAC adds a new sanctioned entity, compliance teams must:
1. Detect the update (manual check of treasury.gov)
2. Find the entity's blockchain addresses (manual search across multiple platforms)
3. For entities without known addresses, use investigative methods to discover them
4. Trace all transactions to/from those addresses and determine association with sanctioned funds

This process currently takes **days to weeks** per entity. OFAC Sentinel automates steps 1, 2, and 4 end-to-end, and assists step 3 with heuristic address discovery.

---

## Architecture

```
┌─────────────────────────────────────────────────────────┐
│                    OFAC Sentinel                         │
├─────────────┬─────────────┬──────────────┬──────────────┤
│  Monitor    │  Resolver   │  Discoverer  │  Tracer      │
│  (Step 1)   │  (Step 2)   │  (Step 3)    │  (Step 4)    │
│             │             │              │              │
│ SDN diff    │ Arkham API  │ Clustering   │ RPC / API    │
│ RSS/news    │ Etherscan   │ heuristics   │ Multi-hop    │
│ Changelog   │ OFAC addrs  │ Pattern      │ Association  │
│ parser      │ Dune labels │ matching     │ scoring      │
└──────┬──────┴──────┬──────┴──────┬───────┴──────┬───────┘
       │             │             │              │
       ▼             ▼             ▼              ▼
┌─────────────────────────────────────────────────────────┐
│                   Shared Data Layer                      │
│  - Entity DB (sanctioned entities + metadata)            │
│  - Address DB (entity → addresses, multi-chain)          │
│  - Transaction DB (traced txs + association scores)      │
│  - Report generator (compliance-ready output)            │
└─────────────────────────────────────────────────────────┘
```

---

## Modules

### 1. Monitor — OFAC SDN Change Detection

**Input:** OFAC SDN list (XML/CSV from treasury.gov)
**Output:** New/modified sanctioned entities with metadata

- Daily cron fetches SDN list from `https://sanctionslistservice.ofac.treas.gov/api/`
- Diffs against stored snapshot, detects new entries
- Extracts: entity name, type (individual/organization), country, sanction program, known blockchain addresses (OFAC now publishes these directly for crypto entities)
- Supplements with RSS monitoring of OFAC press releases and fintech news

**Tech:** Node.js, cron scheduler, XML/CSV parser
**External deps:** OFAC public API (no auth needed)

### 2. Resolver — Known Address Lookup

**Input:** Sanctioned entity name/identifier
**Output:** Known blockchain addresses across chains

Sources queried (in priority order):
1. **OFAC SDN list** — addresses published directly by OFAC
2. **Arkham Intelligence** — `intel.arkm.com` entity pages, largest labeled address database
3. **Etherscan/Tronscan/BTC explorers** — address labels and tags
4. **Dune Analytics** — community-maintained label tables
5. **Chainalysis public data** — sanctioned address lists

For each source, the resolver normalizes addresses to a standard format: `{chain}:{address}`.

**Tech:** Node.js, API clients for each source
**External deps:** Arkham API (free tier available), Etherscan API (free tier), blockchain RPCs

### 3. Discoverer — Heuristic Address Identification

**Input:** Entities where Resolver found zero or incomplete addresses
**Output:** Candidate addresses with confidence scores

Methods:
- **Clustering heuristics:** For exchanges, identify hot wallets by transaction pattern (high frequency, many small inbound txs, few large outbound). Uses common-input-ownership and change-address detection for UTXO chains.
- **Deposit address pattern:** Known exchange deposit addresses share format characteristics — the discoverer checks if the entity operates a known exchange and applies deposit pattern matching.
- **Cross-reference:** If Entity A is sanctioned and Entity B (already labeled) has significant transaction volume with an unlabeled address, that address becomes a candidate for Entity A.

All candidates require **human confirmation** before entering the Address DB.

**Tech:** Node.js, blockchain RPC clients, statistical analysis
**External deps:** Full/archive node access (Ethereum, Tron, Bitcoin) or commercial node providers (Alchemy, Infura)

### 4. Tracer — Transaction Association Analysis

**Input:** Target address (sanctioned entity's address)
**Output:** Association report — all source addresses that sent funds, with sanction match status

Process:
1. Pull all inbound transactions to target address (via RPC or explorer API)
2. Extract unique source addresses
3. For each source address:
   - Direct match: check against sanctioned address DB → **direct association**
   - 1-hop: check if the source received funds from a sanctioned address → **indirect association (1 hop)**
   - 2-3 hop: recursive trace with configurable depth → **indirect association (N hops)**
4. Score each association: direct = 1.0, 1-hop = 0.7, 2-hop = 0.4, 3-hop = 0.2
5. Generate compliance report with transaction hashes, amounts, timestamps, and association paths

**Tech:** Node.js, graph traversal, blockchain RPCs
**External deps:** Archive node access (for historical tx data), or explorer APIs with pagination support

---

## Data Model

```
Entity {
  id: string (SDN entry number)
  name: string
  type: "individual" | "organization" | "vessel"
  country: string
  programs: string[]          // e.g., ["CYBER2", "IRAN"]
  addedDate: date
  source: "ofac_sdn"
  metadata: object            // raw SDN fields
}

Address {
  address: string             // normalized: lowercase, no prefix
  chain: "eth" | "btc" | "trx" | "sol" | ...
  entityId: string            // FK → Entity
  source: "ofac" | "arkham" | "etherscan" | "manual" | "heuristic"
  confidence: number          // 1.0 = confirmed, <1.0 = heuristic
  discoveredAt: date
  confirmedBy: string | null  // human reviewer for heuristic entries
}

Association {
  targetAddress: string       // sanctioned entity's address
  sourceAddress: string       // address that sent funds
  chain: string
  hops: number                // 0 = direct, 1-3 = indirect
  score: number               // association strength
  path: string[]              // tx hashes forming the association chain
  totalValue: string          // total value transferred (in native token)
  firstTx: date
  lastTx: date
}
```

---

## Resource Requirements

### Infrastructure
- **Server:** 1 VPS (2 vCPU, 4GB RAM) — sufficient for monitoring + API calls + light analysis
- **Database:** SQLite for MVP, PostgreSQL for production
- **Blockchain access:** Commercial RPC provider (Alchemy/Infura free tier for MVP, paid for production multi-chain)
- **Scheduler:** Cron or PM2-based task scheduler

### External API Keys (Free Tier Sufficient for MVP)
- Etherscan API key (free, 5 calls/sec)
- Arkham Intelligence (free tier for basic lookups)
- Alchemy/Infura (free tier: 300M compute units/month)
- Optional: Tronscan, BscScan for multi-chain support

### Development
- **Phase 1 (Monitor + Resolver):** ~1-2 weeks
  - SDN diff engine, Arkham/Etherscan integration, basic address DB
  - Deliverable: daily alert of new sanctioned entities + known addresses
- **Phase 2 (Tracer):** ~1-2 weeks
  - Transaction pulling, multi-hop graph traversal, association scoring
  - Deliverable: association reports for any input address
- **Phase 3 (Discoverer):** ~2-3 weeks
  - Clustering heuristics, candidate scoring, human review UI
  - Deliverable: candidate address suggestions with confidence scores
- **Phase 4 (Dashboard + Reports):** ~1 week
  - Web UI for monitoring, address management, report generation
  - Compliance-ready PDF/CSV export

### Team
- 1 full-stack developer (Node.js + blockchain familiarity)
- 1 compliance domain advisor (part-time, for validation and edge cases)
- Agent team (Jessie + Lucy) for development, Boot for QA/DevOps

---

## Tech Stack

- **Runtime:** Node.js (ESM)
- **Database:** SQLite (MVP) → PostgreSQL (production)
- **Blockchain:** ethers.js / web3.js for EVM chains, custom for UTXO
- **Scheduling:** node-cron or PM2 cron
- **API framework:** Hono or Fastify (lightweight)
- **Frontend:** Vanilla HTML/JS (same stack as ClawFeed) or React if dashboard complexity warrants it

---

## MVP Scope

Phase 1 deliverable — the Monitor + Resolver pipeline:
1. Automated daily SDN diff → detect new sanctioned entities
2. Auto-resolve known addresses via Arkham + Etherscan
3. Output: structured JSON feed of `{entity, addresses[], chain}` per new sanction
4. Notification: alert to configured channel (Lark, Telegram, email)

This alone replaces the most time-consuming part of the manual workflow (Steps 1-2) and provides immediate value.

---

## License

MIT
