# Kingfisher — User Guide

Your real-time small-cap catalyst feed. Alerts land in Telegram the moment a
press release or SEC filing hits the wire — already filtered, enriched, and
scored so you can decide in 5 seconds whether to **act, watch, or skip**.

This guide tells you how to read what shows up.

---

## What you see when an alert lands

```
🔴  $TDIC  ·  $1.85  ·  Dreamland Ltd
TDIC Subsidiary signs non-binding MoU for "AI-Powered Image Library"
GlobeNewswire • 05:12:14 ET
──────────────
Float: 3.0M  |  IO: 0.83%  |  MC: $7.7M  |  CTB: 120.0%  |  Short: 35%
⚠️ R/S: 1-for-5 on 2026-04-20
Catalyst: Low · loi_mou (non-binding, off-theme, promo 0.7)
🏷️  #AI #MoU #LowFloat #MicroFloat #NanoCap #SqueezeRisk #RecentRS #RS_Trap_Setup
💡 Non-binding AI MoU on a post-reverse-split shell — fade strength.
🔗 Source
   [👀 Watch]   [🚫 Mute 24h]   [⭐ Flag]
```

Three blocks, top to bottom:

1. **The header** — who, how much, what
2. **The metadata bar** — the structural setup
3. **The judgment** — quality call, tags, one-line read

---

## 1. The header

```
🔴  $TDIC  ·  $1.85  ·  Dreamland Ltd
TDIC Subsidiary signs non-binding MoU for "AI-Powered Image Library"
GlobeNewswire • 05:12:14 ET
```

| Element | What to take from it |
|---|---|
| **🔴 / 🟡 / 🟢** | Traffic light — high-conviction / interesting / watch (only shown when the ranker is trained) |
| **$TICKER** | The stock you'd actually trade |
| **$1.85** | Live last-trade price |
| **Company name** | Confirms the ticker resolved to the right name |
| **Headline** | The verbatim wire headline |
| **Source • Time ET** | Which wire, when it hit (Eastern Time) |

---

## 2. The metadata bar — the structural setup

```
Float: 3.0M  |  IO: 0.83%  |  MC: $7.7M  |  CTB: 120.0%  |  Short: 35%
⚠️ R/S: 1-for-5 on 2026-04-20
```

These five numbers tell you what *kind* of stock this is — before you even
read the headline. Internalize them:

| Field | Meaning | Watch for |
|---|---|---|
| **Float** | Shares actually available for public trading | **< 10M is twitchy. < 5M is explosive.** Low float means a small order can move price hard. |
| **IO** (Institutional Ownership) | What % of shares are held by funds | **< 5%** means no smart money has touched this — pure retail vehicle. **> 40%** means there's real ownership. |
| **MC** (Market Cap) | Total company value | **< $50M** is nano-cap manipulation territory. **< $300M** is small/micro-cap. |
| **CTB** (Cost to Borrow) | What shorts pay to borrow shares, annualized | **> 50%** = shorts are crowded; squeeze risk is real. |
| **Short** (Float Short %) | What % of float is sold short | **> 20%** combined with high CTB and low float = textbook squeeze setup. |

### The R/S warning line

```
⚠️ R/S: 1-for-5 on 2026-04-20
```

Means: this company did a reverse stock split (e.g. every 5 old shares
became 1 new share) on April 20, 2026 — within the last 90 days.

**Why it matters:** Companies do reverse splits when their stock is dying.
They buy time against delisting, then often dump fresh shares into the new
price. The classic pattern is: **R/S → buzzword press release → pump →
offering announcement → dump.** When you see this line, the next press
release from that ticker is almost always promotional, not material.

---

## 3. The judgment

```
Catalyst: Low · loi_mou (non-binding, off-theme, promo 0.7)
🏷️  #AI #MoU #LowFloat #MicroFloat #NanoCap #SqueezeRisk #RecentRS #RS_Trap_Setup
💡 Non-binding AI MoU on a post-reverse-split shell — fade strength.
🔗 Source
```

### Catalyst quality

| Label | What it means | What you typically do |
|---|---|---|
| **High** | Hard, binding, on-theme, material news | Lean in — this can sustain |
| **Medium** | Real news but qualifiers (uncertain, partial, or single-source) | Watch closely; trade the reaction, not the news |
| **Low** | Non-binding, promotional, off-theme, or immaterial | Don't chase; this is fade territory or pure noise |

The qualifiers in parentheses tell you *why* the quality is what it is:

- `non-binding` — LOI, MoU, "intends to," "in talks" (vs. signed/announced)
- `off-theme` — the catalyst has nothing to do with the company's actual business
- `promo 0.7` — the wording is hype-heavy (1.0 = pure pump language)

### The tag glossary

You'll see two kinds of tags — **structural** (computed from numbers) and
**LLM** (judgment from the press release content).

| Tag | What fires it | What it tells you |
|---|---|---|
| `#MicroFloat` | float < 5M | Sharp moves possible on small volume |
| `#LowFloat` | float < 20M | Still tight, watch for momentum |
| `#NanoCap` | market cap < $50M | Manipulation risk; one trader can move it |
| `#MicroCap` | market cap < $300M | Below most institutional mandates |
| `#SqueezeRisk` | high CTB + crowded shorts + low float | Real squeeze setup if catalyst lands |
| `#RecentRS` | reverse split in last 90 days | Failing-company red flag |
| `#RS_Trap_Setup` | RecentRS + themed buzzword PR | The classic R/S → pump pattern firing right now |
| `#ForeignIssuer` | Files SEC 6-K (Hong Kong, China, etc.) | Less regulatory protection, often promotional setups |
| `#Novel` | This headline isn't a duplicate of recent news | Worth a second look — genuinely new |
| `#AI` / `#MoU` / `#Partnership` / `#FDA` / `#Earnings` / etc. | LLM-extracted from content | What the news is actually about |

### The trader read (`💡`)

One sentence summarizing the setup, generated by the model. It's the **5-second
read** — if you only had time to read one line, this is it.

Examples:
- *"Non-binding AI MoU on a post-reverse-split shell — fade strength."*
- *"Confirmatory Phase 3 readout on lead candidate — gap-and-go candidate if early prints clean."*
- *"Routine 10% buyback announcement on a $4B name — likely muted reaction."*

---

## 4. The buttons

```
[👀 Watch]   [🚫 Mute 24h]   [⭐ Flag]
```

Tap any one — the keyboard collapses to a confirmation chip
(`👀 Watched` / `🚫 Muted 24h` / `⭐ Flagged`) so you know it landed.

| Button | What it does | When to use |
|---|---|---|
| **👀 Watch** | Marks this alert as worth following | You're tracking the open price action or waiting for follow-on news |
| **🚫 Mute 24h** | Silences that ticker entirely for 24 hours | A ticker is spamming low-quality news and you don't want to see it again today |
| **⭐ Flag** | Flags for review (helps train the system) | Something looks unusually right or wrong — your feedback teaches the model |

**Mute is per-ticker, not per-bot** — if both bots are running, muting a
ticker silences it everywhere. The mute auto-expires after 24 hours.

---

## 5. DM commands

Send these as a regular message to the bot:

| Command | Reply |
|---|---|
| `/status` | Live system health: how many alerts went out in the last hour, queue depths, active mutes, when the last alert landed |
| `/top` | Top 10 ranked alerts of the last 24h (only populated when the probability ranker is trained) |

---

## 6. When to act, watch, or skip — at a glance

Quick decision matrix based on what you see:

| You see | Likely action |
|---|---|
| 🔴 `Catalyst: High` · binding · `#LowFloat` | **Trade-able** — strong asymmetric setup |
| 🟡 `Catalyst: Medium` · binding · big-cap | **Watch the open** — react to price, not headline |
| 🟢 `Catalyst: Low` · non-binding · off-theme | **Skip or fade** — likely a pump |
| `#RS_Trap_Setup` fired | **Highest skepticism** — classic small-cap dump setup |
| `#SqueezeRisk` + `Catalyst: High` | **Asymmetric long setup** — squeeze fuel + real news |
| No tags, no tickers | **Probably already filtered out** — you shouldn't be seeing this one |
| Float `n/a` + CTB `n/a` + no catalyst line | **Foreign / thinly-covered name** — careful, low data quality |

---

## 7. What you *won't* see in your DM

The bot **only sends alerts that pass quality filters**. By default it drops:

- News without a resolvable ticker (Visa Canada street soccer, etc.)
- News the model judges `low` quality (catalyst not material, off-theme, promo)
- Duplicate alerts for a ticker within 10 minutes
- Alerts on any ticker you've muted in the last 24 hours

This is intentional. Roughly **90% of small-cap news is noise**, and the bot's
job is to surface the 10% that's worth a glance. If you ever feel the feed is
**too quiet**, the operator can loosen the filter; if it's too noisy, they
can tighten it.

---

## 8. Sources behind the alerts

The bot watches five wires and one filing source continuously:

| Source | What it carries |
|---|---|
| **SEC EDGAR** | 8-K (US companies), 6-K (foreign issuers), 424B (offerings/prospectus) |
| **GlobeNewswire** | Press releases, often from small-caps |
| **PRNewswire** | Press releases (broad coverage) |
| **BusinessWire** | Press releases (broader, more general) |
| **Benzinga** | Curated market news, comes with tickers attached |

Each ticker's structural data (float, IO, market cap, short data, etc.) is
pulled live from **Finviz Elite, Finnhub, Polygon, iBorrowDesk, and EDGAR
XBRL** when an alert fires — so the numbers in the bar reflect the moment
the news hit, not yesterday's snapshot.

---

## 9. Things to know

- **Alerts are signal, not advice.** The bot tells you *something happened
  and here's the structural setup*. The trade decision is yours.
- **Off-hours are quieter.** Most material news hits between 4 AM and 9:30
  AM ET (pre-market) and during the regular session. Overnight you may go
  hours without an alert — that's normal.
- **Same alert, two bots.** If both bots are configured, every alert lands
  in both. Tap a button on either one — it counts the same.
- **Buttons don't place trades.** They record your judgment so the system
  learns. Trades happen in your broker.
- **The model can be wrong.** If a catalyst quality call looks off, ⭐ Flag
  it — that feedback gets reviewed.

---

## 10. Quick reference card

```
TRAFFIC LIGHT       🔴 high  ·  🟡 interesting  ·  🟢 watch

QUALITY             High     hard, binding, on-theme, material
                    Medium   real but qualified
                    Low      non-binding, promo, off-theme  → fade/skip

KEY DANGER TAGS     #RS_Trap_Setup    classic post-RS pump pattern
                    #ForeignIssuer    Hong Kong / China shells
                    #NanoCap          < $50M, manipulation risk

KEY OPPORTUNITY TAGS #SqueezeRisk     short squeeze setup
                     #MicroFloat      < 5M float, explosive
                     #Novel           not a rehash

BUTTONS             👀 Watch    bookmark
                    🚫 Mute     silence ticker 24h
                    ⭐ Flag     train the model

COMMANDS            /status   pipeline health
                    /top      top scored alerts (24h)
```

That's it. Read the header, glance the bar, take the 5-second judgment, tap a button if it matters. The bot does the rest.
