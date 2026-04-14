# SAR
## Stateless Adversarial Review
**Author:** Jose Maria Fernandez Gomez
**Date:** April 2026

## Abstract
Agentic AI monitoring fails in two ways: collusion risk (same-lab monitors share blind spots) and iterative attack vulnerability (detection degrades from ~90% to ~60% over 10 rounds). SAR proposes stateless heterogeneous judges. Central hypothesis: R(t) proportional to 1/M(t).

## Architecture
T0: Summary Agent (stateless) / T1: Llama Guard / T2: Fragment Judges x3 (Claude+GPT4o+Gemini) / T3: Composed Review / T4: Audit Layer

## Hypothesis
R(t) prop 1/M(t). Stateless judges: M(t)=0, R constant. Heterogeneity: joint FNR = product of individual FNRs.

## Limitations
Sandbagging, long-horizon coordination, weight poisoning, orchestrator compromise - all out of scope.

## License
CC BY 4.0