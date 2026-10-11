# 🤝 += · Tacto Negotiation Agent

**Autonomous procurement agent for Hollerau Pumpenwerke** — EHL Munich 2026, Tacto track
Team **+=** · [ziminaksu](https://github.com/ziminaksu) · [IlyaKats2005](https://github.com/IlyaKats2005)

![python](https://img.shields.io/badge/python-3.12-blue) ![runner](https://img.shields.io/badge/runner-uv-purple) ![model](https://img.shields.io/badge/LLM-gpt--5--mini%20%2B%20fallbacks-green) ![practice](https://img.shields.io/badge/practice-0.827%20·%203rd-orange)

The agent negotiates by e-mail with supplier bots and closes the deal with the **lowest total cost** for the
buyer — price, payment terms, freight, lateness, quality, index clauses and, where the goal requires it, the
cost of squeezing a supplier below a sustainable margin.

---

## ⚡ Quick start

```bash
uv sync --all-extras          # Python 3.12, https://docs.astral.sh/uv/
cp .env.example .env          # fill in the variables below (never commit .env)
uv run arena data             # case data + the Sunday release
uv run arena me               # checks the arena key
./tools/final_azure.sh        # LIVE FINAL: five waves, 10 cases in parallel, log in hollerau/logs/
```

One practice case: `./tools/run_azure.sh P08` · free alternative: `./tools/run_free.sh P08`

### Environment variables (`.env`)

| Variable | Needed for |
|---|---|
| `ARENA_URL`, `ARENA_API_KEY`, `ARENA_DATA` | arena access and the data package (`data/package`) |
| `AZURE_OPENAI_ENDPOINT`, `AZURE_OPENAI_DEPLOYMENT`, `AZURE_OPENAI_KEY` | primary decision model (gpt-5-mini) |
| `GEMINI_API_KEY` | backup model (free tier) |
| `HF_TOKEN` | optional backup models (gpt-oss-120b, DeepSeek-V4.1) |
| `OPENAI_API_KEY` | optional: `./tools/run_gpt6.sh` with gpt-6-astra and a hard spend cap |

**No model credits? The agent still plays.** Every model call walks a chain — Azure → Hugging Face →
Gemini — and moves on at any provider error (rate limit, exhausted credits, timeout, invalid answer). If no
provider answers, the move is taken from the validated board by code, so a session never stops on an API
problem.

---

## 🧠 How the agent works

1. **Read the case like a buyer.** `hollerau/intel.py` turns the data package into *checked* arguments, each
   with its ID: index moves, our order history, should-cost, delivery and quality records, rival quotes — and
   the traps hidden in contracts and notes:
   - the supplier asks for an index-driven increase although the **framework agreement already indexes the
     price** (`ctr:…#3.2`);
   - its own letter says the **material is bought at fixed prices** (then no index clause);
   - it already **raised the price this year**, its **own recent quote** is lower, the owner **offered to open
     the calculation**, sales **focuses on volume customers**, a **pricing committee** handles larger moves.
2. **Build the board in Python.** Every possible package is priced with the published cost function
   (`arena_kit/cost.py`) and filtered by must-haves and negotiation rules.
3. **The LLM chooses** the move, the packages and at most two arguments per e-mail, and writes the text.
4. **Python checks and sends.** Exact offer IDs, must-haves, only verified figures in the text.

### Negotiation rules (each one comes from a traced loss)

| Rule | Why |
|---|---|
| Open from our data: ~86 % of the ask; 97–97.5 % when the goal is `stabilise` / `secure_supply` | concessions follow the anchor; deep cuts cost a margin penalty in relationship cases |
| Concede on price only; payment and contract never shorter than our last offer | every record kept 90 days / 36 months |
| Concession schedule: ¼ of the gap, then ⅓, then ½ | crawling steps trigger an early final offer |
| Longest contract that covers the horizon | free for us, valuable to the supplier |
| Index clause only if its deadband covers the expected index move | otherwise it costs the whole move |
| Second counter asks for the supplier's management | the largest single concession in our probes |
| RFQ: refresh all threads before each move; accept a competitive final offer at once; never walk away from an offer better than the alternative | while one supplier answers, the others time out |

### Reliability

- One 55 s budget per move (arena limit 60 s); a code fallback is still sent in the last seconds.
- Any failed turn becomes a fallback move from the current board — never silence, never a crashed case.
- Runner hardening: stale offer IDs re-checked, timeout replies picked up, malformed decisions replaced by the
  safest closing move.

---

## 📊 Results (practice)

| P01 | P02 | P03 | P04 | P05 | P07 | P08 | P10 | P11 | P18 | **Mean** |
|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| 0.967 | 0.735 | 0.974 | 0.860 | 0.875 | 0.792 | 0.870 | 0.686 | 0.794 | 0.721 | **0.827** |

Score = share of the possible cost improvement captured (1.0 = arena optimum). Practice leaderboard: **3rd**.
Last full wave in the Live Final setup (ten cases in parallel): **10/10 deals, no late move, no session
timeout**. Every batch, failures included, is in [`docs/evaluation/`](docs/evaluation/).

---

## 🗂️ Repository

| Path | Content |
|---|---|
| `hollerau/analytical/` | the agent — `director.py` board & rules, `router.py` model chain, `agent.py` turn loop & fallbacks |
| `hollerau/intel.py` | checked arguments from the data package |
| `arena_kit/` | arena protocol, runner (`play.py`), published cost function |
| `tools/` | launchers (`final_azure.sh`, `run_azure.sh`, `run_free.sh`, `run_gpt6.sh`) and the scripted probe |
| `docs/` | architecture, decision contract, evaluations |
| `tests/` | `uv run pytest -q` |

Sessions are recorded with [Entire](https://hack.entire.io/ehl-munich-2026/) (`git-refs` checkpoints,
credentials redacted). Starter-kit code: `LICENSE-kit` (MIT). Data: `LICENSE-data` (CC BY 4.0).
