# AI-Drone-Product-Regulation-Assistant
A RAG-based AI assistant for DJI product consultation and UK drone regulation Q&A.

This project explores how retrieval, reranking and grounded generation can improve the reliability of an AI assistant when answering questions that depend on **current product information and regulatory documents**.

---

## 1. Project Overview

The system is designed for a practical user scenario:

> A user wants to choose a DJI drone in the UK and needs to understand both product specifications and relevant flying regulations.

This is a suitable RAG use case because product specifications and regulations can change over time. A general-purpose LLM may rely on outdated knowledge or generate unsupported details.

The knowledge base therefore uses publicly available information from:

- DJI official product documentation
- UK Civil Aviation Authority (CAA) drone regulations

The original source documents are not redistributed in this repository.

---

## 2. System Design

Two pipelines were implemented and compared.

### Baseline RAG

```text
User Query
    ↓
Query Embedding
    ↓
FAISS Vector Retrieval
    ↓
Top-k Chunks
    ↓
Basic Prompt + Local LLM
    ↓
Answer
```

The baseline system uses semantic similarity to retrieve relevant document chunks.

### Enhanced RAG

```text
User Query
    ↓
Hybrid Retrieval
(Vector Search + BM25)
    ↓
Candidate Chunks
    ↓
Cross-Encoder Reranking
    ↓
Top-ranked Evidence
    ↓
Grounded Prompt + Local LLM
    ↓
Answer
```

The enhanced pipeline introduces three improvements:

**Hybrid Retrieval** combines semantic search with keyword matching. This is useful because the knowledge base contains both natural-language explanations and exact terms such as product names, regulatory categories and weight limits.

**Cross-Encoder Reranking** re-evaluates retrieved candidates against the user query so that the most relevant evidence is prioritised before generation.

**Grounded Prompting** instructs the model to answer from retrieved evidence and avoid unsupported claims when the available information is insufficient.

---

## 3. Implementation

The system processes DJI and CAA documents into overlapping text chunks while retaining metadata such as source file and page number for traceability.

Key components include:

- **Embedding:** `all-MiniLM-L6-v2`
- **Vector Store:** FAISS
- **Keyword Retrieval:** BM25
- **Reranking:** Cross-Encoder
- **Generation:** `Qwen2.5-1.5B-Instruct`
- **Document Processing:** Python / PyMuPDF

Using a local open-source generation model also reduces dependence on external APIs.

---

## 4. Evaluation

Evaluation was designed at two levels because good retrieval does not necessarily guarantee a good final answer.

### Retrieval Evaluation

The test set contains 9 queries covering:

- product selection
- product comparison
- simple regulatory facts
- product + regulation questions
- compliance responsibilities
- ambiguous flying scenarios
- missing-information edge cases
- pre-flight safety questions

Retrieval performance was assessed using:

- Keyword Hit Rate
- Precision@K
- manual inspection of retrieved chunks, sources and page numbers

### Generation Evaluation

Representative answers were manually evaluated across:

- **Correctness** — Is the answer factually accurate?
- **Grounding** — Are the claims supported by retrieved evidence?
- **Completeness** — Does the answer address the important parts of the question?

---

## 5. Key Results

The enhanced pipeline did **not** outperform the baseline on every retrieval metric.

| Pipeline | Average Keyword Hit Rate |
|---|---:|
| Baseline | 0.752 |
| Enhanced | 0.696 |

However, keyword hit rate alone did not fully reflect answer quality.

For compliance and scenario-based questions, the enhanced system often retrieved evidence that was more closely aligned with the user's actual intent.

Generation results for three representative queries were:

| Query | Pipeline | Correctness | Grounding | Completeness | Overall |
|---|---|---:|---:|---:|---:|
| Q1 | Baseline | 7 | 8 | 4 | 6.3 |
| Q1 | Enhanced | 3 | 2 | 5 | 3.3 |
| Q5 | Baseline | 6 | 6 | 5 | 5.7 |
| Q5 | Enhanced | 8 | 8 | 8 | 8.0 |
| Q7 | Baseline | 7 | 6 | 7 | 6.7 |
| Q7 | Enhanced | 8 | 7 | 8 | 7.7 |

The enhanced pipeline performed better for Operator ID and location-related compliance questions, but Q1 exposed an important failure mode.

---

## 6. Failure Analysis & Product Insight

In Q1, the enhanced pipeline retrieved relevant UK regulatory information, but the generated answer introduced DJI model and weight details that were not sufficiently supported by the retrieved evidence.

Interestingly, the system behaved better when information was **completely absent**. For a query asking about drone motor repair, it correctly recognised that the documents did not contain the required information instead of inventing repair instructions.

This suggests an important distinction:

```text
No relevant evidence
→ relatively easy to refuse

Partially relevant evidence
→ higher risk of unsupported completion
```

The main lesson from the project is therefore:

> **Better retrieval does not automatically produce a more reliable AI product.**

A production-ready RAG system also needs to determine whether the retrieved evidence is **sufficient** to support the final answer.

Potential improvements include:

- evidence-sufficiency thresholds
- stronger refusal mechanisms
- claim-level citation checking
- post-generation answer verification
- broader bad-case evaluation
- regular knowledge-base updates

---

## 7. Tech Stack

`Python` · `FAISS` · `BM25` · `Sentence Transformers` · `Cross-Encoder` · `Qwen2.5` · `PyMuPDF` · `Pandas`

---

## 8. Repository Structure

```text
drone-rag-assistant/
├── README.md
├── RAG_drone_code.ipynb
├── demo_log.pdf
└── .gitignore
```

The source DJI and CAA documents are publicly available but are not included in this repository.

---

## 9. Project Takeaway

This project started as an attempt to improve a basic RAG pipeline through hybrid retrieval, reranking and grounded prompting.

The evaluation showed that system quality cannot be judged by retrieval metrics alone. The more important challenge is whether the system can recognise when its evidence is incomplete and prevent unsupported claims from reaching the user.

For AI product design, this means evaluation should cover not only **whether the system can answer**, but also **when it should not answer**.
