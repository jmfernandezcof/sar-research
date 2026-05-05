# Preguntas técnicas abiertas

Estado: post-review DeepSeek (2026-05-05). Marca `[v0.4]` = ya tratado en v0.4. `[v0.5-blocker]` = bloquea la publicación de v0.5. `[abierto]` = sin resolver.

## Bloqueadores para v0.5

- **§5.3 overlap claim — bug matemático.** [v0.5-blocker, parcheado en v0.4 con erratum]
  Claim original: "every token seen in at least two contexts" con O = 0.3·L. Falso: con O < L/2 solo los tokens en la banda de solape se ven dos veces; el resto se ven una. Para cobertura ≥2 universal hace falta O ≥ L/2 (y aún así los bordes del primer/último fragmento son singletons). v0.5 debe (a) subir O a L/2 o (b) derivar O del threat model en lugar de heurístico.

- **M2S + Yang como refutación lógica de statelessness.** [v0.5-blocker]
  Cadena: si multi-turn ≈ single-turn compactado/reintentado, statelessness no aporta sobre cualquier juez single-turn robusto. Salida posible: caracterizar la subclase de ataques con dependencias inter-turn que la compactación pierde (cross-turn agent state, side-channels temporales, ataques que requieren que el target ya haya emitido X antes de aceptar Y). Si esa subclase no existe o es marginal → reposicionar SAR (drop "stateless" del título; tesis central pasa a ser heterogeneous sliding-window consensus). **Decisión aplazada hasta completar Fase 1.1 (lectura directa de M2S y Yang).**

- **T0 SPOF — espacio de diseño más amplio del reconocido.** [v0.5-blocker]
  §3.1 plantea falsa dicotomía (T0 vs raw history). DeepSeek expone 4 opciones; la más coherente con el resto del paper es **multi-T0 ensemble**: cada juez recibe summary de un summarizer distinto (3 modelos, 3 estrategias de compresión). SPOF distribuido aplicando heterogeneidad también al compactador. Coste: triplica T0 (que es la capa barata). v0.5 debe reescribir §3.1 con esta opción.

- **Tabla de costes concreta.** [v0.5-blocker]
  v0.4 dice "es caro" sin números. Estimación DeepSeek: ~12-25 API calls/turn SAR vs 1 para B1 → ~$0.02-0.05/turn vs ~$0.002 → 25× más caro. v0.5 debe incluir tabla con precios actuales y argumentar qué cargas justifican el coste (banking, critical infra; no chat).

## Mejoras fuertes (no bloqueadoras pero recomendadas)

- **H6b — fallo correlado bajo white-box.** Hipótesis adicional: un adversario con acceso a las tres familias puede construir inputs que causan fallo correlado a tasas que exceden el modelo de independencia por factor >2. Si cierto, heterogeneidad es marginal.

- **Framing Byzantine k=2/3.** Lamport 1982 asume fallos independientes; nuestros jueces correlan por pretraining. Quitar la cita Lamport o reframe como "majority of 3 with cross-lab requirement". Coherencia académica.

- **Caracterización de la subclase statelessness-relevant.** Conectado con el blocker M2S/Yang. Trabajo conceptual: ataques con cross-turn agent state, dependencias temporales, requisitos de orden de emisión.

## Heredados de v0.2 — estado actualizado

- **T0 paradox.** [reabierto por DeepSeek] La justificación de §3.1 ("o T0 o raw history") es falsa dicotomía. Ver blocker arriba.
- **T3 paradox.** [abierto] Single = SPOF. Ensemble = doble coste. Sin resolución.
- **k consensus.** [v0.4 — k=2/3 cross-lab justificado en §3.3, pendiente validación empírica]
- **30% overlap.** [v0.5-blocker] Ver arriba.
- **M(t) cuantitativo.** [abierto] Sin definición operativa.
- **R(t) vs M(t).** [v0.4 — degradado a principio de diseño]
- **M2S generalización.** [v0.5-blocker] Ver arriba.
- **Retry-equivalence Yang 2025.** [v0.5-blocker] Ver arriba.
- **κ cross-lab vs within-lab (H6).** [v0.4 — reformulado como ratio test]
- **iDecep / para-jailbreaking.** [abierto] Posible blind spot de T3.
- **ASA / SmuggleBench.** [abierto] Replicar contra T0.
- **Hard-halt vs API churn.** [abierto]
- **§6.6 budget.** [abierto] $200–500 USD pendiente.

## Verificación de referencias

- [abierto] Refs [8] Automating Deception (arXiv:2511.19517), [9] iDecep (arXiv:2604.24082), [10] SmuggleBench/ASA (arXiv:2604.06950): listas de autores pendientes de verificar antes de submission formal.

## Decisión arquitectónica pendiente — A vs B

- **A (conservadora):** mantener título "Stateless Adversarial Review", caracterizar subclase donde statelessness importa, fix overlap, multi-T0, tabla coste, H6b.
- **B (honesta-radical):** dropear "stateless" del título. Reposicionar como "Heterogeneous Sliding-Window Consensus for Agentic AI Supervision". Statelessness pasa a propiedad operacional.
- **Camino acordado (A′):** fix del overlap + erratum YA (hecho en v0.4). Decisión A vs B se aplaza hasta completar Fase 1.1 — leer M2S (Ha et al. ACL 2025) y Yang et al. 2025 con criterio propio. Solo entonces se sabrá si la subclase statelessness-relevant existe.
