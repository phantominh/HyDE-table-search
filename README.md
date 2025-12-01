# HyDE-table-search

## Summary

This project extends HyDE (Hypothetical Document Embeddings) to table search, enabling zero-shot table retrieval without domain-specific labeled data.

**Problem**: Dense retrieval methods achieve strong performance but typically require massive labeled datasets (hundreds of thousands of query-document pairs) to train effectively. Creating such labels is expensive and time-consuming for new domains or specialized applications.

**Approach**: Instead of directly encoding queries, an instruction-following LLM generates hypothetical tables that answer the query. These generated tables are encoded using pre-trained contrastive encoders, and real corpus tables are retrieved based on embedding similarity to the hypothetical table.

**Hypothesis**: Generating hypothetical tables will improve zero-shot table retrieval performance compared to directly encoding queries.

**Dataset**: WikiTables - a curated subset from the full WikiTables dataset (1.6M tables). The subset is from the "Semantic Search for Tables" paper, where 5 different search methods were used to filter tables before human annotation.

**Team**: Skylar Hou, Abigale Kim, Minh Phan, Mark Tervo (CS839 Project, November 2025)

## Setup

### Installation

```bash
pip install -r requirements.txt
```

### Project Structure

```
HyDE-table-search/
├── data/
│   ├── wikitables_mini.csv         # 3K WikiTables subset
│   ├── queries.txt                 # Search queries
│   └── qrels.txt                   # Relevance judgments
├── baseline.ipynb                  # Baseline: direct query encoding
├── hyde.ipynb                      # HyDE: LLM-generated descriptions
└── requirements.txt                # Python dependencies
```

## Method Overview

Both baseline and HyDE follow the same 4-step pipeline:

### Step 1: Generate Table Descriptions

Convert each table's metadata into a text description.

- **Baseline**: Simple concatenation of metadata fields (caption, page title, section, headers, sample data)
- **HyDE**: Use LLM to generate natural description from metadata

### Step 2: Embed and Build Index

Encode all table descriptions using MiniLM-L6-v2 (384-dim embeddings) and build a FAISS flat index for exact nearest neighbor search.

### Step 3: Query Processing and Search

Process query and retrieve top-k similar tables.

- **Baseline**: Directly embed the search query
- **HyDE**: Use LLM to generate a hypothetical table description from the query, then embed that description

The key insight of HyDE: instead of embedding "world interest rates table", embed a description like "A table showing interest rates by country with columns for country name, rate percentage, and year."

### Step 4: Evaluation

Compare retrieved tables against ground-truth relevance judgments using:

**Recall@k**: Measures retrieval coverage
- Formula: (# of relevant tables in top-k) / (total # of relevant tables)
- Range: 0 to 1 (higher is better)
- Example: If there are 5 relevant tables and we find 3 in top-10, Recall@10 = 3/5 = 0.6

**nDCG@k** (Normalized Discounted Cumulative Gain): Measures ranking quality
- Accounts for both relevance scores (0, 1, 2) and ranking position
- Formula: DCG@k / IDCG@k, where highly relevant tables ranked higher get more credit
- Range: 0 to 1 (higher is better)
- Rewards retrieving highly relevant tables at top positions

## Usage

### 1. Baseline Retrieval

Open and run [baseline.ipynb](baseline.ipynb):

- Concatenates table metadata into text descriptions
- Encodes queries directly
- Retrieves using FAISS + MiniLM-L6-v2
- Evaluates with Recall@k and nDCG@k

### 2. HyDE Retrieval

Open and run [hyde.ipynb](hyde.ipynb):

- Uses LLM to generate table descriptions from metadata
- Uses LLM to generate hypothetical table descriptions from queries
- Retrieves using FAISS + MiniLM-L6-v2
- Evaluates with Recall@k and nDCG@k

**Note**: HyDE notebook has TODO comments indicating where to add LLM API calls.

