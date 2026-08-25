# ITI Statistics Case Study — "The Placement Report"

A team case study from ITI's Data Analysis & Business Intelligence track. Three clean datasets — the mess was a different case study's lesson. Here, the data is fine, and the real challenge is deciding **which distribution actually describes it and which comparisons are legitimate.**

The workbook opens with a flawed internal report making four confident claims about the graduates. Every part of this case study exists to test one of those claims against the actual statistics, and the final section (Part G, below) rewrites the report with what the numbers really say.

## The Data

- **graduates** — 300 graduates: scores, salaries, track, interviews attended, days to first offer, and number of offers received.
- **helpdesk** — 180 days of support ticket counts logged at the training labs.
- **lab_task** — 120 timed task completions, split across two branches (Cairo / Alexandria).
- **data_dictionary** — column meanings and units for all of the above.

## The One Rule

Every time a formula assumes a distribution — normal, binomial, Poisson — you have to state **why** that assumption is reasonable for that specific column before you're allowed to use it. Assuming the wrong shape is where the original report went wrong.

This was a team case study. Each teammate owned a different part; I'm listing all of them below for context, with my own part (Placement Odds) expanded in full.

---

## Part A — Is the Comparison Even Fair? (Dispersion & CV)

The original report ranked tracks by mean score and mean salary. Problem: raw standard deviation can't be compared across tracks with different spreads, and definitely can't be compared across score (0–100) and salary (thousands of L.E.) — different units entirely.

- **Coefficient of Variation (CV = std/mean × 100)** by track on score: UI/UX 15.8%, Web Dev 12.0%, Data Science 7.5%, Cyber Security 5.5%. UI/UX has almost **3× the relative variability** of Cyber Security.
- Mean salary was consistently higher than median salary across every track — a right-skew signature that the CV alone doesn't reveal.
- **Takeaway:** the mean is a poor summary for UI/UX specifically, because its spread is the widest of all four tracks — it hides both very strong and very weak graduates inside the same average.

## Part B — Standardization & Z-Scores

Two graduates: Candidate A scored 82 in Cyber Security, Candidate B scored 84 in UI/UX. Raw scores say B wins. Z-scores say otherwise.

- Candidate A's Z-score (≈1.67) is higher than Candidate B's (≈1.12) — **A stands relatively stronger** within their own track, even with the lower raw score. The reversal happens because each score is judged against its own track's mean and spread, not a shared scale.
- A sanity check across all 300 standardized scores confirmed the math: mean ≈ 0, SD ≈ 1, as standardization requires.

## Part C — Normality, the Empirical Rule & Skewness

Tests whether a "normal distribution" assumption actually holds for two variables.

- Data Science assessment scores have low skewness (0.048) — close enough to symmetric that the 68–95–99.7 empirical rule is a reasonable approximation. The theoretical upper-tail estimate (2.5% of students above mean + 2 SD) landed close to the observed rate (2.1%).
- Starting salary is right-skewed (skewness 0.024 overall, and Bowley skewness confirms it varies clearly by track — Cyber Security 0.249 vs UI/UX's near-symmetric 0.010). A normal-distribution assumption on salary needs real caution.

## Part D — Box Plots: Lab Task Time, Cairo vs Alexandria

The lab manager claimed "Alexandria is slower." True, but incomplete.

- Alexandria's median completion time is ~1.45 minutes higher than Cairo's — the "slower" claim holds.
- But Alexandria's IQR is more than double Cairo's, and it carries 3 genuine outliers (Cairo has none). Alexandria isn't just slower on average — it's **far less consistent**, which the median-only claim never captured.
- For anything deadline-sensitive, Cairo is the safer branch: lower median *and* far more predictable.

## The Helpdesk Model — Poisson Tickets

A separate dataset (180 days of support tickets) tests whether ticket volume follows a Poisson process.

- Mean (3.71) and variance (3.61) are close — the classic Poisson signature — and two of the three required conditions (discreteness, randomness) clearly hold.
- The weak link is **independence**: a single root cause (an outage, a buggy update) can inflate several consecutive days at once, which a pure Poisson model doesn't expect.
- Used anyway: P(a day exceeds 6-ticket capacity) ≈ 8.2%, which projects to roughly **15 days a term** running over capacity — small odds per day, real enough over a term to justify an on-call plan rather than fixed staffing.

---

## My Part: Placement Odds (Binomial Model)

I worked on the section modeling job-offer probability using the **Binomial distribution**, applied to the `graduates` dataset — the report's claim that "attend enough interviews and an offer is basically guaranteed."

**What it covers:**
- Testing whether the binomial model's four assumptions actually hold here: fixed number of trials, two outcomes, constant probability of success, and independence between trials. The **constant-probability assumption is the one that breaks** — graduates aren't equally prepared, so treating everyone as sharing one success rate is the flaw baked into the model from the start.
- Estimating a pooled success probability (p ≈ 0.292 per interview) and using it to compute offer probabilities across different interview counts — a graduate attending 6 interviews has roughly an **87.4% chance of at least one offer**.
- Checking the model against reality: the model predicts a **19.4% zero-offer rate**; the actual observed rate is **22%**.

**The key finding:** that 2.6-point gap isn't a rounding error — it's a symptom of using one average probability for every graduate. Some candidates are genuinely stronger than others, and pooling everyone into a single *p* hides that spread. Weaker candidates are the ones actually landing at zero offers, and a constant-p model will always understate how many end up there.

![Zero-offer rate: model vs reality](zero_offer_model_vs_reality.png)

---

## Part G — The Corrected Report

The original report made four confident claims. Here's what actually survives contact with the statistics:

**No single track is straightforwardly "strongest" or "weakest."** Data Science leads on median salary with a tight, dependable spread of scores. Cyber Security scores are the most consistent of the four tracks. UI/UX has the lowest median salary and the widest score spread — but its salaries are distributed just as evenly as any other track, and a meaningful share of its graduates outperform the average graduate in stronger-scoring tracks. "Weakest" was the wrong word for UI/UX; **"most variable"** is the accurate one.

**What the report got right vs. wrong, claim by claim:**

| Original claim | Verdict | Why |
|---|---|---|
| "Different scales and spreads" | ✅ Confirmed | CV shows UI/UX has ~3× the relative variability of Cyber Security on scores — a raw-mean comparison across tracks was never fair. |
| "Mean salary is not symmetric" | ✅ Confirmed | Every track shows mean > median; salary is right-skewed, and the median — not the mean — was the correct number to quote. |
| "UI/UX's top students outperform yours" | ⚠️ Partially confirmed | Standardization shows some UI/UX graduates do rank highly within their own track — but UI/UX's median salary and median score are genuinely the lowest of the four. The track isn't unfairly penalized on the numbers; it was just mischaracterized in how those numbers were compared. |
| "Attend enough interviews and an offer is basically guaranteed" | ⚠️ Overstated | 87.4% at 6 interviews is real, but pooling one probability across all graduates hides that some candidates carry meaningfully worse odds than others — the model understates the zero-offer group by design. |

**Bottom line:** every flawed claim in the original report traces back to the same root cause — comparing raw means/std devs across groups that don't share the same scale or shape, without checking the assumption first. That's the rule this whole case study was built to teach.

## Tools

Excel (formulas only, no VBA/scripting) — CV, Z-scores, skewness (Pearson & Bowley), the empirical rule, box-plot fences, binomial and Poisson probability functions, and manual verification of every model against the observed data.

---

Data Analyst | Transforming raw data into clear, actionable insights using Power BI, SQL, Excel, and Python.
📫 Connect with me on [LinkedIn](https://www.linkedin.com/in/mahmoud-lebda728)
