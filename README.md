# A Mobile App for Lottery Addiction: the Probability Engine

Probability functions for an app that shows compulsive gamblers what their odds actually are, built on 3,665 historical Lotto 6/49 draws (Canada, 1982 to 2018).

Dataquest guided project, Data Scientist in Python path.

**The hard part of this problem is communication, not combinatorics.** Computing 1 in 13,983,816 takes one line; the reason lottery addiction survives that number being printed on the ticket is that nobody can feel 0.0000072%. So the notebook builds the four required functions and then goes after the three beliefs that actually sustain the habit.

## The four functions

| Matches | Probability | Odds |
|---|---|---|
| 2 | 13.2378% | 1 in 8 |
| 3 | 1.7650% | 1 in 57 |
| 4 | 0.0969% | 1 in 1,032 |
| 5 (no bonus) | 0.0018% | 1 in 55,491 |
| 5 + bonus | 0.0000429% | 1 in 2,330,636 |
| 6 (jackpot) | 0.0000072% | 1 in 13,983,816 |
| **Any prize** | **15.1016%** | **1 in 6.6** |

That last row is the clinically important one. A player wins *something* on roughly one ticket in seven, and the lowest tier pays a free play. Frequent real reinforcement with money flowing one way is the structure of a slot machine, not an investment.

## The formula that is usually got wrong

Matching exactly `k` numbers requires `k` of your picks to be among the six drawn **and** the other `6-k` to be among the 43 that were not:

```
P(exactly k) = C(6,k) * C(43, 6-k) / C(49,6)
```

The common shortcut `C(6,k) / C(49,6)` drops the second factor. It is wrong by a factor of **123,410 for two matches** and **12,341 for three**, and its seven terms sum to 0.0000046 rather than 1, which is the check that exposes it. The error makes the lottery look worse than it is, and for an app meant to be trusted by clinicians, being wrong in the convenient direction is still being wrong.

## Expected value: the argument that cannot be waved away

"Your odds are tiny" invites the reply that every winner also had tiny odds. Expected value does not have that weakness.

| | Value |
|---|---|
| Ticket price | $3.00 |
| Expected value | **$1.0918** |
| Expected loss per ticket | **$1.9082** |
| Return per dollar staked | $0.3639 |
| **House edge** | **63.61%** |

Roulette runs about 5% and blackjack under 1%. A player who moved their lottery spending to a roulette wheel would lose money roughly twelve times more slowly.

The jackpot contributes only $0.3576 of that $1.09, so the result barely depends on pari-mutuel guesswork. The more robust figure is the **break-even jackpot**, where a ticket stops being a losing bet: **$31,683,923**. The record Lotto 6/49 jackpot is around $64,000,000, so break-even is occasionally exceeded on paper, and it is still a bad bet: a big advertised jackpot attracts more players, so the chance of splitting it rises exactly when it is worth winning.

## "My numbers are due"

| | Value |
|---|---|
| Combinations ever drawn | 3,665 of 13,983,816 |
| Share ever drawn | **0.0262%** |
| Never drawn | 99.9738% |

Finding a combination absent from 36 years of history is the overwhelmingly normal outcome, not a signal.

The same arithmetic kills a second apparent pattern. There are **zero repeated combinations** in 3,665 draws, which sounds like an anti-repetition mechanism. The birthday calculation: 6,714,280 pairs of draws, each matching with probability 1/13,983,816, gives an expectation of **0.48 repeats** and a 38.13% chance of at least one. Observing zero needs no explanation.

| To reach a 50% chance of ever winning the jackpot | |
|---|---|
| Draws needed | 9,692,842 |
| Years at 101.7 draws/year | **95,267** |
| Cost | $29,078,527 |

## "These numbers are hot": tested, not denied

The frequency table does look uneven. Number 31 appeared 499 times, number 14 only 402, a gap of 97 or 21.6% of the expected 448.78. That gap is exactly what a player points at.

Against 10,000 simulated fair histories:

| | Value |
|---|---|
| Observed chi-square | 47.927 |
| Simulated null mean | 42.931 |
| Null 95th percentile | 58.440 |
| **Monte Carlo p-value** | **0.2712** |

No evidence of non-uniformity. The unevenness is the unevenness fair draws produce. Four of 49 numbers fall outside the pointwise 95% band against 2.5 expected, itself within chance for a pointwise band.

The accompanying chart is the most useful output for the app: a player who believes 31 is lucky can see its bar inside the same grey band as every other number, which is harder to dismiss than a p-value.

## A statistical detail that mattered

The simulated null mean is **42.93, not 48**, and that is not simulation noise.

A chi-square goodness-of-fit statistic has mean equal to its degrees of freedom when categories are filled by independent draws. These are not independent: each draw picks six **distinct** numbers, so a number appearing once is excluded from that draw's other five slots. The finite-population correction is `1 - (k-1)/(N-1) = 1 - 5/48 = 0.895833`:

| | Value |
|---|---|
| Predicted mean, `48 x 0.895833` | **43.000** |
| Simulated mean | **42.931** |
| Difference | 0.069 |
| Naive prediction | 48 |

| | p-value |
|---|---|
| Monte Carlo, correct null | **0.2712** |
| Ignoring the constraint | 0.4734 |

The naive figure is off by a factor of 1.7. Both are far from significance so the conclusion survives, but it would have survived by luck rather than by method, and on a dataset where the result was closer it would have mattered. Simulating the null was the correct choice here, not merely the convenient one.

## Method

1. Combinatorial core with explicit factorials, sanity-checked against hand-verifiable values and cross-checked against `itertools`.
2. `one_ticket_probability`, reported four ways because one framing is not enough.
3. Data quality checks on the draw history: numbers in range, no missing values, no duplicate number within a draw.
4. `check_historical_occurrence` with a `frozenset` lookup, asserted against a combination known to be present and one known to be absent.
5. `multi_ticket_probability` across nine orders of magnitude of spending.
6. `probability_less_6` built hypergeometrically, with the sum-to-one check and an explicit comparison against the wrong formula.
7. Expected value, house edge and break-even jackpot, with pari-mutuel tiers flagged as assumptions.
8. Birthday-problem analysis of repeat combinations.
9. Monte Carlo uniformity test, plus the finite-population correction derived and verified.

## Limitations

- Three of the six prize tiers are pari-mutuel, so those amounts are typical rather than fixed. The break-even jackpot is reported because it leans far less on those assumptions.
- Prize splitting is ignored, which overstates expected value. Modelling it needs tickets-sold-per-draw, which this dataset lacks.
- Taxes and the lump-sum discount are ignored. Canadian lottery winnings are untaxed so this matters less here than in the US, but the annuity discount still applies.
- The uniformity test covers the six main numbers pooled across positions. It does not test the bonus number or look for autocorrelation between consecutive draws; a real audit would do both.
- The test has limited power. 3,665 draws would detect a grossly biased ball, not a subtle one. Failing to find bias is not proving its absence.
- The historical lookup has no predictive value whatsoever. It exists because users will want it, and the function says so in its own output rather than letting the feature imply otherwise.
- Nothing here is a clinical intervention. The functions supply accurate numbers; whether accurate numbers change compulsive behaviour is a question for clinicians, and the evidence on numerical risk communication alone is not encouraging.

## Stack

Python, pandas, NumPy, matplotlib, and the standard library (`math`, `itertools`). No scikit-learn or SciPy: the uniformity test is a hand-built Monte Carlo, which turned out to be the more correct approach as well as the only available one.

## Files

- `notebook.ipynb`: the full analysis with executed outputs and figures.