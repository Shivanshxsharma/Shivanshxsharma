<div align="center">

<img src="./assets/banner.svg" alt="Shivansh Sharma - Developer File" width="100%" />

</div>

<table width="100%">
<tr>
<td width="30%" valign="top" align="center">

<img width="1200" height="1404" alt="1791296983-103 216 143 134" src="https://github.com/user-attachments/assets/b99dcb05-b536-4fed-b91f-4c7a381dc27a" />

`Suspect`

</td>
<td width="70%" valign="top">

```
════════════════════════════════════════════════════════════════
 DEVELOPER FILE -- STATUS: OPEN TO OPPORTUNITIES
 REGISTERED: NSUT DELHI          CLEARANCE: PORTFOLIO REVIEW
════════════════════════════════════════════════════════════════

 IDENTIFICATION:   NSUT / ICE-2027
 NAME:              SHIVANSH SHARMA
 TITLE:             FULL-STACK DEVELOPER / AI SYSTEMS
 COLLEGE:           NETAJI SUBHAS UNIVERSITY OF TECHNOLOGY
 DEGREE:            B.TECH, INSTRUMENTATION & CONTROL ENGINEERING
 GRADUATION:        2027
 STATUS:            ACTIVELY INTERVIEWING
 ─────────────────────────────────────────────────────────────
 LANGUAGES:  PYTHON . JAVA . JAVASCRIPT
 STACK:      FASTAPI . LANGGRAPH . NEXT.JS . NODE.JS . MONGODB . PINECONE
 ─────────────────────────────────────────────────────────────
 KEY PROJECTS ON FILE:
 A1.  ALFRED      -- AI AGENT PLATFORM
 A2.  DOCINTEL    -- HYBRID RAG PIPELINE
 A3.  HEX         -- REAL-TIME AI CHAT PLATFORM
 A4.  FOCUS POT   -- TASKBAR FOCUS COMPANION
 A5.  PDFPORTAL   -- OFFLINE PDF TOOLS
 A6.  INSIGHTS    -- SOCIAL ANALYTICS DASHBOARD
════════════════════════════════════════════════════════════════
```

</td>
</tr>
</table>

<img src="./assets/divider.svg" alt="" width="100%" />

## `> PROJECT FILES`

<!-- ============ ALFRED ============ -->
<table width="100%">
<tr>
<td width="15%" align="center">
<img width="938" height="941" alt="Screenshot 2026-07-11 192336" src="https://github.com/user-attachments/assets/e6847592-f76d-4f51-9ee4-8295c933925a" />

</td>
<td width="85%">

### A1 · ALFRED — Personal AI Agent

**CLASSIFICATION:** Solo-built · Full-Stack · Deployed<br>
**STACK:** FastAPI · LangGraph · Next.js · Pinecone · Zustand · Render + Vercel

- `MODEL_REGISTRY` architecture for unified routing across Gemini 2.5 Flash, Llama 3.3 70B, and Cerebras
- LLM Wiki memory system (inspired by Andrej Karpathy) with session-end summarizer and nightly relevancy decay pruning, cutting memory retrieval latency by 50%
- 2-layer token-efficient RAG: chat history queried before Pinecone, MarkItDown chunking improving retrieval accuracy by 30%
- Tavily web search, SSE streaming with abort, JWT auth with refresh tokens, Google OAuth

**LIVE:** [alfred-khaki.vercel.app](https://alfred-khaki.vercel.app)

</td>
</tr>
</table>

<img src="./assets/divider.svg" alt="" width="100%" />

<!-- ============ DOCINTEL + HEX ============ -->
<table width="100%">
<tr>
<td width="50%" valign="top">

### A2 · DOCINTEL — Hybrid RAG Pipeline

**CLASSIFICATION:** Take-home build → HPC deployment<br>
**STACK:** Docling · Pinecone · BM25 · Cohere Rerank

- Document parsing and chunking via Docling
- Dense (Pinecone) + sparse (BM25) retrieval merged with RRF, then Cohere Rerank
- Benchmarked on QASPER against a standard RAG: 63% fewer retrieval misses, +30% MRR
- Deployed and evaluated on client HPC infrastructure

</td>
<td width="50%" valign="top">

### A3 · HEX — AI Personality Chat

**CLASSIFICATION:** Full-stack real-time chat, 7 personas<br>
**STACK:** Node.js · React.js · Socket.IO · Clerk · MongoDB

- Gemini streaming over Socket.IO WebSockets, cutting perceived response latency by 40%
- JWT auth via Clerk, chat history persisted in MongoDB
- Backend deployed on Render with concurrency optimization

</td>
</tr>
</table>

<img src="./assets/divider.svg" alt="" width="100%" />

<!-- ============ FOCUS POT + PDFPORTAL ============ -->
<table width="100%">
<tr>
<td width="50%" valign="top">

### A4 · FOCUS POT — Taskbar Focus Companion

**CLASSIFICATION:** Windows app · MIT<br>
**STACK:** Tauri 2 · Rust · JavaScript

- Lives on the empty left end of the Windows taskbar
- Pomodoro timer, tasks, and media controls for every player
- Clipboard history (opt-in, memory only) and a screenshot organizer
- Windows 11, Material 3, and Liquid Glass themes in an 8 MB app

**REPO:** [focus-pot](https://github.com/Shivanshxsharma/focus-pot) · [Download](https://github.com/Shivanshxsharma/focus-pot/releases/latest/download/FocusPot-Setup.exe)

</td>
<td width="50%" valign="top">

### A5 · PDFPORTAL — Offline PDF Tools

**CLASSIFICATION:** Windows app · AGPL-3.0<br>
**STACK:** Tauri 2 · Rust · Python · PyMuPDF

- Drag a file and shake it: a ring of tools opens under the cursor
- 10 tools: compress, merge, split, rotate, watermark, protect, and more
- Fully offline, so files never leave the PC
- Installer built and tested by GitHub Actions on every release

**REPO:** [pdfportal](https://github.com/Shivanshxsharma/pdfportal) · [Download](https://github.com/Shivanshxsharma/pdfportal/releases/latest/download/PDFportal-Setup.exe)

</td>
</tr>
</table>

<img src="./assets/divider.svg" alt="" width="100%" />

<!-- ============ INSIGHTS ============ -->
### A6 · INSIGHTS — Social Analytics Dashboard

**CLASSIFICATION:** In progress · Multi-platform SaaS<br>
**STACK:** FastAPI · PostgreSQL · modular monolith with background workers

- One dashboard for creators and businesses, starting with YouTube, then Meta, LinkedIn, TikTok, and Google Analytics
- Multi-tenant accounts: individual or company, teams, and per-platform Read/Post roles
- Shared sign-in across separate tools, with rotating refresh-token auth

<img src="./assets/divider.svg" alt="" width="100%" />

<div align="center">

[![GitHub](https://img.shields.io/badge/GITHUB-Shivanshxsharma-FFD400?style=flat-square&labelColor=0a0a0a)](https://github.com/Shivanshxsharma)
[![LinkedIn](https://img.shields.io/badge/LINKEDIN-shivansh--sharma-FFD400?style=flat-square&labelColor=0a0a0a)](https://www.linkedin.com/in/shivansh-sharma-a72283283)
[![HuggingFace](https://img.shields.io/badge/HUGGINGFACE-shivansh012-FFD400?style=flat-square&labelColor=0a0a0a)](https://huggingface.co/shivansh012)

</div>
