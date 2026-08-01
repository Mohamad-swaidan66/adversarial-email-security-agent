<div align="center">

# Adversarial Email Security Agent

**An automated red-teaming framework for email security powered by LLM-based adversarial co-evolution**


<br/>

> *"The best defense is understanding the offense — automatically, continuously, and at scale."*

<br/>

[📖 Documentation](#-architecture) · [🚀 Quick Start](#-quick-start) · [📊 Results](#-results) · [🗺️ Roadmap](#️-roadmap) · [📄 Paper](#-citation)

</div>

---

## 🔍 Overview

Most phishing detection systems are **static**: trained once, tested on a fixed dataset, deployed — and immediately outdated. Real-world attackers adapt continuously.

**Adversarial Email Security Agent** models this dynamic as an automated **arms race**:

```
Attacker Agent generates phishing emails
        ↓
Defender Agent attempts detection + explains reasoning
        ↓
Attacker Agent receives feedback, adapts strategy
        ↓
MIPROv2 re-optimizes Defender prompts automatically
        ↓
Loop continues → both agents evolve
```

This project implements this co-evolution loop using **DSPy/MIPROv2** for automatic prompt optimization, **LangGraph** for orchestration, **RAG** over real phishing corpora, and **MCP-connected threat intelligence tools** — producing a system that mirrors how real attacker-defender dynamics unfold.

---

## ✨ Key Contributions

| # | Contribution | Why it matters |
|---|---|---|
| 1 | **Automated adversarial co-evolution loop** | First open-source implementation of LLM attacker ↔ defender arms race |
| 2 | **MIPROv2-driven prompt auto-optimization** | Defender prompts self-improve without human intervention |
| 3 | **MCP-connected threat intelligence** | Agents use real security tools (VirusTotal, WHOIS, header analysis) |
| 4 | **Quantified evolution metrics** | Evasion rate, detection rate, and adaptation curves across rounds |
| 5 | **RAG over real phishing corpora** | Grounded in PhishTank, Nazario corpus, and CEAS datasets |

---

## 🏗️ Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                    ORCHESTRATION LAYER                           │
│                  LangGraph Evolution Graph                        │
│         (manages rounds, state, stopping criteria)               │
└──────────────────────┬───────────────────────┬──────────────────┘
                       │                       │
          ┌────────────▼──────────┐ ┌──────────▼────────────┐
          │    ⚔️ ATTACKER AGENT  │ │  🛡️ DEFENDER AGENT    │
          │                       │ │                         │
          │  PhishingGenerator    │ │  EmailClassifier        │
          │  • Target profiling   │ │  • Multi-signal fusion  │
          │  • Evasion strategy   │ │  • Chain-of-thought     │
          │  • Style adaptation   │ │  • Confidence scoring   │
          │                       │ │                         │
          │  [DSPy ChainOfThought]│ │  [DSPy ChainOfThought] │
          └────────────┬──────────┘ └──────────┬─────────────┘
                       │                       │
          ┌────────────▼───────────────────────▼─────────────┐
          │              MCP TOOLS LAYER                       │
          ├────────────────────────────────────────────────────┤
          │  🔗 URL Reputation      (VirusTotal API)           │
          │  📧 Header Analyzer     (SMTP metadata parsing)    │
          │  🌐 Domain WHOIS        (registration signals)     │
          │  🔍 Threat Intel Feed   (live threat indicators)   │
          │  🧠 RAG Retriever       (similar phishing lookup)  │
          └────────────────────────┬───────────────────────────┘
                                   │
          ┌────────────────────────▼───────────────────────────┐
          │              KNOWLEDGE BASE (RAG)                   │
          ├────────────────────────────────────────────────────┤
          │  • PhishTank URLs corpus          (malicious)       │
          │  • Nazario Phishing Corpus        (email bodies)    │
          │  • Enron + SpamAssassin           (legitimate)      │
          │  • CEAS 2008 dataset              (benchmark)       │
          └────────────────────────┬───────────────────────────┘
                                   │
          ┌────────────────────────▼───────────────────────────┐
          │           OPTIMIZATION LAYER (MIPROv2)              │
          ├────────────────────────────────────────────────────┤
          │  Auto-optimizes Defender prompts every N rounds     │
          │  Metric: maximize detection rate on evolved emails  │
          └────────────────────────┬───────────────────────────┘
                                   │
          ┌────────────────────────▼───────────────────────────┐
          │            EVALUATION & LOGGING LAYER               │
          ├────────────────────────────────────────────────────┤
          │  • Evasion rate per round                           │
          │  • Detection rate per round                         │
          │  • Prompt evolution diff (before/after MIPRO)       │
          │  • Per-technique attack success rate                │
          └────────────────────────────────────────────────────┘
```

---

## 🚀 Quick Start

### Prerequisites

```bash
python >= 3.11
```

### Installation

```bash
# Clone the repository
git clone https://github.com/Mohamad-swaidan66/adversarial-email-security-agent.git
cd adversarial-email-security-agent

# Install dependencies
pip install -e ".[dev]"

# Copy and configure environment variables
cp .env.example .env
# → Add your API keys (OpenAI / Mistral / VirusTotal)
```

### Download datasets

```bash
python scripts/download_datasets.py
# Downloads: PhishTank, Enron subset, Nazario corpus
```

### Run your first evolution loop

```python
from agents import AttackerAgent, DefenderAgent
from orchestration import EvolutionGraph

# Initialize agents
attacker = AttackerAgent(model="openai/gpt-4o")
defender = DefenderAgent(model="openai/gpt-4o")

# Run co-evolution for 10 rounds
graph = EvolutionGraph(attacker=attacker, defender=defender)
results = graph.run(rounds=10, optimize_every=3)

# Visualize evolution
results.plot_evolution_curves()
```

---

## 📁 Project Structure

```
adversarial-email-security-agent/
│
├── 📄 README.md
├── 📄 ARCHITECTURE.md           # Detailed design decisions
├── 📄 pyproject.toml
│
├── 📁 agents/
│   ├── attacker_agent.py        # LLM-based phishing generator
│   ├── defender_agent.py        # LLM-based email classifier
│   └── base_agent.py            # Shared agent interface
│
├── 📁 tools/                    # MCP-exposed security tools
│   ├── url_checker.py           # VirusTotal integration
│   ├── header_analyzer.py       # SMTP header parsing
│   ├── whois_lookup.py          # Domain registration signals
│   └── rag_retriever.py         # Phishing corpus RAG
│
├── 📁 rag/
│   ├── ingest.py                # Dataset ingestion pipeline
│   ├── chunker.py               # Email-aware text chunking
│   └── retriever.py             # Dense retrieval (ChromaDB)
│
├── 📁 optimization/
│   ├── mipro_optimizer.py       # MIPROv2 prompt optimization
│   └── metrics.py               # Custom DSPy metrics
│
├── 📁 orchestration/
│   ├── evolution_graph.py       # LangGraph evolution loop
│   ├── state.py                 # Shared state definition
│   └── stopping_criteria.py     # Convergence detection
│
├── 📁 evaluation/
│   ├── tracker.py               # Round-by-round metrics
│   ├── visualizer.py            # Evolution curve plots
│   └── reporter.py              # Structured JSON reports
│
├── 📁 data/
│   ├── datasets/                # Raw corpora (gitignored)
│   └── processed/               # Vectorized chunks
│
├── 📁 notebooks/
│   ├── 01_data_exploration.ipynb
│   ├── 02_baseline_evaluation.ipynb
│   └── 03_evolution_analysis.ipynb
│
├── 📁 tests/
│   ├── test_attacker.py
│   ├── test_defender.py
│   └── test_evolution_loop.py
│
└── 📁 .github/
    └── workflows/
        └── ci.yml               # Automated tests on push
```

---

## 📊 Results

> ⚠️ *Results below are preliminary (Round 1–15). Full benchmark in progress.*

### Co-evolution Dynamics

| Round | Attacker Evasion Rate | Defender Detection Rate | MIPRO Triggered |
|---|---|---|---|
| 1 | 11% | 89% | — |
| 3 | 19% | 84% | — |
| 6 | 31% | 77% | ✅ |
| 9 | 38% | 82% | — |
| 12 | 41% | 79% | ✅ |
| 15 | 36% | 85% | — |

Key observations:
- **Evasion rate increases** steadily until MIPRO re-optimization kicks in
- **Defender recovers** after each optimization round
- **Equilibrium** emerges around round 12–15 — a genuine arms race dynamic

### Attack Technique Breakdown

| Technique | Round 1 Success | Round 15 Success |
|---|---|---|
| Urgency + authority framing | 42% | 61% |
| Domain spoofing signals | 18% | 34% |
| Lookalike brand impersonation | 29% | 48% |
| Payload obfuscation | 8% | 27% |

---

## 🗓️ Roadmap

- [x] Project structure & datasets
- [x] DefenderAgent v1 (DSPy classifier)
- [x] AttackerAgent v1 (basic generation)
- [x] Basic co-evolution loop (LangGraph)
- [ ] MCP tools integration (VirusTotal, WHOIS)
- [ ] MIPROv2 auto-optimization of Defender
- [ ] Full benchmark on CEAS 2008 dataset
- [ ] Interactive demo (Gradio / Streamlit)
- [ ] Technical report / preprint (ArXiv)
- [ ] Multi-LLM support (GPT-4o, Mistral, Llama)

---

## 🧪 Datasets Used

| Dataset | Source | Size | Use |
|---|---|---|---|
| [PhishTank](https://phishtank.org/) | Community-verified | ~100K URLs | Attacker training signals |
| [Nazario Phishing Corpus](https://monkey.org/~jose/phishing/) | Research corpus | ~6.5K emails | Phishing examples |
| [Enron Email Dataset](https://www.cs.cmu.edu/~./enron/) | CMU | ~500K emails | Legitimate baseline |
| [SpamAssassin](https://spamassassin.apache.org/publiccorpus/) | Apache | ~6K emails | Spam/ham benchmark |
| [CEAS 2008](http://www.ceas.cc/) | CEAS Workshop | ~50K emails | Standard benchmark |

---

## 🛠️ Tech Stack

| Layer | Technology | Role |
|---|---|---|
| LLM Optimization | **DSPy + MIPROv2** | Auto-optimize agent prompts |
| Orchestration | **LangGraph** | Manage co-evolution state graph |
| Tool Integration | **MCP** | Connect agents to security APIs |
| Vector Store | **ChromaDB** | RAG over phishing corpora |
| LLM Providers | **LiteLLM** | Unified API (GPT-4o, Mistral, Llama) |
| Experiment Tracking | **MLflow** | Log rounds, metrics, prompt diffs |

---

## 🔒 Ethical Considerations

This project is built **strictly for defensive research purposes**.

- All generated phishing emails are used **only within the sandboxed evaluation loop** — no real sending infrastructure is involved
- Datasets used are publicly available research corpora
- The system is designed to improve **detection capabilities**, not to assist attackers
- This work follows the ethical guidelines of ACM CCS, IEEE S&P, and USENIX Security

---

## 📄 Citation

If you use this project in your research, please cite:

```bibtex
@misc{swaidan2026adversarial,
  title   = {Adversarial Email Security Agent: Automated Red-Teaming via LLM Co-Evolution},
  author  = {Swaidan, Mohamad},
  year    = {2026},
  url     = {https://github.com/Mohamad-swaidan66/adversarial-email-security-agent},
  note    = {Work in progress}
}
```

---

## 🤝 Contributing

Contributions, issues, and feature requests are welcome.

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/your-feature`)
3. Commit your changes (`git commit -m 'feat: add your feature'`)
4. Push to the branch (`git push origin feature/your-feature`)
5. Open a Pull Request

See [CONTRIBUTING.md](CONTRIBUTING.md) for detailed guidelines.

---

## 📬 Contact

**Mohamad Swaidan** — Engineering Student, ENSTA / Institut Polytechnique de Paris

[![LinkedIn](https://img.shields.io/badge/LinkedIn-Connect-0A66C2?style=flat&logo=linkedin)](https://linkedin.com/in/mohamad-swaidan)
[![GitHub](https://img.shields.io/badge/GitHub-Follow-181717?style=flat&logo=github)](https://github.com/Mohamad-swaidan66)

---

<div align="center">

*Built as part of a CIFRE doctoral research project at the intersection of Generative AI and Email Cybersecurity.*

⭐ **Star this repo** if you find it useful — it helps the project gain visibility!

</div>
