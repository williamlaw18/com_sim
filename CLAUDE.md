# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project overview

com_sim is a research simulation; no source code yet. **Core question:** do agents develop words with meaning, and do those words express an internal model of the world and of themselves, rather than just what's in front of them? Agents have bodies (energy, damage) they can't directly see, and exchange signals that start out meaningless. The analysis classifies their words against four sources (current senses, true world, memory, hidden body) to find out what the words are about. The decisive tests check whether words follow the agent's belief when that belief is wrong. Motivation: LLM self-reports are tainted by human writing about minds, and these agents never see human language. Full design: [policy/09-meaning-and-inner-model.md](policy/09-meaning-and-inner-model.md).

**Goals:** (1) emergent words from survival pressure; (2) show the words have meaning; (3) show they carry an internal sense of the world (things out of view); (4) show they track the agent's own hidden state; (5) show they follow belief, not just reality; (6) later, communicate with agents using their own validated words.

**Hard rules (do not violate in code or design):**
- Body variables (energy, damage) are never fed directly to the agent's network, except in the explicitly labelled C2 control condition. Any self-model must emerge.
- All agents are equal: same body, same abilities, same kind of network, no special roles. Any difference in what they know comes from where they are, not what they are.
- Language starts as random babbling; any meaning must grow out of noise. Each sound costs a little energy and silence is free. Rewards and costs must be content-neutral: never reward a *particular* sound or a sound in a *particular* situation.
- **Conditions, not outcomes.** We build the world, body, sound physics and learning rules under which language *could* emerge. We never build the language, and never reward it directly. Check new mechanisms against the checklist in policy/10.
- Every recording keeps the four sources separate: senses, true world, memory, body. The core analysis depends on comparing them.
- We learn the agents' language. We never train them on ours. Researcher-sent signals use only validated agent signals and are logged as interventions.
- No human language and no LLM inside the agents (System A).
- Analysis (System B) only reads recorded data. Its outputs never feed back into agents, except through explicitly logged interventions.
- Signals start as meaningless binary sequences. Never hard-code what a signal means.
- Record raw measurements, not human labels.
- Any interpretation is a hypothesis until it is validated.

## Policy docs

Read the relevant doc before designing or changing the related part of the system.

| Doc | Covers |
|---|---|
| [policy/01-vision-and-goals.md](policy/01-vision-and-goals.md) | Vision, goals, non-goals, core principles |
| [policy/02-agent-design.md](policy/02-agent-design.md) | System A: agent anatomy, learning layers, environment design |
| [policy/03-communication-and-recording.md](policy/03-communication-and-recording.md) | Signal channel, capacity comparison populations, data-recording schema and rules |
| [policy/04-analysis-and-interpretation.md](policy/04-analysis-and-interpretation.md) | System B: classifiers (Experiments A/B), clustering, limits on interpretation |
| [policy/05-validation-and-experiments.md](policy/05-validation-and-experiments.md) | Generalisation tests, signal playback, what can and cannot be claimed |
| [policy/06-roadmap.md](policy/06-roadmap.md) | Ordered build stages and definition of done |
| [policy/07-implementation-overview.md](policy/07-implementation-overview.md) | How it's built: tech stack, RL simulation, dataset schema, ML interpretation pipeline |
| [policy/08-prior-work.md](policy/08-prior-work.md) | Existing emergent-communication research, what we borrow, and where com_sim is novel |
| [policy/09-meaning-and-inner-model.md](policy/09-meaning-and-inner-model.md) | **Core research design:** the four-source word analysis, mistake tests (removed objects, illusions, belief injection), evidence levels, talking with agents |
| [policy/10-how-language-emerges.md](policy/10-how-language-emerges.md) | **Simulation design:** emergence stages, world/sound/body/network/learning/reward design, conditions-vs-forcing checklist, experimental switches, build order A–G |

## Working with the user

The user is new to machine learning and reinforcement learning and is exploring this as an idea, not running a funded research programme. When working with them:
- Explain ML/RL terms in plain language the first time they come up, and favour concrete examples, small code sketches and diagrams over jargon.
- Prefer the simplest approach that works (e.g. REINFORCE before PPO, scikit-learn before deep classifiers), and say why when suggesting something more complex.
- Build in small, runnable steps so each piece can be seen working before the next is added.

## Branches

- `main` — stable branch
- `dev` — active development; open PRs from `dev` into `main`

## Commands

_To be added once a build system / language is chosen (build, test, lint, run)._

## Architecture

_To be documented as the codebase takes shape._
