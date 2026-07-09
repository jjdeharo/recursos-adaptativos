# Operational Specification for AI

**Version 2.0**

## Purpose

This document enables an AI to implement Bayesian adaptive educational resources reliably.  
It is a concise operational specification.  
If theoretical background, examples, or mathematical justification are needed, refer to `documentacion.html` and `matematicas.html`.  
If there is any conflict between both documents, this one takes precedence.

## Usage Instruction

Attach this document to the AI and use the following prompt:

> Read the attached document and implement the resource following the operational rules applicable to the requested type of resource.

## Mandatory Workflow

1. Before implementing, check whether you have the following information:
   - topic;
   - year group or age;
   - learning objective;
   - resource type;
   - main purpose;
   - number of hypotheses or levels;
   - interaction type;
   - approximate number of questions or steps;
   - expected final output.
2. If essential information is missing, ask only for what is strictly necessary.
3. If the information has already been provided, do not repeat questions — proceed directly to implementation.
4. The result must be a static web resource in `HTML + CSS + JavaScript`, preferably without a backend, unless another format is explicitly requested. It may be organised into one or several files if that improves clarity, maintainability, or reuse.

## Design Rules

- The resource must not be linear if the purpose requires adaptation.
- Each learner response must modify the system's estimated state.
- Adaptation may affect:
  - the next question;
  - difficulty;
  - activity type;
  - explanation;
  - help or hints;
  - the learning pathway;
  - the moment of termination.
- The final output must be pedagogical, not merely a score.

## Learner State

- Represent the learner's state as a probability distribution over `n` hypotheses.
- If there is no reliable prior information, use a uniform distribution. This rule holds for **level** hypotheses; not for binary **error factors**. A uniform prior over the `2^k` profiles (or over `{present, absent}` for each factor) amounts to claiming that each error has a 50 % prior probability of being present: it is not neutral, it is a strong claim about its prevalence, and it biases the first few questions toward false positives (errors the student does not have shown as "indeterminate" or "probable"). For error factors, start from a moderate informative prior; see "Likelihoods".
- If the hypotheses are hierarchical, assign them ordered, centred `theta` values.
- If the teacher does not set values, use a symmetric scale centred at 0.
- The `theta` scale is fixed and depends only on the number of hypotheses, not on the question bank. With `n` hierarchical hypotheses, use values centred on `0` with intervals of `2`: `theta_i = 2 * (i - 1) - (n - 1)`, so that `theta` runs over `{-(n-1), ..., +(n-1)}` and `theta_max = n - 1`. Examples: `n = 2` → `{-1, +1}`; `n = 3` → `{-2, 0, +2}`; `n = 4` → `{-3, -1, +1, +3}`; `n = 5` → `{-4, -2, 0, +2, +4}`.
- Place the difficulties within the central half of that scale: `b_q` must lie in `[-theta_max / 2, +theta_max / 2]`. Since `theta_max` depends on `n`, the conversion of difficulties also depends on `n`, not only on the number of categories.
- If the teacher gives `k` qualitative difficulty categories, spread them uniformly over that interval: `b_j = (theta_max / 2) * (2 * (j - 1) / (k - 1) - 1)` for `j = 1..k` (if `k = 1`, use `b = 0`). Examples: `n = 3` and `k = 3` → `{-1, 0, +1}`; `n = 3` and `k = 5` → `{-1, -0.5, 0, +0.5, +1}`; `n = 4` and `k = 3` → `{-1.5, 0, +1.5}`; `n = 2` and `k = 5` → `{-0.5, -0.25, 0, +0.25, +0.5}`.
- Do not use a fixed difficulty table independent of `n`: with few hypotheses and many categories it would produce difficulties outside the range of levels (for example, `n = 2` with `b = ±2`) and cancel the separation between scales.
- If the teacher gives numeric `b_q` values outside the interval, clamp them to the interval.
- Never recompute `theta` from the extremes of the bank. A single atypically hard or easy question must not redefine the scale: if you stretch `theta` to accommodate it, you saturate the likelihoods of the rest of the bank (all probabilities pinned to `c_q` or to `1`) and the posterior makes overconfident jumps on a single answer.
- If most of the bank's difficulties fell outside the interval, do not stretch the scale: review the definition of levels and difficulties with the teacher, because the design is inconsistent.
- Comparability invariant: with `a_ef = 1.25` and intervals of `2` between adjacent hypotheses, the product `a_ef * Δtheta = 2.5` equalizes the **maximum slope** of the ICC across formats, whatever `n` is (as `n` grows so does `theta_max`, but the spacing between adjacent hypotheses stays the same). Keep this invariant across resources so that confidences and convergence speeds remain comparable. That comparability is strict only between resources with the same number of hypotheses `n`: with the fixed scale, the margin between the extreme level and the hardest item is `(n - 1) / 2`, so with `n = 2` (mastered / not mastered) the extreme items confirm weakly (the probability that the top level answers the hardest item correctly is ≈ 0.65 for open, ≈ 0.77 for 4 options) and it is advisable to compensate with more questions or to define the scale from a target probability `P*`.
- The invariant equalizes the slope, **not the maximum strength of the evidence**. On a failure, the `(1 - c)` factor cancels in the ratio and the likelihood ratio between adjacent hypotheses tends to `e^(a * Δtheta)` with the **nominal** `a` (`= a_ef / (1 - c)`), not with `a_ef`. That is why a failure on an easy item with higher pseudo-guessing carries far more evidence than the same failure on an open question: with `Δtheta = 2`, the failure likelihood ratio is ≈ 12 for open (`c = 0`), ≈ 28 for 4 options (`c = 0.25`), and ≈ 148 for true/false (`c = 0.5`). To avoid these near-deterministic posterior jumps, apply the mastery ceiling of the ordinal case (see "Likelihoods").

## Bayesian Update

After each response:

1. compute the likelihood of that response under each hypothesis;
2. multiply the prior by the likelihood;
3. normalise;
4. use the result as the new state.

This must be done after each relevant interaction.

## Model Choice

- If the main purpose is to estimate an ordered level of mastery, use ordinal hypotheses and IRT 3PL.
- If the main purpose is to identify a mutually exclusive strategy or error, use nominal hypotheses with explicit likelihoods per question.
- If several errors, skills, or needs can coexist, use separate dimensions or full profiles.
- If the resource must estimate level and diagnose errors, combine a global ordinal distribution with parallel diagnostic distributions.
- Do not force coexisting errors into a single ordinal scale.

## Likelihoods

- If the hypotheses represent ordered levels of mastery, use IRT 3PL.
- Recommended formula:

`P(correct | H_i, q) = c_q + (1 - c_q) / (1 + exp(-a * (theta_i - b_q)))`

- Use by default:
  - `a_ef = 1.25` (target effective discrimination)
  - `a = a_ef / (1 - c_q)`
  - `c_q = 1 / m_q` if guessing is possible
  - `c_q = 0` if it is not
- Do not set `a` directly: fix the target effective discrimination `a_ef = 1.25` and compute `a` per question as `a = a_ef / (1 - c_q) = 1.25 / (1 - c_q)`.
  - Do this **always**, even if all questions have the same number of options. This rule equalizes the ICC's maximum slope; it does not guarantee that every format provides the same expected information. A fixed `a` with different `c_q` produces different and non-comparable real slopes (for example, fixed `a=1.5` gives `a_ef=1.0` with 3 options but `a_ef=1.125` with 4). Information-gain selection will still favor the items that provide more evidence.
  - `a_ef = 1.25` guarantees that `a` stays within the usual psychometric range (0.5–2.5) for any format with 2 or more options: the extreme case, true/false (`c_q=0.5`), gives exactly `a = 2.5`.
  - Values you should obtain: open (`c_q=0`) → `a=1.25`; 5 options (`c_q=0.20`) → `a=1.5625`; 4 options (`c_q=0.25`) → `a≈1.667`; 3 options (`c_q=1/3`) → `a=1.875`; true/false (`c_q=0.5`) → `a=2.5`.
- Apply a **mastery ceiling** in the ordinal case too: bound `P(success | H_i, q) <= 0.95` (equivalently, `P(failure) >= 0.05`) for every hypothesis, including the highest level on easy items. The pure 3PL lets `P(success) → 1` on easy items with higher pseudo-guessing, so a single failure drives the posterior almost deterministically (with `c = 0.5` the failure likelihood ratio between adjacent levels reaches ≈ 148, against ≈ 12 for open). The ceiling models slip, which the 3PL does not account for, and aligns the ordinal case with the nominal one, which already imposes `0.9`–`0.95`, never `1`. Use `0.90` to be more conservative.
- If the learner fails:

`P(incorrect | H_i, q) = 1 - P(correct | H_i, q)`

- If the hypotheses are not hierarchical, distinguish two cases.
- If they are **mutually exclusive alternatives** (for example, strategy A / strategy B / correct mastery), do not use logistic IRT. Generate for each question a vector of `n` likelihoods `P(correct | H_i, q)`, one per hypothesis.
- If the errors or needs **can coexist**, do not force them into a single nominal distribution. Model one dimension per factor (`longer-decimal`, `zero-ignoring`, etc.) or, equivalently, a distribution over **full profiles** (`2^k` combinations for `k` factors).
- **Prior over error factors: not uniform.** Conceptual errors are usually a minority, so assign each factor a low prior probability of being present (by default `P(error) ≈ 0.2`–`0.3`) rather than the uniform `0.5`. A uniform prior here is not neutral: it assumes each student has each error with probability `0.5` and produces false positives in the first few questions. The resource applies this default automatically; the teacher may adjust their group's prevalence if they wish, but is not required to.
- In that multifactorial case, each question should define how each profile responds. The minimum is a probability of success by profile; better still is an option-level distribution `P(R = r | profile, q)` so that the chosen distractor carries evidence, not only correct/incorrect.
- If you start from simple factors, assign each value by asking: "if the student had this error, with what probability would they answer this question correctly?". Low if the question attacks the concept that error distorts; high if the error does not interfere.
- Bound each value: not below the guessing floor `1/m` (m options), except for the case in the next point; the mastery hypothesis or mastery profile around `0.9`–`0.95`, never `1`.
- If a specific distractor is the answer an error produces, the probability of **that option** should rise under that profile, and the probability of success may fall near or below chance: the student is actively drawn toward that wrong option.
- Do not fine-tune the decimal: use buckets (`≈0.9` unaffected / `≈0.5` partial effect / `≈0.15–0.25` distractor capturing the error / `≈1/m` floor when failing by guessing). What matters is that the correct factors or profiles separate clearly in discriminating questions.
- Do not use fixed global tables if each question can generate its own likelihoods.
- Diagnostic quality does not depend only on the algorithm: it also depends on how the hypotheses, categories, difficulties, concepts, and errors are defined. If those classifications are poorly structured, if the relevant errors are not well identified, or if the bank covers important cases poorly, the generated likelihoods may misrepresent reality and bias the adaptation.
- In the final diagnosis do not compute an expected `theta` (meaningless without order). If the model is exclusive nominal, you may report the MAP hypothesis and its probability. If the model is multifactorial, report for each factor whether it is **present**, **absent**, or **undetermined**, together with its marginal probability or confidence.
- With an informative prior (`P(error) ≈ 0.2`–`0.3`), absence already starts at `0.7`–`0.8` with no evidence at all: do not declare a factor **absent** merely because its marginal exceeds the confidence threshold. Also require its minimum sample of evidence, and distinguish in the report between "confirmed absent" (with evidence) and "insufficient evidence" (the prior's default value).

## Partial Credit Responses

- If a response is not simply correct or incorrect but admits degrees (multiple steps, weighted components, partial credit), summarise it as a score `s` between `0` and `1`.
- If you only have an aggregated partial score `s`, construct a geometric likelihood:

`L(H_i) = P(correct | H_i, q)^s * P(incorrect | H_i, q)^(1 - s)`

- Use this `L(H_i)` in the Bayesian update instead of choosing between `P(correct)` and `P(incorrect)`. Normalisation and the rest of the process remain unchanged.
- Edge cases: `s = 1` is equivalent to full credit; `s = 0` to full failure. An `s = 0.5` is not necessarily neutral: it favors the hypotheses for which the item predicts an intermediate score.
- If the item has `J` approximately independent components and `s` is the weighted fraction of correct components, you can preserve the strength of the evidence with `L(H_i) = P(correct | H_i, q)^(sJ) * P(incorrect | H_i, q)^((1 - s)J)`.
- Careful: the exponent `J` treats the response as `J` independent trials with the **same** probability `p_i` of the whole item, so it is only reasonable if the components are of **similar difficulty**. If they differ clearly in difficulty (the usual case in step-by-step tasks), this form **overcounts** the evidence relative to the per-component model; when in doubt, use a `J` smaller than the real number of components (more conservative evidence).
- If you have separate evidence by component, prefer multiplying the component likelihoods instead of reducing everything to a single `s`.
- Define how `s` is calculated in an explicit, self-correctable way: weighted sum of sub-criteria, fraction of correct steps, proximity to the numerical solution, etc. Weights must sum to `1`.
- Do not treat a response that admits degrees as binary: diagnostic information is lost.
- To select the next question, compute information gain over the outcomes the item actually models. For binary or partial-credit items, the approximation with full success and full failure remains acceptable; if you model distractors or full profiles, average over all relevant response outcomes.

## Chance Floor in Composite Items

- If an item is scored across several components with different numbers of options and partial credit is used, the aggregate floor is not the probability of getting the whole item right, but the expected partial score by chance.
- Calculate that expected-score floor as the weighted mean of the chance levels for each component:

`c_q = Σ_j w_j * c_j`  with  `c_j = 1 / m_j`  and  `Σ_j w_j = 1`

- Use that aggregate `c_q` in the logistic function only when the ICC represents the expected score of the composite item.
- If the composite item is scored all-or-nothing, do not use the weighted mean: the probability of full success by chance is the product `Π_j c_j` if the components are guessed independently.
- The weights `w_j` must match those used to calculate the partial credit score `s`.

## Multidimensional Diagnosis

- If you need to know not only the overall level but also which skills or steps are failing, maintain several Bayesian distributions in parallel: one per category or level, and one for each diagnostic dimension (skill, step, error type).
- Update each distribution with the evidence that belongs to it: the overall outcome may feed the level belief; each sub-criterion must feed only its own dimension. Do not use the same global correct/incorrect result to update several independent dimensions if the question requires multiple skills at once, because that duplicates evidence and misattributes the cause of the error.
- Each dimension may have its own chance floor `c` depending on its number of options, so their percentages are not directly comparable across dimensions: the common reference is the latent value `theta`.
- Do not merge into a single distribution dimensions that can coexist: use separate distributions.
- If several diagnostic dimensions interact strongly, you may replace those independent distributions with a single distribution over full profiles; the key is not to force as mutually exclusive factors that can in fact coexist.
- The level by category guides what to practise; the diagnosis by dimension guides what to explain or reinforce.
- If you combine an ordinal level distribution with error factors (combined model), both are parallel marginal estimates fed by the same evidence, not independent findings. Check the coherence of the final result: if the estimated level is high and some error remains "present", present it as a nuance of the level ("masters X, although error Y persists"), not as juxtaposed contradictory conclusions.

## Questions and Activities

Each question or self-correctable interaction must have, where applicable:

- text or statement;
- difficulty `b_q`;
- number of options `m_q`;
- category or concept;
- correction criterion;
- optional help or hint;
- minimum feedback after the response;
- specific explanation, especially if the main purpose is learning, practice, or reinforcement.

If the resource is procedural or tutorial in nature, interactions may not be classical questions, but they must still be self-correctable or explicitly assessable.

## Adaptive Selection

For each available candidate:

1. compute the marginal probability of a correct response;
2. simulate the posterior if correct;
3. simulate the posterior if incorrect;
4. compute the expected posterior entropy;
5. compute the expected information gain.

Select the candidate with the highest expected information gain when the main purpose is diagnostic and the goal is to estimate a global level.

If the state consists of several parallel distributions (error factors or diagnostic dimensions), apply the same procedure with the item's **total gain**: the sum of the expected gains over each distribution (in the factorised representation, the joint entropy is the sum of entropies, so the expected joint reduction is the sum of reductions). Average over the outcomes the item models (per option, if there are diagnostic distractors).

In adaptive practice or reinforcement resources with multiple categories, problem types, or concepts, use a two-phase selection:

1. **Initial diagnostic phase.** Until each relevant category has a minimum sample of attempts, prioritise categories with less evidence. Within them, use expected information gain to choose the most diagnostic question. A reasonable default is to require at least `2` attempts per category before leaving this phase. Bear in mind that `2` is the defensible minimum, not a comfortable value: with active forgetting the per-category marginal is volatile, so raise the minimum sample if the bank allows. If there are many categories, this phase can consume the session (with `10` categories and `2` attempts that is already `20` questions): group related categories into blocks or reduce the minimum sample so that the initial diagnosis does not exceed the practical maximum.
2. **Reinforcement phase.** When all relevant categories have a minimum sample, prioritise the category with the lowest estimated mastery. Within that category, do not automatically choose the most difficult question: select an informative question close to the learner's estimated level. A reasonable rule is `utility = α * IG_norm + (1 - α) * fit`, with both scales defined in `[0, 1]`: `IG_norm = IG(q) / max IG` among the current candidates, and `fit = max(0, 1 - |b_q - E[theta]| / 2)`, which equals `1` when the difficulty matches the estimated level (`E[theta] = Σ p_i * theta_i`) and `0` when it is a full level interval away. Without both scales defined on the same range, the weight `α` means nothing.

Do not use Shannon entropy as the sole permanent criterion when the main purpose is to practise or reinforce. Shannon indicates where there is the most diagnostic uncertainty; reinforcement must also attend — and preferably give priority — to what the learner has least mastered.

In adaptive tests for global diagnosis with a stopping criterion, it is not mandatory to apply the reinforcement phase. In that case, it is enough to maximise expected information, diversifying categories in ties or when several candidates have equivalent utility.

Bear in mind that maximising expected information tends to propose questions with a probability of success close to `50%`. With young learners or learners who struggle, you may open with an accessible question or interleave one with a high chance of success: you lose some informational efficiency in exchange for sustaining motivation. This is an optional pedagogical decision.

If several candidates are practically equivalent:

- break ties with randomisation;
- favour categories or concepts that have been repeated less.

Do not use simple deterministic selection for ties.

Avoid repeating the same question mechanically:

- do not repeat exactly the same item consecutively unless the design explicitly requires an immediate retry;
- apply a frequency penalty or a recent-exclusion window so that a recently used item loses priority for several selections;
- if several candidates have similar utility, prefer the least recent and least repeated one;
- only allow an item to be reused when the bank is small, there are no comparable alternatives, or the pedagogical goal is to deliberately revisit that exact case.

Do not confuse this problem with a simple tie-breaking rule:

- randomisation only helps if several reasonable candidates actually exist;
- to avoid repetition, the bank needs local redundancy: several alternative questions for each relevant category, difficulty level, item type, or error;
- if a diagnostic area has only one useful item, the AI should recognise that the bank is insufficient for varied adaptation in that area;
- in that case do not fake variety with randomisation: reuse the item only when necessary, temporarily shift the pedagogical objective, or mark the result as limited by the size of the bank.

Evidence from a reused item:

- if an item is reused after the learner has seen its correction or explanation (including the immediate retry), a subsequent correct answer is not full evidence of mastery: it may reflect only memory of the feedback. Treat it like hints: partial credit with a reduced `s` (the more explicit the explanation shown, the lower) or, if the explanation gave away the answer, exclude it from the Bayesian update and use it only as practice;
- the best local redundancy is not repeating the same item but having **parameterised variants** of the same type (same concept, difficulty, and format with different data): each variant counts as a new item and carries no feedback contamination.

## Stopping Criterion

Stop the session when reasonable closing criteria are met, for example:

- minimum number of questions already reached;
- entropy below `H_stop`;
- most probable hypothesis above `p_min`;
- no useful questions remaining;
- the best remaining question provides very little information;
- the practical maximum is reached.

If using `p_min`, calculate the indicative threshold:

`H_stop = -p_min * log2(p_min) - (1 - p_min) * log2((1 - p_min) / (n - 1))`

After the minimum number of questions has been met, check the diagnostic's confidence. When `H_stop` is derived from the same `p_min`, the condition `max(p_i) >= p_min` already implies `H <= H_stop`: checking both is harmless but **redundant**, it adds no requirement. The control that does add something different is the **separation** from the second hypothesis:

`P(winner) - P(second) >= Δ_min`

However, the separation only adds a requirement if `Δ_min > 2 * p_min - 1`. With `p_min = 0.80`, requiring `max(p_i) >= p_min` already forces a separation `>= 0.60`, so a `Δ_min` of 0.3–0.4 would be as redundant as the entropy. That is why separation is useful mainly **as an alternative to a high `p_min`, not as an addition**: when there are many hypotheses and requiring `max(p_i) >= 0.80` is impractical, close with a moderate `max(p_i)` (for example `>= 0.50`) **and** `Δ_min >= 0.3–0.4`, which requires the winner to be clearly ahead of the second even if the remaining mass is spread out. With `n = 2`, separation and `p_min` are equivalent (`sep = 2 * max - 1`).

In multifactorial models, evaluate stopping **per factor**: a factor is decided when its marginal leaves the undetermined zone (for example, `P(error) >= 0.7` present, `<= 0.3` absent) and has its minimum sample of evidence. Close when all factors are decided, when no item provides an appreciable gain on the undetermined ones, or when the practical maximum is reached; undecided factors are reported as undetermined, not forced.

If the session closes due to the maximum number of questions, an exhausted bank, or low marginal utility without reaching the chosen confidence criterion, present the result as provisional.

Minimum rules:

- do not close too early;
- do not artificially extend the session when marginal utility is already low;
- if uncertainty remains high, the result must be indicated as provisional.

## Continuous Practice Without Stopping

- In open practice or reinforcement resources there may be no stopping criterion: the session continues while the learner practises.
- In that mode, the estimated state is not a closed diagnosis but a live estimate that updates with each response.
- Do not apply `H_stop` or `p_min` to close the session; use them, if at all, only to report the level of confidence reached.
- Maintain the two-phase selection throughout the session: minimum diagnosis per category, then reinforcement of what is least mastered.
- The learner's state can change while they practise: apply exponential forgetting so the estimate tracks their current state. Before each update, attenuate the posterior **toward the prior** `pi_i` and renormalise: `p_i <- p_i^lambda * pi_i^(1 - lambda)` (divided by the sum). With a uniform prior this coincides with the simple form `p_i <- p_i^lambda / Σ_j p_j^lambda`.
- **Do not attenuate toward the uniform when the prior is informative.** The simple form has the uniform distribution as its fixed point: an error factor with prior `P(error) ≈ 0.25` that goes some time without receiving evidence drifts on its own toward 50 % (≈ 0.40 after 20 steps with `lambda = 0.95`) and reappears as "undetermined" or "probable" without the learner having done anything — the very false positive the informative prior is there to avoid. Anchored to the prior, forgetting discards old evidence (evidence from `k` steps ago weighs `lambda^k`) but the prior always keeps full weight, and a distribution with no new evidence stays at its prior instead of degrading.
- Apply the attenuation to every distribution at every step of the session, including those that receive no evidence from that response: this way all of them track the passage of time and, thanks to the anchoring, unobserved ones return to their prior, not to the uniform.
- Use `lambda ≈ 0.9`–`0.98` in continuous practice or reinforcement (practical rule: `lambda = 1 - 1/W`, where `W` is the number of recent responses that should dominate the estimate). In short-session diagnostic resources use `lambda = 1` (no forgetting): there it would only add noise.
- With forgetting active, count the minimum sample per category or dimension over a recent window (attempts within the last `W ≈ 1/(1 - lambda)` responses), not over the whole session: evidence expires with forgetting, but a cumulative counter does not, and a category sampled only at the start would still count as diagnosed while its posterior has already degraded.
- With forgetting active, present the confidence as referring to the learner's recent state.
- If you need to model learning explicitly (for example, a higher probability of moving up a level right after an explanation), use a transition model (Bayesian Knowledge Tracing); see `matematicas.html §3.5`.

## Stage-Based Pathways

If the resource has successive phases, techniques, or stages:

- distinguish between overall estimation and local estimation per stage;
- do not promote a stage using only the accumulated global belief;
- decide whether a stage has been passed based on evidence generated within that stage.

To consider a stage passed, it is advisable to require at least:

- sufficient local confidence;
- sufficiently low local entropy;
- and an explicit minimum of observed performance in that stage, measured on evidence that is **not** selected by maximum information.

Be careful with the raw percentage of correct answers as a criterion: if within the stage you select items by maximum information gain, every learner's success rate tends by design toward `(1+c)/2` (≈ 50 % with no guessing, ≈ 62 % with four options), so a fixed threshold such as "60 % correct" may block learners who do master the stage, and its effect depends on the item format. To measure performance in a comparable way, use one of these two routes:

- **Exit items (recommended):** require passing 1-2 items of difficulty representative of the stage goal, selected **without** an informative criterion (not by maximum IG). By fixing the difficulty, a correct answer does inform about mastery. Format matters: avoid true/false for exit items (a single T/F item lets most non-masters through by chance, `P(correct | no mastery) ≈ 0.6`); with 4-5 options require passing both; with open-ended items bear in mind that requiring 2 out of 2 blocks ≈ 1 in 4 learners who do master the stage. The exit filter complements the local confidence `p_min`, which already screens: its role is to catch bad calibration, not to decide on its own.
- **Model consistency:** require that the observed rate not fall well below the rate expected under the local-mastery hypothesis (this is the `l_z` person-fit statistic from the foundations applied as a stage criterion). With the few items of a stage, the normal approximation of `l_z` is weak: compare observed versus expected correct answers with a margin of ~1 standard deviation (or an exact binomial test) and treat it as an indicative signal, not a hard block.

The resource itself applies all of this automatically, from the difficulties it already knows: it does not require the teacher to understand the methodology or to configure anything, unless they choose to.

Reasonable example:

- `p_min = 0.80`
- passing 1-2 exit items of representative difficulty, instead of a fixed percentage-correct threshold

If the learner repeats a stage:

- reset the local estimate for that stage, unless there is an explicit pedagogical justification against doing so.

Completing a stage does not necessarily imply having passed it.

## Recovery and Reinforcement

- If the learner shows difficulty, the system should be able to:
  - offer hints;
  - show an explanation;
  - propose reinforcement;
  - change the activity type;
  - temporarily reduce difficulty;
  - allow retry or review.
- If the learner shows mastery, the system may:
  - advance;
  - extend;
  - increase complexity;
  - reduce assistance.
- Effect of hints on the evidence: if the learner answers correctly **after a hint**, do not record it as a full success in the Bayesian update. A hint raises that item's probability of success, so treat it as partial credit with `s < 1` (the smaller the more decisive the hint; if the hint practically gives the answer, the evidence of mastery is almost nil) and apply the geometric partial-credit likelihood. Response times and the use of aids may be recorded for information, but this version defines no likelihood for them: they do not by themselves alter the update.

## Final Output

The final output must include, depending on the resource type:

- diagnosis or estimated level;
- degree of confidence;
- detected difficulties;
- observed strengths;
- pedagogical recommendation;
- next step.

If it is a learning pathway or activity, also add:

- the route followed;
- stages passed or not;
- help used;
- areas to reinforce.

Do not declare high mastery based on very few attempts. If the result makes claims by category or dimension, require a minimum sample in those categories or dimensions before presenting them as firm. If the estimate is global, the minimum sample may refer to the session as a whole; limit or mark the estimate as provisional when evidence is scarce.

Do not return only a score or label.

If two hypotheses end with close probabilities, show the full posterior distribution (for example, a bar chart), not just the winning label.

In the student's view, the pedagogical recommendation and the next step should carry **more visual weight** than the level label. Phrase the result in terms of a task ("it would help you to practise X before Y"), not of a trait ("you are basic level"): the literature on expectations indicates that the trait label carries more risk. The level label with its probability is better reserved for the teacher's view.

If the model diagnoses a specific error (for example, "confuses mass with weight", with its probability), communicate it to the student as a **hypothesis to check together**, not as a verdict ("let's check whether…"), especially in primary school. The error label with its probability is appropriate for the teacher's view.

## Implementation Constraints

- The interface must be comprehensible to both learners and teachers.
- It must clearly show progress and feedback.
- If visible formulae are used, accompany them with a readable interpretation.
- Avoid depending on a backend unless one has been requested.
- Avoid long open-ended questions without reliable automatic correction.
- Minimum accessibility by default, even if the teacher does not ask for it: sufficient contrast, keyboard navigation, not conveying information through colour alone, resizable text, and no time limit by default (unless the design requires it and warns about it).
- Privacy by default: do not send the student's data outside the browser. The static resource without a backend already guarantees this and is the preferred option when handling minors' data. If persistence of results is requested, warn about data protection and prefer local export (for example, downloading a file) over uploading them to a server.

## Recommended Default Values

If the teacher does not specify parameters:

- `n = 3` hypotheses or levels;
- target effective discrimination `a_ef = 1.25`; always compute `a = 1.25 / (1 - c_q)` per question (with `c_q = 0` it gives `a = 1.25`; maximum `a = 2.5` for true/false);
- `p_min = 0.80`;
- minimum number of questions: between `4` and `6`;
- minimum diagnostic sample per category in adaptive practice: `2` attempts;
- practical maximum: between `10` and `20`, depending on the resource type;
- weight of information gain in the reinforcement phase: `α = 0.65` (reasonable range `0.6`–`0.7`);
- exponential forgetting: `lambda = 0.95` in continuous practice or reinforcement (effective memory ≈ `20` responses); `lambda = 1` in short-session diagnosis;
- partial credit: sub-criterion weights defined by the design, summing to `1`;
- minimum sample per category or dimension before showing high mastery: `2`–`4` attempts.

## What the AI Must Not Do

- Do not confuse a theoretical explanation with a mandatory implementation rule.
- Do not use the global belief to promote local stages in pathways.
- Do not close the resource without justifying the certainty reached.
- Do not present as definitive a result with high uncertainty.
- Do not limit the output to a bare score.
- Do not treat as binary a response that admits degrees: use partial credit.
- Do not declare high mastery with an insufficient sample.

## Validation and reliability (optional)

These checks are not part of the mandatory flow: they are an optional layer that increases the honesty of the diagnosis without requiring empirical data. Apply them when the purpose is diagnostic and the result will be used to make decisions.

- **Individual pattern fit (person-fit).** When closing a session, assess whether the response pattern is coherent with the estimated level. Compute the standardized index `l_z` from the probabilities of a correct answer that the model assigns to the answered questions under the most probable hypothesis. If `l_z` is strongly negative (as a rough guide, `< -2`), flag the diagnosis as unreliable even if the posterior is high: it usually indicates responses inconsistent with difficulty, slips, or guessing. It is a caution signal, not a formal test; with few questions it is only indicative.
- **Detecting random answering or anxiety during the session (not only at close).** Do not wait until the close to compute the fit: if during the session the person-fit or an implausible streak (failing easy questions and getting hard ones right, or answering too fast) suggests random answering or anxiety, the resource should be able to **pause and redirect** ("you seem to be going very fast, shall we continue calmly?") instead of continuing to consume the bank. Do it tactfully, without accusing, and resume normally.
- **Design separability (Monte Carlo).** As a property of the test (not of the student), estimate how reliably the bank distinguishes the levels: generate synthetic respondents located at the `theta` of each hypothesis, run the test on them reusing the same adaptive selection and the same stopping criterion, and build the confusion matrix (true level versus diagnosed). Present it as reliability under the model, never as empirical validity: the respondents come from the model itself, so it measures whether the design discriminates the levels, not whether the parameters reflect reality. **This validation is a tool for the resource's author, not for students: it must not be shown in the student interface.** Implement it in a separate file or utility that the author can run when designing or reviewing the test, not in the material the student receives.

See `matematicas.html §11.7–§11.8` for the formulas and the full framing.

## Note on the Model

The IRT 3PL function described here is used as an initial likelihood generator when no empirical data are available. It is not equivalent to a validated psychometric calibration. If a sufficient number of real responses accumulates, it is advisable to recalibrate difficulties, discrimination, and the pseudo-chance parameter.
