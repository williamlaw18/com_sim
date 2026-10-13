# 04 — Analysis and Interpretation (System B)

System B is the researchers' toolkit for inspecting agent communication. It is **strictly separate** from System A.

## Separation rule

- System B reads recorded data; it never writes to agents.
- Classifier outputs and interpretations are never fed back into agents during core experiments — doing so would contaminate the concepts we are trying to discover.
- Any experiment that deliberately crosses this boundary (e.g. signal playback, see [05](05-validation-and-experiments.md)) must be explicitly designed and logged as an intervention.

## Tooling by stage

1. **Initial:** ordinary statistics and visualisation — frequencies, co-occurrence, timing.
2. **Core:** classifiers trained on the two recorded streams (see below).
3. **Unsupervised:** clustering of communication and experience to find recurring patterns *before* assigning any interpretation.
4. **Later, optional:** an LLM may help researchers brainstorm interpretations. Its output is a **hypothesis, never a dictionary**, and must be tested against agent behaviour.

## The two core experiments

| Experiment | Input → Prediction | If it works, it shows… |
|---|---|---|
| **A** | Communication → experience/internal state | Communication carries information about situations |
| **B** | Experience/internal state → communication | Communication is systematically related to experience |

Run both; together they are much stronger evidence than either alone. Evaluate on **held-out, unseen situations**.

## Discovering concepts without defining them

- Feed the classifier **raw** environmental and physiological measurements, not human categories.
- Look for disparate situations that produce the same signal pattern (e.g. predator, aggressive stranger, being trapped, resource vanishing), then investigate what they share.
- Look for signals best explained by a *combination* of factors (internal state + social relationship + anticipated change + history) — candidate concepts with no single human equivalent.

## The four-source comparison (core analysis)

Experiments A and B are extended into the main analysis of the project, specified in [09](09-meaning-and-inner-model.md). Train one classifier per source to predict the agent's words, and compare them:

| Words best predicted by | They are about |
|---|---|
| **Senses** | What's in front of the agent |
| **True world** | Reality, including things inferred |
| **Memory** | Its internal picture: what it remembers, believes or expects |
| **Body** | Itself |

Every classifier must beat a shuffled baseline and the same analysis run on an untrained, random-weight agent, and is scored on held-out episodes. The decisive cases are moments where belief and world disagree (see the mistake tests in [09](09-meaning-and-inner-model.md)).

## Interpretive caution

Always distinguish:

- **A category our classifier found** vs. **a category the agents actually represent or use**.
- **Correlation** vs. **meaning**. A classifier may exploit a confound.

Classifier findings are candidates. They become claims only after validation.
