# 08 — Prior Work and Novelty

com_sim builds on an established research field called **emergent communication**: agents that develop their own signalling systems through learning. This doc records what has already been done, what we borrow, and where com_sim aims to go further.

> **Status:** an early survey, not a full literature review. Core papers were checked against arXiv/ACL listings in October 2026. Before claiming novelty publicly, run a deeper search (especially on point 3 under "Where com_sim differs").

## What already exists

### Foundations

| Work | What it established |
|---|---|
| David Lewis, *Convention* (1969) | The **signalling game**: a sender and receiver can arrive at meaningful conventions with no prior agreement. |
| Brian Skyrms, *Signals* (2010) | Simple learning and evolutionary dynamics reliably produce signalling conventions in Lewis games. |
| Luc Steels, Talking Heads experiments (late 1990s) | Physical robots developed shared vocabularies for things they saw through repeated language games. |

### Neural agents inventing signals

| Work | What it showed |
|---|---|
| Foerster, Assael, de Freitas & Whiteson (2016), *Learning to Communicate with Deep Multi-Agent Reinforcement Learning* (NeurIPS 2016) | Deep RL agents can learn communication protocols to solve cooperative tasks. It introduced two methods: **RIAL** (messages learned by ordinary RL, like our REINFORCE approach) and **DIAL** (training signals passed back through the communication channel during training). |
| Lazaridou, Peysakhovich & Baroni (2017), *Multi-Agent Cooperation and the Emergence of (Natural) Language* (ICLR 2017) | In an image referential game (the sender knows which of two images is the target and sends a symbol from a fixed vocabulary, then the receiver picks), simple networks learn to coordinate. This is essentially our Stage 0 with images. |
| Mordatch & Abbeel (2018), *Emergence of Grounded Compositional Language in Multi-Agent Populations* (AAAI 2018) | Agents in a 2D world developed a discrete symbol language with a basic vocabulary and syntax to reach shared goals. When speech was unavailable, they also used non-verbal signalling (pointing, guiding). |
| Kajić et al. (2020), *Learning to cooperate: Emergent communication in multi-agent navigation* | In a grid world, signals emerged meaning things like "left", "up", "upper-left room", clustering space with minimal built-in bias. Closest prior work to our Stage 1 grid world. |
| Lazaridou & Baroni (2020), *Emergent Multi-Agent Communication in the Deep Learning Era* | A survey of the field. Read this first. |

### Interpreting and measuring emergent signals (closest to our System B)

| Work | Relevance to com_sim |
|---|---|
| Andreas, Dragan & Klein (2017), *Translating Neuralese* (ACL 2017) | Translated agent messages into English with no parallel data, treating a message and a phrase as equivalent if they produce the same belief in a listener. Closest prior work to our "decode from context" idea. Note that it maps onto **human** language, which we deliberately avoid early on. |
| Lowe et al. (2019), *On the Pitfalls of Measuring Emergent Communication* | Defined **positive signalling** (messages depend on the sender's situation) and **positive listening** (messages change the receiver's behaviour), and showed common metrics can mislead. It also proposed measuring the **causal influence** of messages. These map directly onto our Experiments A/B and signal-playback interventions. |
| Bouchacourt & Baroni (2018), *How agents see things* (EMNLP 2018) | Agents communicated successfully, but their shared representations did **not** capture the object properties humans would assume. This is evidence that agent concepts can diverge from human ones, which supports our "concepts we don't have" goal. |
| Bouchacourt & Baroni (2019), *Miss Tools and Mr Fruit* (ACL 2019) | Agents developed separate "idiolects" (personal dialects). Interpretation can't assume one shared language across a population. |

### Tooling and real-world parallels

- **EGG** (Meta AI, Kharitonov et al. 2019) is an open-source PyTorch toolkit for emergent-communication games. It's useful as a reference implementation for Stage 0.
- **Project CETI** is applying the same decode-from-context approach to sperm whale vocalisations: a real-world version of System B.
- **Field direction:** recent work (e.g. the EMNLP 2022 tutorial by Baroni, Dessì & Lazaridou) is shifting toward letting **large language models** communicate with each other. Small agents with no human language, as in com_sim, are becoming *less* common, not more.

## What we borrow

| From | We use |
|---|---|
| Lewis/Skyrms, Lazaridou et al. | Stage 0 signalling game as a known-working baseline ([07](07-implementation-overview.md)) |
| Kajić et al., Mordatch & Abbeel | Grid-world design with information asymmetry |
| Lowe et al. | Positive signalling/listening and causal influence as standard metrics; channel ablation as a sanity check |
| Bouchacourt & Baroni | Probing internal representations, not just symbol use; expecting per-agent dialects |
| Andreas et al. | The principle that meaning is defined by effect on a listener's beliefs and behaviour |
| EGG | Reference code to check our implementation against |

## Where com_sim differs

The basic mechanism (agents invent signals from reward) is well covered. Our contribution is in how the setting is designed and what we ask of it. "Less explored" below means we didn't find much in an early search, not that it has never been done.

### 1. Survival ecology instead of a referential quiz

Most emergent-communication work uses **referential games**: a fixed set of objects, one right answer per round, success measured as accuracy. com_sim agents have **needs** (hunger, energy, damage), live and die, face threats, and are selected across generations. Communication has to earn its place in a messy life, not a tidy quiz. That is closer to how animal communication arose, and it gives agents things to talk about besides external objects.

### 2. Decipherment without an answer key

In referential games, researchers know the full set of possible referents and simply check messages against them. com_sim treats the agents like an **unknown species**:
- Record raw experience, not a list of candidate meanings.
- Discover categories **unsupervised** first, before any human label is applied.
- Only then validate them with interventions.

Andreas et al. come closest, but they translate into human language. We hold off on human language until meanings are independently validated ([05](05-validation-and-experiments.md)).

### 3. Words that express an inner model of the world and self (the core of com_sim)

Prior work overwhelmingly studies signals about the **outside world**: objects, locations, directions. An early search found almost no learning experiments where agents come to signal their **own internal states** (depletion, damage, uncertainty). The closest work we found is in robotics, where artificial "interoception" is proposed as a model rather than tested as something agents learn on their own.

com_sim goes further. Body variables are **hidden from the network**, so any self-representation has to emerge. We classify the agents' words against their senses, the true world, their memory and their body. Then we deliberately make belief and world disagree, to test whether words express what the agent *believes* ([09](09-meaning-and-inner-model.md)). This needs things that are rarely put together:
- simulated internal needs that matter for survival,
- recorded hidden states (what the agent represents), and
- recorded messages (what it says), plus
- full ground truth of the world, including what the agent can't see.

The motivation is the LLM "gaming problem": LLM self-reports can't be trusted because LLMs are trained on human writing about minds. These agents never see human language, and we have the ground truth LLM research lacks.

### 4. Deliberately small, human-language-free agents

As the field moves toward LLM-based agents, com_sim stays with small networks that have never seen human language. That keeps any concepts that emerge uncontaminated by human categories, which an LLM-based setup can't guarantee.

## Honest assessment

- Stages 0–1 ([06](06-roadmap.md)) **reproduce** known results. That's intentional: they validate the pipeline.
- The novel ground starts with hidden bodies: does a self-model emerge, is it communicated, and can we talk to the agents about it using their own signals?
- No single component is new on its own. The **combination**, together with the strict separation between the agents and the analysis, is what makes com_sim distinctive.

## Sources

- Foerster et al. (2016), *Learning to Communicate with Deep Multi-Agent RL*: https://arxiv.org/abs/1605.06676 · https://proceedings.neurips.cc/paper/2016/hash/c7635bfd99248a2cdef8249ef7bfbef4-Abstract.html
- Lazaridou, Peysakhovich & Baroni (2017), *Multi-Agent Cooperation and the Emergence of (Natural) Language*: https://arxiv.org/abs/1612.07182
- Mordatch & Abbeel (2018), *Emergence of Grounded Compositional Language in Multi-Agent Populations*: https://arxiv.org/abs/1703.04908 · https://ojs.aaai.org/index.php/AAAI/article/view/11492
- Andreas, Dragan & Klein (2017), *Translating Neuralese*: https://arxiv.org/abs/1704.06960 · https://aclanthology.org/P17-1022/
- Lowe et al. (2019), *On the Pitfalls of Measuring Emergent Communication*: https://arxiv.org/abs/1903.05168
- Bouchacourt & Baroni (2018), *How agents see things*: https://arxiv.org/abs/1808.10696
- Bouchacourt & Baroni (2019), *Miss Tools and Mr Fruit*: https://aclanthology.org/P19-1380/
- Lazaridou & Baroni (2020), *Emergent Multi-Agent Communication in the Deep Learning Era*: https://arxiv.org/abs/2006.02419
- Kajić et al. (2020), *Learning to cooperate: Emergent communication in multi-agent navigation*: https://arxiv.org/abs/2004.01097
- Review of applications of deep-learning emergent communication (2024): https://arxiv.org/abs/2407.03302
- Baroni, Dessì & Lazaridou, EMNLP 2022 tutorial: https://aclanthology.org/2022.emnlp-tutorials.3/
- Not yet re-checked (cited from general knowledge): Lewis (1969), Skyrms (2010), Steels' Talking Heads, EGG (Kharitonov et al. 2019), Project CETI.
