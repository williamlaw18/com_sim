# 01 — Vision and Goals

## The question

> **Do agents develop words with meaning — and do those words express an internal model of the world and of themselves, rather than just what's in front of them?**

Agents live in a simulated world. They can only exchange signals that start out meaningless, and they have bodies (energy, damage) they can't directly see. We record what they sense, what's really in the world, what's in their memory, and what their bodies are doing. Then we classify their words against each of those to find out what the words are about.

The detailed research design is in [09](09-meaning-and-inner-model.md).

### In plain terms

A group of identical robots wander a world looking for cookies. Each can only see a little way, but they can hear beeps from further off. At first they beep at random: just noise. Over a long time, a robot that happens to beep near a cookie brings friends over, everyone eats, and that beep starts to stick. Eventually beeps mean things. We secretly record everything, then play detective to work out what each beep means. The trick question: if a robot sees a cookie, then we secretly take it away, does it still beep "cookie"? If so, it's talking about the cookie in its head. That means it has an inner picture of the world.

## Why do this

We'd like to know whether a neural network can develop meaning, an internal sense of the world, and internal thinking about itself.

**LLMs can't answer this cleanly.** They've been trained on vast amounts of human writing about minds, thoughts and consciousness. When an LLM describes an inner life, we can't tell whether it's reporting or reciting what humans wrote. Philosophers call this the "gaming problem".

**com_sim agents have never encountered human language.** Anything they come to say about the world or themselves, they developed on their own. And unlike with an LLM, we know the ground truth: what the world contains, what the agent could sense, what's in its memory, and what its body is doing.

## Goals

1. **Emergent words.** Agents develop a signalling system from survival pressure alone. ([02](02-agent-design.md), [03](03-communication-and-recording.md))
2. **Meaning.** Show their words consistently refer to things, and validate this with playback. ([04](04-analysis-and-interpretation.md), [05](05-validation-and-experiments.md))
3. **Internal sense of the world.** Show their words carry an inner picture of the world, including things out of view. ([09](09-meaning-and-inner-model.md))
4. **Internal thinking about themselves.** Show their words track their own hidden state. ([09](09-meaning-and-inner-model.md))
5. **Belief, not just reality.** Show their words follow what they believe, even when that belief is wrong. ([09](09-meaning-and-inner-model.md))
6. **Dialogue (later).** Communicate with agents using their own validated words.

## Non-goals

- **Proving consciousness.** com_sim tests meaning, world models and self-models through what agents say. Whether they subjectively experience anything is a separate question these methods can't settle on their own.
- **Building a chatbot.** The agents are not language models and are not trained on human language.
- **Teaching agents our language.** We learn theirs. Training them on human-chosen meanings would contaminate the result.
- **Guaranteeing a result.** No meaning, no inner model, or an inner model that's never voiced are all valid findings.

## Known weaknesses (and how we handle them)

| Weakness | Handling |
|---|---|
| Small agents aren't LLMs, so results don't directly tell us what LLMs have | Frame com_sim as showing what *can* emerge cleanly, and as a test bench where ground truth is known |
| We design the body, rewards and environment, so human ideas still leak in | Claim *reduced* contamination, not none; document design choices; prefer unsupervised discovery |
| Tracking a simple state like energy is a low bar | Require the higher evidence levels in [09](09-meaning-and-inner-model.md), especially words that follow a *wrong* belief |
| Agents say what's useful, not necessarily what's true | Compare words against body and world ground truth to detect it |
| Null results are hard to interpret | Pre-register predictions and null criteria before each experiment |

## Core principles

| Principle | Meaning |
|---|---|
| Words are the instrument | Every core question is answered by analysing what the agents say. See [09](09-meaning-and-inner-model.md). |
| Body is designed, mind must emerge | Body variables are never fed directly to the network (except the C2 control). |
| No human language in the loop | Agents never see human language or LLM output. See [02](02-agent-design.md). |
| Meaning is inferred, not assigned | We never label signals for the agents; we observe and test. |
| Two systems, strictly separated | System A (agents) and System B (analysis) never feed back into each other, except through logged interventions. See [04](04-analysis-and-interpretation.md). |
| Correlation is a hypothesis | Every interpretation is provisional until validated. See [05](05-validation-and-experiments.md). |
| Start small, scale deliberately | Begin with a small learnable channel; add expressiveness as a controlled variable. See [03](03-communication-and-recording.md). |
