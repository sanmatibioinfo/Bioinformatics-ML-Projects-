# Where Do LLMs Go Wrong in Pathway Analysis?
### A Benchmarking Study Across Autoimmune Diseases

---

## Overview

Pathway enrichment analysis is one of the most common steps in transcriptomics — you get a list
of differentially expressed genes and ask: what biology is actually happening here? Tools like
fgsea and ORA answer this statistically and are well validated. But increasingly, researchers are
also feeding gene lists directly into LLMs like ChatGPT and asking the same question, informally
and sometimes in published work.

Nobody has systematically tested how good this actually is.

This project benchmarks LLM-based pathway analysis against fgsea and ORA on real gene expression
data from two autoimmune diseases — Lupus (SLE) and Multiple Sclerosis. The goal is not just to
measure accuracy, but to understand the *type* of errors LLMs make: do they hallucinate pathways,
confuse disease signatures, or return results that are technically correct but biologically
uninformative?

llm model was compared — **grok** (general-purpose reasoning) against **Fgsea** and **ORA**

---

## Pipeline Architecture

```
  GEO Datasets (Lupus · MS)
           │
           ▼
  Preprocessing + DESeq2
  (top 200 DEGs per disease)
           │
     ┌─────┼─────┐
     ▼     ▼     ▼
   fgsea  ORA   LLM
  (ranked) (ORA) (Grok<img width="960" height="768" alt="Sanity_Score" src="https://github.com/user-attachments/assets/6a8d914b-fb4b-44bf-8071-85ecd9ce55d0" />
)
     └─────┼─────┘
           │
           ▼
     Benchmarking Layer
  Jaccard · Precision · Recall
  (fgsea treated as ground truth)
           │
           ▼
   Failure Mode Analysis
   Hallucination · Disease confusion
   Ranking failure · Vagueness
           │
           ▼
  Biological Sanity Layer
  Gold standard pathway recall
  (literature-confirmed pathways)
           │
           ▼
  Cross-Disease Robustness
  Blinded vs informed LLM test
```

---

## How It Works

### 1. Differential Expression
Expression matrices for Lupus (GSE72326, 996 samples) and MS (GSE41850) are downloaded from GEO.
DESeq2 is run on each dataset to identify differentially expressed genes between disease and
healthy control samples. The top 200 genes ranked by effect size and significance are taken
forward for pathway analysis.

### 2. Parallel Pathway Analysis
The same gene list is analyzed by three methods in parallel:

- **fgsea** — ranked gene set enrichment against MSigDB Hallmark gene sets. Used as the
  statistical ground truth throughout because it uses the full gene ranking, not just a cutoff.
- **ORA** — overrepresentation analysis using `clusterProfiler` against GO Biological Process
  terms. A simpler, threshold-based approach included as a classical baseline.
- **LLM** — top 100 gene symbols sent to grok via API. The model is asked to
  return the top 10 enriched pathways in structured JSON with confidence levels and supporting
  genes. Each call is repeated 3 times to measure output consistency.

### 3. Benchmarking
Each method is scored against fgsea using Jaccard similarity (pathway set overlap), Spearman
rank correlation (agreement on pathway importance), precision and recall. This is done separately
for Lupus and MS — differences between diseases are part of the result.

### 4. Failure Mode Analysis
LLM errors are categorized into four types:

| Failure Type | Description |
|---|---|
| Hallucination | Pathways named by LLM that do not exist in MSigDB or literature |
| Disease confusion | MS-specific terms appearing in Lupus analysis and vice versa |
| Ranking failure | Correct pathways returned in wrong order of importance |
| Vagueness | Real but generic terms (e.g. "immune response") that add no analytical value |

### 5. Biological Sanity Layer
Statistical benchmarking against fgsea only tells you consistency with one method. This layer
scores each method against a manually curated gold standard — pathways that are independently
confirmed in the primary literature for each disease. A method can score poorly against fgsea
but still recover the right biology, and vice versa. That gap is one of the key findings.

<p align="center"><img width="398" height="312" alt="image" src="https://github.com/user-attachments/assets/9c4c7403-0a97-4a7d-b883-8ccfee639144" /></p>

**Lupus gold standard:** Type I interferon signaling · Complement activation · NETosis · B cell dysregulation

**MS gold standard:** T cell immunity · Myelin degradation · Oligodendrocyte apoptosis · Neuroinflammation

### 6. Cross-Disease Robustness
The LLM is run twice on the same gene list — once told the disease name, once given only
"autoimmune disease, unspecified." The drop in biological sanity score between informed and
blinded conditions reveals how much the model is reasoning from the gene data versus reciting
what it already knows about the disease.

---

## Tech Stack & Setup

**Language:** R

**Key packages:**

| Purpose | Package |
|---|---|
| Data download | `GEOquery` |
| Differential expression | `DESeq2` |
| Pathway enrichment | `fgsea`, `clusterProfiler`, `msigdbr` |
| Gene annotation | `org.Hs.eg.db` |
| Visualization | `ggplot2`, `pheatmap` |
| API calls + parsing | `httr`, `jsonlite` |

**Install:**
```r
BiocManager::install(c("GEOquery", "DESeq2", "fgsea",
                       "clusterProfiler", "msigdbr", "org.Hs.eg.db"))

install.packages(c("tidyverse", "ggplot2", "pheatmap", "jsonlite", "httr"))
```

**API keys** (set before running `R/06_llm_calls.R`):
```r
Sys.setenv(GROQ_API_KEY = "your-key-here")

```

**Data:** Not included. Download GSE72326 and GSE41850 from
[NCBI GEO](https://www.ncbi.nlm.nih.gov/geo/) or run `R/01_data_download.R`.



---

## Author
**Sanmati Ganesh** <br>
sanmati.bioinfo@gmail.com <br>
https://www.linkedin.com/in/sanmati-ganesh-701008273


---

## License
MIT
