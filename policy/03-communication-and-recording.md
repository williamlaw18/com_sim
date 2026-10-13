# 03 — Communication Channel and Data Recording

## Communication channel

### Signals

Signals start as meaningless **binary sequences** (e.g. `00101`, `11000`). Any meaning — a location, object, action — emerges through learning and is inferred by researchers from when signals are sent and how receivers respond.

### Sounds, not messages

All agents can make sounds, and all agents can hear. Each step, an agent either stays silent or makes one sound (a symbol from a small set). Sounds are **broadcast**: every agent within hearing range hears them, and hearing range is larger than sight range. Sounds therefore carry information beyond what an agent can see, which is what makes them worth evolving. Agents start out babbling randomly ([02](02-agent-design.md)).

Listeners perceive each sound's **symbol, direction, distance and voice** (a fixed, arbitrary per-agent feature). Sounds can be quiet or loud, overlapping sounds interfere, and predators can hear them too. There's no fixed message length: sequences are just sounds on consecutive steps. Full sound physics is in [10](10-how-language-emerges.md).

### Channel capacity is an experimental variable

- A **fixed, small signal set** constrains what can develop.
- An **unrestricted bit stream** is hard to interpret and computationally expensive.

**Policy:** start with a small, learnable channel; increase expressive capacity deliberately. Run comparison populations:

| Population | Channel |
|---|---|
| P1 | Small, fixed signal set |
| P2 | Can learn combinations of signals |
| P3 | Can modify its communication system more freely |

This reveals whether expressive communication emerges from survival pressure alone or needs particular architectural features.

### Hypothesised progression (not guaranteed)

| Phase | What happens |
|---|---|
| 1. Random signalling | Signals produced with no established system |
| 2. Useful associations | Signals become linked to situations or actions |
| 3. Shared conventions | Agents interpret one another's signals consistently |
| 4. Combinations | Signals combined to convey more complex information |
| 5. Abstraction | Signals refer to things not present (past events, future threats) |
| 6. Complex communication | Increasingly expressive conventions, possibly grammar-like structure |

Stalling at any phase is a valid, reportable outcome.

## Data recording

All data is recorded **as three independent streams**, synchronised by timestamp, so analysis cannot simply learn signal-to-signal mappings. Between them they capture the four sources the core analysis compares the agent's words against: **senses**, **true world**, **memory** and **body** ([09](09-meaning-and-inner-model.md)).

### Stream A — Experience

- **Senses:** exactly what the agent perceived this step (including any illusions we injected)
- **True world:** the full state of the world, including everything out of the agent's view (researcher-only ground truth)
- **Body variables (researcher-only ground truth):** energy, damage, other simulated needs. Recorded every step, but never shown to the agent
- **Bodily cues:** whatever indirect signal the agent *did* receive about its body (depends on interoception condition C0/C1/C2)
- **Actions:** approaching, fleeing, cooperating, sharing
- **Social interactions:** approaches, help, threats, resource sharing

### Stream B — Communication

- Signals sent and received
- Signal sequences
- Timing and frequency
- Sender/receiver identities

### Stream C — Internal representation

- The network's memory vector at every step (mandatory, not optional; it's the main evidence for a self-model)
- Model checkpoints at regular intervals during training, so we can see *when* a self-model forms

### Recording rules

- Record **raw measurements**, not human labels. Labels like "predator" or "fear" may exist in researcher-side ground truth but must never reach agents.
- Every record carries agent ID, timestamp, generation, and population/condition.
- Logs must be sufficient to replay or reconstruct an episode.
- Treat the dataset as the primary research artefact: version it, and never mutate historical runs.
