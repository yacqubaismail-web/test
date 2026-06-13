---
name: ict-pd-array-strategy
description: ICT PD Array Matrix trading strategy the user wants automated in Python (source PDF in Downloads)
metadata: 
  node_type: memory
  type: project
  originSessionId: ae0fc10a-480d-4953-b2c9-04af4c1284ad
---

User wants to automate the **ICT PD Array Matrix** (Premium & Discount Array) strategy, starting in **Python** (backtest first). Source: `C:\Users\GBT A520M RADEON\Downloads\ICT PD Array PDF Download.pdf` (6 pages).

**Core idea:** Draw a Fibonacci over the ICT "dealing range" (established old high → old low). 0.5 = equilibrium. Above 0.5 = **premium** (sell zone), below 0.5 = **discount** (buy zone). Sell at premium, buy at discount. PD arrays = ICT entry tools located inside the right zone.

**PD array tools (entry triggers):** Fair Value Gap (FVG), Inversion FVG, Order Block, Breaker Block, Mitigation Block, Unicorn, NWOG (New Week Opening Gap), NDOG (New Day Opening Gap).
- Bullish FVG = BISI; Bearish FVG = SIBI.

**Workflow:**
1. Daily bias (1D TF): structure shifted up + price in discount → look for buys; shifted down + price in premium → look for sells.
2. Entry (15m/5m TF): after a Market Structure Shift (MSS), find a PD array in the correct zone, wait for price to test it and reject, then enter. Always use a stop loss.

**Markets:** built for NASDAQ/NQ & S&P E-mini; also works on GBP/USD, EUR/USD, XAU/USD.

**Important caveat:** the PDF is conceptual, not mechanical. To code it I must define exact algorithmic rules for FVG/OB/MSS/dealing-range detection. Default codeable definitions agreed: FVG = 3-candle imbalance (bull: candle1.high < candle3.low; bear: candle1.low > candle3.high); Order Block = last opposite candle before impulsive move; MSS = break of recent swing high/low against short-term trend; dealing range = swing high→low fib with 0.5 split. Related: [[tradingview-claude-setup]].

## User's precise mechanical rules (taught live on MNQM2026 daily, 2026-06-13)
- **Dealing range / fib:** drawn over **last 20 daily candles**, levels 0 / 0.25 / 0.5 / 0.75 / 1. The **high-probability zone is between 0.25 and 0.75, split at equilibrium (0.5)**: the **0.5→0.75 band = premium side**, the **0.5→0.25 band = discount side**. PD arrays in these two bands are where the higher-probability bias is found; ignore the deep extremes (>0.75, <0.25).
- **PD array = FVG + Volume Imbalance COMBINED.** When 3 candles form an FVG *and* there is an adjacent volume imbalance (body-to-body gap where wicks overlap but bodies don't), **add the volume imbalance to the FVG range** — the combined zone is the PD array. (Volume imbalance ≠ FVG: VI is the open/close body gap; FVG is the full no-overlap gap.)
- **Inversion FVG (IFVG):** a prior FVG flips polarity once price trades through and **closes** beyond it after a **liquidity sweep**. Live example: price broke the low and took **sellside liquidity at 28,622.25**, then closed back into the premium of the FVG while market was in discount (0.25–0.5) → the bearish FVG **inverts to bullish support** = bullish strength.
- **Respect / invalidation rule (this is the stop logic):** for the bullish IFVG to stay valid, price may **wick down to the 50% of the FVG but must NOT close below 50%**. A candle **close below 50% of the FVG invalidates** the idea. → Stop = below 50% of the FVG / on close below it.
- **Multi-timeframe confirmation:** Daily bias confirmed by a **lower-TF FVG in the same direction** (live: a **4H bullish FVG = the red box** supported the daily bullish idea).
- **50% midpoint = the universal line in the sand (applies on EVERY TF incl. 4H):** for a bullish FVG, candle **closes must stay ABOVE the 50% midpoint** of the FVG. Price showing **weakness/unwillingness to penetrate deeper** into the FVG (closing in the upper half) = **bullish strength**. A **close below 50% = weakness/invalidation**. (Live: the 4H red-box FVG had ~4 candles closing above its 50% = strong buy signal.)
- **SMT divergence (ES vs NQ) — checked at the liquidity sweep, part of HTF confirmation:** compare the two correlated index futures (ES = S&P E-mini, NQ = Nasdaq E-mini). **CONFIRMED DIRECTION (bullish case, live example):** **ES MADE a lower low (Thursday) but NQ did NOT** → NQ is the stronger index refusing the new low → **bullish SMT → buy NQ/MNQ.** (General rule: the instrument that FAILS to make the new low is the stronger one; trade in its direction.) Requires pulling BOTH ES and NQ data when coding.
- **CISD = Change in State of Delivery (the confirmation right after SMT):** after the SMT, the market **breaks structure (BOS/MSS)** → this break is the **evidence of a change in the state of price delivery** (bearish delivery → bullish delivery). **MSS/CISD is NOT a 1H-only thing — it is evidence of the HIGHER-TIMEFRAME bias and shows on the 4H too (and 1H).** Once present we have a **STRONG ONE-SIDED daily bias**. Sequence: Daily bias → sweep → SMT (ES/NQ) → **BOS = CISD (visible 4H & 1H)** → strong one-sided daily bias.
- **Overbought / Oversold (ICT definition, NOT an indicator like RSI):** **Overbought = price reaches an opposing PD array, or is in premium. Oversold = price reaches an opposing PD array, or is in discount (the reverse).**
- **1H = weekly price flow / weekly profile lens.** Use the 1H to read where price is within the **weekly range**. **Time-of-week filter:** if price is already **overbought late in the week (Friday)**, further upside is **less likely even when the bias is bullish** (weekly range often already delivered).
- **PURPOSE of the whole HTF cascade = establish a ONE-SIDED DAILY BIAS.** You *can* trade against the HTF, it's just lower probability. The edge: **get the HTF (daily) bias, then take a 1-minute entry in the SAME direction = higher probability.** The 1m is purely the execution layer aligned to the HTF bias.
- **Current live bias (2026-06-13): BULLISH** on MNQM2026 — sellside sweep at 28,622.25 + daily bullish IFVG (grey box, ~equilibrium) + 4H bullish FVG (red box). Expecting higher prices.

**Still to confirm before coding:** exact volume-imbalance code definition, entry trigger (enter at IFVG retrace?), take-profit target (high 30,806 / buyside?), and which TF's close counts for the 50% invalidation (daily?).

## Project goal & approach
End goal: a **full Python automation** of this strategy. Plan: **backtest first**, then live later (broker TBD). The strategy is a **top-down multi-timeframe SERIES** — conditions cascade Daily → 4H → 1H/15m → 5m → **1-minute execution**. Each TF feeds the next; all must align before entry fires.

## ✅ CONSOLIDATED HTF BIAS CHECKLIST (user's final HTF summary, 2026-06-13)
To establish a **high-probability one-sided daily bias**, on the Daily:
1. **Lookback 20 days**, draw fib: 1 / 0.75 (premium) / 0.5 (equilibrium) / 0.25 / 0.
2. Work the **0.5→0.75 (premium)** and **0.5→0.25 (discount)** bands only.
3. In those bands find an **FVG** (add the **volume imbalance** if present) **or an IFVG** — for bullish, an **IFVG at discount that supports price up**.
4. Confirm with **SMT (ES vs MNQ/NQ)** — stronger index fails to make the new low/high.
5. Confirm with **4H and 1H** in the same direction (FVG support, BOS/CISD).
6. All of the above aligning = the daily bias. Then **execute on 1m in that direction = high probability.**

## ⚡ EXECUTION LAYER — 1-minute entry model (user taught 2026-06-13, bullish case)
Once the one-sided daily bias is set (bullish here), execute on the **1-minute**:
1. **Time:** entries ONLY in the **NY AM killzone 09:30–11:00 ET** (confirmed). All times New York.
2. **Mark liquidity:** the **session highs & lows** of the real sessions — **Asian, London, and New York pre-market** — are the liquidity pools. **All times = New York time (ET), always.** Execution is **purely on the 1-minute** (15m/5m have NO role).
3. **Wait for the trigger:** since we want higher price, **wait for price to sweep an obvious LOW** (take sellside liquidity).
4. **CISD:** after the sweep, price makes a **CISD inside the up-leg** that breaks structure. **The breaking (displacement) leg MUST contain an FVG.**
5. **Entry — FVG path:** when price retraces and **touches that FVG → BUY.** **Stop loss = at/below the sweep low** (the low that took liquidity).
6. **Entry — no-FVG path:** if the breaking leg has **no FVG**, use the **candle that closed the low** (the down-close candle = order block). It must support price upward. Enter only if price **respects** it → **other candles must NOT close below its 50%** (same 50% rule).
- **Target (TP) = FIXED 2R, no matter what (1:2).** TP = entry + 2× risk; risk (1R) = entry − sweep-low SL. **Ignore swings** — it is a hard 2R. (User: "just 2R no matter what.")
- **ALL ANSWERED (spec complete 2026-06-13):** liquidity = **Asian 20:00–00:00 / London 02:00–05:00 / NY pre-market 07:00–09:30** session highs & lows (ET). Trading window = **NY AM killzone 09:30–11:00 ET only**. Execution = **1-minute only**. Target = **hard 2R**.
- **BOTH SIDES confirmed (logic mirrors exactly):**
  - **LONG:** sweep an obvious session **LOW** → bullish **CISD** (up-leg containing an FVG) → **BUY** when price retraces to the FVG → **SL below the sweep low** → **TP = entry + 2R**.
  - **SHORT:** sweep an obvious session **HIGH** → bearish **CISD** (down-leg containing an FVG) → **SELL** when price retraces to the FVG → **SL above the sweep high** → **TP = entry − 2R**.
  - No-FVG fallback (either side): use the CISD/order-block candle; valid only if other candles don't close beyond its 50%.

## 🏗️ BUILD STATUS (updated 2026-06-13)
- **Project location:** `C:\Users\GBT A520M RADEON\ict-pd-array-bot\` (package `ictbot/`). Run: `python run_backtest.py` (also `run_demo.py`, `debug_exec.py`).
- **Data choice:** user picked **free yfinance** for now (NQ=F + ES=F). Reality: only ~6 days of 1m history → engine proven but NOT enough for real backtest stats. Deeper data source still needed later for validation.
- **v0.1 BUILT & WORKING END-TO-END:** data loader (NQ+ES 1m in ET, daily/1h/4h) · detectors (swings, FVG, volume-imbalance, IFVG, order block, sweep, session levels) · bias engine (20-day dealing range + fib + heuristic + manual override + SMT) · 1m execution (sweep→CISD-with-FVG→entry→SL→2R, both sides, OB fallback) · backtest + stats. Execution engine verified detecting real setups on 06-10/11/12.
- **Known gaps to refine WITH user:** (a) `heuristic_bias` is approximate — only flagged 1 directional day; the real HTF bias is discretionary → validate against user's live chart reads / use BIAS_OVERRIDE. (b) tune mechanical defs of sweep/swing/CISD against live reads. (c) get deeper 1m data for meaningful stats. (d) eventually live broker (none chosen yet).
- **Next:** validate engine vs the user's known 2026-06-12 bullish read (compare detected 1m setup to their chart), then iterate on bias accuracy.
- Live chart access: TradingView MNQM2026 via CDP — see [[tradingview-claude-setup]]. Helpers in project cwd: `tv_cdp.py` (info/screenshot/eval), `tv_setsymbol.py <SYMBOL>` (switch symbol, e.g. ESM2026). Screenshots: tv_mnq_daily.png, tv_mnq_4h.png, tv_mnq_1h.png, tv_es_4h.png.
