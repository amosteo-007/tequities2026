# Tokenized Equities on Base: Cross-Venue Liquidity and Weekend Price Discovery

**Plan version:** v0.2 (30 September 2026)  
**Owner:** Amos  
**Status:** Research plan for discussion; collection not yet implemented

## 1. Objective

Understand where tokenized-equity liquidity resides on Base, how executable prices differ across venues, and what happens when the underlying equity market closes.

**Central question:** when weekend prices move, are multiple venues incorporating new information, or is limited trading moving a market with deteriorating liquidity?

The dashboard should answer:

1. Where do trades execute, and where can a user actually buy or sell a specified quantity?
2. How do spreads, depth, quote availability, and cross-venue gaps change by session?
3. When NBBO is unavailable, what reference information remains, and how old is it?
4. Do weekend moves persist into the next underlying trading session, or reverse as liquidity returns?
5. Do mint/redemption constraints, concentrated liquidity, or collateral liquidations explain these patterns?

Backing integrity remains a supporting control. Regulatory scenario analysis is deferred until the applicable primary texts and product scope are verified.

## 2. Evidence and assumptions

[Base's stock directory](https://www.base.org/stocks) lists AAPLc, NVDAc, METAc, and GOOGLc, links to contracts and prospectuses, and identifies Aerodrome and several trading interfaces. This is a discovery starting point, not proof of pool depth, successful execution, or complete venue coverage.

Distinguish three records:

- **Tape:** observed swaps and fills. Our reconstructed token tape is not automatically the underlying stock's consolidated tape.
- **Executable liquidity:** pool curves, standing orders, and available maker quotes at a stated size and time.
- **References:** underlying stock quotes/trades, official closes, and oracle reports, each with source, timestamp, and session.

Lower weekend volume, wider spreads, LP withdrawals, and larger gaps are hypotheses to test. No new trade means no new transaction price; it does not prove unchanged fair value.

[FINRA's extended-hours explanation](https://www.finra.org/investors/insights/extended-hours-trading) distinguishes regular-session NBBO from fragmented extended-hours trading. Label vendor quotes by coverage and session; a returned price alone does not establish a live national benchmark.

Verify mint/redemption and underlying-market hours separately. [Ondo describes weekend minting/redemption](https://ondo.finance/blog/real-24-7-trading-for-tokenized-stocks) for selected products; this does not establish the same capability for Coinbase-issued tokens or every Ondo asset.

## 3. Proposed stock universe

### Initial cohort

| Underlying | Candidate Base token | Purpose |
|---|---|---|
| AAPL | AAPLc | Familiar underlying benchmark; compare token liquidity with underlying liquidity |
| NVDA | NVDAc | Candidate for semiconductor/AI news event studies and weekend repricing |
| META | METAc | Different business/news profile within the original cohort; cross-sectional comparison |
| GOOGL | GOOGLc | Completes the original cohort; company/regulatory news event candidates |

These are analytical roles, not measured liquidity rankings. Verify contracts, share class, decimals, multiplier, transfer restrictions, and activation dates before ingestion. Map GOOGL to the correct Alphabet share class, not GOOG.

**Pilot:** backfill all four, then select the deepest and thinnest token using a fixed seven-day discovery window: median two-sided $1k/$10k quote availability and depth, with trade count secondary. Freeze selection before evaluating later weekends. Keep zero-trade periods and failed quotes.

### Conditional expansion

| Underlying | Purpose | Admission condition |
|---|---|---|
| TSLA | Additional event-driven weekend cases | Verified token and measurable Base liquidity |
| AMZN or MSFT | Broaden the initial cohort | Verified listing and adequate history |
| SPY or QQQ ETF | Broad-market comparison with single-name moves | Verified token; record ETF structure separately |
| COIN | Explore links to crypto-market activity | Verified token; retain employee-policy/publication review |

Do not invent suffixes or assume expansion names are available on Base. The initial cohort is technology-heavy and cannot establish results for all equities. A second issuer's version of an initial stock is the first cross-issuer priority.

## 4. Venue shortlist and comparison design

| Priority | Venue/service | Role | Evidence and admission check |
|---|---|---|---|
| P0 | Aerodrome, including Slipstream where applicable | Direct pool execution and liquidity | Listed by Base; enumerate factory-created pools for each verified token and confirm activity |
| P0 conditional | Uniswap v3 on Base; v4 if relevant pools are discovered | Independent direct-pool comparison | [Deployment documentation](https://developers.uniswap.org/docs/protocols/v3/deployments) establishes deployment, not stock-pool availability; verify pools and any hooks |
| P1 | 0x API / Matcha route | Routed execution versus direct pools | [0x API](https://docs.0x.org/api-reference/api-overview) supports Base; test exact tokens, fees, eligibility, and sources |
| P1 | 1inch Classic Swap API | Second router comparison | [Quote API](https://business.1inch.com/portal/documentation/apis/swap/classic-swap/methods/v6.1/chainId/quote/method/get) supports Base; validate tokens and routes |
| P2 conditional | CoW Swap | Intent/auction execution | Listed by Base; verify token support, Base settlement, quote semantics and accessible history |
| P2 conditional | 01 exchange or True Markets | Candidate order-book/other execution mechanism | Listed by Base; actual mechanism, live stock markets and L2 access remain unverified |
| P2 conditional | Ondo product and its verified execution venues | Same underlying across issuers | Verify chain, token, economic rights, conversion ratio, schedule and quote access |

**Start with:** all relevant Aerodrome pools versus verified Uniswap pools for the same token, plus 0x and 1inch route quotes. If no second direct venue exists, report concentration and compare pools/routes; do not manufacture a cross-venue result.

Keep three layers separate:

1. **Same contract, different pools/venues:** cleanest fragmentation comparison.
2. **Same contract, different routers:** execution-quality comparison; shared pool liquidity cannot be added twice.
3. **Same underlying, different issuers/chains:** economic-exposure comparison, adjusted for rights and conversion ratios. A gap is not automatically executable arbitrage.

Use USDC pairs first. Include other stablecoin/WETH routes where material, converting with contemporaneous quote-asset/USD prices and recording stablecoin deviations. Lending protocols supply collateral/liquidation context, not spot venue depth.

## 5. Session model and weekend mechanisms

Store UTC and classify equity sessions using America/New_York, DST, holidays, early closes, and instrument-specific halts. Singapore time is an optional display.

| State | Reference treatment | Question |
|---|---|---|
| Regular | Valid contemporaneous NBBO and underlying trades | How closely do token executions track the underlying? |
| Pre/post-market | Labelled extended-hours quotes/trades with coverage | How does fragmentation change? |
| Overnight | Verified ATS/overnight feed if available | Does token pricing follow an observable overnight market? |
| Underlying fully closed | Official close and last valid later-session reference, separately timestamped | What happens without a current underlying execution reference? |
| Halt | Explicit halt status | Does token trading continue under venue rules? |
| Data unavailable | Missing reference distinct from closure | Is this an ingestion gap rather than a market phenomenon? |

[Chainlink's market-hours documentation](https://docs.chain.link/data-streams/market-hours) describes Sunday evening reopening for supported 24/5 equity streams. Use the actual instrument/feed schedule and marketStatus, not timestamps alone. Do not label the entire Friday regular-close-to-Monday-open interval as fully closed.

Mechanisms to test:

- AMMs can continue swapping against deposited inventory, subject to contract/access rules. An oracle existing elsewhere does not automatically anchor the pool.
- Concentrated positions can move out of range or be withdrawn/repositioned. TVL can remain high while nearby depth falls. [Aerodrome documents range-based liquidity](https://github.com/aerodrome-finance/docs/blob/main/content/liquidity.mdx).
- Book makers can cancel, widen, reduce size, or skew quotes without a trade.
- RFQ providers can reduce size, widen prices, or stop responding. Indicative quotes are not necessarily firm commitments.
- Underlying hedging can become unavailable while token-to-token arbitrage continues. Record creation/redemption access, inventory constraints, fees, eligibility and settlement timing.
- Oracle-dependent execution/lending can retain old references, apply limits, or pause. Verify actual implementation.

## 6. Dashboard and metric definitions

### A. Where liquidity lives

Show token -> venue -> pool, protocol version, quote asset, fee tier, TVL, active liquidity, trade count, notional, and LP concentration. Show router attribution separately.

Maintain pool-leg volume and reconstructed user-order volume separately. Multi-hop legs, router events, and pool events must not all count as separate user purchases. Preserve ambiguity when reconstruction is uncertain. Addresses are not unique people.

### B. Executable prices and depth

Collect both sides at approximately $1k, $10k, and $100k. Convert these targets into fixed share-equivalent quantities using a recorded reference, then compare the same quantity across venues/sides. For exact-input-only buy APIs, solve for target output or keep notional quotes separate from equal-quantity comparisons.

- ask_v(q) = total quote currency paid / share-equivalent quantity received.
- bid_v(q) = total quote currency received / share-equivalent quantity sold.
- spread_bps(q) = 10,000 × (ask_v(q) − bid_v(q)) / ((ask_v(q) + bid_v(q)) / 2).
- Buy premium = 10,000 × (ask_v(q) / P_ref − 1).
- Sell discount = 10,000 × (1 − bid_v(q) / P_ref).
- Depth = maximum buy/sell size within 25/50/100 bps of a stated pre-trade reference, with fee treatment explicit.
- Quote availability = valid responses / scheduled attempts, by side/size; distinguish no liquidity, access failure, timeout, rate limit and collector downtime.

Show trading fees and curve impact in execution prices; itemize gas/router fees and provide all-in prices. Do not deduct fees twice. Simulated execution and gas estimates are not realized fills.

Use the actual pool invariant/quoter. Concentrated pools require ticks, liquidity and fees; reserve ratios and trade-size/reserve approximations are not universal. AMM bid/ask here is a size-dependent synthetic quote, not a literal order book.

### C. Tape, quote age and cross-venue gaps

- Plot fills, last-trade age and no-trade intervals alongside current buy/sell curves. Empty trade candles stay empty; forward-filled prices must not imply fresh trades.
- Measure inter-trade times, size distribution, buy/sell imbalance and active pools/participants.
- At synchronized snapshots calculate max_v bid_v(q) − min_v ask_v(q) for the same transferable token and quantity.
- A positive result is a candidate crossed quote, not proven profit: account for gas, timing, eligibility, shared pool state and route interference.
- Report dispersion, gap duration and return to a predeclared tolerance. Calibrate tolerances to fees and normal-session behavior.
- Track oracle observation age/status, underlying quote age, collection delay and latest pool state separately.

### D. Weekend and reopening study

Compare per-hour volume/counts, spread distributions, depth, quote availability and LP changes by session. Normalize duration rather than equating one weekend with one trading day.

Show drift from Friday's official close and the last valid later-session reference separately. These are historical anchors, not weekend fair value. Track synchronous independent-pool moves, isolated outliers, stablecoin changes and timestamped company news.

Study first valid overnight reopening, Monday pre-market and regular opening separately, skipping holidays. Record the underlying opening auction where available, first valid quotes, and basis at +1/+5/+30 minutes. Describe persistence/reversal; neither proves prediction of the opening price.

### E. Backing and collateral context

- Supply × verified share-equivalent multiplier versus attested holdings at matching timestamps; document cash/settlement treatment.
- Mint/burn versus issuance/redemption records where available; a burn alone does not prove completed underlying redemption.
- Corporate actions based on product terms; do not hard-code the previous draft's dividend fee/withholding assumptions without verification.
- Lending exposure, liquidations, contemporaneous oracle state and pool changes.
- Unknown custody data remains unknown; chain data alone does not establish backing.

## 7. Data sources and access proposal

**Recommended stack:** Dune for discovery/backfill, archive-capable Base RPC for state, prospective quote collection, and Massive for licensed underlying data. Add actual Chainlink feed history and Blue Ocean overnight data as access permits. These are proposed providers; no subscriptions or credentials have been obtained.

| Need | Source / retrieval | Limitation or action |
|---|---|---|
| Canonical tokens/terms | [Base directory](https://www.base.org/stocks), linked prospectuses, issuer attestations and BaseScan/code | Save dated evidence; ticker matching is insufficient |
| Swaps/transfers/LP events | [Dune catalog](https://dune.com/data): dex.trades filtered to Base and verified addresses; raw/decoded Base logs as fallback | Verify schema/protocol coverage; new tokens may be missing; check USD valuations independently |
| Raw reconciliation | [Alchemy Base eth_getLogs](https://www.alchemy.com/docs/chains/base/base-api-endpoints/eth-get-logs), receipts | API key, range limits and history retention need testing |
| Pool depth | Archive RPC historical eth_call, verified pool/quoter ABI, ticks and liquidity events | Test historical state access before choosing a plan; logs alone are not a full depth snapshot |
| Router offers/routes | [0x](https://docs.0x.org/api-reference/api-overview), [1inch](https://business.1inch.com/portal/documentation/apis/swap/classic-swap/methods/v6.1/chainId/quote/method/get), Base chain ID 8453 | Keys/limits; capture prospectively; historical fills cannot recover unaccepted offers |
| Book/RFQ data | Admitted venue's official API: snapshots, incremental updates, sequences, timestamps; maker response logs | Access/mechanisms unverified; fills cannot recover canceled or offchain orders |
| Underlying NBBO/trades | Preferred: [Massive quotes](https://www.massive.com/docs/rest/stocks/trades-quotes/quotes) and [SIP flat files](https://massive.com/docs/flat-files/stocks/overview) | Request four-symbol sample/business terms; verify conditions, corrections, session coverage and retention; bars cannot measure spreads |
| Alternative equities vendor | [Databento](https://databento.com/docs), exact dataset/schema to be selected | Require coverage specification; direct-exchange/partial BBO is not automatically consolidated NBBO |
| Overnight underlying | Blue Ocean direct/vendor feed; [L1/L2 through Bloomberg](https://blueocean-tech.io/2025/09/30/pr-newswire-blue-ocean-technologies-real-time-overnight-trading-data-now-available-via-bloomberg-solutions/) | Confirm machine-readable historical entitlement/symbols; one ATS is a venue reference, not a national market |
| Oracle reports | [Chainlink Data Streams](https://docs.chain.link/data-streams) or actual onchain Data Feed proxy/history | Identify feed/schema and consumer; Streams history may need credentials; chain history omits reports never submitted |
| Calendars/halts/actions | Vendor reference products and official exchange/issuer notices | Retain point-in-time events/raw prices; apply token adjustments separately |
| Custody/issuance | Published attestations; authorized internal records | Public cadence may preclude intraday reconciliation; private records stay internal |

Obtain business-use, display, derived-data and redistribution terms before putting licensed data into a public dashboard. Request provider-specific costs; retail pricing is not a business budget. Aggregator charts cannot substitute for fills or depth.

### Collection specification

1. Registry: chain/token address, issuer, underlying/share class, decimals, multiplier history, pools, protocol version, quote asset, access rules and evidence.
2. Backfill from verified deployment dates; reconcile curated swaps with raw events on active and quiet samples.
3. Capture both sides/three sizes every 60 seconds; use 5-second sampling around selected closure/reopening/events where limits permit. Record every attempt. Four names × two routers × three sizes × two sides = 48 requests/minute before retries, excluding direct pools.
4. Pin direct-pool quotes to one block. Store router request/response times, available block references, TTL, fees, latency, routes and taker assumptions. Label comparisons outside the declared timing tolerance asynchronous.
5. Reconstruct historical AMM quotes only where state/logic permit and label them simulated. They are not historical firm RFQ or router offers.
6. Join the latest valid underlying observation at or before each timestamp; never future quotes or minute-bar closes. Assess sensitivity to chain timestamp granularity and rapid price moves.
7. Retain raw data separately from derived tables, with versions, collection health, block hashes, transaction/log identifiers and reorg/finality handling.
8. Use Dune for public-chain aggregates; access-controlled storage/BI for high-frequency quotes, licensed references and internal records.

**Backfillable:** chain events and, subject to archive support, pool state; licensed equity history where entitled. **Prospective or specialist historical access required:** unfilled RFQs, router offers and offchain book changes. Begin those collectors before the full dashboard is ready.

## 8. Study design and delivery

Target four complete prospectively observed weekends; extend to eight if sparse. Historical chain data supplements but cannot replace missing quote history. Separate quiet/news weekends, holidays, incentives and launch periods. Report counts and day/weekend-level uncertainty rather than treating every minute as independent.

| Phase | Timing | Deliverable / acceptance |
|---|---|---|
| Discovery/access | Week 1 | Verified registry, four-name venue coverage matrix, vendor samples; unavailable venues marked |
| Capture/backfill | Weeks 1–2 | Tape, pool replay pilot, quote collectors; event reconciliation and missingness documented |
| Session/reference joins | Weeks 2–3 | Price/depth panels; no future joins; closure, DST, holiday and missing-data checks |
| Weekend analysis | After four complete captured weekends, typically Weeks 5–6 | Matched session metrics, per-weekend counts and coverage |
| Extensions | Weeks 6–8, conditional | Order books/RFQ, second issuer, backing/liquidations only where access and comparability established |
| Review pack | After analysis | Dated findings with reproducible methods; legal/comms review before publication |

First demo: one deep and one thin token, verified pools, three-size buy/sell curves, fills, reference age and a complete closure/reopening window. A single-pool market is a concentration finding.

## 9. Governance and retained follow-ups

- **Two builds:** public-chain plus licensed reference data only as permitted for the public build; authorized private issuance/custody/maker records stay in the internal Coinbase build.
- **Publication:** retain the original plan's legal/comms sign-off requirement before external analysis of Coinbase's product.
- **Trading:** observational study; quote collection requires no executed trades. Confirm employee restrictions before any separate trading activity.
- **Backing:** supporting workstream; unavailable private records should not block public liquidity analysis.
- **Regulatory appendix:** the prior draft's SEC release, symbol/ADV caps, file number and disclosure obligations require primary-text verification before reuse. Any eventual scenario must define jurisdiction/product applicability, volume, affiliate aggregation and reporting day; a shadow scenario is not a compliance finding.

## 10. Immediate next steps

- [ ] Verify four canonical contracts, accounting multipliers and terms; archive evidence.
- [ ] Enumerate Aerodrome/Uniswap pools and rank measured activity/depth.
- [ ] Test historical state and raw-event reconciliation for one token/weekend.
- [ ] Obtain 0x/1inch access; test all sides/sizes and record unsupported tokens/access restrictions.
- [ ] Request Massive/Databento samples and business terms; assess overnight access separately.
- [ ] Identify actual Chainlink feeds/consumers and closed-market behavior.
- [ ] Confirm issuer/AP mint/redemption hours, weekend size limits, commitments and settlement timing.
- [ ] Start prospective quotes once access is available; freeze pilot selection after discovery.
- [ ] Verify whether candidate book venues provide actual stock L2 data and recoverable history.
- [ ] Confirm internal sponsor/data budget; build the first demo from available public data.

The first discovery decision is whether a meaningful same-token multi-venue comparison exists. The first research result is whether weekend repricing coincides with broad price changes or reduced executable liquidity. Neither is assumed in advance.
