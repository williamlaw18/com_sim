# 10 — How Language Emerges: Conditions, Not Outcomes

This doc turns a model of how human language may have emerged into a concrete simulation design. The guiding rule:

> **We build the conditions under which language *could* emerge. We never build the language, and never reward it directly.**

Every mechanism below is either **physics** (how the world works), **body** (what agents can do), or **learning** (how agents change). None of them say what any sound means, or reward any particular sound.

---

## Part 1 — The emergence model

How language may plausibly have developed in humans (drawn from language-evolution research; competing theories, not settled fact), and the condition each stage needed:

| Stage | What happens | Condition it needs |
|---|---|---|
| **1. Cues** | Bodies make incidental noise (crunching, panting). Listeners learn to read it; makers learn because listeners react. A cue becomes a signal. | Some actions are audible |
| **2. "Attention: here"** | A noise means "something worth coming to is where I am". Meaning comes from the sound's *location*, not the sound itself. | Listeners can tell **where** a sound came from; some things are worth coming to |
| **3. Different calls** | Different sounds for situations needing different responses (food → approach, predator → flee). | Situations demanding **opposite responses** |
| **4. Conventions** | A group converges on the same sound for the same thing, and passes it on. | **Generations**, and learning from what elders do |
| **5. Combinations** | Sounds combine into sequences whose parts carry meaning. | **More things to say than sounds**, and a **learning bottleneck** between generations |
| **6. Displacement** | Talking about what isn't here: food that *was* there, a threat that went out of sight. | Information that **leaves view but still matters** |

Running through all of it is **trust**. Signals only work if listeners can trust them, which is most likely among family, where helping others also helps you.

---

## Part 2 — The design

### The world (physics)

| Feature | Design | Why (stage) |
|---|---|---|
| **Grid** | ~32×32 cells, optional walls that block sight but not sound | Room for a population; walls create out-of-view information (6) |
| **Small food** | Scattered, scarce, respawns slowly. One agent can eat it alone | Basic survival |
| **Large food** | Rarer. Needs **2+ agents** adjacent to eat; feeds them all | A reason to bring others over: "come here" pays off (2) |
| **Predators** | Wander and chase the nearest agent. Agents escape by fleeing or reaching shelter cells | Needs the *opposite* response to food (3) |
| **Predators hear sounds** | Predators are attracted to calls in their hearing range | Calling has a natural risk, so silence has value, without us deciding when to call |
| **Movement** | Food and predators sometimes move or get blocked by walls | Things leave view but still matter (6) |

### Sound (physics)

| Feature | Design | Why |
|---|---|---|
| **Inventory** | K abstract symbols (start K=8) plus silence. Each step: at most one sound | Small and fixed, so to express more, agents must combine (5) |
| **Volume** | Quiet (short range, cheap) or loud (long range, costly, attracts predators from further away) | "How much to say" is a learned trade-off |
| **Direction and distance** | Listeners perceive which direction a sound came from and how far (by loudness) | **The single most important ingredient.** Makes "attention: here" possible (2) |
| **Voice** | Each agent has a distinctive voice: listeners perceive a fixed, arbitrary per-agent voice feature | Lets agents learn *whom* to trust, without us labelling anyone (trust) |
| **Interference** | When several sounds overlap, a listener hears only the 2 loudest | Constant noise costs the group, not just the caller |
| **Self-hearing** | Agents hear their own sounds | Needed to build and keep track of sequences (5) |
| **Sequences** | No message length. A "sequence" is just sounds on consecutive steps | Combinations can emerge in time rather than being a fixed slot structure (5) |
| **Cost** | Quiet sound: small energy cost. Loud: larger. Silence: free | Content-neutral pressure on whether to speak |

### The body

| Feature | Design | Why |
|---|---|---|
| **Energy** | Drops each step and with movement and sound; food restores it; zero = death. **Hidden from the network** (except C2 control, see [09](09-meaning-and-inner-model.md)) | Survival pressure; self-model test |
| **Damage** | From predator contact; slows movement; heals slowly. Hidden | Self-state worth signalling |
| **Sight** | Small radius (e.g. 2 cells), blocked by walls | Information asymmetry from situation |
| **Hearing** | Larger radius (e.g. quiet = 4, loud = 10), not blocked by walls | Sound carries what sight can't |
| **Give** | An agent can transfer some energy to an adjacent agent | Pressure to communicate about *own* state ("I'm starving") |
| **Lifespan** | Agents age and die of old age even if fed | Forces generations (4) |

### The agent's network

```
 inputs                                   network                     outputs (all learned)
 ──────                                   ───────                     ─────────────────────
 what it sees (small grid patch)  ─┐                              ┌─► move: N/S/E/W/stay
 sounds heard: symbol, direction, ─┤                              ├─► interact: eat / give / none
   distance, voice                 ├─► [encoder] → [GRU memory] ──┼─► sound: silence or one of K symbols
 its own last action and sound    ─┤                              └─► volume: quiet / loud
 bodily cue (C0: none, C1: noisy) ─┘
```

Everything is learned: whether to speak, what, how loud, how often, when to stop, how to react to what's heard. The GRU memory is what lets sequences, expectations and an internal picture of the world form. It's also what we record and probe ([09](09-meaning-and-inner-model.md)).

**Not an input:** reward, true energy (except C2), anything researcher-only.

### Learning (three layers)

**1. Lifetime reinforcement learning.**
Each agent has its **own network** and learns from its own experience with a standard RL algorithm. Start simple, as in [07](07-implementation-overview.md), then move to PPO.

**2. Social learning: a tendency to copy, not a lesson.**
Humans are born vocal imitators; most animals aren't. We give agents the *tendency*, as a switchable condition:
- During **childhood** (the first part of life), when a young agent hears an elder's sound, its own sound output is nudged slightly toward producing that sound *in its own current situation*.
- The nudge fades as the agent grows up.
- A child only learns from what it **happens to hear**: whoever's nearby, whenever they speak. That is the **learning bottleneck**: each generation reconstructs the language from a partial sample, which research suggests favours reusable, systematic parts (5).
- This copies what elders *do*. It never pushes toward a "correct" sound, because we never define one.

**3. Generations.**
- Population is fixed (e.g. 16–24 agents in 3–4 families).
- When an agent dies, a newborn takes its place. The newborn joins a family chosen in proportion to how well each family has been doing (food gathered, members surviving). Successful families grow; unsuccessful ones shrink.
- Newborns get a **fresh network**: they inherit nothing learned. Any language that persists has to be passed on culturally, by copying. (Inheriting starting weights with small mutations, i.e. evolution, is an optional later condition.)

### Reward (who benefits)

```
reward for agent i = (i alive this step) + λ × (number of i's family members alive this step)
```

- **No reward for eating, sounding, or anything else directly.** Food matters only because it keeps you and your family alive.
- **λ (family weight)**, e.g. 0.5, makes helping family pay, so honest signalling can emerge among kin. Strangers get nothing, so deception between families is possible, and that's interesting to watch.
- Families are recognisable by sight (a visible family marker) as a default. Agents still have to *learn* what it means.

---

## Part 3 — Conditions vs. forcing

The checklist that keeps us honest. For each stage: what we provide, and what we must never do.

| Stage | ✅ We provide | ❌ We never |
|---|---|---|
| 1. Cues | (Condition only) eating makes an involuntary crunch | Make cues for anything else by default; reward reacting to cues |
| 2. Attention | Sound direction/distance; large food needing 2+ agents | Reward approaching sounds; reward calling near food |
| 3. Different calls | Food and predators needing opposite responses | Reward distinct sounds; give food and predators their own channels |
| 4. Conventions | Generations, family groups, copying tendency | Provide a target vocabulary; reward agreement between agents |
| 5. Combinations | Small inventory, many things worth saying, learning bottleneck, sequences in time | Fix a message length or grammar; reward sequences |
| 6. Displacement | Walls, moving food and predators, memory | Reward talking about absent things |
| Self-state | Hidden body, give action, family reward | Show the body to the network (except C2); reward sounds tied to hunger |

**General rules**
- All rewards and costs are **content-neutral**: the same for every sound and every situation.
- Every design choice is recorded in the run config, so we can always say what *we* contributed.
- Each condition we add is also run **switched off** at least once, so we can tell whether it mattered.

---

## Part 4 — Experimental switches

| Switch | Default | Comparison | Tests |
|---|---|---|---|
| Accidental crunch (cue-first) | Off | On | Does a cue route speed up stage 1→2? |
| Sound direction | On | Off | Is location really what bootstraps meaning? |
| Copying tendency | On | Off | Are conventions possible without imitation? |
| Family reward λ | 0.5 | 0, 1 | Does trust/kinship matter for honest signals? |
| Predators hear sounds | On | Off | Does risk shape *how much* agents say? |
| Interference | 2 loudest | Unlimited | Does crowding favour turn-taking/brevity? |
| Large food | On | Off | Is "come here" needed to start language? |
| Inventory K | 8 | 4, 16 | Does a small inventory force combinations? |
| Body cue | C0 | C1, C2 (control) | Self-model emergence ([09](09-meaning-and-inner-model.md)) |

---

## Part 5 — How we detect each stage

All measured from recorded data ([03](03-communication-and-recording.md)), in System B ([04](04-analysis-and-interpretation.md)):

| Stage | Signature in the data |
|---|---|
| Start of language | **Channel ablation:** scrambling sounds starts to hurt survival |
| 2. Attention | Agents move toward sounds more than chance; callers near large food more than chance |
| 3. Different calls | Sounds predict situation (food vs predator) above shuffled baseline; listeners respond differently to different sounds |
| 4. Conventions | Different agents use the same sound for the same situation (cross-agent classifier transfer); conventions persist after all original agents have died |
| 5. Combinations | Parts of sequences carry consistent meaning wherever they appear; similar meanings get similar sequences (*topographic similarity*: correlation between how alike two situations are and how alike their sequences are) |
| 6. Displacement | Sounds explained by memory but not current senses (four-source analysis, [09](09-meaning-and-inner-model.md)) |
| Self-state | Sounds explained by the hidden body, and by belief in mistake tests ([09](09-meaning-and-inner-model.md)) |

Track these **over time**. A plot of each signature across training and generations is the "story" of how the language emerged.

---

## Part 6 — Build order

Add one mechanism at a time, and check it works before adding the next. Each step is a runnable milestone.

| Step | Add | Check before moving on |
|---|---|---|
| **A** | Stage 0 signalling game ([07](07-implementation-overview.md)) | Pipeline works: train → record → classify |
| **B** | World: grid, small food, energy, 4 agents, **no sound** | Agents learn to forage. This is the survival baseline |
| **C** | Sound with direction + cost; large food | Channel ablation hurts survival → language has started (stage 2) |
| **D** | Predators (that hear) + volume | Different sounds for food vs predator (stage 3) |
| **E** | Lifespan, generations, families, copying | Conventions survive full population turnover (stage 4) |
| **F** | More things to say: food types, predator types, directions | Signs of combinations (stage 5) |
| **G** | Walls, moving things, hidden body, give action | Core experiments in [09](09-meaning-and-inner-model.md) (stage 6, self-state) |

Recording runs from step C onward. Each step's switches are run on and off where relevant.

---

## Honest risks

- **Slow.** Each step may need very long training runs. Steps C and D alone would be a meaningful result.
- **Compute.** Separate networks per agent are slower to train than one shared network. Keep networks small (e.g. 64-unit memory) and populations modest. Optimise only once something works.
- **Our fingerprints.** Choosing large food, predators, families and copying shapes what *can* emerge, even though it doesn't dictate meanings. The switches table and the run configs exist to make our influence visible and testable.
- **It might not happen.** Agents may survive fine without language, or stall at stage 2. That's a result too, and the switches help explain why.
