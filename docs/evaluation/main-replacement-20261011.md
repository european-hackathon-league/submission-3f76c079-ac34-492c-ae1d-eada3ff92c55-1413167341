# Why main runs the `ksu-best` agent (2026-10-11, 10:30)

Same setup for every wave: all ten hosted practice cases in parallel (Live Final runner, `hollerau.cli run`),
gpt-5-mini on Azure with Hugging Face / Gemini backups, `tools/env_azure.sh`.

| Code | Wave mean | Zeros |
|---|---:|---|
| Ksu `a8705b9`..`08c5d10` (before PR #8) | 0.747 / 0.715 / 0.740 | none |
| PR #8 merge `15d0b94` + `592b809` | 0.477 | P04 (model proposal gave away 90 -> 30 days) |
| PR #8 + proposal filter `2e4cffd` | 0.600 | none; P01 effective date given away, P08 index alternative above the ask |
| `ksu-best` wave 1 (08c5d10 + hedge/opening rules) | 0.649 | P05 (expired budget, nothing sent - fixed) |
| **`ksu-best` wave 2** | **0.714** | none; P02 0.734 (was 0.37) |

`ksu-best` = `08c5d10` plus general rules from the scripted probes: fixed prices when the supplier's own letter
says its material is hedged; "price already raised" leads the opening; no package above the supplier's
standing price; an index clause whose deadband covers the expected index move opens the negotiation;
relationship openings at 97-97.5 % of the ask; a python fallback is still sent in the last seconds of the
arena window. PR #8 commits stay in history; its evaluation documents are kept in `docs/evaluation/`.
