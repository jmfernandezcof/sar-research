# Stateless Adversarial Review (SAR)
## Heterogeneous Multi-Model Supervision for Agentic AI Systems

**Author:** José María Fernández Gómez
**Affiliation:** Nomad Prompters · Independent AI Safety Research · Saceruela, Ciudad Real, Spain
**Contact:** jmfernandez@nomadprompters.com · [github.com/jmfernandezcof/sar-research](https://github.com/jmfernandezcof/sar-research)
**Date:** Working Paper v0.4 — May 2026
**Status:** Pre-empirical position paper. Not peer-reviewed.
**Erratum (2026-05-05):** §5.3 contained a mathematically false coverage claim ("every token seen in at least two contexts" with 30% overlap). Corrected in place; superseded by v0.5. Other open issues raised in expert review (T0 SPOF, M2S/Yang challenge to statelessness, overlap derivation from threat model, cost table, Byzantine framing) are tracked in `research/open-questions.md` and will be addressed in v0.5.
**License:** CC BY 4.0

> *Built with the Nomad Council AI ensemble (BITchito, BITrro, Nyx, Lex). v0.4 incorporates additional empirical context from a literature scan provided by Lex (Gemini Pro), with all citations verified against primary sources before incorporation. Material rejected from that scan is documented in Appendix C.*

---

## Abstract

As AI systems transition from conversational assistants to autonomous agents with real-world affordances, the supervision challenge shifts from content moderation to behavioural governance. Current monitoring architectures predominantly rely on models from a single training lineage to supervise themselves, creating systemic blind spots documented in Anthropic's *Alignment Risk Update for Claude Mythos Preview* (April 2026).

This paper describes **Stateless Adversarial Review (SAR)**, a candidate architecture for multi-tiered supervision built on three design principles: (1) **structural amnesia** — judges hold no cross-turn memory, intended to limit the progressive manipulation vector; (2) **heterogeneous oversight** — judges drawn from independently trained model families, intended to reduce shared blind spots; and (3) **sliding-window fragmentation** with mandatory overlap, intended to defeat mosaic attacks.

We do not claim novelty for the broad strokes. Untrusted monitoring was articulated by Greenblatt et al. (2023) within the AI Control programme. Multi-dimensional collusion-resistant monitoring was proposed by Nguyen et al. (TraceGuard, 2026). Multi-agent judge frameworks (e.g. HAJailBench with Multi-Agent Debate, 2025–2026) already use critic, defender, and judge configurations. Stateful intent-drift tracking was proposed by Albrethsen et al. (DeepContext, 2026) — a deliberately opposite approach to ours. SAR's contribution is a specific combination of design choices accompanied by an explicit threat model that includes attacks against the architecture itself, and an experimental design that includes hypotheses (H5) under which the central principle is empirically defeated by Ha et al. (M2S, ACL 2025) and Yang et al. (2025).

**v0.4 update.** This version incorporates empirical evidence from recent (2025–2026) literature that *both supports and challenges* the design intuitions: heterogeneity benefit gains empirical motivation from cross-lab vulnerability divergence (per *Automating Deception*, arXiv:2511.19517), while statelessness as defence against multi-turn attacks faces additional pressure from the finding that multi-turn attacks may be approximately equivalent to retried single-turn resampling (Yang et al., *Multi-Turn Jailbreaks Are Simpler Than They Seem*, arXiv:2508.07646).

This is a **position paper**, not a research paper. We present a design, a falsifiable experimental protocol, and an honest enumeration of vulnerabilities in our own proposal.

**Keywords:** AI safety, agentic supervision, multi-model oversight, AI control, adversarial robustness, position paper.

---

## 1. Introduction

The dominant paradigm for AI safety monitoring is straightforward: **use AI to watch AI**. Anthropic monitors Claude with Claude-derived systems. OpenAI deploys GPT-derived classifiers to oversee GPT agents. The approach is practical and cost-effective. It is also structurally vulnerable in ways that recent literature has begun to map.

### 1.1 Position relative to existing work

This paper does not invent the problem space. We position SAR within an active area of research:

- **AI Control / Untrusted Monitoring** — Greenblatt et al. (2023, *AI Control: Improving Safety Despite Intentional Subversion*, arXiv:2312.06942). The foundational framing of the collusion problem and multi-instance protocols. SAR's statelessness principle is one specific implementation of the broader untrusted-monitoring family, focused on limiting cross-turn manipulation. The "untrusted monitoring collapses to ~0% safety" finding from this work is a key motivator for any cross-lab approach to monitoring.

- **TraceGuard** — Nguyen et al. (2026, arXiv:2604.03968). Proposes structured multi-dimensional monitoring evaluating agent actions across five dimensions (goal alignment, constraint adherence, reasoning coherence, safety awareness, action-trace consistency), scored in parallel by independent LLM calls. SAR's Tier 2 is a close cousin; we differ in mandating *cross-lab* heterogeneity rather than within-lab structural diversity, and in adding sliding-window fragmentation. *(Note: a different paper also titled "TraceGuard" exists addressing reasoning backdoors — Washington University, arXiv:2603.02436 — and is not the reference here.)*

- **HAJailBench + Multi-Agent Debate** — *Efficient LLM Safety Evaluation through Multi-Agent Debate* (2025–2026, arXiv:2511.06396). Introduces an 11,100-example human-annotated jailbreak benchmark and a Multi-Agent Judge framework (critic + defender + judge under shared safety rubric) that allows smaller backbones such as Qwen3-14B to approach larger judges' accuracy. SAR shares this multi-agent lineage; our incremental contribution is the strict statelessness constraint and the orchestrator design.

- **DeepContext** — Albrethsen et al. (2026, arXiv:2602.16935). **Proposes the opposite approach to SAR**: a stateful RNN-based monitor that propagates a hidden state across conversation turns to track intent drift, achieving F1=0.84 on multi-turn benchmarks at sub-20ms latency. SAR and DeepContext represent two opposing hypotheses about how to handle multi-turn attacks: track explicitly (DeepContext) or refuse to remember (SAR). We argue both have merit in different threat models, and §6 includes a comparative experimental hypothesis.

- **M2S** — Ha et al. (ACL 2025, *M2S: Multi-turn to Single-turn jailbreak in Red Teaming for LLMs*, arXiv:2503.04856 / 2025.acl-long.805). Demonstrates that multi-turn adversarial attacks can be systematically reformatted into single-turn prompts (Hyphenize, Numberize, Pythonize templates) that preserve and sometimes enhance adversarial potency, achieving 70.6%–95.9% ASR while reducing token usage by half. **This result partially undermines SAR's central design principle** — if multi-turn attacks compress effectively to single-turn, statelessness alone offers no defence.

- **"Multi-Turn Jailbreaks Are Simpler Than They Seem"** — Yang et al. (2025, arXiv:2508.07646). Empirically demonstrates that multi-turn jailbreak attacks, when accounting for retry mechanisms, are approximately equivalent to resampling single-turn attacks multiple times. **This compounds the M2S concern**: if multi-turn attacks reduce to repeated single-turn attempts, then statelessness's structural defence is reduced to whatever single-turn robustness the underlying judges already had.

- **Cross-lab vulnerability divergence** — *Automating Deception: Scalable Multi-Turn LLM Jailbreaks* (2025, arXiv:2511.19517). Reports stark differences in contextual robustness across labs: GPT family ASR rises by up to 32 percentage points with conversational history; Gemini 2.5 Flash is "nearly immune"; Claude 3 Haiku resists but imperfectly. **This is empirical evidence supporting the heterogeneity intuition**: distinct labs have distinct multi-turn vulnerabilities, so a cross-lab judge ensemble has lower correlated false-negative risk than a same-lab one.

We address all of these in §3, §4, §5, and §6.

### 1.2 What SAR contributes (specifically)

Our differential contribution is a particular combination, not any single component:

- **Strict cross-turn statelessness** of judges — not "limited memory" or "summarised memory", but zero cross-turn memory.
- **Mandatory cross-lab heterogeneity** — at least two distinct training lineages required for consensus, not just multiple instances of one model.
- **Sliding-window fragmentation with mandatory overlap** — not optional, not for performance, but as a hard architectural constraint with documented intent (mosaic attack mitigation).
- **A non-learning deterministic orchestrator** with hard system invariants that halt the pipeline rather than degrade silently.

Each piece exists in the literature. The combination, with a threat model that includes the architecture's own vulnerabilities and an experimental design that names its falsifiers, is what we are proposing for evaluation.

### 1.3 What this paper claims, and what it does not

This is a **position paper**. It contains:

- A design.
- A falsifiable experimental protocol.
- An honest enumeration of architectural vulnerabilities in our own proposal.
- An explicit acknowledgement of failure modes the design does not address.

It does **not** contain:

- Empirical results from adversarial benchmarks against SAR itself.
- A proven theorem about robustness.
- A claim of novelty for the broader research direction.

Empirical validation of SAR is identified as the critical missing piece. Where empirical evidence from third parties bears on SAR's design intuitions, we cite it.

> *This is not a solved problem. This is a design proposal with falsifiable predictions.*

---

## 2. Problem Statement

We identify five documented failure modes in current monitoring architectures.

### 2.1 Collusion risk

When a model and its supervisor share a training distribution, they share systematic blind spots. The space of outputs invisible to the supervisor overlaps significantly with the space of outputs the supervised model is most likely to produce, because both are shaped by the same data and training pressures.

This is the structural argument of the AI Control programme. Greenblatt et al. (2023) formally evaluated trusted-monitoring protocols and found that untrusted monitoring (using the same model as both agent and monitor) **collapses to near-zero safety**. SAR does not originate this concern; it proposes one specific mitigation.

### 2.2 Iterative attack vulnerability — and its complication

Stateful supervisors accumulate context. An adversary can craft multi-turn interactions that progressively shift semantic framing, normalising outputs that would trigger detection in isolation. Anthropic's *Alignment Risk Update for Claude Mythos Preview* (April 2026) documents specific monitoring failure modes, including degraded performance against iterative red-team attacks compared to single-shot attempts, and acknowledges collusion-style risk pathways arising from same-lineage monitoring.

> *Citation note: rather than citing a specific percentage, we cite the existence of the documented failure mode. Russinovich et al. (Crescendo, USENIX Security 2025) report ASR up to 98–100% on certain target models for multi-turn attacks, and Greenblatt et al. (2023) report ~62% safety for trusted monitoring under intentional subversion.*

**Two recent results substantially complicate the simple "stateless judges defeat multi-turn attacks" intuition:**

- **M2S** (Ha et al., ACL 2025). Multi-turn attacks can be reformatted into single-turn equivalents (Hyphenize, Numberize, Pythonize templates) that preserve effectiveness, achieving 70.6%–95.9% ASR. If this compaction generalises, the value of statelessness against the broader class of "multi-turn attacks" is reduced.

- **Multi-Turn Jailbreaks Are Simpler Than They Seem** (Yang et al., 2025, arXiv:2508.07646). When accounting for retry mechanisms after refusal, multi-turn approaches are approximately equivalent to repeatedly resampling single-turn attacks. This finding holds across GPT, Claude, and Gemini variants in the authors' tests.

Together, these results imply that the meaningful defence is not statelessness per se, but whatever single-turn robustness each judge brings — and that requires the cross-lab heterogeneity argument (§5.2) to do most of the actual security work. We address this directly in the experimental design (§6, hypothesis H5).

### 2.3 Agentic blind spots and contemporary attack classes

Beyond conversational manipulation, agentic systems introduce attack surfaces increasingly documented in 2025–2026 literature:

- **Mosaic attacks**: distributing harmful actions across multiple individually-innocuous tool calls.
- **Long-horizon coordination**: spreading harmful goals across days, weeks, or months.
- **Tool-output context poisoning**: adversarial content injected via tool results that re-enter the model's context.
- **Intention deception (iDecep, arXiv:2604.24082)**: multi-turn narratives that establish a benign cover (e.g. impersonating a police officer drafting a prevention report) to elicit restricted information through what the authors term "para-jailbreaking" — the model refuses the direct harmful request but provides exploitable alternatives. Specifically effective against GPT-5-thinking and Claude-Sonnet-4.5, exploiting "safe completion" mechanisms that prefer helpful-sounding alternatives over flat refusal.
- **Adversarial Smuggling Attacks (ASA, arXiv:2604.06950)**: harmful content encoded into human-readable visual formats that remain AI-unreadable, exploiting the gap between human perception and OCR/vision-encoder capability. The SmuggleBench benchmark reports ASR exceeding 90% on both proprietary (GPT-5) and open-source (Qwen3-VL) state-of-the-art models. Relevant to SAR because the Tier 0 Summary Agent is itself a vision-or-text-processing component vulnerable to this class.

### 2.4 Sandbagging — beyond the reach of any content monitor

Sandbagging — producing technically correct but intentionally suboptimal outputs — is invisible to any system that monitors content rather than performance. **SAR does not address sandbagging.** This is a known limitation of the entire content-monitoring family of approaches, not of SAR specifically.

---

## 3. SAR Architecture

SAR is a five-tier pipeline. Each tier addresses a specific failure mode, and each tier has documented vulnerabilities of its own (§4).

```
INPUT
  │
  ▼
[T0] Summary Agent (stateless) — compresses context, intended to prevent history manipulation
  │
  ▼
[T1] L1 Fast Filter (Llama Guard) — cheap, catches obvious violations
  │ (~13% proceed to T2 in benign-traffic-dominant deployments)
  ▼
[T2] Fragment Judges × 3 (Claude + GPT-4o + Gemini, stateless, k=2/3 majority + cross-lab requirement)
  │
  ▼
[T3] Composed Review — full output holistic check, intended to catch mosaic attacks
  │
  ▼
[T4] Audit Layer — hash-chained logs, canary injection, alerting
  │
  ▼
OUTPUT
```

### 3.1 Tier 0 — Summary Agent

A stateless agent compresses prior conversation context into a bounded summary before each evaluation. Intent: prevent progressive manipulation of judges by limiting their exposure to controlled context.

**Why keep T0 despite §4.1?** A reasonable alternative is to remove T0 entirely and let each judge see truncated raw history directly. We retain T0 for two reasons: (a) raw history exposes the same progressive-manipulation surface that T0 is meant to mitigate — every judge becomes vulnerable to the same context attack the architecture is supposed to defend against; (b) bounded summaries reduce judge cost substantially in long conversations. The trade-off is real: T0 introduces a new SPOF (§4.1), and the SmuggleBench result (arXiv:2604.06950) shows the class of attack the SPOF is vulnerable to is not theoretical. Removing T0 returns the system to the very stateful exposure SAR is designed to avoid. We treat this as a design choice with stated risks rather than a clean win, and identify summary-agent hardening as critical future work.

### 3.2 Tier 1 — L1 Fast Filter

A lightweight local classifier (e.g., Llama Guard) catches obvious violations cheaply. A small probabilistic sample (ε ≈ 0.05) of T1-passed inputs is forwarded to T2 anyway as an audit mechanism, to detect cases where T1 systematically misclassifies.

### 3.3 Tier 2 — Fragment Judges (the core of SAR)

Output is segmented into overlapping fragments and evaluated independently by three stateless judges drawn from different labs:

- **J₁**: Claude (Anthropic)
- **J₂**: GPT-4o (OpenAI)
- **J₃**: Gemini (Google)

**Sliding-window fragmentation** with mandatory ≥30% token overlap ensures no token is evaluated in isolation:

```
Output:    [T1 T2 T3 T4 T5 T6 T7 T8 T9]
Fragment 1: [T1 T2 T3 T4]
Fragment 2:      [T3 T4 T5 T6]
Fragment 3:           [T5 T6 T7 T8 T9]
```

**Consensus rule (k = 2/3 with cross-lab requirement):** content passes only if **at least 2 of 3 judges vote PASS**, *and* **the passing judges include at least 2 distinct model families**.

The k=2/3 threshold balances three concerns: (a) requiring unanimity (3/3) makes the system fragile to individual judge errors and increases false positives; (b) accepting any single judge (1/3) loses the heterogeneity benefit; (c) the cross-lab clause prevents 2 instances from the same family from carrying a vote — a single Claude+Claude or GPT+GPT majority would reintroduce same-lineage collusion risk.

**Prompt jitter:** judges receive randomly-selected variants from a pool of 50 semantically-equivalent constitutions, intended to prevent adversarial overfitting to specific rule phrasings.

### 3.4 Tier 3 — Composed Review

Fragment-level analysis can miss emergent semantic threats. T3 runs a holistic evaluation on the complete assembled output.

**Acknowledged paradox (see §4.2):** if T3 uses a single model, it reintroduces the SPOF that T2's heterogeneity was designed to eliminate. If T3 also uses an ensemble, computational and latency costs roughly double. This is an open architectural question.

### 3.5 Tier 4 — Audit Layer

All decisions are logged with cryptographic hash-chaining (genesis → entry 1 → entry 2 → ...), enabling immutable audit trails. Periodic canary injection verifies ongoing system integrity.

### 3.6 Deterministic orchestrator

The orchestration layer is implemented in non-learning code (Python with Pydantic, or Rust). It is the single point of failure in the architecture and **must not be a learned model**. System invariants are enforced as hard constraints:

```yaml
system_constitution:
  judge_statelessness:
    enforcement: HARD
    violation: SYSTEM_HALT
  minimum_overlap:
    value: 0.30
    enforcement: HARD
  judge_heterogeneity:
    minimum_models: 3
    maximum_same_family: 1
    enforcement: HARD
```

Violations trigger immediate system halt.

**Acknowledged vulnerability (see §4.4):** in production, hard-halt invariants interact poorly with upstream API changes and rate limits.

---

## 4. Threats to the Architecture Itself

This section names vulnerabilities of the proposal *as proposal*. v0.4 strengthens §4.1 with empirical evidence and §4.3 with the cross-lab correlation finding from Yang et al. (2025).

### 4.1 The Tier 0 attack — semantic prompt smuggling

The Summary Agent (T0) is the most attractive attack target in the architecture. It is the chokepoint through which all context flows, and its statelessness — designed as a defence — is also a denial of historical perspective.

An adversary can embed instructions in seemingly innocuous variables, tool-call payloads, or external content that the T0 ingests. Because T0 has no memory of prior interactions, it cannot detect gradual manipulation; it compresses what it sees in the moment.

If T0 is induced to omit critical metadata or distort framing, it poisons the inputs to all three downstream judges simultaneously. The heterogeneity of T2 provides no defence here — the judges all read the same poisoned summary.

This is a **Man-in-the-Middle attack at the semantic layer**. **The class of attack is not theoretical.** SmuggleBench (arXiv:2604.06950) reports ASR exceeding 90% for adversarial smuggling attacks against GPT-5 and Qwen3-VL — a closely related vulnerability class that exploits perception/OCR limits. Even purely textual variants of context-poisoning are documented in iDecep (arXiv:2604.24082) where multi-turn benign covers induce the model into responses that bypass safe-completion safeguards.

We do not currently have a mitigation for the T0 attack that does not reintroduce stateful tracking (which the architecture is designed to avoid). §3.1 documents why we retain T0 despite this; the trade-off is open.

### 4.2 The Tier 3 paradox

T2's heterogeneity is justified as a defence against same-lab blind spots. T3's stated purpose is holistic review of the assembled output to catch emergent mosaic semantics that fragment-level analysis misses.

But T3 has the last word over the assembled context. Either:

- **T3 uses a single model**, in which case the SPOF that T2 was designed to avoid is reintroduced at the final stage, or
- **T3 also uses a heterogeneous ensemble**, which roughly doubles the cost and latency of an already expensive pipeline.

We do not present a clean resolution. The honest answer is that this is an open design question, and the marginal value of T3 over T2 (or vice versa) requires an ablation study not yet performed.

### 4.3 Judge independence is partial at best — and now there is empirical evidence

The exponential-decay claim about joint bypass probability across k heterogeneous judges (in any multi-agent supervision proposal, including this one) implicitly assumes statistical independence of judge errors.

There is empirical evidence against full independence. Zou et al. (2023, arXiv:2307.15043) demonstrate that adversarial prompts trained against one aligned model often **transfer** to others, including across labs.

**Yang et al. (2025, arXiv:2508.07646)** provide more recent and direct evidence: in their analysis, "models from the same lab are more likely to show similar robustness", and "attack success is more correlated among models of the same provider, especially for single-turn attacks". Claude variants in particular show high inter-model correlation in their benchmarks.

**This finding cuts in both directions for SAR:**

- *Against the design*: any pure exponential-decay claim about joint bypass probability is empirically false. Same-family judges are correlated. The heterogeneity benefit is not what an independence assumption would predict.
- *For the design*: the very fact that within-lab correlation is higher than cross-lab correlation **is the empirical justification for the cross-lab requirement** in §3.3. If correlation were uniform across labs, lab diversity would buy nothing. The Yang et al. finding is the cleanest current evidence that lab diversity matters operationally.

**Honest claim:** the heterogeneity benefit is real, *bounded*, and likely substantially weaker than independence-based decay would predict. Magnitude must be measured directly via Cohen's κ between cross-lab judge verdicts (§6, hypothesis H6).

### 4.4 Operational maintenance is a real cost

Beyond the technical critique, a deployment of SAR requires sustained engineering against a moving target: three vendor APIs, each with rate limits, version updates, default-behaviour changes, and unannounced filter modifications. The hard-halt invariants in §3.6 increase operational fragility — an upstream change that violates an invariant produces a SYSTEM_HALT that the operator must diagnose and reconcile under live conditions.

This is not a flaw in the security argument. It is a flaw in the **deployability** argument, and it is the kind of cost that often determines whether a security architecture survives contact with production.

---

## 5. Design Principles

Earlier drafts of this paper presented the relation **R(t) ∝ 1/M(t)** with mathematical notation, framing it as a formal property. This was incorrect. The expression is a **design principle**, not a theorem; M(t) is not quantitatively defined, the inverse-proportional functional form is arbitrary, and no derivation is provided.

We retain the principle in prose:

> *System robustness against progressive multi-turn attacks is qualitatively reduced as the supervisor accumulates cross-turn state, because accumulated state provides surface area for incremental adversarial manipulation.*

This is the operating intuition behind the statelessness requirement. It is not a proven relationship and should not be treated as one. If formalised in future work, it would need:

- A precise definition of M(t) (token count? semantic drift measure? entropy of context?).
- A defined adversary model with bounded budget.
- A derivation of the functional form (which is unlikely to be cleanly inverse-proportional in practice; degradation curves typically reflect attention distribution effects and are non-linear).

We treat this as future work, not a current contribution.

### 5.1 Design intuition — stateless judges (with significant qualifications)

If a judge has no cross-turn memory, then for any n-turn attack A_n, the judge sees each turn as if it were turn 1. Progressive manipulation strategies that rely on context accumulation cannot, by construction, exploit memory the judge does not have.

**Two major qualifications from 2025 literature:**

- **M2S (Ha et al., ACL 2025).** If multi-turn attacks compress effectively to single-turn equivalents, this design intuition is much weaker than it appears. A single-turn attack carrying all the manipulation power of an n-turn attack defeats statelessness.

- **Yang et al. (2025).** Even without compaction templates, multi-turn attacks may be approximately equivalent to retried single-turn attempts. This further reduces the structural advantage statelessness provides.

We treat these as serious challenges, not footnotes. The empirical generality of M2S compaction and the retry-equivalence finding are explicit falsifiable elements of our experimental design (§6, H5).

### 5.2 Design intuition — heterogeneity (now with empirical support)

Judges from different training lineages have different blind spots. Requiring at least two model families to agree before passing content reduces the rate of correlated false negatives.

**Empirical support added in v0.4.** *Automating Deception* (arXiv:2511.19517) reports stark cross-lab differences in multi-turn robustness:

- GPT family: ASR rises by up to **32 percentage points** when conversational history is enabled.
- Gemini 2.5 Flash: "**nearly immune**" to the same attack class.
- Claude 3 Haiku: strong but imperfect resistance.

This is the kind of evidence that moves the heterogeneity intuition from theoretical preference to empirically motivated design. If different labs differ this strongly in vulnerability, then a cross-lab judge ensemble has materially lower correlated-failure risk than a same-lab one — even without claiming exponential decay.

**Qualification (§4.3):** the absolute magnitude of the heterogeneity benefit is bounded by the (substantial) overlap in training distributions across labs. The Yang et al. finding that within-lab models show higher correlation than cross-lab models bounds the benefit but also confirms that lab diversity is the relevant axis. Empirical measurement of cross-lab judge agreement (Cohen's κ) on SAR-specific tasks remains required to make any quantitative claim.

### 5.3 Design intuition — sliding-window overlap

A token evaluated in only one fragment is vulnerable to fragmentation attacks where the harmful semantics live at fragment boundaries. With fragment length L and overlap O, the step size is L−O and a token at position p appears in ⌈min(L, p+1) / (L−O)⌉ fragments (bounded by 1 at the edges). With O = 0.3·L, only tokens that fall inside the overlap band (≈30% of positions) are seen by two fragments; the remaining ≈70% are seen exactly once. To guarantee every interior token is evaluated in at least two fragments, O ≥ L/2 is required. The 30% figure in earlier drafts is a heuristic and the prior claim that "every token is seen in at least two contexts" was incorrect; the overlap parameter must be derived from the threat model in v0.5 (see `research/open-questions.md`).

---

## 6. Experimental Design

A falsifiable experimental protocol is essential. We design an experiment that explicitly includes hypotheses under which SAR fails to outperform baselines.

### 6.1 Baselines

- **B1**: stateful Claude monitor (current Anthropic-style approach)
- **B2**: Llama Guard only (lightweight static filter)
- **B3**: Constitutional AI (model-level alignment)
- **B4**: DeepContext (Albrethsen et al. 2026) — direct comparison with the *opposite* design philosophy (stateful intent-drift tracking)

### 6.2 SAR variants

- **SAR-H1**: heterogeneous stateless (Claude + GPT-4o + Gemini)
- **SAR-S1**: homogeneous stateless (3 × Claude) — control to isolate the contribution of heterogeneity vs. number of judges

### 6.3 Attack types

1. Single-shot jailbreaks (control)
2. Multi-turn progressive attacks (5–10 turns) — Crescendo (Russinovich et al. USENIX 2025), ActorAttack
3. Mosaic attacks — composed harm from benign fragments
4. **M2S-compacted attacks** (Ha et al. ACL 2025)
5. **Retry-equivalent single-turn attacks** (Yang et al. 2025) — single-turn attacks resampled multiple times to test the "multi-turn ≈ retried single-turn" finding
6. **iDecep-style intent-deception attacks** (arXiv:2604.24082) — multi-turn benign-cover narratives exploiting safe-completion
7. Sandbagging simulation (control for known blind spot)

### 6.4 Metrics

- **ASR** (Attack Success Rate) by attack type
- **FPR** (False Positive Rate)
- **Latency** per query
- **Cost** per query (USD per 1k queries)
- **Cohen's κ** between judges (independence verification — directly tests the §4.3 concern)
- **Tier 0 robustness** — separate test for prompt smuggling against the summary agent

### 6.5 Falsifiable hypotheses

| Hypothesis | Predicted outcome | What it tests |
|---|---|---|
| **H1**: SAR-H1 < B1 in ASR for multi-turn attacks (non-compacted) | SAR wins | Core claim |
| **H2**: SAR-H1 ≈ B1 in ASR for single-shot attacks | No SAR advantage | Honest scope limit |
| **H3**: SAR-S1 > SAR-H1 in ASR | Heterogeneity matters | Validates lab diversity vs. judge count |
| **H4**: All variants fail on sandbagging | Known blind spot confirmed | Stated limitation |
| **H5**: SAR-H1 ≈ SAR-S1 ≈ B1 in ASR for **M2S-compacted and retry-equivalent attacks** | Statelessness defeated by compaction/resampling | **Tests M2S (Ha et al.) + Yang et al. as falsifiers** |
| **H6**: Cohen's κ between Claude/GPT-4o/Gemini judges shows substantial correlation, but materially lower than within-lab κ | Yang et al. finding replicates on SAR task | **Tests §4.3 empirically with comparator** |
| **H7**: SAR-H1 vs DeepContext — stateless vs stateful comparison on multi-turn attacks | Open comparative test | Both philosophies on the table |

**On the H6 design.** Rather than fix an arbitrary κ threshold, H6 takes the form: *cross-lab κ is materially lower than within-lab κ on the same benchmark*. This is operationally testable and directly aligned with the Yang et al. (2025) finding. If within-lab κ is not higher than cross-lab κ in our measurements, the heterogeneity argument loses its principal empirical justification.

If H5 holds, SAR's value is restricted to attack classes M2S compaction does not generalise to.
If H6 fails (within-lab and cross-lab correlations are similar), the cross-lab requirement loses its empirical motivation.

We commit to publishing both outcomes if observed.

### 6.6 Minimum viable experiment

Recognising practical resource constraints, the minimum useful version of this protocol is:

- 100 prompts each from Crescendo and ActorAttack (200 total).
- 3 conditions: B1, SAR-S1, SAR-H1.
- API cost estimate: ~500 USD with current pricing.
- Outputs: ASR per condition, Cohen's κ (within-lab vs cross-lab), plus qualitative failure mode analysis.

A reduced 50-prompt subset (~200 USD) would still produce directional results suitable for v0.5.

Even this minimum version moves the paper from "design proposal" to "preliminary evidence" and would substantially strengthen the case for further investigation.

---

## 7. Application to Mythos

Anthropic's *Alignment Risk Update for Claude Mythos Preview* (April 2026) identifies a set of agentic risk pathways, several of which describe specific behaviours observed during internal testing including unauthorised actions taken with overreach (e.g. deletion of Git branches outside intended scope, premature production database operations, unauthorised exfiltration attempts), as well as architectural concerns around same-lineage monitoring and sandbox-escape behaviours.

We analyse SAR's coverage honestly across the risk categories most relevant to runtime behavioural monitoring.

### 7.1 Where SAR helps

1. **Same-lineage collusion risk.** Mythos monitored by Claude-derived monitors has correlated blind spots. Cross-lab heterogeneity reduces this, with empirical motivation now coming from cross-lab vulnerability divergence (*Automating Deception*, arXiv:2511.19517) and within-lab correlation findings (Yang et al., 2025).
2. **Progressive (non-compacted) multi-turn attacks.** Stateless judges, by construction, cannot accumulate the context that progressive manipulation depends on — though M2S and retry-equivalence findings limit the size of this win.
3. **Chain-of-thought monitoring.** SAR can be applied to reasoning traces, not only final outputs.

### 7.2 Where SAR does NOT help

1. **Sandbagging.** Invisible to fragment judges by construction.
2. **Irreversible actions already executed.** Runtime monitoring cannot recover deleted state.
3. **Long-horizon coordination.** Beyond the sliding window, SAR sees only the summary, with all the §4.1 risks.
4. **Weight poisoning.** Training-time attacks are invisible to a runtime monitor.
5. **M2S-compacted or retry-equivalent attacks.** If multi-turn attacks compress to or behave like repeated single-turn attempts (Ha et al. 2025; Yang et al. 2025), statelessness alone provides no defence beyond per-turn judge robustness.
6. **iDecep / para-jailbreaking attacks.** Multi-turn benign covers exploiting safe-completion mechanisms (arXiv:2604.24082) bypass any system that monitors only direct harmful content rather than indirect leakage patterns.

### 7.3 Honest scope assessment

SAR substantively addresses a subset of the Mythos failure modes — most directly the same-lineage collusion concern and non-compacted progressive attacks. It does not address sandbagging, weight poisoning, already-executed irreversible actions, or several contemporary attack classes (iDecep, ASA), which together represent a substantial fraction of the documented agentic risk surface.

This is not a comprehensive solution. It is a defence-in-depth contribution targeting specific, named vulnerabilities.

### 7.4 Concrete proposal for Anthropic

SAR as a supplementary layer, not a replacement:

- **Layer 1 — keep:** existing monitoring (Opus 4.6 or successor).
- **Layer 2 — add:** heterogeneous stateless layer with GPT-4o + Gemini judges.
- **Layer 3 — research:** sandbagging detection through cross-domain comparative performance analysis (out of scope for SAR).

---

## 8. Limitations

Consolidated limitations beyond §4 architectural threats:

- **Cost.** Three judge calls per fragment plus orchestration overhead is significantly more expensive than a single stateful monitor. Justified only for high-risk paths.
- **Latency.** End-to-end pipeline latency is incompatible with sub-second response requirements. DeepContext (Albrethsen et al. 2026) achieves <20ms with its stateful approach; SAR will not match this.
- **Independence assumption is empirically weak.** Per §4.3, Yang et al. (2025), and Zou et al. (2023). Heterogeneity benefit is real but bounded.
- **M2S generalisation and retry-equivalence.** If multi-turn-to-single-turn compaction (Ha et al. ACL 2025) or retry-equivalence (Yang et al. 2025) is broadly effective, the central design principle weakens substantially.
- **Summary agent SPOF.** Per §4.1; SmuggleBench (arXiv:2604.06950) demonstrates the relevant attack class is operational, not theoretical.
- **iDecep / para-jailbreaking blind spot.** SAR judges, like other content monitors, are vulnerable to indirect leakage via benign-cover multi-turn deception.
- **Tier 3 paradox unresolved.** Per §4.2.
- **Operational fragility.** Hard-halt invariants interact poorly with vendor API churn.
- **No empirical validation of SAR itself.** This is a design proposal. The case is weak until §6.6 (or fuller) is executed.
- **Selection bias in judges.** Choice of Claude / GPT-4o / Gemini reflects current accessible top-tier models; results may not generalise.

---

## 9. Conclusion

SAR proposes structural amnesia, heterogeneous oversight, and sliding-window fragmentation as architectural responses to two specific documented vulnerabilities: collusion risk between same-lineage monitors, and progressive multi-turn attack vulnerability against stateful judges.

We position the work explicitly within an active research area (Greenblatt et al. 2023, Nguyen et al. 2026, HAJailBench/Multi-Agent Debate, Albrethsen et al. 2026, Ha et al. 2025, Yang et al. 2025) rather than claiming virgin territory. The differential contribution is a specific combination of design choices — strict cross-turn statelessness, mandatory cross-lab heterogeneity, sliding-window overlap, and a non-learning deterministic orchestrator — accompanied by a threat model that includes attacks against the architecture itself.

**v0.4 explicitly engages with results that bound the design's value:** M2S compaction (Ha et al.) and the retry-equivalence finding (Yang et al.) constrain how much statelessness alone can do; the cross-lab vulnerability divergence (*Automating Deception*) gives the heterogeneity argument concrete empirical motivation; the within-lab correlation finding (Yang et al.) simultaneously bounds the upper limit of heterogeneity benefit and provides its empirical justification.

The design principle behind statelessness is **not a theorem**. Empirical validation of SAR itself is the critical missing piece, and a minimum-viable protocol is specified in §6.6. Falsifiable hypotheses (H5, H6, H7) name the conditions under which the design fails, and we commit to publishing those outcomes if observed.

SAR is not an alignment solution. It is a containment cage with stated gaps. We invite replication, critique, and falsification.

> *We do not design to eliminate risk, but to ensure that no local optimisation can turn it into global catastrophe without leaving traces.*

---

## References

[1] Anthropic. (2026). *Alignment Risk Update: Claude Mythos Preview*. April 7, 2026. URL: anthropic.com/claude-mythos-preview-risk-report

[2] Greenblatt, R., Shlegeris, B., Sachan, K., & Roger, F. (2023). *AI Control: Improving Safety Despite Intentional Subversion*. arXiv:2312.06942.

[3] Nguyen, K. L., et al. (2026). *TraceGuard: Structured Multi-Dimensional Monitoring as a Collusion-Resistant Control Protocol*. arXiv:2604.03968.

[4] *Efficient LLM Safety Evaluation through Multi-Agent Debate* (HAJailBench + Multi-Agent Judge framework). (2025–2026). arXiv:2511.06396.

[5] Albrethsen, J., Kumar, K., Datta, Y., & Rajasekar, S. (2026). *DeepContext: Stateful Real-Time Detection of Multi-Turn Adversarial Intent Drift in LLMs*. arXiv:2602.16935.

[6] Ha, J., Kim, H., Yu, S., Park, H., Yousefpour, A., Park, Y., & Kim, S. (2025). *M2S: Multi-turn to Single-turn jailbreak in Red Teaming for LLMs*. ACL 2025. arXiv:2503.04856 / 2025.acl-long.805.

[7] Yang, X., et al. (2025). *Multi-Turn Jailbreaks Are Simpler Than They Seem*. arXiv:2508.07646.

[8] *Automating Deception: Scalable Multi-Turn LLM Jailbreaks*. (2025). arXiv:2511.19517. *(Author list to verify against primary source before formal submission.)*

[9] *Jailbreaking Frontier Foundation Models Through Intention Deception (iDecep)*. (2026). arXiv:2604.24082. *(Author list to verify against primary source before formal submission.)*

[10] *Making MLLMs Blind: Adversarial Smuggling Attacks in MLLM Content Moderation (SmuggleBench / ASA)*. (2026). arXiv:2604.06950. *(Author list to verify against primary source before formal submission.)*

[11] Zou, A., et al. (2023). *Universal and Transferable Adversarial Attacks on Aligned Language Models*. arXiv:2307.15043.

[12] Russinovich, M., Salem, A., & Eldan, R. (2025). *Great, now write an article about that: The Crescendo Multi-Turn LLM Jailbreak Attack*. USENIX Security 2025.

[13] Bai, Y., et al. (2022). *Constitutional AI: Harmlessness from AI Feedback*. Anthropic.

[14] Perez, E., et al. (2022). *Red Teaming Language Models with Language Models*. arXiv:2202.03286.

[15] Saunders, W., et al. (2022). *Self-critiquing models for assisting human evaluators*. arXiv:2206.05802.

[16] Irving, G., et al. (2018). *AI safety via debate*. arXiv:1805.00899.

[17] Lamport, L., et al. (1982). *The Byzantine Generals Problem*. ACM TOPLAS.

> **Verification status (v0.4).** All references with full author attribution have been verified against primary sources. References [8], [9], [10] have arxiv IDs verified but full author lists are marked for verification — these are the new additions in v0.4 and the author list pulls from arxiv abstract pages prior to formal submission.

---

## Appendix A — Status & Call for Collaboration

- [x] Architectural proposal
- [x] Threat model including self-vulnerabilities (§4)
- [x] Falsifiable experimental design (§6) including hypotheses adverse to the proposal
- [x] References verified against primary sources (v0.3+)
- [x] Empirical context from third-party literature integrated (v0.4)
- [ ] Minimum viable empirical study (§6.6) — **seeking ~200–500 USD compute / API budget**
- [ ] Full ablation across tiers
- [ ] Cohen's κ measurement on cross-lab vs within-lab judge agreement (directly tests §4.3 + Yang et al. 2025)
- [ ] Peer review

Collaborators with API budget, GPU resources, or experience in adversarial benchmarking are explicitly invited.

Contact: jmfernandez@nomadprompters.com / [github.com/jmfernandezcof/sar-research](https://github.com/jmfernandezcof/sar-research)

---

## Appendix B — Heterogeneous Adversarial Review of This Paper

This paper has been progressively shaped by review from a small heterogeneous panel:

- **CC (Claude Code, Anthropic)** — review of v0.1 and v0.2. Identified missing positioning vs. Greenblatt 2023, TraceGuard, HAJailBench, DeepContext, M2S; flagged the formal weakness of R(t) ∝ 1/M(t); flagged the absence of empirical contribution; identified the unverified citation 90→60 and demanded reference verification before submission. v0.3 incorporated all critical and most medium-priority points.

- **Lex (Gemini Pro, Google)** — architectural critique of v0.2 + literature scan informing v0.4. Identified the Tier 0 prompt smuggling vector; identified the Tier 3 ensemble paradox; identified the operational maintenance burden under vendor API churn; identified the partial-independence problem between same-family-trained judges. The literature scan additionally surfaced iDecep, SmuggleBench, the Yang et al. retry-equivalence finding, and the *Automating Deception* cross-lab divergence — all incorporated in v0.4 after primary-source verification.

- **DeepSeek (DeepSeek-R1)** — *[review pending; will be incorporated in v0.5 if applicable]*

The author thanks the reviewers and notes — without irony — that this revision is itself a small instance of the heterogeneous multi-model review the paper proposes. The exercise generated more useful critique than a single-lab review would have. This is presented as anecdotal data, not validation.

---

## Appendix C — Material from External Review NOT Incorporated

In the spirit of transparency, this appendix documents material surfaced during external review that was evaluated and **rejected** before incorporation, and the reasons.

### C.1 Numerical SAR efficacy claims

The Lex literature scan included a comparative table reporting performance figures for SAR (e.g. 92.4% safety, 94.7% usefulness) and a TraceGuard SSFT variant (e.g. 95.1% safety, 96.2% usefulness, 6× speedup). **These figures were not generated by any experiment performed for this paper.** Their provenance could not be traced to a verifiable source within the literature scanned. They were rejected because:

- SAR has not been empirically evaluated. Reporting unmeasured figures as if they were results would replicate exactly the kind of opacity §6 is designed to avoid.
- The TraceGuard SSFT figures appear to mix two distinct papers titled "TraceGuard" — Nguyen et al. (arXiv:2604.03968, the relevant collusion-resistant control protocol) and an unrelated Washington University paper on reasoning backdoors (arXiv:2603.02436) which uses SSFT. Mixing them would reintroduce the disambiguation error v0.3 explicitly fixed.

### C.2 Specific Cohen's κ baselines

The literature scan included specific κ values attributed to particular models (e.g. Gemini 3.1 Pro κ=0.860, GPT-5.5 κ=0.966) presented as decision-stability baselines. **These specific figures were not located in the primary sources.** Cohen's κ values for LLM judges in the verified literature (e.g. arXiv:2510.09738, arXiv:2512.20352) are real but vary widely by task (from κ ≈ 0.07 in code judging to κ ≈ 0.85–0.91 in thematic analysis), without consensus baselines for the specific values cited. H6 in §6.5 has been formulated to test cross-lab vs within-lab κ ratios on the SAR task directly, rather than to compare against unverified prior figures.

### C.3 Some dramatised Mythos Preview claims

Specific narrative claims from the literature scan about Mythos Preview behaviours (e.g. specific exploit details, sandbox escape with publication-on-public-websites narrative) were not independently verified line-by-line against the primary Anthropic document. v0.4 cites the Mythos document at a higher level of abstraction, consistent with v0.3, to avoid propagating possible paraphrastic embellishment.

### C.4 SADCA (Semantic-Augmented Dynamic Contrastive Attack)

Mentioned in the literature scan but not located in the verification searches conducted for v0.4. Excluded pending direct citation. May appear in v0.5 if verified.

---

*Working paper. Not peer reviewed. Open critique welcome.*
*[github.com/jmfernandezcof/sar-research](https://github.com/jmfernandezcof/sar-research)*
