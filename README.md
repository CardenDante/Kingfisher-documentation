# Kingfisher — Features
---

## 1. The alert (what you see in Telegram)

Every alert that passes the signal filter is posted to **both configured bots**
in real time. Format:

```
🔴 $TDIC · $1.85 · Dreamland Ltd
TDIC Subsidiary signs non-binding MoU for "AI-Powered Image Library"
GlobeNewswire • 05:12:14 ET
──────────────
Float: 3.0M | IO: 0.83% | MC: $7.7M | CTB: 120.0% | Short: 35.00%
⚠️ R/S: 1-for-5 on 2026-04-20
Catalyst: Low · loi_mou (non-binding, off-theme, promo 0.7)
🔥 P(>30%): 0.62 | P(>100%): 0.18         ← only when ranker is trained
🏷️ #AI #MoU #LowFloat #MicroFloat #NanoCap #SqueezeRisk #RecentRS #RS_Trap_Setup #Novel
💡 Non-binding AI MoU on a post-reverse-split shell — fade strength.
🔗 Source
[👀 Watch]  [🚫 Mute 24h]  [⭐ Flag]
```

Line-by-line:

| Line | What | Source |
|---|---|---|
| `🔴/🟡/🟢` prefix | Severity (high / interesting / watch) | Phase 4 ranker |
| `$TICKER` | Ticker(s) extracted from the article | Resolver: cashtag / `(NASDAQ: TICK)` / SEC CIK lookup |
| `$1.85` | Last trade price | Polygon snapshot |
| Company name | Registrant | Finnhub `/stock/profile2` |
| Headline | Verbatim | News source |
| Source • Time | Wire and timestamp | News source (ET-localized) |
| Metadata bar | Float, IO, MC, CTB, Short Float | Finviz Elite (ownership view), iBorrowDesk |
| `⚠️ R/S` | Reverse split within 90 days | Polygon splits + headline regex parser |
| `Catalyst:` | LLM judgment | Ollama Qwen2.5-7B-Instruct |
| `🔥 P(...)` | Probability badge | LightGBM ranker (when trained) |
| `🏷️ #tags` | LLM tags + deterministic taggers + R/S Trap + novelty | See [§5 Detectors](#5-detectors-and-taggers) |
| `💡 trader_read` | One-sentence actionable read | LLM |
| Inline buttons | Feedback | See [§7 Inline buttons](#7-inline-buttons) |

---

## 2. News ingest

Five independent pollers, each running as an async service inside
`kingfisher-ingest`. All publish to the Redis stream `raw_news` with
cross-source deduplication by stable `event_id`.

| Source | Poll interval | Dedup key | Notes |
|---|---|---|---|
| **SEC EDGAR** `getcurrent` Atom | 3s | accession number | Polls 8-K, 6-K, 424B form types in parallel |
| **GlobeNewswire** RSS | 5s | entry GUID | |
| **PRNewswire** RSS | 5s | entry GUID | |
| **BusinessWire** RSS | 5s | entry GUID | Home feed — mostly noise, will be filtered downstream |
| **Benzinga** REST `/api/v2/news` | 5s | Benzinga article id | Tickers come pre-resolved from `stocks[]` |

Each ingestor wraps its source in resilience: a slow or failing fetch
**skips that source for the cycle** without losing the others, exponential
backoff on errors, jittered intervals so the network isn't hit in lockstep.

---

## 3. Ticker resolution

Resolved at ingest time, in this priority order:

1. **Provider-supplied tickers** (Benzinga `stocks[]`).
2. **CIK → ticker** lookup (for EDGAR filings) against the SEC
   `company_tickers.json` index — 10,354 tickers, refreshed each restart.
3. **Cashtags** like `$AAPL` in the headline/body.
4. **Exchange-prefix convention**: `(NASDAQ: VIAV)`, `NYSE: BA`, `OTCMKTS: XYZ`,
   `TSX: ABC`, etc. — the universal PR convention, trusted as authoritative.
5. **Company-name match** against the SEC index, with legal suffix stripping
   (`Acme Holdings Inc` → matches `acme holdings`) and a 6-character minimum.

**Boilerplate blocklist:** `nasdaq`, `nyse`, `amex` are excluded from the
name-match index so generic PR text like "Nasdaq Global Select Market" doesn't
falsely tag every release with `NDAQ`.

---

## 4. Enrichment (metadata bar)

Every event with a resolved ticker fans out **six provider calls in parallel**
in the `kingfisher-enrich` service. Each provider has its own cache TTL and
rate-limiter. A failed provider degrades the corresponding field to `n/a`
without blocking the alert.

| Field | Primary source | Fallback | Cache | Rate limit |
|---|---|---|---|---|
| **Shares outstanding** | EDGAR XBRL `companyfacts` (authoritative) | Finviz, then Finnhub | 24h | SEC 10 req/s |
| **Float** | Finviz Elite (v=131 ownership) | none | 12h | 0.6s min interval (~100/min) |
| **Institutional ownership %** | Finviz Elite | none | 12h | as above |
| **Short float %** | Finviz Elite | none | 12h | as above |
| **Market cap** | Finviz Elite | Finnhub | 12h | as above |
| **Company name / sector** | Finnhub `/stock/profile2` | none | 6h | — |
| **Price + volume** | Polygon `/v2/snapshot` | none | 60s | — |
| **Reverse splits (90d)** | Polygon `/v3/reference/splits` | Headline regex | 12h | — |
| **Cost to borrow** | iBorrowDesk (Cloudflare-protected scrape) | none | 1h | 1s min interval (~60/min) |

The rate limiters smooth out the once-per-cycle backlog drains that used to
produce HTTP 429 storms on ingest startup.

**Headline R/S parser** runs only when Polygon doesn't report a recent split —
catches PR-announced splits before Polygon ingests them, including foreign 6-K
issuers Polygon doesn't cover. Supports `1-for-5`, `1 for 10`, `1:8`, `1-15`,
`1/12`, `5-for-1` ratio styles.

---

## 5. Detectors and taggers

Three layers, applied in order after classification:

### R/S Trap detector — `kingfisher/detect/rs_trap.py`

Fires when a ticker had a reverse split in the last 90 days **and** the current
catalyst looks "themed" (partnership, MoU, contract award, uplisting, or tags
hinting at AI/blockchain/crypto/quantum, or the LLM marked it off-theme).
Result: appends `#RS_Trap_Setup` and bumps severity to at least `interesting`
(never downgrades a higher rank).

### Halt-Resume detector — `kingfisher/detect/halt_resume.py`

Scaffolded, awaiting a halt data source (e.g. Nasdaq Trader RSS halts feed).
The integration point in the sender is already in place — drop the detection
logic into `apply()` to activate.

### Deterministic taggers — `kingfisher/detect/tags.py`

Computed from enrichment numbers — fire regardless of what the LLM tagged:

| Tag | Trigger |
|---|---|
| `#MicroFloat` | float < 5M shares |
| `#LowFloat` | float < 20M |
| `#NanoCap` | market cap < $50M |
| `#MicroCap` | market cap < $300M |
| `#SqueezeRisk` | any two of: CTB ≥ 50%, short float ≥ 20%, float < 20M |
| `#RecentRS` | reverse split within 90 days (any catalyst) |
| `#ForeignIssuer` | EDGAR source + form 6-K |

---

## 6. LLM classification

Service `kingfisher-classify` calls the OpenAI-compatible chat endpoint at
`LLM_BASE_URL` (Ollama by default) for every enriched event. Output is a
strict JSON object validated against the `Classification` schema:

```json
{
  "catalyst_type": "loi_mou",
  "binding": false,
  "off_theme_for_company": true,
  "promotional_language_score": 0.7,
  "catalyst_quality": "low",
  "expected_pattern": "fade",
  "tags": ["AI", "MoU", "LowFloat"],
  "trader_read": "Non-binding AI MoU on a post-reverse-split shell — fade strength."
}
```

Catalyst types: `offering`, `registered_direct`, `atm`, `ma`, `partnership`,
`loi_mou`, `clinical_data`, `fda_regulatory`, `earnings`, `guidance`,
`contract_award`, `uplisting`, `going_concern`, `investigation_litigation`,
`buyback`, `insider_activity`, `stock_split`, `index_inclusion`, `other`.

Quality: `low | medium | high`. Pattern: `runner | fade | dump | muted | unclear`.

**Graceful degradation:** if the LLM endpoint is down or returns malformed JSON,
the classifier returns a default `Classification` with `model=None`. The alert
still fires (assuming it passes the filter); the catalyst section is just
omitted from the message.

**Novelty check (opt-in):** when `EMBEDDINGS_URL` is set, every headline is
embedded with bge-m3 (or whatever model is configured), compared against the
last 90 days in Qdrant via cosine similarity, and tagged `#Novel` when the
similarity is below 0.85 (i.e. novelty score ≥ 0.15). The check naturally
collapses multilingual wire echoes — translated copies of the same release
embed close together and the second one isn't novel.

---

## 7. Inline buttons

Every alert ships with a three-button inline keyboard on **every configured
bot**. Taps land in `kingfisher-feedback`, which polls each bot's
`getUpdates` queue concurrently.

| Button | Action | Storage |
|---|---|---|
| 👀 **Watch** | Mark this alert as interesting | `alerts_sent.user_action='watch'` (DB) |
| 🚫 **Mute 24h** | Suppress this ticker for 24 hours | `mute:{TICKER}` Redis key, TTL 24h |
| ⭐ **Flag** | Flag for later review (labeled-dataset signal) | `alerts_sent.user_action='flag'` |

After a tap the keyboard collapses to a single confirmation chip
(`👀 Watched` / `🚫 Muted 24h` / `⭐ Flagged`) so it's visually clear the action
landed. The same news_event_id is updated regardless of which bot you tapped on.

---

## 8. Telegram commands

DM either bot:

| Command | Result |
|---|---|
| `/top` | Top 10 ranked alerts of the last 24h (empty until the ranker is trained) |
| `/status` | Stream depths, alerts last hour / last 24h, last-alert time, active mutes |

Both commands work from either configured bot — the feedback service handles
each bot's update queue independently and replies in the same chat.

---

## 9. Signal filter (noise control)

The sender drops any classified event that doesn't meet quality bars before
posting to Telegram. Defaults are tuned for the small-cap catalyst use case:

| Setting | Default | Behavior |
|---|---|---|
| `SENDER_MIN_QUALITY` | `medium` | Drops `low` catalysts |
| `SENDER_REQUIRE_TICKER` | `true` | Drops events with no resolved ticker |

**R/S Trap detector overrides both filters** — if the trap fires, the alert
goes out regardless of LLM quality call or ticker resolution.

**Per-ticker rate limit:** one alert per ticker per 10 minutes (Redis
`ratelimit:alert:{ticker}` with TTL 600).

**Mute:** Redis `mute:{ticker}` (24h) — checked before the rate limit.

Drop accounting lives in the `kingfisher_events_processed_total{outcome="skipped"}`
Prometheus counter, with `sender.filtered` log lines carrying the specific reason.

---

## 10. Multi-bot broadcast

The sender ships every alert to all configured destinations
(currently two), each with its own interactive keyboard. The feedback
service polls each bot's `getUpdates` in parallel — taps on either bot
write to the same `alerts_sent` row and the same Redis mute keys.

```
# .env
TELEGRAM_BOT_TOKEN=…         # primary
TELEGRAM_CHANNEL_ID=…
TELEGRAM_BOT_2_TOKEN=…       # optional secondary
TELEGRAM_BOT_2_CHAT_ID=…
```

Adding a third bot is a matter of adding `TELEGRAM_BOT_3_TOKEN`/`_CHAT_ID`
and a couple of lines in `alert/run.py` and `alert/feedback.py` to load it.

---

## 11. Pipeline architecture

Five independent services connected by four Redis streams. Each service owns
exactly one stage and is independently restartable.

```
ingest    ──> raw_news        ──> enrich    ──> enriched_news
                                                       │
                                                       v
                          classified_news <──  classify
                                  │
                                  v
                                alert       ──> Telegram (both bots)
                                                 ▲     │
                                                 │     v
                                              feedback (button taps + commands)

   any failure  ───────────────────────────────> dead_letter
```

- **All streams** are capped at 50k entries (`maxlen=50000`, approximate).
- **All consumer groups** start at `$` — only new entries are processed after
  a fresh service start, not stale backlog.
- **Failed events** go to `dead_letter` with service, reason, and original
  payload — nothing is silently dropped.

---

## 12. Observability

Each service exposes a Prometheus endpoint and a `/health` probe:

| Service | Port |
|---|---|
| `kingfisher-ingest` | 9101 |
| `kingfisher-enrich` | 9102 |
| `kingfisher-classify` | 9103 |
| `kingfisher-alert` | 9104 |
| `kingfisher-feedback` | 9105 |

Counters / gauges shipped:

| Metric | Labels | What |
|---|---|---|
| `kingfisher_events_published_total` | `stream`, `source` | XADDs to each stream |
| `kingfisher_events_processed_total` | `service`, `outcome` | `ok / skipped / failed` |
| `kingfisher_provider_calls_total` | `provider`, `outcome` | Outbound API call results |
| `kingfisher_stream_depth` | `stream` | XLEN, refreshed every 15s by ingest |
| `kingfisher_dead_letter_total` | `service`, `reason` | DLQ entries |

`/health` returns `200 ok` if the service's HTTP loop is responsive.

Structured logs go to stdout as JSON (one event per line), captured by systemd
journal: `journalctl -u 'kingfisher-*' -f`.

---

## 13. Backtest harness

Replay historical news through the live scoring + detection code paths into a
separate `backtest_alerts` table, with same-day open→high move from Polygon
as the label. Two commands:

```bash
# Predict + label
python -m kingfisher.backtest.replay \
    --id 2026-Q1 --from 2026-01-01 --to 2026-03-31

# Evaluate
python -m kingfisher.backtest.evaluate --id 2026-Q1
```

Output:
- Total alerts, label coverage, mover counts
- Top-decile recall: of all >30% movers in the window, what fraction landed in
  the top 10% of scores
- Precision@K/day: average daily precision in the top-K-scored alerts

Designed to be rerun monthly to detect model drift.

---

## 14. Configuration

All runtime configuration is environment-based and loads from
[.env](.env.example). Categories:

- **Data providers:** Benzinga, Polygon, Unusual Whales, Finviz Elite, Finnhub
- **Telegram:** primary bot token + chat_id, optional secondary bot
- **SEC EDGAR:** required `EDGAR_USER_AGENT` contact header
- **Infrastructure:** `DATABASE_URL`, `REDIS_URL`, `QDRANT_URL` (defaults match
  the bundled docker-compose)
- **LLM:** `LLM_BASE_URL`, `LLM_MODEL`, `LLM_API_KEY` (Ollama by default)
- **Embeddings (opt-in for novelty):** `EMBEDDINGS_URL`, `EMBEDDINGS_MODEL`
- **Ranker:** `RANKER_MODEL_DIR` (default `training/ranker_models`)
- **Sender filter:** `SENDER_MIN_QUALITY`, `SENDER_REQUIRE_TICKER`

Each service reads `.env` at startup; restart a service to pick up config changes.

---

## 15. Operations

Production deployment is via systemd — units in [ops/systemd/](ops/systemd/).
Install once:

```bash
sudo cp ops/systemd/kingfisher-*.service /etc/systemd/system/
sudo systemctl daemon-reload
sudo systemctl enable --now \
    kingfisher-ingest kingfisher-enrich kingfisher-classify \
    kingfisher-alert kingfisher-feedback
```

After that:

```bash
journalctl -u 'kingfisher-*' -f                # tail every service
systemctl status kingfisher-alert              # one service
sudo systemctl restart kingfisher-alert        # pick up new code / .env
sudo systemctl stop 'kingfisher-*'             # overnight pause
```

Each unit has `Restart=on-failure` with a 5-second back-off — crashes
self-heal, the cluster survives reboots.

---

## 16. Data stores

| Store | Purpose | Persistence |
|---|---|---|
| Postgres + TimescaleDB | `tickers`, `news_events`, `corporate_actions`, `market_snapshots` (hypertable), `alerts_sent`, `backtest_alerts` | Docker volume `kingfisher_timescale-data` |
| Redis | Pipeline streams (`raw_news`/`enriched_news`/`classified_news`/`dead_letter`), provider caches, rate-limit + mute keys, dedup keys | Docker volume `kingfisher_redis-data` (AOF on) |
| Qdrant | 1024-dim headline embeddings, 90-day cosine-similarity window for novelty | Docker volume `kingfisher_qdrant-data` |

Schema lives in [kingfisher/store/sql/](kingfisher/store/sql/), applied
automatically on fresh DB init. The Phase 5 `backtest_alerts` table is also
created idempotently on first `replay.py` run for already-initialized DBs.

---

## 17. Out-of-the-box vs. needs-training

| Capability | Today | What unlocks the rest |
|---|---|---|
| News ingest, dedup, ticker resolution | ✅ live | — |
| Metadata bar (float / IO / MC / CTB / short / R/S) | ✅ live | — |
| Catalyst classification + tags + trader read | ✅ live (base Qwen2.5-7B) | LoRA fine-tune for higher consistency ([training/README.md](training/README.md) stage 2) |
| R/S Trap, deterministic tagging, signal filter | ✅ live | — |
| Multi-bot broadcast + inline button feedback | ✅ live | — |
| `/top`, `/status` commands | ✅ live | — |
| Backtest replay + evaluation | ✅ runnable | Accumulated data or Polygon flat-file backfill |
| Probability badge + severity emoji | ⏳ scaffolded | Train the LightGBM ranker ([training/README.md](training/README.md) Phase 4 section) |
| Novelty `#Novel` tag | ⏳ opt-in | Serve `bge-m3` embeddings and set `EMBEDDINGS_URL` |
| Halt-Resume detector | ⏳ scaffolded | Wire to Nasdaq Trader halts RSS feed |
