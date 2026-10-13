# 02 — Agent Design (System A)

System A is the population of agents that live, learn, and communicate. Everything here must avoid smuggling in human concepts.

## Agent anatomy

Each agent is a learning system (initially a neural network) with:

| Component | Contents |
|---|---|
| **Body** | Simulated variables (energy, damage). Designed by us, **hidden from the network** |
| **Inputs** | Sensory information about the environment + signals received from other agents + (depending on condition) indirect bodily cues |
| **Internal representation** | The network's learned memory. We choose its size, never its meaning. This is where a self-model may or may not emerge |
| **Outputs** | Actions in the environment + signals sent to other agents |
| **Learning mechanism** | Adjusts behaviour from experience and outcomes |

## All agents are equal

Every agent has the same body, the same abilities (see a little, hear further, move, eat, make sounds) and the same kind of network. There are no special roles like "scout" or "gatherer". Each agent has its own separate network weights, so each one learns individually, as separate creatures would.

Differences in what agents know come only from **where they are**. The one that happens to stumble on food knows something the others don't. That situational asymmetry is the reason to communicate.

## Language starts as babbling

At the start, every agent has some chance each step of making a random sound (one of K symbols) or staying silent. These sounds mean nothing. Language is expected to emerge slowly, roughly the way early human language may have:

1. **Noise.** Random sounds, ignored by everyone.
2. **Accidental usefulness.** An agent happens to make a sound near food, another agent happens to move toward it, and both eat. Rewarded behaviour becomes slightly more likely on both sides.
3. **Tied to things.** Over many generations of episodes, particular sounds become reliably linked to particular situations.

Rules:
- **Every sound costs a small amount of energy; silence is free.** This is the only pressure on *whether* to make a sound. Without it, agents may babble every step and fill the channel with noise. The cost is an experimental setting to tune: too low and babbling lasts a long time; too high and agents fall permanently silent before language can start.
- **Rewards and costs must be content-neutral.** A cost for making *any* sound is allowed, because real animals pay energy and risk to call. A reward that depends on *which* sound is made, or *when*, is forbidden, because that would be us teaching meanings.
- Noise is also self-correcting: an agent that always shouts carries no information, so others learn to ignore it, and its shouting becomes pure cost.
- **Optional, sound interference:** if several agents call at once, a listener hears only the nearest. This makes noise costly to the group, not just the caller.
- **Expect this to be slow.** Early on, a sound is useless unless someone listens, and listening is useless unless sounds mean something. This chicken-and-egg problem is well known in emergent-communication research. It's handled with patience, family rewards, populations and the conditions in [10](10-how-language-emerges.md), not by hand-designing meanings.

## Everything is learned

Agents learn every part of their behaviour from survival alone: whether to make a sound, which one, how loud, how often, when to stop, and how to react to what they hear. The full design is in [10](10-how-language-emerges.md).

## Learning layers

1. **Lifetime learning:** each agent has its own network and learns by reinforcement from its own experience.
2. **Social learning:** a *tendency* to copy sounds heard from elders during childhood, in the child's own situation. It copies what elders do, never a "correct" sound. It's a switchable condition.
3. **Generations:** agents age and die; newborns with fresh networks join families in proportion to family success. Language persists only if it's passed on.

Reward is survival-based: being alive, plus a share (λ) for each living family member. Nothing else is rewarded directly. See [10](10-how-language-emerges.md).

## Rules

- **No predefined language.** Agents start with no human-designed communication ability or signal meanings.
- **No LLM in System A.** No LLM is used to drive, train, or interpret agents during development.
- **Signals must have consequences.** Agents must be able to learn a link between sending/receiving a signal and its outcome. Arbitrary signal emission with no feedback loop is not sufficient.
- **No fuel gauge.** Body variables (energy, damage) are never fed directly to the network, except in the C2 control condition. The agent learns about its body only through consequences or noisy cues. See [09](09-meaning-and-inner-model.md).
- **No analysis feedback.** Outputs from System B (classifiers, interpretations) must never be fed back into agents during core experiments.

## Environment design

The environment is the only teacher. To encourage richer communication *without teaching it*, it must present problems where more expressive signals give a genuine survival advantage, e.g.:

- Resources discovered by one agent but not others (food-finding scenario)
- Threats that one agent sees before others (predators, hazards)
- Tasks requiring coordination between agents
- Situations involving things not immediately present (past events, future threats) — needed for abstraction to pay off
- **Information that goes out of view**, e.g. an agent loses sight of food it already spotted, so remembered information is worth sharing (pressure for words about an inner picture of the world)
- **Decisions that depend on your own state**, e.g. risky long foraging trips that only pay off when you have energy to spare (pressure to model yourself)
- **Decisions that depend on others' states**, e.g. food sharing where the group survives best if the hungriest eats (pressure to communicate about yourself)

## Canonical starting scenario

1. A few identical agents wander a grid. Food appears in random places.
2. Each step, every agent may make a random sound or stay silent; nearby agents hear it.
3. Sometimes, by chance, a sound made near food draws another agent over, and both survive longer.
4. Survival is the only reward, so over a long time, sounds that helped become more common and start to carry meaning.
