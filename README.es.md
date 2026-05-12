# SAR Research Project

**Stateless Adversarial Review** — Heterogeneous Multi-Model Supervision for Agentic AI Systems.

Autor de la idea: José María Fernández Gómez (Nomad Prompters).
Desarrollo técnico: en colaboración iterativa con LLMs (Claude, Gemini, GPT, DeepSeek).

## Estado actual

- Paper v0.4 redactado (mayo 2026) con erratum visible en cabecera: §5.3 contenía un claim matemáticamente incorrecto sobre la cobertura del sliding-window overlap. Corregido en sitio; se mantiene el documento como working paper.
- v0.5 pendiente. Bloqueadores documentados en `research/open-questions.md` (T0 SPOF, M2S/Yang vs statelessness, derivación del overlap desde threat model, tabla de costes).
- **No enviado a ningún venue.** Decisión consciente: no se publica hasta que el autor humano pueda defender técnicamente cada afirmación. Plan de formación en `learning/00-plan.md`.

## Estructura del repo

- `paper/` — Versiones del paper.
- `research/` — Trabajo técnico de los modelos: verificación de referencias, preguntas abiertas, diseño de experimentos.
- `learning/` — Formación del autor humano. Plan en 3 fases hasta poder defender el paper.

## Próximo paso

Empezar Fase 1 del plan de formación: entender el problema antes de defender la solución.
Ver `learning/00-plan.md`.
