# Legal/RTI Document Assistant

**Corrective RAG + GraphRAG pipeline for accurate, citation-grounded answers on RTI law**

An AI-powered assistant that helps citizens navigate the Right to Information (RTI) Act, state RTI rules, court judgments, and CIC decisions — without the hallucinations that plague typical LLM-based legal Q&A tools.

Built during an AI/ML Product Engineering internship at **Vedas Tech** (9 June 2026 – 10 August 2026).

---

## Why this exists

RTI law in India is scattered across the Central Act, state-specific rules, Delhi High Court judgments, and CIC decisions. Legal language is dense, and manually tracing which precedent applies to which clause is slow and error-prone. Off-the-shelf LLMs answer confidently but often invent citations or misquote sections.

This project indexes the source documents properly, retrieves at clause-level precision, and verifies every answer against its sources before returning it — so answers come with real citations, and clauses are linked to the case law that interprets them.

## Features

- **Section/clause-level retrieval** — not just "which document," but which exact sub-clause (e.g. Section 8(1)(j))
- **Hybrid search** — combines keyword matching (BM25) with semantic vector search for better recall
- **Corrective RAG loop** — a self-grading step re-retrieves when the first pass looks weak, instead of answering from poor context
- **Hallucination / faithfulness check** — every generated answer is checked against its source chunks before being returned, with retry-on-fail
- **Citation graph (GraphRAG)** — links Act clauses to the judgments and CIC decisions that cite them, so users can trace precedent
- **Citation-grounded output** — answers reference the specific section/clause and case they came from

## Current coverage

- Central RTI Act, 2005
- Delhi RTI Act, 2001
- Delhi High Court judgments (RTI-related)
- CIC (Central Information Commission) decisions

Designed to extend to additional state Acts and judgments over time.

## Architecture

```
Raw PDFs (Act text, judgments, CIC orders)
        │
        ▼
Text extraction & cleaning  (pdfplumber; OCR/font-mapping fixes)
        │
        ▼
Parent-child chunking       (parent = Section, child = clause, e.g. 8(1)(j))
        │
        ▼
Indexing                    (Qdrant vector DB + BM25 index)
        │
        ▼
Hybrid retrieval             (BM25 + vector search)
        │
        ▼
Self-grading / re-retrieval  (corrective loop on weak matches)
        │
        ▼
Answer generation            (Groq LLM inference, with citations)
        │
        ▼
Faithfulness check            (hallucination detection, retry-on-fail)
        │
        ▼
Citation graph layer          (links clauses ↔ citing case documents)
        │
        ▼
Final answer with citations
```

## Tech stack

| Purpose | Tool |
|---|---|
| Language | Python |
| PDF extraction | `pdfplumber` |
| Embeddings | `sentence-transformers` |
| Keyword retrieval | `rank-bm25` |
| Vector database | Qdrant |
| LLM inference | Groq |
| Config | `python-dotenv` |
| Dev tools | VS Code, Git |

## Methodology

1. **Data collection** — RTI Act 2005, Delhi RTI Act 2001, court judgments, CIC decisions
2. **Text extraction & cleaning** — resolve OCR issues and font-mapping bugs
3. **Parent-child chunking** — attach section/subsection/clause metadata to each chunk
4. **Indexing** — load chunks into Qdrant
5. **Hybrid retrieval + generation** — BM25 + vector search, answers generated via Groq with citations
6. **Verification** — hallucination/faithfulness check with retry-on-fail
7. **Citation graph** — link Act clauses to citing case documents
8. **Testing** — forward/reverse queries, no-graph fallback cases

## Evaluation

Accuracy is measured with **RAGAS** metrics:

- Faithfulness
- Answer relevancy
- Context precision
- Context recall

## Roadmap

- [ ] Add more state RTI Acts beyond Delhi
- [ ] Expand judgment/CIC decision coverage
- [ ] Broaden automated test coverage (forward/reverse query cases)

## Acknowledgements

Developed as part of an internship at [Vedas Tech](https://vedastech.io/) — Empowering AI & Language Services since 2021.

## References

- Right to Information Act, 2005 (Government of India)
- Delhi Right to Information Act, 2001 (Government of NCT of Delhi)
- Qdrant Documentation — https://qdrant.tech/documentation/
- Sentence-Transformers Documentation — https://www.sbert.net/
- Groq API Documentation — https://groq.com/
- RAGAS: Evaluation Framework for Retrieval-Augmented Generation — https://docs.ragas.io/
