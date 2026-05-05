# Plan de formación — del autor humano

Objetivo: que José María pueda explicar, defender y criticar SAR con sus propias palabras antes de publicar nada.

Regla: **no se publica hasta que se complete Fase 3.**

---

## Fase 1 — Entender el problema (1-2 semanas)

Antes de defender la solución, hay que poder articular el problema. Cuatro bloques:

1. **Jailbreaks multi-turn** — qué son, por qué funcionan, por qué un monitor con memoria es vulnerable.
   Ataques de referencia: Crescendo (Russinovich), ActorAttack, Foot-In-The-Door.
2. **Untrusted monitoring (Greenblatt 2023)** — el problema de colusión, por qué usar el mismo modelo para vigilarse a sí mismo es estructuralmente débil.
3. **Ataques mosaico** — fragmentos individualmente benignos, daño en la composición.
4. **Sandbagging** — output correcto pero deliberadamente subóptimo, invisible a monitores de contenido.

**Criterio de paso a Fase 2:** José María explica cada uno con un ejemplo nuevo (no el del libro) y un modelo le hace preguntas; si responde sin pánico, pasamos.

---

## Fase 2 — Entender la propuesta (1-2 semanas)

Las tres ideas centrales de SAR. Para cada una: intuición, ataque que la motiva, objeción más fuerte.

1. **Statelessness** — borrar memoria del juez. Amenaza: M2S compaction.
2. **Heterogeneidad cross-lab** — tres labs distintos. Amenaza: Zou 2023 transferencia adversarial.
3. **Sliding window con overlap** — bordes como zona de ataque. Decisiones: 30% overlap, k de n.

**Criterio de paso a Fase 3:** José María articula la objeción más fuerte contra cada idea.

---

## Fase 3 — Defender (1 semana)

Q&A adversario simulado. Los modelos jugamos a revisor escéptico de NeurIPS / LessWrong y hacemos las 10 preguntas más duras. José María responde sin asistencia.

**Criterio de publicación:** 8 de 10 preguntas respondidas con solvencia. Las 2 que no, se reconocen como límites en el paper.

---

## Cómo trabajamos en cada sesión

- 30-45 min por sesión. Cortas y frecuentes mejor que largas y espaciadas.
- Un modelo explica, José María escribe con sus palabras lo que entiende.
- El modelo critica lo que José María ha escrito. No copy-paste de respuestas modelo.
- Referencias se leen, no se citan a ciegas.

---

## Estado

- [ ] Fase 1.1 — Jailbreaks multi-turn
- [ ] Fase 1.2 — Untrusted monitoring
- [ ] Fase 1.3 — Ataques mosaico
- [ ] Fase 1.4 — Sandbagging
- [ ] Fase 2.1 — Statelessness
- [ ] Fase 2.2 — Heterogeneidad
- [ ] Fase 2.3 — Sliding window
- [ ] Fase 3 — Q&A adversario
