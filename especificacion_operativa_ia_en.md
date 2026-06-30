# Operational Specification for AI

**Version 1.3**

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
- If there is no reliable prior information, use a uniform distribution.
- If the hypotheses are hierarchical, assign them ordered, centred `theta` values.
- If the teacher does not set values, use a symmetric scale centred at 0.
- When using centred difficulties `b_q`, make the `theta` scale span a wider range than the difficulty scale. Practical rule: `theta_max = 2 * max(abs(b_q))`.

## Bayesian Update

After each response:

1. compute the likelihood of that response under each hypothesis;
2. multiply the prior by the likelihood;
3. normalise;
4. use the result as the new state.

This must be done after each relevant interaction.

## Likelihoods

- If the hypotheses represent ordered levels of mastery, use IRT 3PL.
- Recommended formula:

`P(correct | H_i, q) = c_q + (1 - c_q) / (1 + exp(-a * (theta_i - b_q)))`

- Use by default:
  - `a = 1.5`
  - `c_q = 1 / m_q` if guessing is possible
  - `c_q = 0` if it is not
- If the learner fails:

`P(incorrect | H_i, q) = 1 - P(correct | H_i, q)`

- If the hypotheses are not hierarchical, do not use logistic IRT. Define specific diagnostic likelihoods.
- Do not use fixed global tables if each question can generate its own likelihoods.

## Partial Credit Responses

- If a response is not simply correct or incorrect but admits degrees (multiple steps, weighted components, partial credit), summarise it as a score `s` between `0` and `1`.
- Construct the likelihood by interpolating between correct and incorrect:

`L(H_i) = s * P(correct | H_i, q) + (1 - s) * P(incorrect | H_i, q)`

- Use this `L(H_i)` in the Bayesian update instead of choosing between `P(correct)` and `P(incorrect)`. Normalisation and the rest of the process remain unchanged.
- Edge cases: `s = 1` is equivalent to full credit; `s = 0` to full failure; `s = 0.5` provides no information and leaves the posterior almost unchanged.
- Define how `s` is calculated in an explicit, self-correctable way: weighted sum of sub-criteria, fraction of correct steps, proximity to the numerical solution, etc. Weights must sum to `1`.
- Do not treat a response that admits degrees as binary: diagnostic information is lost.

## Chance Floor in Composite Items

- If an item is scored across several components with different numbers of options, the probability of a correct response by chance is not `1/m`.
- Calculate the aggregate chance floor as the weighted mean of the chance levels for each component:

`c_q = Σ_j w_j * c_j`  with  `c_j = 1 / m_j`  and  `Σ_j w_j = 1`

- Use that aggregate `c_q` in the logistic function when generating the likelihoods for the full item.
- The weights `w_j` must match those used to calculate the partial credit score `s`.

## Multidimensional Diagnosis

- If you need to know not only the overall level but also which skills or steps are failing, maintain several Bayesian distributions in parallel: one per category or level, and one for each diagnostic dimension (skill, step, error type).
- Update all relevant distributions with the same response: the overall outcome feeds the level belief; each sub-criterion feeds the belief for its own dimension.
- Each dimension may have its own chance floor `c` depending on its number of options, so their percentages are not directly comparable across dimensions: the common reference is the latent value `theta`.
- Do not merge into a single distribution dimensions that can coexist: use separate distributions.
- The level by category guides what to practise; the diagnosis by dimension guides what to explain or reinforce.

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

In adaptive practice or reinforcement resources with multiple categories, problem types, or concepts, use a two-phase selection:

1. **Initial diagnostic phase.** Until each relevant category has a minimum sample of attempts, prioritise categories with less evidence. Within them, use expected information gain to choose the most diagnostic question. A reasonable default is to require at least `2` attempts per category before leaving this phase.
2. **Reinforcement phase.** When all relevant categories have a minimum sample, prioritise the category with the lowest estimated mastery. Within that category, do not automatically choose the most difficult question: select an informative question close to the learner's estimated level. A reasonable rule is to combine information gain with difficulty appropriateness, penalising questions too far from the working zone.

Do not use Shannon entropy as the sole permanent criterion when the main purpose is to practise or reinforce. Shannon indicates where there is the most diagnostic uncertainty; reinforcement must also attend — and preferably give priority — to what the learner has least mastered.

In adaptive tests for global diagnosis with a stopping criterion, it is not mandatory to apply the reinforcement phase. In that case, it is enough to maximise expected information, diversifying categories in ties or when several candidates have equivalent utility.

If several candidates are practically equivalent:

- break ties with randomisation;
- favour categories or concepts that have been repeated less.

Do not use simple deterministic selection for ties.

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

After the minimum number of questions has been met, a firm diagnostic close should ideally verify both conditions: `H <= H_stop` and `max(p_i) >= p_min`. If the session closes due to the maximum, an exhausted bank, or low marginal utility without meeting them, present the result as provisional.

Minimum rules:

- do not close too early;
- do not artificially extend the session when marginal utility is already low;
- if uncertainty remains high, the result must be indicated as provisional.

## Continuous Practice Without Stopping

- In open practice or reinforcement resources there may be no stopping criterion: the session continues while the learner practises.
- In that mode, the estimated state is not a closed diagnosis but a live estimate that updates with each response.
- Do not apply `H_stop` or `p_min` to close the session; use them, if at all, only to report the level of confidence reached.
- Maintain the two-phase selection throughout the session: minimum diagnosis per category, then reinforcement of what is least mastered.

## Stage-Based Pathways

If the resource has successive phases, techniques, or stages:

- distinguish between overall estimation and local estimation per stage;
- do not promote a stage using only the accumulated global belief;
- decide whether a stage has been passed based on evidence generated within that stage.

To consider a stage passed, it is advisable to require at least:

- sufficient local confidence;
- sufficiently low local entropy;
- and an explicit minimum of observed performance in that stage.

Reasonable example:

- `p_min = 0.80`
- minimum of `60 %` correct responses in the stage

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

## Implementation Constraints

- The interface must be comprehensible to both learners and teachers.
- It must clearly show progress and feedback.
- If visible formulae are used, accompany them with a readable interpretation.
- Avoid depending on a backend unless one has been requested.
- Avoid long open-ended questions without reliable automatic correction.

## Recommended Default Values

If the teacher does not specify parameters:

- `n = 3` hypotheses or levels;
- `a = 1.5`;
- `p_min = 0.80`;
- minimum number of questions: between `4` and `6`;
- minimum diagnostic sample per category in adaptive practice: `2` attempts;
- practical maximum: between `10` and `20`, depending on the resource type;
- weight of information gain in the reinforcement phase: `α = 0.65` (reasonable range `0.6`–`0.7`);
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
- **Design separability (Monte Carlo).** As a property of the test (not of the student), estimate how reliably the bank distinguishes the levels: generate synthetic respondents located at the `theta` of each hypothesis, run the test on them reusing the same adaptive selection and the same stopping criterion, and build the confusion matrix (true level versus diagnosed). Present it as reliability under the model, never as empirical validity: the respondents come from the model itself, so it measures whether the design discriminates the levels, not whether the parameters reflect reality. **This validation is a tool for the resource's author, not for students: it must not be shown in the student interface.** Implement it in a separate file or utility that the author can run when designing or reviewing the test, not in the material the student receives.

See `matematicas.html §11.7–§11.8` for the formulas and the full framing.

## Note on the Model

The IRT 3PL function described here is used as an initial likelihood generator when no empirical data are available. It is not equivalent to a validated psychometric calibration. If a sufficient number of real responses accumulates, it is advisable to recalibrate difficulties, discrimination, and the pseudo-chance parameter.
