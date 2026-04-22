# MedLit AI — Biomedical Literature Mining Pipeline

> Automated extraction of genes, drugs, and clinical insights from PubMed papers using LLM-powered NLP.

---

## Overview

Biomedical researchers face an overwhelming volume of scientific literature. Manually reading and summarizing dozens of PubMed papers just to identify relevant genes, drug targets, and key findings is time-consuming and does not scale.

**MedLit AI** solves this by taking a disease name as input and automatically fetching the most relevant PubMed papers, extracting biological entities using an LLM, scoring papers by scientific richness and recency, and delivering a structured research report — reducing hours of manual review to seconds.

---

## Pipeline Architecture

```
User Input (Disease Name)
        │
        ▼
┌───────────────────────────────────────────┐
│  PubMed Fetch                             │
│  NCBI Entrez API · top 10 by relevance    │
└────────────────────┬──────────────────────┘
                     │
                     ▼
┌───────────────────────────────────────────┐
│  LLM Entity Extraction (per paper)        │
│  Groq API · qwen/qwen3-32b               │
│  → Genes · Drugs · 1-line Summary        │
└────────────────────┬──────────────────────┘
                     │
                     ▼
┌───────────────────────────────────────────┐
│  Relevance Scorer                         │
│  Entity density + publication recency     │
└────────────────────┬──────────────────────┘
                     │
                     ▼
┌───────────────────────────────────────────┐
│  Report Generator                         │
│  Top 5 papers · saved as <disease>.txt   │
└───────────────────────────────────────────┘
```

---

## How It Works

**1. Fetch** — PubMed is queried with the disease name combined with biomedical keywords (gene, protein, mutation, drug, therapy). Titles, abstracts, and publication years are parsed from the XML response.

**2. Extract** — Each paper's title and abstract are sent to `qwen/qwen3-32b` via Groq with a strict prompt that returns a JSON object containing gene symbols (e.g. `APP`, `PSEN1`), drug names (e.g. `Donepezil`), and a 1–2 sentence summary. A regex parser safely isolates the JSON block from the response.

**3. Score** — Each paper gets a composite relevance score: +2 per gene, +2 per drug, +3 if published within 2 years, +1 if within 5 years.

**4. Report** — The top 5 papers are written to a `.txt` file:

```
--- REPORT: ALZHEIMER---

1. The evolving landscape of Alzheimer's disease therapy: From Aβ to tau.
   Score: 9
   Genes: ['APP', 'PSEN1', 'MAPT']
   Drugs: []
   Summary: The paper discusses advancements in Alzheimer's disease therapy, highlighting the transition from amyloid β (Aβ) to tau-targeting approaches, progress in disease-modifying tau therapies, and future directions including combination therapies and targets beyond Aβ/tau.
```

---

## Tech Stack & Setup

| | |
|---|---|
| Language | Python 3.x |
| Literature Source | NCBI PubMed · Entrez API (BioPython) |
| LLM Provider | Groq API |
| LLM Model | `qwen/qwen3-32b` |
| LLM Framework | LangChain (`langchain-groq`) |

**Install dependencies**
```bash
pip install biopython langchain-groq python-dotenv
```

**Configure environment** — create a `.env` file:
```env
NCBI_EMAIL=your_email@example.com
GROQ_API_KEY=your_groq_api_key
```
Get a free Groq key at [console.groq.com](https://console.groq.com) · NCBI email registration at [ncbi.nlm.nih.gov](https://www.ncbi.nlm.nih.gov)

**Run**
```bash
python MedLit_AI.ipynb
# Enter disease name when prompted: Alzheimer
```
---

## Author

**Sanmti** · Bioinformatics & AI
📧 sanmti.bioinfo@gmail.com · GitHub: [@your-username](https://github.com/your-username)

---

## License

Released under the [MIT License](LICENSE) — free to use, modify, and distribute with attribution.
