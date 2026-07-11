# DocIntel — Document Intelligence System
> N-ERGY Take-Home Assignment | Shivansh Sharma

---

## Optimization Choice: Accuracy

This system is optimized for **retrieval accuracy**, not raw latency. The core reasoning: the assignment's primary deliverable is *answers with cited sources* — a fast but incorrectly cited answer is worse than a slightly slower correct one. Every architectural decision below is made in service of that goal.

---

## Architecture Overview

```
PDF Upload
   │
   ▼
Docling Parser (OCR off, tables on)
   │  layout analysis → clean Markdown with heading hierarchy
   ▼
MarkdownHeaderTextSplitter
   │  section-aware splits on #/##/###/#### headings
   ▼
RecursiveCharacterTextSplitter
   │  size-bounds long sections, prevents orphan tail chunks
   ▼
Section breadcrumb prepended to each chunk
   │  injects section identity into embedded text, not just metadata
   ▼
┌─────────────────────────┬──────────────────────────┐
│   Dense Index           │   Sparse Index           │
│   Gemini Embeddings     │   BM25 sparse vectors    │
│   → Pinecone            │   → Pinecone (hybrid)    │
└─────────────┬───────────┴──────────┬───────────────┘
              │                      │
              ▼                      ▼
         Top-50 dense          Top-50 sparse
              │                      │
              └──────────┬───────────┘
                         ▼
              Reciprocal Rank Fusion (RRF, k=60)
                         │
                         ▼
              Cohere Rerank → Top-5 chunks
                         │
                         ▼
              Gemini 1.5 Flash (structured output)
                         │
                         ▼
              {answer, citations, insights, next_questions}
                         │
                         ▼
              Plain HTML/JS/CSS Frontend
```

---

## Technology Decisions

| Component | Choice | Alternatives Considered | Why This |
|---|---|---|---|
| **PDF Parsing** | Docling (IBM) | PyPDFLoader, pdfplumber | PyPDFLoader flattened two-column academic layouts into scrambled interleaved text (author names mashed into emails). Docling reconstructs reading order, preserves heading hierarchy, and handles tables as proper Markdown. This directly feeds section-aware chunking — the foundation everything else builds on. |
| **Chunking strategy** | MarkdownHeaderTextSplitter + RecursiveCharacterTextSplitter | Fixed-size character splits, semantic chunking | Two-stage: first split on heading boundaries (section identity preserved), then size-bound only if a section is genuinely long. A section only slightly over the size limit is kept whole rather than shedding a contentless orphan tail. Semantic chunking was considered but Docling's heading-aware output already achieves most of what semantic chunking provides. |
| **Chunk enrichment** | Section breadcrumb prepended to chunk body | Metadata-only, contextual embeddings (Anthropic) | Breadcrumb prefix ("filename > Section > Subsection") injects section identity into the embedded vector itself, not just metadata. This means deep fragments stay findable by their section context even when their body text doesn't repeat section-level keywords. Anthropic's contextual embeddings (35-67% retrieval failure reduction) was considered but excluded due to per-chunk LLM call overhead at ingestion time — the breadcrumb approach achieves similar benefits at zero LLM cost. |
| **Dense retrieval** | Pinecone + Gemini text-embedding-004 | FAISS, Chroma, OpenAI embeddings | Pinecone removes in-memory constraints for 50-document scale, provides managed ANN search, and pairs naturally with Gemini embeddings (same provider as generation model — consistent embedding space). Chroma was considered for simplicity but Pinecone's cloud-managed index is more robust for a production-like demo. |
| **Sparse retrieval** | Pinecone native sparse vectors (BM25) | rank-bm25 in-memory, TF-IDF, Elasticsearch | Pinecone's hybrid index stores BM25 sparse vectors alongside dense vectors in the same managed index — incremental upserts work correctly without full corpus recomputation, no in-memory RAM constraint, and sparse vectors persist identically to dense vectors. Catches exact terminology matches (model names, metric values, proper nouns) that dense embeddings miss when vocabulary diverges between query and source. |
| **Fusion** | Reciprocal Rank Fusion (RRF, k=60) | Linear score combination, simple union | RRF operates on rank positions, not raw scores — so it correctly fuses results from two retrievers with incompatible score scales (cosine similarity vs BM25 term frequency). k=60 is the standard literature default, empirically robust across domains. |
| **Reranking** | Cohere Rerank API | Local cross-encoder (sentence-transformers), LLM-based scoring | Cohere's reranker uses a cross-encoder that jointly reads query + chunk together, fixing rank-ordering errors that RRF cannot — specifically: short, superficially keyword-matching chunks (e.g. copyright footers) that BM25 over-scores get demoted; substantive but semantically matched chunks get promoted. Tested and confirmed: the "authors of this paper" query had the correct chunk at rank 3 before reranking; reranking promotes it to rank 1. |
| **Generation** | Gemini 1.5 Flash (structured output) | GPT-4o, Claude Haiku, local LLM | Gemini Flash is fast, cost-efficient, and supports native structured output (Pydantic schema enforcement) — critical for reliably getting answer + citations + insights + next_questions as a consistent JSON blob rather than parsing free text. Same provider as embeddings keeps the stack coherent. |
| **Frontend** | Plain HTML/JS/CSS (single file) | React + Vite, Next.js | Assignment explicitly states "function over form, not evaluating design." A single `index.html` with no build step opens in any browser instantly, has zero dependency installation, and is demonstrably faster to ship in an 8-hour window than a React app — without sacrificing any required functionality. |
| **GraphRAG** | Not used | — | Considered explicitly. GraphRAG adds value for entity-relationship-heavy corpora (org charts, knowledge graphs, relational enterprise data). Arbitrary uploaded PDFs have no guaranteed schema or entity structure — this is the textbook case for vector RAG, not graph RAG. Wrong shape of problem. |

---

## Key Design Decisions

### Why two-stage chunking instead of fixed-size splits

Fixed-size character splitting is fast but semantically blind — it breaks mid-sentence, mid-equation, and mid-table with no awareness of document structure. Our two-stage approach:

1. **Stage 1 (MarkdownHeaderTextSplitter):** splits on actual section boundaries from Docling's heading hierarchy. Keeps section content together.
2. **Stage 2 (RecursiveCharacterTextSplitter):** only applied to sections that are genuinely large, with a guard against shedding orphan tail chunks under ~25 tokens (which are usually formula placeholders or stray punctuation with no retrieval value).

This was validated empirically: a section containing all four fractal feature definitions (Higuchi, Petrosian, Katz, DFA) at 1,260 chars was being split at CHUNK_SIZE=1,200 into a 955-char useful chunk and a 305-char contentless orphan. The orphan was being surfaced by the reranker at position [5] instead of the useful chunk, making it look like the feature names were missing when they were intact in the adjacent chunk. The fix: keep sections within a reasonable tolerance of the size limit whole.

### Why we switched from PyPDFLoader to Docling mid-build

Initial testing with PyPDFLoader on a two-column IEEE paper revealed a fundamental flaw: the library reads PDF text in stream order (left-to-right across the full page width), which interleaves the two columns incorrectly. Author names were mashed into email addresses, section headers appeared mid-paragraph, and table content was scrambled across lines. Docling's layout analysis reconstructs the correct reading order from the document's actual visual structure, fixing all of these issues in one swap.

### Why we skipped contextual embeddings

Anthropic's contextual retrieval technique prepends a 1-2 sentence LLM-generated context blurb to each chunk before embedding, reducing retrieval failures by 35-67% depending on whether reranking is also used. This was seriously considered. The reason it was excluded: it requires one LLM call per chunk at ingestion time. For 50 PDFs generating ~1,000+ chunks, this adds substantial ingestion latency and API cost. The breadcrumb prefix achieves similar section-identity injection at zero LLM cost. With more time, contextual embeddings would be the highest-ROI next addition.

---

## Tradeoffs

| Tradeoff | Decision | What was given up |
|---|---|---|
| Accuracy vs latency | Accuracy | Query responses take 2-4s (retrieval + reranking + generation). Latency-mode would skip reranking and use a smaller k, cutting this to ~800ms. |
| Cohere reranker vs local cross-encoder | Cohere API | Network dependency on Cohere at query time. Local cross-encoder (ms-marco-MiniLM) would eliminate this but requires model loading (~80MB) on the server. |
| Pinecone vs local Chroma | Pinecone | Added network/API dependency. Chroma would be zero-dependency but in-memory, non-persistent across server restarts. |
| Full contextual embeddings vs breadcrumb prefix | Breadcrumb prefix | 35-67% retrieval failure reduction left on the table. Would add it with more time. |
| Streaming vs structured output | Structured output | No typewriter effect in the UI. Cleaner JSON contract between frontend and backend, more reliable citation extraction. |

---

## What Would Break at Scale (10k+ Documents)

| Component | How it breaks | Fix |
|---|---|---|
| **Cohere reranker at query volume** | Cohere Rerank is a network call per query. At high concurrent query volume (many users, many requests/sec), this becomes a latency bottleneck and a cost scaling concern — Cohere charges per rerank call. | Move to a local cross-encoder (ms-marco-MiniLM, ~80MB) to eliminate the network dependency, or cache rerank results for repeated queries. |
| **Synchronous ingestion** | PDF parsing (Docling) + embedding + upsert runs synchronously in the upload request. At 50 PDFs this is ~30-60s. At 10k it's hours. | Async job queue (Celery + Redis), return a job ID immediately, poll for completion. |
| **Pinecone free tier** | Free tier has namespace/vector limits. At 10k docs the index size could hit plan limits. | Move to paid Pinecone plan or self-hosted Qdrant/Weaviate. |
| **Single Pinecone namespace** | All documents share one namespace — no per-user isolation. At scale, user A's documents contaminate user B's retrieval. | Namespace per user/session. |
| **No eval infrastructure** | No retrieval quality metrics (MRR, NDCG, hit@k), no answer quality evals. Can't detect when accuracy degrades as the corpus grows. | Add RAGAs or a custom eval harness with a golden Q&A set per document type. |

---

## What I Would Improve With More Time

1. **Contextual embeddings** — Anthropic's technique, one LLM call per chunk at ingestion, 35-67% retrieval failure reduction. Highest ROI next addition.
2. **Local reranking** — replace the Cohere Rerank API call with a local cross-encoder (ms-marco-MiniLM) to eliminate the network dependency and per-call cost at high query volume.
3. **Eval harness** — generate a golden Q&A set from test documents, measure hit@k and MRR before/after each pipeline change. Currently all tuning is qualitative (manual query testing).
4. **Async ingestion queue** — Celery + Redis so large PDF batches don't block the upload endpoint.
5. **Metadata filtering** — allow users to scope queries to a specific uploaded document ("only search in report.pdf") rather than always searching the full corpus.
6. **Semantic caching** — cache query-answer pairs, return cached answer when a new query is >0.95 cosine similarity to a cached one. Reduces redundant LLM calls.
7. **Per-user namespacing** — isolate each user's uploaded documents in separate Pinecone namespaces for multi-tenant correctness.
8. **OCR support** — currently text-only PDFs are supported. Adding Docling's OCR pipeline (Tesseract/EasyOCR) would handle scanned documents.

---

## Running the System

### Backend

```bash
cd nergy-clean
python -m venv venv
source venv/bin/activate      # Linux/Mac
pip install -r requirements.txt
cp .env.example .env          # fill in your keys
uvicorn api:app --host 0.0.0.0 --port 8000
```

### Frontend

Open `frontend/index.html` directly in a browser. No build step required.

Backend must be running on `http://localhost:8000` before using the UI.

### Environment Variables

```
PINECONE_API_KEY=       # Pinecone API key
PINECONE_INDEX_NAME=    # e.g. hybrid-search
GEMINI_API_KEY=         # Google AI Studio API key
COHERE_API_KEY=         # Cohere API key (for reranking)
```

---

## API Endpoints

| Method | Endpoint | Description |
|---|---|---|
| `GET` | `/health` | Liveness check |
| `POST` | `/upload` | Upload a PDF — ingests, chunks, embeds, indexes |
| `POST` | `/query` | Ask a question — returns answer + citations + insights + next questions |

### POST /upload
- Accepts: `multipart/form-data` with `file` field (PDF only)
- Returns: `{status, filename, chunks_added}`
- Edge cases: non-PDF files → 400, empty/unreadable PDF → 200 with `chunks_added: 0`

### POST /query
- Accepts: `{"question": "..."}`
- Returns:
```json
{
  "answer": "...",
  "citations": [{"filename": "...", "page": 1, "section": "..."}],
  "insights": ["...", "..."],
  "next_questions": ["...", "..."]
}
```
- Edge cases: empty question → 400, no indexed documents → answer explains this

---

*Built by Shivansh Sharma — github.com/Shivanshxsharma*
#   S h i v a n s h x s h a r m a  
 