# SAR Research Project

**Stateless Adversarial Review** — Heterogeneous Multi-Model Supervision for Agentic AI Systems.

Idea author: José María Fernández Gómez (Nomad Prompters).
Technical development: in iterative collaboration with LLMs (Claude, Gemini, GPT, DeepSeek).

## Current status

- Paper v0.4 drafted (May 2026), with a visible erratum on the header: §5.3 contained a mathematically incorrect claim about sliding-window overlap coverage. Corrected in place; document kept as a working paper.
- v0.5 pending. Open blockers documented in `research/open-questions.md` (T0 SPOF, M2S/Yang vs statelessness, overlap derivation from threat model, cost table).
- **Not submitted to any venue.** Deliberate decision: the paper will not be published until the human author can technically defend every claim. Training plan in `learning/00-plan.md`.

## Repo structure

- `paper/` — Paper versions.
- `research/` — Technical work from the model ensemble: reference verification, open questions, experiment design.
- `learning/` — Human author training. Three-phase plan toward being able to defend the paper.

## Next step

Begin Phase 1 of the training plan: understand the problem before defending the solution.
See `learning/00-plan.md`.

---

*Spanish version: see [README.es.md](README.es.md).*
