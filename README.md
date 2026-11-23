# Cubilete DP Calculator

An interactive calculator of the optimal strategy for one turn of **Cubilete**, computed
exactly by dynamic programming. It was built for the MATH4045 group report
*"Cubilete: What is the optimal strategy?"* (University of Nottingham).

**▶ Run the calculator in your browser:** https://maoyidingnb-png.github.io/cubilete/

You can also download `index.html` and open it locally.

---

## 1. How to use it

1. Choose the five dice of your first roll from the drop-down menus.
2. The calculator shows the optimal action for that roll:
   - **green dice are kept**,
   - **red dice are re-rolled**,
   - and the **expected final score** if you play optimally from now on.
3. Press **Next roll** to re-roll the red dice at random. The calculator then shows
   the optimal action for the new hand, with the kept dice locked.
4. After the third roll, or when the optimal action is to keep all five dice, the
   final hand and its score are shown. Press **New turn** to start again.

## 2. Game rules used

- Five six-sided dice, at most three rolls per turn.
- After each of the first two rolls, the player chooses which dice to keep and
  re-rolls the rest. **Dice that have been kept are locked and cannot be re-rolled.**
- Only the final hand is scored. Aces (ones) are wild.

| Final hand | Score |
|---|---|
| Five aces | 10 |
| Five natural sixes (no aces) | 5 |
| Only aces and sixes (at least one of each) | 2 |
| Any other five-of-a-kind, aces wild (e.g. 1, 3, 3, 3, 3) | 1 |
| Anything else | 0 |

## 3. Model

### State
A hand is stored as a sorted list of five faces, which is equivalent to a count vector
**m** = (m₁, …, m₆), where mⱼ is the number of dice showing face j. Because order does
not matter, there are C(10, 5) = **252** possible hands.

A state is (**s**, **l**, r):
- **s** – the current hand,
- **l** – the dice already locked from earlier rolls (lⱼ ≤ mⱼ),
- r – the number of re-rolls left (2 after the first roll, then 1, then 0).

### Actions
An action **b** is the set of dice to keep. It must contain the locked dice and cannot
contain dice that are not in the hand:

  B(**s**, **l**) = { **b** : lⱼ ≤ bⱼ ≤ mⱼ for every face j }.

Keeping all five dice means stopping.

### Transition probabilities
If k = 5 − Σ bⱼ dice are re-rolled, the counts **c** = (c₁, …, c₆) of the new dice follow
a multinomial distribution:

  P(**c**) = k! / (c₁! ⋯ c₆!) · (1/6)ᵏ.

The next hand is **m′** = **b** + **c**, and the kept dice become locked: **l′** = **b**.

### Bellman equation
Let g(**s**) be the score of a final hand and V_r(**s**, **l**) the highest expected final
score from state (**s**, **l**) with r re-rolls left. Then

  V₀(**s**, **l**) = g(**s**),

  V_r(**s**, **l**) = max over **b** in B(**s**, **l**) of Σ_**c** P(**c**) · V_{r−1}(**b** + **c**, **b**).

The optimal action is the **b** that attains the maximum.

## 4. Algorithm

The Bellman equation is solved by **memoised recursion**, which is equivalent to backward
induction: each value V_r(**s**, **l**) is computed once, stored, and reused.

For the current state the calculator:

1. lists every allowed keep set **b** in B(**s**, **l**);
2. for each **b**, enumerates every outcome **c** of the k re-rolled dice with its
   multinomial probability;
3. adds up P(**c**) · V_{r−1}(**b** + **c**, **b**), calling itself recursively for the
   next state;
4. keeps the **b** with the highest expected value. If two actions give the same value
   (within 10⁻¹²), the one that keeps more dice is chosen.

The values are exact; no simulation is used to choose actions. Random numbers are only
used to generate the next roll when you press **Next roll**.

### Code map (`index.html`)

| Function | Role |
|---|---|
| `score(a)` | Score g(**s**) of a final hand, using the table above |
| `keeps(s, locked)` | All allowed keep sets B(**s**, **l**) |
| `outcomes(n)` | All results of re-rolling n dice with their multinomial probabilities |
| `find(s, l, r)` | Optimal value V_r(**s**, **l**) and the optimal keep set (memoised) |
| `rerolled(s, k)` | The dice that are re-rolled when keeping k |
| `render()`, `next()` | Display the result and simulate the next roll |

## 5. Checks

| Check | Result |
|---|---|
| Expected score of the optimal policy over all first rolls | **0.5037** |
| First roll {1, 1, 2, 6, 6} | keep {1, 1, 6, 6}, expected score 1.1111 |
| First roll {1, 3, 3, 3, 6} | keep {1, 3, 3, 3}, expected score 0.5556 |

The value 0.5037 matches the exact dynamic programming result and the Monte Carlo
estimate (0.5041 over 2,000,000 simulated turns) reported in the group report.

## 6. Running locally

The calculator is a single self-contained HTML file with no dependencies. Download
`index.html` and open it in any web browser.
