# Challenge v1.0.0 design

Challenge is an **independent discriminative track** layered on top of the longitudinal Core set. Core is retained unchanged; Challenge does not replace it.

## Composition

Challenge v1.0.0 contains 80 newly authored short-context items, exactly 8 in each Core category:

| Category | Items |
|---|---:|
| advanced_japanese | 8 |
| analytical_reasoning | 8 |
| mathematics_statistics | 8 |
| coding | 8 |
| physics_engineering | 8 |
| chemistry_materials | 8 |
| life_information | 8 |
| humanities_social_economics | 8 |
| hallucination_resistance | 8 |
| instruction_following | 8 |
| **Total** | **80** |

Grader distribution:

| Grader | Items |
|---|---:|
| numeric | 35 |
| choice | 21 |
| json_exact | 16 |
| python_unit_test | 8 |
| **Total** | **80** |

## Difficulty philosophy

The track is designed to be harder without consuming long context. Difficulty is increased primarily by:

- multi-step inference rather than longer prose;
- necessary/sufficient-condition and counterfactual distinctions;
- causal-identification traps and adversarial distractors;
- coupled numerical reasoning and unit/constraint handling;
- coding edge cases, state augmentation, dynamic programming, graph algorithms and parsing;
- scientific problems requiring more than one transformation or physical assumption;
- epistemic-discipline tasks that separate supported, partially supported, underdetermined and invalid causal claims;
- instruction-following tasks with several simultaneous constraints and cross-field consistency.

The intended use is to reduce ceiling effects among modern ~30B reasoning models. **No target accuracy is guaranteed in advance.** A nominal 40–70% range is a calibration goal, not a grading threshold. The dataset must not be edited after inspecting model-specific results merely to increase separation; any future calibration change requires a new dataset version and hash.

## Execution

`./run.sh challenge` runs Challenge only under the same canonical 8K / 16:7 / all-GPU / F16-KV controlled profile and skips Core, reasoning-effort reruns, performance probes and long-context probes.

Challenge uses medium reasoning effort and the same deterministic sampling profile as Core.
