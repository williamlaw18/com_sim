# 05 — Validation and Experiments

A candidate meaning (e.g. "signal X relates to danger") is a hypothesis until it survives controlled tests.

## Validation methods

### Generalisation tests
- Introduce a **novel** instance of the category (a threat type never seen) — is the same signal used?
- Introduce a **harmless lookalike** — does the signal appear anyway?
- **Remove** the key factor while keeping other features — does communication change accordingly?

### Interventions
- **Signal playback:** play a recorded signal to an agent in a controlled context and measure behavioural change.
- A signal that reliably shifts another agent's behaviour in a contextually appropriate way supports a communicative function.

### Mistake tests (the decisive experiments)
Create moments where the agent's belief and the world disagree, then check whether its words follow the world or the belief. Full table in [09](09-meaning-and-inner-model.md).
- **Removed object:** the agent sees food, then it's secretly removed. Does it keep naming it?
- **Illusion:** show phantom food. Does it name it?
- **Belief injection:** nudge its memory into the "starving" pattern while it's fed. Do its words say "starving"?
- **Body mismatch:** make its noisy body cue mislead it. Do its words track the belief or the truth?

### Cross-condition comparison
- Compare results across agent architectures, channel capacities (P1/P2/P3), interoception conditions (C0/C1/C2), and environments.

## What validation can and cannot establish

- It can show a signal has a **communicative function** and what situations it tracks.
- It does **not** yield a single fixed translation — a signal may encode a category, instruction, prediction, social relation, or a mix.
- It does **not** establish subjective experience.

## The three questions (keep separate)

| Question | How we address it |
|---|---|
| What information does the agent experience? | Recorded inputs, internal variables, interactions — measurable processes, not proof of experience |
| What does the agent communicate about it? | Prediction, behavioural experiments, interventions |
| Does the agent subjectively experience anything? | Not settled by the above; out of scope for claims |

## Human ↔ agent communication

The listen → speak → ask → translate sequence and its rules are in [09](09-meaning-and-inner-model.md). In short, we use only the agents' own validated signals and never train them on ours. Every researcher-sent signal is logged as an intervention. Any English mapping is researcher-side only and must be shown to **preserve agent meanings** rather than impose English ones.
