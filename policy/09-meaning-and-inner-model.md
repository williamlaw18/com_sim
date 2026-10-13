# 09 — Meaning, World Model and Self-Model (Core Research Design)

This is the central research design for com_sim. The rest of the spec exists to support it.

## The research question

> **Do agents develop words with meaning — and do those words express an internal model of the world and of themselves, rather than just what's in front of them?**

It breaks into three parts:

1. **Meaning.** Do their invented signals consistently refer to things?
2. **Internal sense of the world.** Do their signals carry an inner picture of the world, including things they can't currently see?
3. **Internal thinking about themselves.** Do their signals express their own internal state, which they were never directly shown?

**The method:** classify the agents' words against what they sense, what is really in the world, what's in their memory, and what their hidden body is doing. Whichever source best explains the words tells us what the words are *about*.

### In plain terms

A group of identical robots wander a world looking for cookies. Each sees only a little way, but can hear beeps from further off. They start out beeping at random. Over a long time, beeps that happen to help get repeated and start to mean things. We secretly record everything they see, what's really there, what's in their heads, and what they beep. Then we play detective. The decisive trick: let a robot see a cookie, then secretly remove it. If it still beeps "cookie", it's talking about the cookie **in its head**, which shows it has an inner picture of the world.

## Why agents, and not LLMs

An LLM has been trained on vast amounts of human writing about minds, thoughts and consciousness. When it describes an inner life, we can't tell whether it's reporting or reciting. (Researchers call this the "gaming problem".) com_sim agents have never encountered human language. Anything they come to say about the world or themselves, they had to develop on their own.

We also have something LLM research lacks: **ground truth.** We know exactly what the world contains, what the agent could sense, what its memory holds, and what its body is doing. That lets us check what its words refer to, rather than guess.

## The four sources

Every step, the recorder ([03](03-communication-and-recording.md)) captures four things alongside the agent's words:

| Source | What it is | Agent can see it? |
|---|---|---|
| **S — Senses** | What the agent currently perceives | Yes |
| **W — World** | The true state of the world, including things out of view | Only the part in its senses |
| **M — Memory** | The network's internal state (its learned memory vector) | It *is* the agent's mind |
| **B — Body** | Hidden body variables (energy, damage) | **No.** Never fed directly to the network |

### Body is designed, mind must emerge

We design the body, but the agent's representation of it must be learned. Body variables are researcher-only ground truth. The agent is never handed a "fuel gauge", only the consequences (moving slower when weak, dying at zero). If its memory and words come to track its own energy anyway, it worked that out itself.

How much the agent can sense about its own body is an experimental condition:

| Condition | What the agent senses about its body | Purpose |
|---|---|---|
| **C0 — none** | Only consequences | Strongest test of emergence |
| **C1 — noisy cue** | A weak, noisy bodily signal | Closer to biology |
| **C2 — direct** | The true number | **Control only.** Never the basis of a self-model claim |

## The core analysis: what explains the words?

Train a separate classifier for each source, each trying to predict the agent's words from that source alone. Then compare how well each one does.

| If words are best predicted by… | They are about… |
|---|---|
| **S** (current senses) | What's in front of it. Reactions |
| **W** (true world) | Reality, including things it inferred |
| **M** (memory) | Its **internal** picture: what it remembers, believes or expects |
| **B** (hidden body) | **Itself** |

Every classifier is checked against:
- a **shuffled baseline** (words randomly reassigned), which it must clearly beat;
- a **random-network baseline**, the same analysis on an untrained agent, to rule out patterns that appear by accident;
- **held-out episodes** the classifier never saw during training.

These are ordinary scikit-learn classifiers ([07](07-implementation-overview.md)). The design is in the comparison, not in clever models.

## The three questions, tested

### 1. Meaning
- Words predict situations far above the shuffled baseline.
- The relationship holds in new, unseen situations.
- **Playback:** sending a word to another agent changes its behaviour in a way that fits the word's apparent meaning ([05](05-validation-and-experiments.md)).

### 2. Internal sense of the world
Look for words about things **not currently sensed**: food seen earlier, a predator that moved behind a wall, where something is likely to be.

- **Evidence:** M (memory) predicts these words, but S (current senses) can't.

### 3. Internal thinking about themselves
Look for words that track the agent's **hidden body state**, which it was never shown (condition C0/C1).

- **Evidence:** B and M predict these words, beyond what S and W explain.

## The decisive tests: mistakes

Words that always match reality could be reflexes. Words that match a **wrong belief** can only come from inside. So we deliberately create moments where the agent's belief and the world come apart:

| Test | What we do | Words follow the world | Words follow the belief |
|---|---|---|---|
| **Removed object** | Agent sees food; it's secretly removed | Stops naming it (only possible if it can sense the removal) | **Keeps naming it**, so it's talking about its inner picture |
| **Illusion** | Show a phantom food item that isn't real | Doesn't name it | **Names it**, so it's talking about its percept |
| **Belief injection** | Nudge its memory into the pattern it has when starving, while it's actually fed | Words say "fine" | **Words say "starving"**, so its words express its internal state |
| **Body mismatch** (C1) | Make the noisy body cue mislead it | Words track true energy | **Words track what it believes** about its energy |

Each test is logged as an **intervention run**, kept separate from natural data ([05](05-validation-and-experiments.md)).

The strongest single result in com_sim is words that follow the agent's belief **when that belief is wrong**.

## Environment requirements

Agents only talk about things it pays to talk about. The environment must make these words useful:

- **Out-of-view information:** an agent sometimes loses sight of food it already spotted, so remembered information is worth sharing.
- **Prediction:** food or threats that move, so expectations about where things will be are useful.
- **Self-state matters to others:** food sharing where the group survives best if the hungriest agent eats. Nobody can see anyone else's energy, so it pays to signal your own.

## Levels of evidence

Reports must state which level was reached for each question.

| Level | Claim | Evidence |
|---|---|---|
| 1 | Words carry information | Classifier beats shuffled and random-network baselines |
| 2 | Words have meaning | Plus generalisation and playback |
| 3 | Words express an internal world/self model | Plus M or B explaining words beyond S and W |
| 4 | Words express **belief**, not just the world | Plus words follow the belief in the mistake tests |

## Talking with them (later stage)

Once meanings are validated, researchers can join the conversation, using only the agents' own words:

1. **Listen:** decode what agents say about the world and themselves.
2. **Speak:** send validated words and check agents respond as they would to a peer.
3. **Ask:** if agents developed a way to ask each other about state, use that same word to ask them.
4. **Translate:** keep a researcher-side mapping from agent words to English descriptions. Agents never see it.

Rules: we learn their language, and we never train them on ours. Every researcher-sent word is logged as an intervention.

## Honest limits

- **Correlation isn't meaning.** Classification identifies candidates. Playback and the mistake tests turn them into claims.
- **We're not neutral.** We design the body, rewards and environment, and we choose what to record. Contamination by human ideas is reduced, not eliminated. Design choices should be documented, and unsupervised discovery preferred where possible.
- **Words are shaped by reward.** Agents say what's *useful*, which may not always be what's true (e.g. exaggerating hunger to be fed first). The mistake tests and the body ground truth let us detect this.
- **Small agents aren't LLMs.** Results show what can emerge in clean conditions; they don't directly tell us what an LLM has. The value for LLM research is as a test bench where ground truth is known.
- **Pre-register.** Before each experiment, write down predictions and what would count as a null result. No meaning, no inner model, or an inner model that's never voiced are all valid findings.
