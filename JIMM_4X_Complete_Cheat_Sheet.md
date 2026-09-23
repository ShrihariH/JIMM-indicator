# JIMM 4X — Practical Trading Cheat Sheet
### For use with the author’s JIMM 4X backtest workbook

> **Purpose:** Convert the patterns found in the supplied JIMM 4X backtest workbook into a disciplined, risk-controlled operating playbook.
>
> **Important:** The workbook supports the *signal/timeframe/parameter* observations below. It does **not** publish a complete audited stop-loss, position-sizing, drawdown, Sharpe, expectancy or portfolio-equity methodology. The risk-management rules in this document are therefore a **proposed execution wrapper**, not claims about the author’s exact rules.

---

## 1. The core setup

### Primary setup: 1H regime + 15M execution

**Recommended baseline for Indian index/large-stock trading**

| Component | Setting |
|---|---|
| Higher timeframe | **1 Hour** |
| Execution timeframe | **15 Minutes** |
| J | **200** |
| M | **0.1 & 0.01** |
| MD | **48, 104, 36** |
| ICL / IBL | **20 / 60** as the balanced baseline |
| Direction | Prefer **Long-only** for the more robust Nifty behaviour |
| Entry | Only after the 15M candle closes and the setup is still valid |
| Initial risk | **0.50% of account equity per trade** |
| Hard daily loss limit | **1.5% of equity or 3R, whichever comes first** |
| Max correlated positions | **2** |
| Default profit management | Scale at 1R; trail the remainder |

### Why 20/60 rather than blindly using the fastest or slowest setting?

The workbook repeatedly tests 10/30, 20/60 and 40/120. The 20/60 setting is a sensible **middle ground** for a live implementation:

- 10/30 is more responsive and often produces stronger raw Nifty 15M numbers.
- 40/120 is slower and is useful when you want fewer/noisier-signal trades.
- 20/60 sits between them and reduces the danger of overfitting to one extreme.

This is a **risk-management choice**, not a claim that the author proved 20/60 is mathematically optimal.

---

# 2. What the workbook actually says about timeframe

This is the strongest part of the evidence.

For Nifty annual intraday testing, the 15M timeframe is remarkably persistent:

- Long-only: **44 of 48** populated target/ICL/IBL cells were positive.
- The yearly Long-only 15M blocks are approximately:
  - 2021: **9/12 positive**
  - 2022: **11/12 positive**
  - 2023: **12/12 positive**
  - 2024: **12/12 positive**

The 10M timeframe is much less stable and shows an unusual deterioration relative to 15M.

The key lesson is:

> **Do not assume that more frequent signals are better. JIMM shows a strong timeframe dependency and a non-linear timeframe response.**

### Practical hierarchy

```text
1H        = regime / direction
15M       = primary trigger / execution
30M       = slower alternative / confirmation
5M        = tactical only; not primary
1M        = avoid as a core JIMM timeframe
```

---

# 3. The complete entry rule

## A. First: determine the regime on 1H

For a LONG:

1. 1H JIMM must be bullish/aligned.
2. Price should not be in an obvious breakdown against that regime.
3. Do not enter because of a single intrabar colour change.
4. Wait for the 1H state to be confirmed on candle close.

For a SHORT:

1. 1H JIMM must be bearish/aligned.
2. 15M must independently confirm the bearish direction.
3. Do not short merely because a 15M candle turns red against a bullish 1H regime.

**Conservative mode:** trade only in the direction of the 1H regime.

---

## B. Then: wait for the 15M setup

The preferred sequence is:

```text
1H directional regime
        ↓
15M JIMM alignment
        ↓
pullback / pause
        ↓
15M continuation confirmation
        ↓
ENTRY
```

### Preferred LONG entry

Wait for:

- 1H bullish regime
- 15M bullish JIMM alignment
- a pullback/pause rather than buying a highly extended candle
- a 15M confirmation candle that closes back in the bullish direction

Then enter:

> **Above the high of the 15M confirmation candle**, preferably using a stop entry rather than anticipating the move.

### Preferred SHORT entry

Mirror the process:

- 1H bearish
- 15M bearish
- pullback/pause
- bearish confirmation candle
- enter below the confirmation candle low

---

# 4. The most important entry filter: DON'T CHASE

A good JIMM signal can still be a bad entry if the move is already extended.

Skip the trade when:

- the signal candle is abnormally large,
- price has already travelled a large distance away from the recent structure,
- the stop would have to be unusually wide,
- the entry is immediately into a major nearby resistance/support zone.

### Simple mechanical rule

Before entering, calculate:

**ATR(14) on 15M**

If the proposed stop distance is:

- **< 0.8 ATR:** probably too tight → widen to structure-based stop.
- **0.8–1.5 ATR:** acceptable zone.
- **> 1.5 ATR:** **skip the trade**.

This prevents the strategy from turning a good signal into a poor risk/reward entry.

---

# 5. Stop-loss rule

The workbook does not provide a definitive author SL formula. Use the following as a separate risk wrapper.

## LONG

Set the initial SL:

> **Below the most recent meaningful 15M swing low, with a small volatility buffer.**

Practical formula:

```text
SL = Swing Low − 0.20 to 0.25 × ATR(14, 15M)
```

Then check the resulting entry-to-SL distance.

### Accept only when

```text
0.8 ATR ≤ risk distance ≤ 1.5 ATR
```

If the stop is wider than 1.5 ATR:

> **Do not reduce the stop just to make the position fit. Reduce/skip the trade.**

## SHORT

Mirror:

```text
SL = Swing High + 0.20 to 0.25 × ATR(14, 15M)
```

---

# 6. Position sizing — the part that actually controls survival

Never choose quantity first.

Choose the **rupee risk** first.

### Formula

```text
Maximum trade risk = Account Equity × 0.50%

Position Size =
Maximum Trade Risk ÷ Stop Distance
```

For an ₹10,00,000 account:

```text
Maximum risk = ₹5,000
```

If the trade has a ₹25 stop:

```text
Quantity = ₹5,000 / ₹25
         = 200 shares
```

For derivatives, incorporate:

- lot size,
- premium,
- brokerage,
- taxes/charges,
- slippage.

Do not size from margin available.

---

# 7. Profit target: don't make the target the strategy

This is one of the strongest conclusions from the workbook.

The author tests different intraday targets including roughly:

**0.6%, 1%, 1.5%, 2%**

and the Nifty 15M behaviour remains positive across many of these variations.

That suggests:

> **JIMM's directional edge is more important than a magical fixed target.**

Therefore use **R-based management** rather than worshipping one percentage target.

---

# 8. Recommended exit framework

## Exit 1 — hard stop

Exit immediately.

No averaging down.

No moving the stop farther away.

---

## Exit 2 — partial at 1R

At approximately **+1R**:

- book **30–50%**
- reduce remaining risk

Example:

```text
Entry = 100
SL = 98
Risk = 2 points

1R = 102
2R = 104
3R = 106
```

At 102:

- book part of the position
- move remaining stop to approximately breakeven **after accounting for costs/slippage**

---

## Exit 3 — let the runner follow JIMM

For the remaining position:

Stay in the trade while:

- 15M JIMM remains aligned,
- price holds the trailing structure,
- the 1H regime remains supportive.

Trail below successive 15M swing lows for LONG.

Trail above successive 15M swing highs for SHORT.

---

## Exit 4 — 1H regime failure

For the remaining position:

> **A confirmed 1H reversal is a hard exit signal.**

Do not wait for a large percentage loss after the higher-timeframe thesis has failed.

---

# 9. Optional time-stop

This is **not from the author's published backtest**; it is a risk-control addition.

When a trade goes nowhere:

```text
After ~6–8 15M candles
≈ 90–120 minutes
```

review it.

If:

- price has not reached roughly +0.5R,
- momentum has visibly stalled,
- the JIMM alignment is weakening,

close the position rather than tying up capital indefinitely.

Do **not** apply this blindly to a strongly trending trade.

---

# 10. The preferred risk/reward structure

A clean trade should look like:

```text
             2R–3R+ runner
                  ↑
                  │
        ──────────┤
                  │
               1R partial
                  ↑
                  │
ENTRY ────────────┤
                  │
                  │
                -1R
                  │
                 SL
```

### Minimum quality gate

Do not take a trade where the logical first target is less than about:

> **1.5R**

Prefer:

> **2R+ potential**

because the strategy needs room for the runner to compensate for inevitable losing trades and slippage.

---

# 11. The “A+” JIMM trade

A high-quality setup has this sequence:

```text
1H JIMM
   ↓
clear directional regime
   ↓
15M JIMM agrees
   ↓
controlled pullback
   ↓
structure survives
   ↓
confirmation candle closes
   ↓
stop < 1.5 ATR
   ↓
first objective ≥ 1.5R
   ↓
ENTER
```

The worst setup is essentially the opposite:

```text
1H unclear
   +
15M fighting 1H
   +
large extended candle
   +
wide stop
   +
nearby resistance
   ↓
SKIP
```

---

# 12. When NOT to trade JIMM

Avoid initiating a new position when:

### 1. 1H regime is unclear

If the higher timeframe is switching repeatedly, wait.

### 2. 15M is repeatedly flipping

This is a classic chop condition.

### 3. Stop is too wide

If the structural stop exceeds about 1.5 ATR, the trade is usually not attractive enough for this framework.

### 4. Price has already run

Don't enter after the majority of the impulse has already happened.

### 5. Major event risk

Avoid opening a fresh leveraged trade immediately before major scheduled market-moving events.

### 6. You are trying to recover a previous loss

The JIMM signal should decide the trade, not the previous P/L.

---

# 13. 15M vs 30M

Use the two timeframes differently.

## 15M

Best when:

- you want earlier entries,
- you can monitor the market,
- you want more opportunities.

## 30M

Best when:

- you want fewer signals,
- you want less noise,
- you can tolerate later entries.

The workbook shows meaningful positive behaviour on both, but the **15M Nifty results are particularly persistent**.

---

# 14. 5M: tactical, not the default

The author's final worksheet contains several securities for which 5M is listed as usable.

However, the broader Nifty tests do not support treating 5M as universally superior.

Therefore:

> Use 5M only after the 1H + 15M thesis already exists.

A good hierarchy is:

```text
1H → direction
15M → trade thesis
5M → optional entry refinement
```

Not:

```text
5M signal → immediate trade
```

---

# 15. Long-only vs Long & Short

The workbook contains an important asymmetry.

For Nifty's annual intraday testing:

- Long-only 15M is positive in approximately **44/48** cells.
- Long & Short 15M is positive in approximately **39/48** cells.

That does not prove that shorting is bad.

It does show that the **Long-only variant is more consistently positive in this particular Nifty test set**.

### Practical implementation

For a conservative system:

> **Longs are the primary mode.**

For a two-sided system:

> Add shorts only when the 1H regime is clearly bearish and the 15M setup independently confirms.

---

# 16. Swing version

For a slower swing implementation:

```text
1D = primary regime
2H / 1H = execution
```

Use the same principles:

- trend alignment,
- pullback,
- confirmation,
- structural stop,
- R-based sizing,
- trailing rather than relying entirely on a fixed target.

The workbook tests swing targets around:

**7%, 10%, 15%, 20%, 25%**

but the data does not establish one universal optimal target.

Therefore a live system should not assume that, for example, 20% is intrinsically superior simply because some assets produced larger raw P/L under it.

---

# 17. JIMM parameter card

Keep the core parameters unchanged unless you deliberately run a new out-of-sample test.

```text
J = 200
M = 0.1 & 0.01

ICL / IBL:
10 / 30
20 / 60   ← balanced baseline
40 / 120  ← slower confirmation

MD:
48, 104, 36
```

### Important

Do not continuously optimize these after every few weeks of live data.

That turns a strategy into a moving target and creates parameter overfitting.

---

# 18. What the manual-trade section warns us about

The manual sample in the workbook is surprisingly instructive.

Many trades:

- moved favourably after entry,
- showed meaningful MFE,
- but still ended in losses.

This suggests that **entry accuracy alone is not enough**.

The lesson for live trading is:

> **A JIMM signal is only half the system. Trade management determines how much of the favourable excursion you actually keep.**

That is why the proposed 1R partial + trailing runner is important.

---

# 19. Daily risk protocol

### Conservative

```text
Risk / trade       = 0.50%
Max open risk      = 1.00%
Daily loss limit   = 1.50%
Max new trades     = 3
```

### Aggressive

```text
Risk / trade       = 0.75%
Max open risk      = 1.50%
Daily loss limit   = 2.00%
```

Avoid going above this simply because the last few trades were profitable.

The JIMM workbook does not provide enough drawdown statistics to justify a highly leveraged risk model.

---

# 20. Three-strike discipline

Stop initiating new trades for the day after:

```text
3 consecutive full-SL losses
OR
daily loss limit reached
OR
JIMM regime becomes consistently choppy
```

This is a risk-control rule, not an assertion about JIMM's statistical losing streak.

---

# 21. The one-page execution checklist

## BEFORE MARKET

```text
[ ] 1H JIMM direction identified
[ ] Market is not in obvious 1H chop
[ ] 15M JIMM agrees
[ ] ICL/IBL setting = 20/60 baseline
[ ] J = 200
[ ] M = 0.1 & 0.01
[ ] MD = 48,104,36
[ ] Nearby support/resistance checked
[ ] ATR(14) 15M calculated
```

## BEFORE ENTRY

```text
[ ] Pullback/pause occurred
[ ] Confirmation candle closed
[ ] No chasing
[ ] Logical structural SL identified
[ ] SL distance <= ~1.5 ATR
[ ] Potential reward >= 1.5R
[ ] Position size calculated from risk
[ ] Trade does not violate daily risk limit
```

## AFTER ENTRY

```text
[ ] Hard SL placed immediately
[ ] Never widen SL
[ ] At +1R → partial profit
[ ] Remaining position trailed
[ ] 15M structure monitored
[ ] 1H regime monitored
```

## EXIT

```text
[ ] Hard SL
OR
[ ] 1R partial + trailing exit
OR
[ ] 15M structure breaks
OR
[ ] 1H regime reverses
OR
[ ] Optional time-stop after ~90–120m if stagnant
```

---

# 22. The simplest version to remember

```text
                JIMM 4X
                   │
             1H = DIRECTION
                   │
                   ▼
             15M = SETUP
                   │
             Pullback
                   │
             Confirmation
                   │
                   ▼
                ENTRY
                   │
             SL = SWING
             + ATR BUFFER
                   │
                   ▼
                +1R
             Take Partial
                   │
                   ▼
           TRAIL THE REST
                   │
       ┌───────────┴───────────┐
       ▼                       ▼
  15M structure            1H reversal
       │                       │
       └──────────┬────────────┘
                  ▼
                 EXIT
```

---

# 23. What is actually proven vs inferred

## Strongly supported by the workbook

- 15M is an unusually persistent Nifty timeframe in the tested annual intraday data.
- 1M is poor in the tested Nifty data.
- 10M is unstable relative to 15M.
- ICL/IBL materially affects results.
- Fixed target changes are less important than the signal/timeframe interaction.
- Long-only has stronger consistency than Long & Short in the Nifty 15M annual sample.
- Different assets behave very differently.
- The report's headline “return” is not the same thing as CAGR.
- The workbook contains spreadsheet/data-quality issues that require caution.

## Not proven by the workbook

- A specific stop-loss percentage is optimal.
- A specific ATR multiplier is optimal.
- 0.50% account risk is optimal.
- 1R/2R/3R exits are the author's exact exit methodology.
- A specific position-sizing method is the author's method.
- Any one setup is guaranteed to remain profitable live.

---

# 24. Final live-trading configuration

### CORE JIMM 15M MODEL

```text
REGIME       : 1H
EXECUTION    : 15M
J            : 200
M            : 0.1 & 0.01
MD           : 48,104,36
ICL/IBL      : 20/60
DIRECTION    : Prefer LONG when using conservative mode

ENTRY:
1H aligned
→ 15M aligned
→ pullback
→ confirmation candle close
→ enter above/below confirmation structure

STOP:
Beyond 15M swing
+ 0.20–0.25 ATR buffer
Accept ~0.8–1.5 ATR
Skip if wider

RISK:
0.50% equity / trade

MANAGEMENT:
+1R → partial
Remaining → trail 15M structure

FULL EXIT:
Hard SL
OR 1H regime failure
OR 15M structure/JIMM reversal
OR optional time-stop when stagnant

MINIMUM TRADE QUALITY:
~1.5R potential
Prefer 2R+
```

---

## Bottom line

The workbook does **not** justify the idea that there is one magical “JIMM best parameter.”

What it does support is a much more useful conclusion:

> **JIMM appears strongest when used as a slower directional/filtering framework and then executed on a timeframe where the signal is stable. For the Nifty tests supplied, 15M is the clearest robust execution zone, while 1H is a sensible higher-timeframe regime filter.**

The safest way to deploy that observation is **not** to maximize leverage. It is to combine the signal with a fixed account-risk budget, a volatility-aware structural stop, partial profit-taking and a trailing exit.

