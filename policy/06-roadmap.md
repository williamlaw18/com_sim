# 06 — Roadmap

Ordered build plan. Add one mechanism at a time; each step should produce recorded data and a short write-up before the next begins. Full detail on steps A–G, including the conditions-vs-forcing checklist and experimental switches, is in [10](10-how-language-emerges.md).

## Build the world and let language emerge

| Step | Add | Check before moving on |
|---|---|---|
| **A** | Stage 0 signalling game ([07](07-implementation-overview.md)) | Pipeline works: train → record → classify |
| **B** | Grid world, small food, energy, a few equal agents, **no sound** | Agents learn to forage (survival baseline) |
| **C** | Sound with direction and distance, sound cost, large food needing 2+ agents. **Recording starts here** ([03](03-communication-and-recording.md)) | Channel ablation hurts survival: language has started |
| **D** | Predators that hear sounds; quiet/loud volume | Different sounds for food vs predator |
| **E** | Lifespan, generations, family groups, copying tendency | Conventions survive full population turnover |
| **F** | More things to say: food types, predator types, directions | Signs of combinations |
| **G** | Walls, moving food/predators, hidden body (C0/C1, C2 control), give action | Ready for the core experiments |

Analysis grows alongside: basic statistics from step C, classifiers and clustering from step D ([04](04-analysis-and-interpretation.md)), and validation with playback and generalisation tests from step D ([05](05-validation-and-experiments.md)).

## Core research (after step G)

1. **Four-source analysis.** Classify words against senses, true world, memory and body; report the evidence level reached ([09](09-meaning-and-inner-model.md)).
2. **Mistake tests.** Removed objects, illusions, belief injection, body mismatch: do words follow belief or world?
3. **Dialogue.** Researchers listen, speak and (if a query word emerged) ask, using agent-invented words.

Pre-register predictions before core steps 1 and 2.

## Definition of done for a step

- Code and config committed on `dev`
- Run data versioned and reproducible; config records every switch setting
- Relevant switches run on and off
- Findings documented, with interpretations explicitly marked as hypotheses or validated
