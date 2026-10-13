# 07 — Implementation Overview

This doc explains how the ideas in [01](01-vision-and-goals.md)–[06](06-roadmap.md) would actually be built. It assumes no prior machine-learning experience: terms are explained the first time they appear.

The plants do not need to be followed verbatim, this is more to give an overview of how things could look.

There are three parts, which match the two systems from the policy docs plus the data that sits between them:

```
 ┌──────────────────────────┐     writes     ┌──────────────────┐     reads     ┌──────────────────────────┐
 │  1. SIMULATION (System A) │ ─────────────► │  2. DATASET      │ ────────────► │  3. ANALYSIS (System B)   │
 │  world + learning agents  │                │  log files       │               │  stats + classifiers      │
 └──────────────────────────┘                └──────────────────┘               └──────────────────────────┘
            ▲                                                                                │
            └──────────── never (except logged interventions, see 05) ───────────────────────┘
```

## Suggested tech stack

| Tool | What it's for |
|---|---|
| **Python 3.11+** | Main language; almost all ML research uses it |
| **PyTorch** | Builds the agents' neural networks and trains them |
| **NumPy** | Fast number arrays; the simulated world's grid lives here |
| **pandas + pyarrow** | Tables of recorded data, saved as Parquet files (compact, fast tables on disk) |
| **scikit-learn** | Ready-made classifiers and clustering for the analysis |
| **matplotlib** (+ optionally **UMAP**) | Charts and 2D maps of high-dimensional data |
| **Jupyter notebooks** | Interactive exploration of results |

Everything runs on a normal laptop at first. A GPU only matters once the environments get large.

## Suggested project layout

```
com_sim/
  sim/
    env.py          # the world: grid, food, predators, rules
    agents.py       # the neural networks
    train.py        # the reinforcement-learning loop
  recording/
    recorder.py     # writes experience + communication logs
  analysis/
    notebooks/      # exploration, charts, classifiers
  data/
    runs/<run_id>/  # one folder per training run (config, logs, model checkpoints)
```

---

## Part 1 — The reinforcement-learning simulation

### What reinforcement learning (RL) is

An agent repeatedly:
1. **observes** its situation,
2. **chooses** an action (and here, also a signal to send),
3. receives a **reward** (a number: positive is good, negative is bad),
4. **adjusts** itself so that choices which led to more reward become more likely.

No one tells the agent the right answer. It learns only from rewards. That's exactly why it fits this project: nobody tells the agents what their signals mean.

### What an agent is, concretely

An agent is a small neural network. A neural network is a function with many adjustable numbers (called *weights*) that turns an input list of numbers into an output list of numbers. Training means nudging the weights.

```
inputs (numbers)                       network                        outputs (probabilities)
─────────────────                      ───────                        ───────────────────────
what it sees nearby   ─┐                                          ┌─► action: up/down/left/right/stay/eat
bodily cues*         ─┼─►  [ layers of weights + a memory ]  ────┤
signal it just heard  ─┘                                          └─► signal: one of K symbols (0..K-1)
```

- **\*Not the energy number itself.** The body's energy is hidden from the network; at most it gets indirect or noisy cues, depending on the interoception condition. Whether the network works out its own state anyway is the core research question ([09](09-meaning-and-inner-model.md)).
- **Signals are just integers** (e.g. symbol 0 to 7). To the network, symbol 3 is just "input slot 3 is switched on". It carries no built-in meaning, which is the point.
- **Memory:** a recurrent layer (a *GRU*) lets the agent keep an internal state between steps. That internal state is a list of numbers, also recorded for analysis (see Part 2).
- The network outputs **probabilities**, and the agent samples from them. Early on it's close to random (the "random signalling" phase in [03](03-communication-and-recording.md)). Learning sharpens the probabilities.

### Stage 0: the signalling game (start here)

Before building a world, start with the simplest possible communication problem, known in research as a **Lewis signalling game**:

- A **speaker** sees one of 5 hidden objects.
- It sends one of 5 symbols.
- A **listener** sees only the symbol and guesses the object.
- Both get reward 1 if the guess is right and 0 if it's wrong.

At first symbols are random. After a few thousand rounds they settle into a shared convention, such as "symbol 2 ↔ object 4", that nobody designed. The whole thing is about 30 lines of PyTorch and trains in seconds:

```python
import torch, torch.nn as nn, torch.nn.functional as F

N_OBJECTS, VOCAB = 5, 5
speaker  = nn.Sequential(nn.Linear(N_OBJECTS, 32), nn.ReLU(), nn.Linear(32, VOCAB))
listener = nn.Sequential(nn.Linear(VOCAB, 32), nn.ReLU(), nn.Linear(32, N_OBJECTS))
opt = torch.optim.Adam([*speaker.parameters(), *listener.parameters()], lr=1e-2)
baseline = 0.0  # running average reward, used to judge "better or worse than usual"

for step in range(5000):
    obj = torch.randint(N_OBJECTS, (64,))                     # 64 games at once
    speak = torch.distributions.Categorical(logits=speaker(F.one_hot(obj, N_OBJECTS).float()))
    msg = speak.sample()                                      # speaker picks a symbol
    guess_dist = torch.distributions.Categorical(logits=listener(F.one_hot(msg, VOCAB).float()))
    guess = guess_dist.sample()                               # listener guesses
    reward = (guess == obj).float()

    # REINFORCE: make choices that beat the average more likely, others less likely
    advantage = reward - baseline
    loss = -(advantage * (speak.log_prob(msg) + guess_dist.log_prob(guess))).mean()
    opt.zero_grad(); loss.backward(); opt.step()
    baseline = 0.9 * baseline + 0.1 * reward.mean().item()

    if step % 500 == 0:
        print(step, "success rate:", reward.mean().item())
```

The learning rule here is **REINFORCE**, the simplest RL algorithm: "whatever you just did, do it more if the outcome beat the average, and less if it didn't." Later stages can switch to **PPO**, a more stable variant built on the same idea.

Stage 0 deliberately breaks the "all agents equal" rule: it has a fixed speaker and listener because it's a **test of the tools**, not the experiment. A variant where both agents take turns at each role is a good next step. This stage is worth doing first because it already contains the whole pipeline in miniature. Agents develop meaningless symbols into a convention, you can log (object, symbol) pairs, and you can check whether a classifier recovers the mapping.

### Stage 1: a grid world

Next, put a few **equal** agents in a small world where communication could help survival:

```
  . . . . . . . . . . . .      a, b, c = identical agents
  . . . . . . F . . . . .      F = food
  . . a . . . . . . . . .
  . . . . . . . . . . . .      every agent: sees 2 cells around it,
  . . . . . . . . . c . .                   hears sounds within 6 cells,
  . . . . b . . . . . . .                   can move, eat, and make a sound
```

Each **step** (one tick of simulated time):
1. Each agent gets its observation: what it can see and the signal it received last step. (In Stage 1 it may see its energy directly to keep things simple. Hiding it comes in the self-model stages.)
2. Each network outputs an action and either a sound or silence. Early on these are random: the agents are babbling.
3. The world updates: agents move, food is eaten, energy drops by 1.
4. Rewards are handed out: each agent is rewarded for being alive, plus a share for each living family member. Eating only matters because it keeps you alive. See [10](10-how-language-emerges.md).

An **episode** is one life: it runs until death or a step limit (e.g. 200 steps), then the world resets with food in new places. Training runs thousands of episodes.

**The key design rule is information asymmetry, from situation rather than role.** All agents are equal, but they're in different places. Whoever stumbles on food knows something the others don't, and because hearing reaches further than sight, a sound can share it. If everyone could see everything, sounds would be useless and no language would emerge.

**Expect it to be slow.** Starting from pure babbling, it can take a very large number of episodes before any sound means anything. Watch the channel-ablation check (below): the moment scrambling sounds starts to hurt survival is the moment language has begun.

**Who benefits: families.** Agents live in family groups and are rewarded when family members survive (weight λ). Helping family pays, so honest signalling can emerge among kin. Strangers get nothing, which leaves room for deception between families. Setting λ to 0 (pure individual reward) or 1 (family counts fully) are comparison conditions. This is a simplified version of the full design; the complete world, sound, learning and generations design is in [10](10-how-language-emerges.md).

### Later stages

- **Predators.** Introduce danger, so signals may come to mean "flee" as well as "food".
- **Longer messages.** Instead of one symbol, send a sequence of 3 symbols from a vocabulary of 4, which opens the door to combinations. These are the P1/P2/P3 populations in [03](03-communication-and-recording.md).
- **Populations and evolution.** Run many agents, and every N episodes copy the best-performing networks (with small random changes to their weights) to replace the worst. This adds the evolutionary layer from [02](02-agent-design.md).

### Sanity check: is communication actually being used?

Run the trained agents twice: once normally, and once with every message replaced by a random symbol (**channel ablation**). If reward drops sharply, the agents rely on their signals. If it doesn't, they've learned to survive without talking, and the environment needs more pressure to communicate.

---

## Part 2 — Dataset collection

While the trained agents run (and optionally during training too), a **recorder** writes one row per agent per step into two separate tables, as required by [03](03-communication-and-recording.md).

### `experience.parquet` — what the agent was going through

| run_id | episode | step | agent | x | y | energy | food_visible | food_dx | food_dy | predator_dist | action | reward |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| r01 | 412 | 37 | 0 | 2 | 2 | 61 | 1 | +4 | −1 | 9 | stay | 0 |
| r01 | 412 | 37 | 1 | 8 | 4 | 58 | 0 | +2 | −3 | 7 | left | 0 |

### `communication.parquet` — what was said

| run_id | episode | step | sender | heard_by | sound |
|---|---|---|---|---|---|
| r01 | 412 | 37 | 0 | `[1, 2]` | 3 |
| r01 | 412 | 37 | 2 | `[0]` | silent |

### `hidden_states.npy` — the agent's internal state

The GRU memory vector (e.g. 64 numbers) for each agent at each step, stored as a separate array keyed by the same (run, episode, step, agent). This is the closest thing to "what the agent is thinking", and it's what lets you later ask whether agents represent their own state.

### Notes

- **Keys join everything.** `(run_id, episode, step, agent)` links all three files, so the analysis can line up "what was happening" with "what was said" at any moment.
- **Researcher-only fields.** Columns like `food_dx` (true direction to the nearest food) are ground truth that the agent usually can't see. They exist so we can test what signals refer to. They must never be fed to the agents.
- **Scale.** 1,000 episodes × 200 steps × 2 agents is 400,000 rows, a few megabytes. Size is not a concern early on.
- **Reproducibility.** Each run folder also stores its config (world size, vocabulary size, random seed) and model checkpoints, so any result can be regenerated.

---

## Part 3 — Machine-learning interpretation

This is System B from [04](04-analysis-and-interpretation.md). It works from the saved files, never from the live agents.

### Step 1 — Look at the data (no ML yet)

- **Symbol frequency.** Which symbols are used, and does usage settle down over training?
- **Co-occurrence heatmap.** A table with symbols as rows, situations as columns (e.g. food direction: N/S/E/W), and counts in each cell. A strong diagonal-like pattern is the first sign that symbols track something.
- **Mutual information.** A single number for "how much does knowing the symbol reduce uncertainty about the situation?" Zero means unrelated; higher means more informative.

### Step 2 — Experiments A and B (classifiers)

A **classifier** is a model that learns to predict a category from inputs, by being shown many examples. Start with simple, interpretable ones from scikit-learn (logistic regression, random forest).

| | Inputs | Predicts | Example question |
|---|---|---|---|
| **Experiment A** | the sounds an agent made | that agent's experience | "From symbol alone, can we tell where the food was?" |
| **Experiment B** | an agent's experience | the sound it will make | "From the situation, can we predict what it'll say?" |

How to tell whether a result means anything:

- **Train/test split by episode.** Fit the classifier on some episodes and score it on episodes it has never seen. Splitting individual rows randomly would leak near-identical neighbouring steps into the test set and inflate scores.
- **Compare against a shuffled baseline.** Re-run with symbols randomly shuffled between rows. The real classifier must clearly beat the shuffled one (e.g. 85% vs 26%). Otherwise the signal carries no information.
- **Inspect which inputs matter.** Random forests report *feature importance*. If `food_dx` dominates the prediction of symbol 3, that's a candidate meaning, "food to the east", which still needs validating ([05](05-validation-and-experiments.md)).

### Step 3 — Discover categories without naming them (unsupervised)

Here no labels are given at all:

- Take every moment a given symbol was sent, and the full experience vector at that moment.
- Use **UMAP** (a technique that squashes many-number vectors into 2D points so similar ones land close together) and plot them, coloured by symbol.
- Run **clustering** (e.g. k-means), which groups similar points automatically.

If symbol 5 forms one tight cluster spread across situations a human would call different (a predator near, being cornered, food vanishing), that's a candidate agent concept with no single English word, the most interesting kind of finding in [04](04-analysis-and-interpretation.md).

### Step 4 — Probe the internal state

Train a simple classifier (a **linear probe**) to read facts out of the saved hidden-state vectors: "is energy low?", "is food nearby?". If a fact is easy to read out, the agent represents it internally.

If the agent was **never given** its energy but a probe can read it from its memory, the network worked it out itself. Always compare against the same probe on a random, untrained network.

The core analysis of the project ([09](09-meaning-and-inner-model.md)) combines this with Steps 2–3: train one classifier per source (senses, true world, memory, body) to predict the agent's words, and see which explains them best. Words explained by memory but not senses mean the agent is talking from an inner picture.

### Step 5 — Validate with interventions

Correlation isn't enough ([05](05-validation-and-experiments.md)). Load the trained agents and run controlled tests:

- **Signal playback.** Play sound 3 from a fixed spot regardless of where the food is, and record where nearby agents go. If they head toward it every time, sound 3 *causes* food-seeking.
- **Novel situations.** Place food in configurations never seen in training, or add a harmless predator lookalike, and check whether the symbols generalise.

These runs are logged as intervention runs, kept separate from normal data.

---

## What a first milestone looks like

1. Stage 0 signalling game trains to roughly 100% success.
2. The recorder saves (object, symbol) pairs.
3. A co-occurrence heatmap shows a clean one-to-one mapping.
4. Experiment A's classifier recovers the object from the symbol far above the shuffled baseline.

That's the full pipeline (simulate → record → interpret) working end to end on the smallest possible problem. Every later stage reuses the same pipeline with a richer world.

## Common pitfalls

| Pitfall | Symptom | Fix |
|---|---|---|
| No reason to talk | Channel ablation doesn't hurt reward | Increase information asymmetry |
| Nobody listens yet | Sounds are informative but nobody reacts to them | Normal early on (the chicken-and-egg stage). Train longer; make sure hearing reaches further than sight |
| Language never starts | After a very long run, scrambling sounds still doesn't hurt survival | Make food scarcer or harder to find alone; add more agents; lower the cost of making sounds |
| Collapse to one symbol | Every message is the same | Add a small *entropy bonus* (reward for keeping choices varied early on) |
| Leaky evaluation | Classifier scores look too good | Split train/test by episode or run, never by row |
| Human labels sneak in | Agents receive researcher-only fields | Keep ground-truth columns out of observation code; review `env.py` against [02](02-agent-design.md) |
