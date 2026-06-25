# Market Structure Engine Specification

## Title
Market Structure Engine — Complete Mathematical Specification

## Purpose
This document defines every concept, rule, condition, and algorithm required to implement the Market Structure Engine (MSE) in Pine Script. Every definition is deterministic. No subjective language is used. No discretionary interpretation is required by the implementor.

## Status
Draft

## Version
0.1.0

## Last Updated
2026-06-25

## Author
mrmtsuruya

---

## Table of Contents

1. [Notation and Conventions](#1-notation-and-conventions)
2. [Configuration Parameters](#2-configuration-parameters)
3. [Primitive Definitions](#3-primitive-definitions)
   - 3.1 [Swing High](#31-swing-high)
   - 3.2 [Swing Low](#32-swing-low)
4. [Structural Labels](#4-structural-labels)
   - 4.1 [Higher High (HH)](#41-higher-high-hh)
   - 4.2 [Lower High (LH)](#42-lower-high-lh)
   - 4.3 [Higher Low (HL)](#43-higher-low-hl)
   - 4.4 [Lower Low (LL)](#44-lower-low-ll)
5. [Break of Structure (BOS)](#5-break-of-structure-bos)
   - 5.1 [BOS Definition](#51-bos-definition)
   - 5.2 [Strong BOS](#52-strong-bos)
   - 5.3 [Weak BOS](#53-weak-bos)
   - 5.4 [Failed BOS](#54-failed-bos)
6. [Change of Character (CHoCH)](#6-change-of-character-choch)
7. [Market Structure Shift (MSS)](#7-market-structure-shift-mss)
8. [Trend State](#8-trend-state)
9. [Market Regime](#9-market-regime)
10. [Displacement](#10-displacement)
    - 10.1 [ATR Requirements](#101-atr-requirements)
    - 10.2 [Body Ratio](#102-body-ratio)
    - 10.3 [Wick Ratio](#103-wick-ratio)
11. [Structure Score](#11-structure-score)
12. [Edge Cases](#12-edge-cases)
13. [Invalid Cases](#13-invalid-cases)
14. [Worked Examples](#14-worked-examples)
15. [Pseudo-Code](#15-pseudo-code)

---

## 1. Notation and Conventions

### 1.1 Bar Indexing

- Bars are indexed by integer `i`, where `i = 0` is the current (most recently closed) bar.
- `i = 1` is one bar prior to the current bar.
- Higher `i` values denote older bars.
- All bars referenced in this specification are **closed bars** unless explicitly stated.

### 1.2 OHLC Notation

For bar at index `i`:

| Symbol | Meaning |
|--------|---------|
| `O[i]` | Open price of bar `i` |
| `H[i]` | High price of bar `i` |
| `L[i]` | Low price of bar `i` |
| `C[i]` | Close price of bar `i` |

### 1.3 Derived Values

| Symbol | Formula | Meaning |
|--------|---------|---------|
| `Body[i]` | `abs(C[i] - O[i])` | Absolute body size |
| `Range[i]` | `H[i] - L[i]` | Full candle range |
| `UpperWick[i]` | `H[i] - max(O[i], C[i])` | Upper wick size |
| `LowerWick[i]` | `min(O[i], C[i]) - L[i]` | Lower wick size |
| `Bullish[i]` | `C[i] > O[i]` | Candle direction is bullish |
| `Bearish[i]` | `C[i] < O[i]` | Candle direction is bearish |
| `Doji[i]` | `Body[i] / Range[i] < DOJI_BODY_RATIO` | Candle is a doji (see §2) |

### 1.4 ATR Notation

`ATR[i]` denotes the Average True Range at bar `i`, computed over `ATR_PERIOD` bars using Wilder's smoothing method:

```
TR[i] = max(H[i] - L[i], abs(H[i] - C[i+1]), abs(L[i] - C[i+1]))
ATR[0] = (1/ATR_PERIOD) * sum(TR[j], j=1..ATR_PERIOD)   [seed]
ATR[i] = ((ATR_PERIOD - 1) * ATR[i+1] + TR[i]) / ATR_PERIOD
```

### 1.5 Lookback Windows

The swing detection window is `N` bars to the left and `N` bars to the right of a candidate pivot bar, where `N = SWING_LOOKBACK`. A pivot is confirmed only after `N` bars to the right have closed.

### 1.6 Ordered Swing Lists

- `SH_LIST`: ordered list of confirmed Swing Highs, from oldest (index 0) to newest (index k).
  - Each entry: `{ bar_index: int, price: float }`
- `SL_LIST`: ordered list of confirmed Swing Lows, same structure.
- `SH_LIST[-1]` and `SL_LIST[-1]` denote the most recent confirmed swing.
- `SH_LIST[-2]` and `SL_LIST[-2]` denote the second-most-recent confirmed swing.

---

## 2. Configuration Parameters

All parameters are defined at engine initialization. Defaults are provided. All values must satisfy the stated constraints.

| Parameter | Type | Default | Constraint | Description |
|-----------|------|---------|------------|-------------|
| `SWING_LOOKBACK` | int | 5 | ≥ 2 | Number of bars left and right required to confirm a swing pivot |
| `ATR_PERIOD` | int | 14 | ≥ 1 | Period for ATR computation |
| `DISPLACEMENT_ATR_MULT` | float | 1.5 | > 0.0 | Minimum candle range as multiple of ATR to qualify as displacement |
| `DISPLACEMENT_BODY_RATIO` | float | 0.6 | (0.0, 1.0] | Minimum `Body / Range` ratio for a displacement candle |
| `DISPLACEMENT_WICK_RATIO` | float | 0.2 | [0.0, 1.0) | Maximum `(UpperWick + LowerWick) / Range` ratio for a displacement candle |
| `DOJI_BODY_RATIO` | float | 0.05 | (0.0, 1.0) | Maximum `Body / Range` ratio for a bar to be classified as a doji |
| `STRONG_BOS_ATR_MULT` | float | 1.0 | > 0.0 | Minimum close distance beyond broken level (in ATR) for BOS to be Strong |
| `STRUCTURE_SCORE_WINDOW` | int | 20 | ≥ 5 | Number of most-recent confirmed swings used to compute Structure Score |
| `MIN_SWING_SIZE_ATR` | float | 0.5 | > 0.0 | Minimum size of a swing (price distance from prior swing) in ATR units to be considered valid |
| `MAX_EQUAL_PRICE_TICKS` | int | 2 | ≥ 0 | Maximum tick difference between two prices to be considered equal (handles float precision) |
| `TICK_SIZE` | float | instrument-specific | > 0.0 | Minimum price increment of the instrument |

### 2.1 Parameter Validation

At engine initialization, the following assertions must hold:

```
assert SWING_LOOKBACK >= 2
assert ATR_PERIOD >= 1
assert 0.0 < DISPLACEMENT_ATR_MULT
assert 0.0 < DISPLACEMENT_BODY_RATIO <= 1.0
assert 0.0 <= DISPLACEMENT_WICK_RATIO < 1.0
assert DISPLACEMENT_BODY_RATIO + DISPLACEMENT_WICK_RATIO <= 1.0
assert 0.0 < DOJI_BODY_RATIO < 1.0
assert 0.0 < STRONG_BOS_ATR_MULT
assert STRUCTURE_SCORE_WINDOW >= 5
assert MIN_SWING_SIZE_ATR > 0.0
assert MAX_EQUAL_PRICE_TICKS >= 0
assert TICK_SIZE > 0.0
```

---

## 3. Primitive Definitions

### 3.1 Swing High

**Definition.** Bar at index `p` is a **Swing High** if and only if all of the following conditions hold:

**Condition SH-1 (Local Maximum):**
```
H[p] > H[p+j]  for all j in {1, 2, ..., SWING_LOOKBACK}
H[p] > H[p-j]  for all j in {1, 2, ..., SWING_LOOKBACK}
```

**Condition SH-2 (Confirmation Delay):**  
Bar `p` can only be evaluated after bar `p - SWING_LOOKBACK` has closed. That is, the current bar index `i` must satisfy:
```
i >= p + SWING_LOOKBACK
```
The swing is considered **confirmed** at the bar when `i = p + SWING_LOOKBACK`.

**Condition SH-3 (Minimum Swing Size):**  
Let `prev_SL` be the most recently confirmed Swing Low prior to bar `p`. The Swing High is valid only if:
```
H[p] - prev_SL.price >= MIN_SWING_SIZE_ATR * ATR[p]
```
If no prior Swing Low exists, this condition is waived for the first detected swing.

**Condition SH-4 (Range Validity):**  
```
Range[p] > 0
```
A zero-range bar (inside tick bar) cannot be a Swing High.

**Tie-breaking rule (equal highs):** If `H[p] == H[p+j]` for any `j` in `{1, ..., SWING_LOOKBACK}`, the tie is resolved using `MAX_EQUAL_PRICE_TICKS`:
```
EqualHigh(p, q) := abs(H[p] - H[q]) <= MAX_EQUAL_PRICE_TICKS * TICK_SIZE
```
If `EqualHigh(p, p+j)` is true for any `j`, bar `p` is **not** a Swing High. Equality on the right side of the window disqualifies the pivot.

**Formal notation:**
```
isSwingHigh(p) :=
    (∀ j ∈ [1, SWING_LOOKBACK]: H[p] > H[p+j]  AND  NOT EqualHigh(p, p+j))
    AND
    (∀ j ∈ [1, SWING_LOOKBACK]: H[p] > H[p-j])
    AND Range[p] > 0
    AND SwingSize(p) >= MIN_SWING_SIZE_ATR * ATR[p]
```

**Output record:**
```
SwingHigh := {
    bar_index  : p,
    price      : H[p],
    atr_at_bar : ATR[p],
    confirmed_at_bar : p + SWING_LOOKBACK
}
```

---

### 3.2 Swing Low

**Definition.** Bar at index `p` is a **Swing Low** if and only if all of the following conditions hold:

**Condition SL-1 (Local Minimum):**
```
L[p] < L[p+j]  for all j in {1, 2, ..., SWING_LOOKBACK}
L[p] < L[p-j]  for all j in {1, 2, ..., SWING_LOOKBACK}
```

**Condition SL-2 (Confirmation Delay):**
```
i >= p + SWING_LOOKBACK
```

**Condition SL-3 (Minimum Swing Size):**  
Let `prev_SH` be the most recently confirmed Swing High prior to bar `p`:
```
prev_SH.price - L[p] >= MIN_SWING_SIZE_ATR * ATR[p]
```

**Condition SL-4 (Range Validity):**
```
Range[p] > 0
```

**Tie-breaking rule (equal lows):**
```
EqualLow(p, q) := abs(L[p] - L[q]) <= MAX_EQUAL_PRICE_TICKS * TICK_SIZE
```
If `EqualLow(p, p+j)` is true for any `j` in `{1, ..., SWING_LOOKBACK}`, bar `p` is **not** a Swing Low.

**Formal notation:**
```
isSwingLow(p) :=
    (∀ j ∈ [1, SWING_LOOKBACK]: L[p] < L[p+j]  AND  NOT EqualLow(p, p+j))
    AND
    (∀ j ∈ [1, SWING_LOOKBACK]: L[p] < L[p-j])
    AND Range[p] > 0
    AND SwingSize(p) >= MIN_SWING_SIZE_ATR * ATR[p]
```

**Output record:**
```
SwingLow := {
    bar_index  : p,
    price      : L[p],
    atr_at_bar : ATR[p],
    confirmed_at_bar : p + SWING_LOOKBACK
}
```

---

## 4. Structural Labels

Structural labels classify each new confirmed swing relative to the immediately preceding swing of the same type (High-to-High or Low-to-Low).

**Prerequisite:** At least two confirmed Swing Highs must exist before HH/LH can be assigned. At least two confirmed Swing Lows must exist before HL/LL can be assigned.

### 4.1 Higher High (HH)

**Definition.** The most recently confirmed Swing High `SH_LIST[-1]` is labeled **Higher High** if and only if:
```
SH_LIST[-1].price > SH_LIST[-2].price + MAX_EQUAL_PRICE_TICKS * TICK_SIZE
```

That is, the current Swing High is strictly greater than the prior Swing High by more than the equality tolerance.

**Label:** `HH`

---

### 4.2 Lower High (LH)

**Definition.** The most recently confirmed Swing High `SH_LIST[-1]` is labeled **Lower High** if and only if:
```
SH_LIST[-1].price < SH_LIST[-2].price - MAX_EQUAL_PRICE_TICKS * TICK_SIZE
```

**Label:** `LH`

---

**Equal High case:** If neither HH nor LH is assigned (prices are within `MAX_EQUAL_PRICE_TICKS * TICK_SIZE`), the swing is labeled **EH** (Equal High) and is excluded from structural sequence analysis. `SH_LIST[-2]` is not updated; the new equal high does not replace the previous reference point.

---

### 4.3 Higher Low (HL)

**Definition.** The most recently confirmed Swing Low `SL_LIST[-1]` is labeled **Higher Low** if and only if:
```
SL_LIST[-1].price > SL_LIST[-2].price + MAX_EQUAL_PRICE_TICKS * TICK_SIZE
```

**Label:** `HL`

---

### 4.4 Lower Low (LL)

**Definition.** The most recently confirmed Swing Low `SL_LIST[-1]` is labeled **Lower Low** if and only if:
```
SL_LIST[-1].price < SL_LIST[-2].price - MAX_EQUAL_PRICE_TICKS * TICK_SIZE
```

**Label:** `LL`

---

**Equal Low case:** Labeled **EL** (Equal Low). Excluded from structural sequence analysis. `SL_LIST[-2]` is not updated.

---

## 5. Break of Structure (BOS)

### 5.1 BOS Definition

A **Break of Structure** event occurs when price closes beyond a previously established structural level. The level is defined by a confirmed Swing High or Swing Low that is part of the active trend sequence.

**Active High Level (`AHL`):** The price of the most recent confirmed Swing High that carried the label HH in a bullish trend, or the most recent confirmed Swing High that is the last HH before a trend reversal signal in a bearish trend. Formally, in trending conditions:

- **Bullish trend active:** `AHL = SH_LIST[-1].price` (the last HH)
- **Bearish trend active:** `AHL = SH_LIST[-1].price` (the last LH, used as resistance)

**Active Low Level (`ALL`):** The price of the most recent confirmed Swing Low.

- **Bullish trend active:** `ALL = SL_LIST[-1].price` (the last HL, used as support)
- **Bearish trend active:** `ALL = SL_LIST[-1].price` (the last LL)

**BOS Bullish:** A Bullish BOS occurs at bar `i` when:
```
C[i] > AHL
AND
C[i-1] <= AHL    [close of prior bar did not already breach the level]
```

**BOS Bearish:** A Bearish BOS occurs at bar `i` when:
```
C[i] < ALL
AND
C[i-1] >= ALL
```

**Note:** Only the **closing price** triggers a BOS. An intra-bar wick that temporarily exceeds the level without a closing breach does not constitute a BOS.

**Output record:**
```
BOS := {
    bar_index      : i,
    direction      : BULLISH | BEARISH,
    broken_level   : AHL | ALL,
    close_price    : C[i],
    close_distance : abs(C[i] - broken_level),
    atr_at_bar     : ATR[i],
    strength       : STRONG | WEAK,   [assigned in §5.2 and §5.3]
    displacement   : bool             [whether the BOS bar is also a displacement candle per §10]
}
```

---

### 5.2 Strong BOS

**Definition.** A BOS is classified as **Strong** if the closing distance from the broken level satisfies:
```
abs(C[i] - broken_level) >= STRONG_BOS_ATR_MULT * ATR[i]
```

Additionally, the BOS bar must **not** be a doji:
```
NOT Doji[i]
```

Both conditions must hold simultaneously:
```
StrongBOS :=
    BOS_condition_satisfied
    AND abs(C[i] - broken_level) >= STRONG_BOS_ATR_MULT * ATR[i]
    AND NOT Doji[i]
```

---

### 5.3 Weak BOS

**Definition.** A BOS is classified as **Weak** if the closing distance from the broken level does not satisfy the Strong BOS threshold:
```
WeakBOS :=
    BOS_condition_satisfied
    AND (abs(C[i] - broken_level) < STRONG_BOS_ATR_MULT * ATR[i]
         OR Doji[i])
```

---

### 5.4 Failed BOS

**Definition.** A **Failed BOS** is a BOS event that is subsequently negated. A BOS at bar `i` is declared **Failed** when, within `SWING_LOOKBACK * 2` bars after bar `i`, the following reversal condition is met:

**For a Bullish BOS at bar `i`:**
```
FailedBOS_Bull :=
    (∃ k ∈ [i+1, i + SWING_LOOKBACK*2] : C[k] < broken_level)
```

**For a Bearish BOS at bar `i`:**
```
FailedBOS_Bear :=
    (∃ k ∈ [i+1, i + SWING_LOOKBACK*2] : C[k] > broken_level)
```

A Failed BOS is retroactively applied to the original BOS event record. The field `failed` is set to `true` and `failed_at_bar` is set to `k`.

**Note:** A Failed BOS does **not** automatically trigger a CHoCH or MSS. It is a standalone event that reduces confidence in the original BOS signal.

---

## 6. Change of Character (CHoCH)

**Definition.** A **Change of Character** (CHoCH) is a BOS event that occurs against the current dominant trend direction. It signals a potential reversal but does not yet confirm a new trend.

**Formal definition:**

Let `current_trend` be the current Trend State (see §8).

**CHoCH Bullish** (potential reversal of bearish trend):  
```
CHoCH_Bull :=
    current_trend == BEARISH
    AND BOS_Bullish at bar i    [close above last confirmed Swing High in bearish sequence]
```

**CHoCH Bearish** (potential reversal of bullish trend):
```
CHoCH_Bear :=
    current_trend == BULLISH
    AND BOS_Bearish at bar i    [close below last confirmed Swing Low in bullish sequence]
```

**Key distinction from BOS:**
- A BOS in the **direction** of the current trend is a **trend-continuation BOS**.
- A BOS **against** the current trend is a **CHoCH**.

A CHoCH alone does not change the Trend State. Trend State changes only on MSS (§7).

**Output record:**
```
CHoCH := {
    bar_index    : i,
    direction    : BULLISH | BEARISH,
    broken_level : float,
    close_price  : C[i],
    strength     : STRONG | WEAK,   [same rules as BOS §5.2/5.3]
    prior_trend  : BULLISH | BEARISH | RANGING
}
```

---

## 7. Market Structure Shift (MSS)

**Definition.** A **Market Structure Shift** (MSS) is a confirmed reversal of the dominant trend, established by a sequence of two structural conditions:

**MSS Bullish (Bear-to-Bull reversal):**

Condition 1: A CHoCH Bullish has occurred at bar `j`.  
Condition 2: After bar `j`, a new Swing Low is confirmed at bar `q` (where `q > j`) such that:
```
SL_LIST_new[-1].price > SL_LIST[-1].price    [new HL formed after CHoCH]
```
Condition 3: Price closes above the Swing High that was used as the level for the CHoCH at bar `j`:
```
∃ k ∈ [j+1, ∞) : C[k] > CHoCH_Bull[j].broken_level
```

When all three conditions are met at bar `k`, MSS Bullish is confirmed at bar `k`.

**MSS Bearish (Bull-to-Bear reversal):**

Condition 1: A CHoCH Bearish has occurred at bar `j`.  
Condition 2: A new Swing High is confirmed at bar `q` (where `q > j`) such that:
```
SH_LIST_new[-1].price < SH_LIST[-1].price    [new LH formed after CHoCH]
```
Condition 3: Price closes below the Swing Low that was used as the level for the CHoCH:
```
∃ k ∈ [j+1, ∞) : C[k] < CHoCH_Bear[j].broken_level
```

MSS Bearish is confirmed at bar `k`.

**MSS invalidation:** The pending MSS is invalidated (discarded) if, before conditions 2 and 3 are met, price extends in the prior trend direction and creates a new HH (bearish MSS pending) or LL (bullish MSS pending) that exceeds the CHoCH level by more than `STRONG_BOS_ATR_MULT * ATR`.

**Output record:**
```
MSS := {
    bar_index        : k,
    direction        : BULLISH | BEARISH,
    choch_bar        : j,
    prior_trend      : BULLISH | BEARISH,
    new_trend        : BEARISH | BULLISH,
    confirmation_bar : k
}
```

---

## 8. Trend State

**Definition.** The **Trend State** is a discrete state variable updated on each confirmed structural event. It has three possible values:

| Value | Code | Meaning |
|-------|------|---------|
| Bullish | `1` | Market is in a confirmed bullish trend |
| Bearish | `-1` | Market is in a confirmed bearish trend |
| Ranging | `0` | No confirmed trend; structural sequence is ambiguous |

**Initial state:** `RANGING` until at least two confirmed Swing Highs and two confirmed Swing Lows exist.

**State transition rules:**

```
RANGING → BULLISH  :  When two consecutive HH and two consecutive HL are confirmed
RANGING → BEARISH  :  When two consecutive LH and two consecutive LL are confirmed
BULLISH → BEARISH  :  When MSS Bearish is confirmed (§7)
BEARISH → BULLISH  :  When MSS Bullish is confirmed (§7)
BULLISH → RANGING  :  When one LH is followed by one LL without a prior MSS
BEARISH → RANGING  :  When one HL is followed by one HH without a prior MSS
RANGING → RANGING  :  All other conditions
```

**Formal bullish entry from RANGING:**  
```
TrendState = BULLISH  iff
    len(SH_LIST) >= 2
    AND SH_LIST[-1].label == HH
    AND SH_LIST[-2].label == HH
    AND len(SL_LIST) >= 2
    AND SL_LIST[-1].label == HL
    AND SL_LIST[-2].label == HL
```

**Formal bearish entry from RANGING:**  
```
TrendState = BEARISH  iff
    len(SH_LIST) >= 2
    AND SH_LIST[-1].label == LH
    AND SH_LIST[-2].label == LH
    AND len(SL_LIST) >= 2
    AND SL_LIST[-1].label == LL
    AND SL_LIST[-2].label == LL
```

The Trend State is evaluated on each new confirmed swing. It is **not** updated on every bar; it is a function of the swing sequence.

---

## 9. Market Regime

**Definition.** The **Market Regime** classifies the broader price behavior using volatility and trend strength. It is computed on each closed bar.

Regimes are mutually exclusive:

| Regime | Code | Definition |
|--------|------|-----------|
| Trending | `T` | Trend State ≠ RANGING AND Structure Score ≥ 0.6 |
| Ranging | `R` | Trend State == RANGING AND Structure Score < 0.4 |
| Volatile | `V` | `ATR[0] > VOLATILE_ATR_MULT * ATR_LONG_MA[0]` |
| Transitional | `X` | None of the above apply |

Where:
- `ATR_LONG_MA[i]` = simple moving average of ATR over `ATR_PERIOD * 3` bars.
- `VOLATILE_ATR_MULT` = 1.5 (hardcoded multiplier for volatility regime detection; not a user parameter).

**Regime priority:** When multiple regime conditions are simultaneously true, apply in order: `V > T > R > X`.

**Output:** Updated each bar as `MarketRegime ∈ {T, R, V, X}`.

---

## 10. Displacement

**Definition.** A bar at index `i` is a **Displacement** candle if all three sub-conditions hold simultaneously:

### 10.1 ATR Requirements

```
D1 :=  Range[i] >= DISPLACEMENT_ATR_MULT * ATR[i]
```

The full candle range must be at least `DISPLACEMENT_ATR_MULT` times the current ATR.

### 10.2 Body Ratio

```
D2 := Body[i] / Range[i] >= DISPLACEMENT_BODY_RATIO
```

The body must occupy at least `DISPLACEMENT_BODY_RATIO` of the total range. This ensures the candle is not primarily wick.

### 10.3 Wick Ratio

```
D3 := (UpperWick[i] + LowerWick[i]) / Range[i] <= DISPLACEMENT_WICK_RATIO
```

Total wick length must not exceed `DISPLACEMENT_WICK_RATIO` of the total range.

**Combined displacement condition:**
```
isDisplacement(i) := D1 AND D2 AND D3 AND NOT Doji[i] AND Range[i] > 0
```

**Displacement direction:**
```
DisplacementDir(i) :=
    BULLISH   if isDisplacement(i) AND Bullish[i]
    BEARISH   if isDisplacement(i) AND Bearish[i]
    NONE      otherwise
```

**Output record:**
```
Displacement := {
    bar_index  : i,
    direction  : BULLISH | BEARISH,
    range      : Range[i],
    body       : Body[i],
    body_ratio : Body[i] / Range[i],
    wick_ratio : (UpperWick[i] + LowerWick[i]) / Range[i],
    atr_mult   : Range[i] / ATR[i]
}
```

---

## 11. Structure Score

**Definition.** The **Structure Score** is a normalized scalar in `[0.0, 1.0]` representing the quality and clarity of the current market structure over the `STRUCTURE_SCORE_WINDOW` most recent confirmed swings.

### 11.1 Component Scores

Let `W = STRUCTURE_SCORE_WINDOW`. Take the `W` most recent confirmed swings from the combined ordered list of Swing Highs and Swing Lows.

**Component S1: Sequence Consistency**  
Count the fraction of consecutive same-type label pairs that are trend-consistent:
```
bullish_pairs = count of (HH, HL) adjacent pairs in last W swings
bearish_pairs = count of (LH, LL) adjacent pairs in last W swings
total_pairs   = W - 1
S1 = max(bullish_pairs, bearish_pairs) / total_pairs
```

**Component S2: Swing Size Uniformity**  
Let `swing_sizes = [abs(swing[k].price - swing[k-1].price) for k in 1..W-1]`  
Let `μ = mean(swing_sizes)`, `σ = std(swing_sizes)`:
```
S2 = 1.0 - min(σ / (μ + TICK_SIZE), 1.0)
```

**Component S3: BOS Success Rate**  
Within the `STRUCTURE_SCORE_WINDOW`, count:
- `n_bos` = total number of BOS events
- `n_failed` = number of Failed BOS events
```
S3 = 1.0 - (n_failed / max(n_bos, 1))
```

**Component S4: Trend Alignment**  
```
S4 = 1.0  if TrendState != RANGING
S4 = 0.5  if TrendState == RANGING
```

### 11.2 Weighted Aggregation

```
StructureScore = 0.40 * S1 + 0.25 * S2 + 0.20 * S3 + 0.15 * S4
```

All weights sum to 1.0. The result is clamped to `[0.0, 1.0]`.

---

## 12. Edge Cases

Each edge case below defines the exact handling rule. No interpretation is required.

### EC-01: Insufficient History

**Condition:** Number of closed bars `< ATR_PERIOD + SWING_LOOKBACK * 2`  
**Rule:** All engine outputs are `UNDEFINED`. No events are emitted. No labels are assigned. Engine runs silently until sufficient history is available.

### EC-02: Zero-Range Bar

**Condition:** `Range[i] == 0` (open equals high equals low equals close)  
**Rule:**  
- Bar cannot be a Swing High or Swing Low.  
- `isDisplacement(i) = false`.  
- `Doji[i]` evaluation is skipped (division by zero guard): `Doji[i] = false` for zero-range bars.  
- `Body[i] / Range[i]` is defined as `0.0` (no body).

### EC-03: Equal Adjacent Swings

**Condition:** Two consecutive Swing Highs or Swing Lows have prices within `MAX_EQUAL_PRICE_TICKS * TICK_SIZE`.  
**Rule:** The newer swing is labeled EH (Equal High) or EL (Equal Low). It is appended to the list but is **not** used as the reference for structural label comparisons. The prior non-equal swing remains the reference.

### EC-04: Swing High and Swing Low on the Same Bar

**Condition:** Bar `p` satisfies both `isSwingHigh(p)` and `isSwingLow(p)` simultaneously.  
**Rule:** This is impossible by construction (a bar cannot have `H[p] > all neighbors` and `L[p] < all neighbors` and also satisfy the minimum swing size in both directions from the same prior swing). However, as a defensive rule: if this condition somehow arises, the bar is labeled **neither** a Swing High nor Swing Low and is logged as an engine error.

### EC-05: No Prior Swing of Opposite Type

**Condition:** A new Swing High is confirmed but no Swing Low has yet been confirmed (or vice versa).  
**Rule:** Condition SH-3 / SL-3 (minimum swing size check against prior opposite swing) is **waived** for the first swing in each direction. The swing is confirmed based on SH-1, SH-2, SH-4 only.

### EC-06: ATR of Zero

**Condition:** `ATR[i] == 0` (possible on instruments with no true price movement in the ATR period).  
**Rule:** All ATR-ratio checks default to `false` (displacement is false, strong BOS is false, minimum swing size check is waived). Log as engine warning.

### EC-07: Gap Open (Price Gaps Through a Level)

**Condition:** A gap open causes `O[i]` to already be beyond `AHL` or `ALL`, so no bar close can be the first to breach the level.  
**Rule:** The BOS is triggered at bar `i` using the opening price as the effective breach price:  
```
BOS_price = O[i]
close_distance = abs(C[i] - broken_level)
```
The BOS record is emitted at bar `i`. All strength calculations use `C[i]` as normal.

### EC-08: Multiple BOS Levels Available

**Condition:** Multiple prior Swing Highs or Swing Lows exist that price could have broken simultaneously.  
**Rule:** Only the **most recent** structural level (highest-priority) is used for BOS detection. Older levels are not evaluated unless the most recent level is resolved (either broken or replaced by a new swing).

### EC-09: CHoCH Before Two Confirmed Swings

**Condition:** CHoCH logic is invoked but fewer than two confirmed Swing Highs or Swing Lows exist.  
**Rule:** CHoCH cannot be emitted. The engine returns `UNDEFINED` for CHoCH. BOS logic continues to apply.

### EC-10: Pending MSS Invalidated by New Trend Extreme

**Condition:** A CHoCH has been emitted but price resumes in the prior trend direction, creating a new structural extreme.  
**Rule:** The pending MSS record is discarded. Its `status` is set to `INVALIDATED`. The Trend State is not changed.

---

## 13. Invalid Cases

The following inputs or states cause the engine to reject processing and emit an error:

| Code | Condition | Error Type |
|------|-----------|------------|
| INV-01 | `H[i] < L[i]` (high below low) | Fatal: malformed OHLC |
| INV-02 | `H[i] < max(O[i], C[i])` | Fatal: high does not encompass body |
| INV-03 | `L[i] > min(O[i], C[i])` | Fatal: low does not encompass body |
| INV-04 | Any OHLC value is `NaN` or `Inf` | Fatal: invalid price data |
| INV-05 | `TICK_SIZE <= 0` | Fatal: invalid configuration |
| INV-06 | `ATR_PERIOD < 1` | Fatal: invalid configuration |
| INV-07 | `SWING_LOOKBACK < 2` | Fatal: invalid configuration |
| INV-08 | `DISPLACEMENT_BODY_RATIO + DISPLACEMENT_WICK_RATIO > 1.0` | Fatal: overlapping ratios |

For Fatal errors, the engine halts processing for the affected bar and emits an error record. It resumes on the next bar.

---

## 14. Worked Examples

### Example 1: Swing High Confirmation

**Setup:** `SWING_LOOKBACK = 3`, `ATR[7] = 10.0`, `MIN_SWING_SIZE_ATR = 0.5`

Bar sequence (highs only):

| Bar | H |
|-----|---|
| 10  | 100.0 |
| 9   | 105.0 |
| 8   | 108.0 |  ← candidate pivot
| 7   | 104.0 |
| 6   | 102.0 |
| 5   | 99.0 |

At bar `i = 5` (current), candidate bar `p = 8`.

**SH-1 check:**  
- Right side: `H[8] = 108 > H[9] = 105` ✓ and `H[8] = 108 > H[10] = 100` ✓ (but we need j=1,2,3; bar 11 not shown, assume 97) ✓  
- Left side: `H[8] = 108 > H[7] = 104` ✓, `H[8] = 108 > H[6] = 102` ✓, `H[8] = 108 > H[5] = 99` ✓  

**SH-2 check:** `i = 5 = p - SWING_LOOKBACK = 8 - 3`. Confirmed at bar `5`. ✓  

**SH-3 check:** Prior SL price = 95.0 (assumed). `108 - 95 = 13.0 >= 0.5 * 10.0 = 5.0` ✓  

**SH-4 check:** `Range[8] > 0` ✓  

**Result:** `SwingHigh { bar_index=8, price=108.0, confirmed_at_bar=5 }` is emitted.

---

### Example 2: Strong vs Weak BOS

**Setup:** `STRONG_BOS_ATR_MULT = 1.0`, `ATR[0] = 20.0`

- `AHL = 1.2000`
- Bar `i`: `C[i] = 1.2150`
- `close_distance = 1.2150 - 1.2000 = 0.0150`
- `1.0 * ATR[0] = 0.0200` (assuming ATR is expressed in price units)
- `0.0150 < 0.0200` → **Weak BOS**

- Bar `j`: `C[j] = 1.2250`
- `close_distance = 1.2250 - 1.2000 = 0.0250`
- `0.0250 >= 0.0200` and `NOT Doji[j]` → **Strong BOS**

---

### Example 3: CHoCH in Bearish Trend

**Setup:** Current Trend State = `BEARISH`. Swing sequence (most recent first):  
`LH → LL → LH → LL`

- `SH_LIST[-1].price = 1.3050` (last LH)
- `ALL = SL_LIST[-1].price = 1.2800` (last LL)

Bar `i`: `C[i] = 1.3075`  
- `C[i] > SH_LIST[-1].price = 1.3050`  
- `C[i-1] = 1.3040 <= 1.3050`  
- Trend State = BEARISH  
→ **CHoCH Bullish** emitted at bar `i`.

The Trend State remains `BEARISH` until MSS conditions (§7) are fully met.

---

### Example 4: Displacement Candle

**Setup:** `DISPLACEMENT_ATR_MULT = 1.5`, `DISPLACEMENT_BODY_RATIO = 0.6`, `DISPLACEMENT_WICK_RATIO = 0.2`, `ATR[i] = 30 pips`

Bar `i`: `O = 1.2000, H = 1.2060, L = 1.1990, C = 1.2055`

- `Range[i] = H - L = 0.0070 = 70 pips`
- `ATR = 30 pips`
- `D1: 70 >= 1.5 * 30 = 45` ✓
- `Body[i] = |C - O| = 0.0055 = 55 pips`
- `D2: 55/70 = 0.786 >= 0.6` ✓
- `UpperWick = H - max(O,C) = 1.2060 - 1.2055 = 0.0005 = 5 pips`
- `LowerWick = min(O,C) - L = 1.2000 - 1.1990 = 0.0010 = 10 pips`
- `D3: (5+10)/70 = 0.214 <= 0.2` ✗ (0.214 > 0.2)

**Result:** `isDisplacement(i) = false`. Wick ratio exceeds threshold.

---

### Example 5: Structure Score Calculation

**Setup:** `STRUCTURE_SCORE_WINDOW = 10`, last 10 swings:

```
SL(LL), SH(LH), SL(LL), SH(LH), SL(HL), SH(LH), SL(HL), SH(LH), SL(HL), SH(LH)
```

**S1:** Adjacent pairs (LH,LL), (LL,LH), (LH,LL), (LL,HL)*, (HL,LH), (LH,HL), (HL,LH), (LH,HL), (HL,LH)  
- Bearish consistent pairs (LH→LL): (LH,LL)×2 = 2  
- Bullish consistent pairs (HL→HH): 0  
- Total pairs = 9  
- `S1 = 2/9 = 0.222`

**S2:** (Assume swing sizes are 40,35,38,42,30,45,33,38,41 pips; μ=38.0, σ=4.6): `S2 = 1 - 4.6/(38.0 + 0.0001) = 0.879`

**S3:** (Assume 4 BOS, 1 failed): `S3 = 1 - 1/4 = 0.75`

**S4:** TrendState = RANGING → `S4 = 0.5`

```
StructureScore = 0.40*0.222 + 0.25*0.879 + 0.20*0.75 + 0.15*0.5
               = 0.0888 + 0.2198 + 0.150 + 0.075
               = 0.533
```

---

## 15. Pseudo-Code

The following pseudo-code is implementation-language-agnostic. It defines the processing order for each new closed bar.

```
FUNCTION process_bar(O, H, L, C, bar_index):

    # Step 0: Validate OHLC
    if H < L or H < max(O,C) or L > min(O,C):
        emit_error(INV-01 or INV-02 or INV-03)
        return

    if isNaN(O) or isNaN(H) or isNaN(L) or isNaN(C):
        emit_error(INV-04)
        return

    # Step 1: Compute ATR
    TR = max(H - L, abs(H - prev_close), abs(L - prev_close))
    ATR = update_atr(TR)

    if ATR == 0:
        emit_warning(EC-06)
        # Continue with ATR-dependent checks defaulting to false

    # Step 2: Detect Swing Highs and Swing Lows
    # The pivot bar being checked is bar_index - SWING_LOOKBACK
    p = bar_index - SWING_LOOKBACK

    if p >= 0:
        if check_swing_high(p):
            sh = create_swing_high(p)
            if validate_swing_high(sh):
                SH_LIST.append(sh)
                label = assign_high_label(sh)  # HH, LH, or EH
                sh.label = label
                emit_event(SWING_HIGH, sh)

        if check_swing_low(p):
            sl = create_swing_low(p)
            if validate_swing_low(sl):
                SL_LIST.append(sl)
                label = assign_low_label(sl)   # HL, LL, or EL
                sl.label = label
                emit_event(SWING_LOW, sl)

    # Step 3: Update Trend State
    prev_trend = TrendState
    TrendState = compute_trend_state(SH_LIST, SL_LIST)

    # Step 4: Compute Active Levels
    AHL = get_active_high_level(SH_LIST, TrendState)
    ALL = get_active_low_level(SL_LIST, TrendState)

    # Step 5: Check BOS
    bos = None
    if AHL is not None and prev_close <= AHL and C > AHL:
        bos = create_bos(BULLISH, AHL, bar_index, C, ATR)
        bos.strength = classify_bos_strength(bos, ATR)
        bos.displacement = isDisplacement(bar_index)
        emit_event(BOS, bos)

    elif ALL is not None and prev_close >= ALL and C < ALL:
        bos = create_bos(BEARISH, ALL, bar_index, C, ATR)
        bos.strength = classify_bos_strength(bos, ATR)
        bos.displacement = isDisplacement(bar_index)
        emit_event(BOS, bos)

    # Step 6: Classify BOS as CHoCH if against trend
    if bos is not None:
        if (bos.direction == BULLISH and prev_trend == BEARISH) or \
           (bos.direction == BEARISH and prev_trend == BULLISH):
            choch = create_choch(bos, prev_trend)
            emit_event(CHOCH, choch)
            pending_mss = initiate_mss(choch)

    # Step 7: Check for MSS confirmation
    if pending_mss is not None:
        mss_result = evaluate_mss(pending_mss, SH_LIST, SL_LIST, C, bar_index)
        if mss_result == CONFIRMED:
            emit_event(MSS, pending_mss)
            TrendState = pending_mss.new_trend
            pending_mss = None
        elif mss_result == INVALIDATED:
            pending_mss.status = INVALIDATED
            emit_event(MSS_INVALIDATED, pending_mss)
            pending_mss = None

    # Step 8: Check for Failed BOS (retroactive, on any prior pending BOS)
    for pb in pending_bos_list:
        if bar_index <= pb.bar_index + SWING_LOOKBACK * 2:
            if (pb.direction == BULLISH and C < pb.broken_level) or \
               (pb.direction == BEARISH and C > pb.broken_level):
                pb.failed = True
                pb.failed_at_bar = bar_index
                emit_event(FAILED_BOS, pb)
                pending_bos_list.remove(pb)

    # Step 9: Compute Displacement for current bar
    disp = compute_displacement(O, H, L, C, ATR, bar_index)
    if disp.direction != NONE:
        emit_event(DISPLACEMENT, disp)

    # Step 10: Compute Structure Score
    StructureScore = compute_structure_score(
        SH_LIST, SL_LIST, bos_history, TrendState, STRUCTURE_SCORE_WINDOW
    )

    # Step 11: Compute Market Regime
    ATR_LONG_MA = update_long_atr_ma(ATR)
    MarketRegime = compute_regime(TrendState, StructureScore, ATR, ATR_LONG_MA)

    # Step 12: Store state for next bar
    prev_close = C

    return {
        trend_state    : TrendState,
        market_regime  : MarketRegime,
        structure_score: StructureScore,
        atr            : ATR,
        events         : emitted_events_this_bar
    }
```

```
FUNCTION classify_bos_strength(bos, ATR):
    dist = abs(bos.close_price - bos.broken_level)
    if dist >= STRONG_BOS_ATR_MULT * ATR and NOT Doji(bos.bar_index):
        return STRONG
    else:
        return WEAK

FUNCTION compute_displacement(O, H, L, C, ATR, bar_index):
    if H == L:
        return Displacement(direction=NONE)
    range_ = H - L
    body   = abs(C - O)
    upper_wick = H - max(O, C)
    lower_wick = min(O, C) - L
    D1 = range_ >= DISPLACEMENT_ATR_MULT * ATR
    D2 = body / range_ >= DISPLACEMENT_BODY_RATIO
    D3 = (upper_wick + lower_wick) / range_ <= DISPLACEMENT_WICK_RATIO
    doji = body / range_ < DOJI_BODY_RATIO
    if D1 and D2 and D3 and not doji:
        direction = BULLISH if C > O else BEARISH
        return Displacement(bar_index, direction, range_, body, body/range_,
                            (upper_wick+lower_wick)/range_, range_/ATR)
    return Displacement(direction=NONE)

FUNCTION compute_trend_state(SH_LIST, SL_LIST):
    sh = [s for s in SH_LIST if s.label in (HH, LH)]
    sl = [s for s in SL_LIST if s.label in (HL, LL)]
    if len(sh) < 2 or len(sl) < 2:
        return RANGING
    if sh[-1].label == HH and sh[-2].label == HH \
       and sl[-1].label == HL and sl[-2].label == HL:
        return BULLISH
    if sh[-1].label == LH and sh[-2].label == LH \
       and sl[-1].label == LL and sl[-2].label == LL:
        return BEARISH
    if sh[-1].label == LH and sl[-1].label == LL:
        return BEARISH_TRANSITIONAL
    if sh[-1].label == HH and sl[-1].label == HL:
        return BULLISH_TRANSITIONAL
    return RANGING

FUNCTION compute_structure_score(SH_LIST, SL_LIST, bos_history, TrendState, W):
    # Merge and sort last W swings by bar_index
    swings = sorted(SH_LIST[-W:] + SL_LIST[-W:], key=lambda s: s.bar_index)[-W:]

    # S1: Sequence consistency
    bull_pairs = 0
    bear_pairs = 0
    for k in range(1, len(swings)):
        a, b = swings[k-1], swings[k]
        if a.label == HL and b.label == HH:
            bull_pairs += 1
        if a.label == LH and b.label == LL:
            bear_pairs += 1
    total_pairs = max(len(swings) - 1, 1)
    S1 = max(bull_pairs, bear_pairs) / total_pairs

    # S2: Swing size uniformity
    sizes = [abs(swings[k].price - swings[k-1].price) for k in range(1, len(swings))]
    mu = mean(sizes) if sizes else 0.0
    sigma = std(sizes) if sizes else 0.0
    S2 = 1.0 - min(sigma / (mu + TICK_SIZE), 1.0)

    # S3: BOS success rate
    recent_bos = [b for b in bos_history if b.bar_index >= swings[0].bar_index]
    n_bos = len(recent_bos)
    n_failed = sum(1 for b in recent_bos if b.failed)
    S3 = 1.0 - (n_failed / max(n_bos, 1))

    # S4: Trend alignment
    S4 = 1.0 if TrendState != RANGING else 0.5

    score = 0.40 * S1 + 0.25 * S2 + 0.20 * S3 + 0.15 * S4
    return clamp(score, 0.0, 1.0)
```

---

*End of Market Structure Engine Specification v0.1.0*
