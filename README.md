# RAG Apprentice Project — 3-Week Full-Stack Build

> A starter repo and project brief for apprentice teams building a **Retrieval-Augmented Generation (RAG)** app — "chat with your documents" — using **React + Flask + Ollama**. 100% local, 100% free, no API keys.

**Who this is for:** Apprentices who have just finished the Software Engineering program and are about to start the AI Data Science program. This three-week group project is a visible way to show you're integrating AI into your learning, while practicing a professional software development lifecycle (SDLC) and Git workflow.

**How to use this repo:** Fork or clone it. The README is your requirements doc and definition of "done." [`CONTRIBUTING.md`](./CONTRIBUTING.md) is your Git and team workflow. Build your app on top of it.

---

## Table of contents
1. [What you're building](#1-what-youre-building)
2. [Why RAG, and how it works](#2-why-rag-and-how-it-works)
3. [Tech stack (fixed)](#3-tech-stack-fixed)
4. [Quick start](#4-quick-start)
5. [Project menu — pick one](#5-project-menu--pick-one)
6. [Requirements / definition of done](#6-requirements--definition-of-done)
7. [Evaluating your AI](#7-evaluating-your-ai-the-habit-that-matters-most)
8. [Three-week timeline](#8-three-week-timeline)
9. [Team roles](#9-team-roles)
10. [Deployment & demo](#10-deployment--demo)
11. [Git & team workflow](#11-git--team-workflow)

---

## 1. What you're building

A web app where a user uploads documents, asks questions in a chat interface, and gets **answers grounded in those documents with citations back to the source**. The "intelligence" comes from retrieval plus a local language model — you are *not* training a model.

The app idea is the vehicle, not the destination. In three weeks nobody expects a polished product. What we expect is evidence that you can work like a real engineering team: a clean Git history, reviewed pull requests, a backlog that maps to what you built, tests that run in CI, a running app anyone can spin up, and a demo with an honest retro. A modest app that's well-engineered beats an ambitious one that only runs on one laptop.

---

## 2. Why RAG, and how it works

RAG is the sweet spot for this cohort: a genuine AI application where the AI is *a model call and a retrieval step*, not a model you train. It teaches the concepts that matter most heading into the data science program — **embeddings, vector search, prompt construction, grounding/citations, and evaluation.**

```
                    ┌─────────── INGESTION (when a doc is uploaded) ───────────┐
  Upload doc  ─▶  parse  ─▶  chunk  ─▶  embed (Ollama)  ─▶  store vectors (pgvector)
                    └──────────────────────────────────────────────────────────┘

                    ┌─────────── QUERY (when a question is asked) ─────────────┐
  Question ─▶ embed query (Ollama) ─▶ similarity search ─▶ top-k chunks
                                                          │
                                  build prompt (question + retrieved chunks)
                                                          │
                                          LLM generates answer (Ollama)
                                                          │
                              return answer + cited source chunks ─▶ React chat UI
                    └──────────────────────────────────────────────────────────┘
```

The whole pipeline runs locally through **Ollama** — both the embedding model and the chat model. No accounts, no keys, no spend.

---

## 3. Tech stack (fixed)

The stack is locked so mentors can support every team and teams can help each other. Don't burn three days choosing a framework.

| Layer | Choice | Notes |
|-------|--------|-------|
| Frontend | **React (Vite)** | Chat UI, document upload, a panel showing the source chunks behind each answer. |
| Backend | **Python + Flask** | REST API: ingest, list documents, query/chat. Keeps you in the language you'll use in the AI program. |
| Database | **Postgres + [pgvector](https://github.com/pgvector/pgvector)** | One database for both your app data and your embeddings. |
| Embeddings | **Ollama → `nomic-embed-text`** | Local, free. 768-dimension vectors → `vector(768)` column in pgvector. |
| LLM | **Ollama → `llama3.1:8b`** (or `llama3.2`) | Local, free. Runs the generation step. |
| Auth | **Flask-Bcrypt** + sessions/JWT | Hash passwords; users see only their own docs. |
| Running it | **Local dev servers** | Run the Vite, Flask, Postgres, and Ollama processes directly on your machine. Docker is an optional stretch (see [Deployment & demo](#10-deployment--demo)). |

> **Architecture rule:** put your model calls behind a single `generate(prompt)` and `embed(text)` interface. Today they call Ollama; swapping to a hosted API later becomes a one-file change. That abstraction is itself a good engineering lesson — and worth pointing out in your demo.

### Recommended models for learning (MacBook-friendly)

Apple Silicon Macs share memory between the CPU and GPU ("unified memory"), so the amount of **RAM** is what decides which models run comfortably. Rough rule of thumb: a model needs roughly its download size in free RAM, plus a couple of GB of headroom.

| Your Mac's RAM | Comfortable chat models | Notes |
|----------------|-------------------------|-------|
| **8 GB** | `llama3.2:3b`, `gemma2:2b`, `phi3.5` | Stay at 3B params or below. Close other apps. |
| **16 GB** | `llama3.1:8b`, `mistral:7b`, `qwen2.5:7b` | The sweet spot — `llama3.1:8b` is a great default. |
| **32 GB+** | `gemma2:9b`, `qwen2.5:14b` | You can run larger models, but you rarely need to for this project. |

**Chat / generation models** (pick one to start):

| Model | Pull command | Size | Good for |
|-------|--------------|------|----------|
| Llama 3.2 1B | `ollama pull llama3.2:1b` | ~1.3 GB | Wiring up the pipeline fast; runs on anything |
| Llama 3.2 3B | `ollama pull llama3.2:3b` | ~2 GB | Best small model for 8 GB Macs |
| Gemma 2 2B | `ollama pull gemma2:2b` | ~1.6 GB | Tiny but surprisingly capable |
| Phi-3.5 | `ollama pull phi3.5` | ~2.2 GB | Strong reasoning for its size |
| Mistral 7B | `ollama pull mistral` | ~4.1 GB | Solid all-rounder on 16 GB |
| Llama 3.1 8B | `ollama pull llama3.1:8b` | ~4.7 GB | **Recommended default** (16 GB+) |
| Qwen 2.5 7B | `ollama pull qwen2.5:7b` | ~4.7 GB | Excellent at following instructions |

**Embedding models** (you need exactly one — match the `vector(N)` dimension in your schema):

| Model | Pull command | Size | Vector dim |
|-------|--------------|------|------------|
| nomic-embed-text | `ollama pull nomic-embed-text` | ~274 MB | 768 ← **recommended default** |
| mxbai-embed-large | `ollama pull mxbai-embed-large` | ~670 MB | 1024 (higher quality) |
| all-minilm | `ollama pull all-minilm` | ~45 MB | 384 (tiniest, fastest) |

> **Practical tip:** develop against a *tiny* model like `llama3.2:1b` so iteration is fast and your laptop stays cool, then switch to `llama3.1:8b` for your evaluation runs and the final demo. Because your model calls sit behind the `generate()`/`embed()` interface, switching is a one-line config change (`OLLAMA_LLM_MODEL` in `.env`).
>
> Model names and sizes here are a known-good starting set — browse [ollama.com/library](https://ollama.com/library) for newer releases, and run `ollama list` to see what you've already pulled.

---

## 4. Quick start

> Prerequisites: [Node.js](https://nodejs.org/) (18+), [Python](https://www.python.org/) (3.11+), [Postgres](https://postgresapp.com/) with the [pgvector](https://github.com/pgvector/pgvector) extension, and [Ollama](https://ollama.com/download) — all installed locally. No Docker required.

```bash
# 1. Clone your team's repo
git clone <your-team-repo-url>
cd <your-team-repo>

# 2. Pull the models (one time, a few GB) — Ollama runs as a local service
ollama pull llama3.1:8b
ollama pull nomic-embed-text

# 3. Copy env template and fill in values
cp .env.example .env

# 4. Create the database and enable pgvector (one time)
createdb ragdb
psql ragdb -c "CREATE EXTENSION IF NOT EXISTS vector;"

# 5. Start the backend (terminal 1)
cd server
python -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt
flask run                       # http://localhost:5000

# 6. Start the frontend (terminal 2)
cd client
npm install
npm run dev                     # http://localhost:5173
```

Ollama serves on `11434` and Postgres on `5432` by default. You'll run the frontend and backend in two terminals during development.

> The app folders, dependency files, and starter pipeline are **yours to build** — this repo ships the requirements and workflow, and your first issues are to scaffold them. See the suggested structure below.
>
> *Prefer one command?* Wrapping all of this in Docker Compose is an optional stretch goal — see [Deployment & demo](#10-deployment--demo).

### Suggested repo structure
```
.
├── README.md            ← this file (requirements & brief)
├── CONTRIBUTING.md      ← Git & team workflow — read this
├── .env.example         ← copy to .env; documents required config
├── .gitignore
├── docker-compose.yml   ← optional (stretch goal)
├── client/              ← React app (Vite)
│   └── ...
├── server/              ← Flask API
│   ├── app.py
│   ├── ingestion.py     ← parse → chunk → embed → store
│   ├── retrieval.py     ← embed query → similarity search
│   ├── llm.py           ← generate() / embed() behind one interface
│   └── tests/
└── docs/
    ├── ARCHITECTURE.md  ← your architecture diagram + decisions
    └── eval.md          ← your evaluation set & results
```

---

## 5. Project menu — pick one

Every option is the *same RAG engine* pointed at a different document set with a tailored UI. The shared shape is what makes teams' work comparable at presentations even though the use cases differ. **Each team picks a different one.**

### Option A — Study Buddy (course notes & textbooks)
Upload lecture notes, PDFs, and slides; ask questions and get cited answers for exam prep.
*Hook:* a "quiz me" mode that generates questions from the retrieved material.

### Option B — Codebase / Docs Assistant
Ingest project documentation (or an open-source repo's docs); answer "how do I…" questions with citations to the exact section.
*Hook:* code-snippet rendering and deep links back to the source file.

### Option C — Policy & Handbook Helper
Ingest an employee handbook, terms of service, or benefits docs; answer policy questions with the exact clause cited.
*Hook:* highlights the supporting clause; refuses to answer when the docs don't cover it.

### Option D — Research Paper Companion
Upload academic papers; ask for summaries, methods, and findings with citations.
*Hook:* side-by-side answer and source passage; "explain like I'm new to this field."

### Option E — Support Knowledge Bot
Ingest help-center articles / FAQs; answer customer-style questions and escalate ("I don't know") when confidence is low.
*Hook:* a "sources found" confidence indicator and a fallback to "contact support."

> **Tip:** Options C and E teach the most important RAG lesson — *gracefully saying "I don't know" when the answer isn't in the documents.* At least one team should take a grounding-strict use case so the cohort sees hallucination mitigation in action.

---

## 6. Requirements / definition of done

This section is the rubric in disguise. Your **problem statement** (one paragraph: who is this for, what corpus, what can they now ask?) goes at the top of your fork's README before you write code.

### MVP — must-have (all teams)
1. **Document ingestion** — upload at least `.txt`/`.md` (then `.pdf`). The app parses, **chunks**, embeds via Ollama, and stores vectors in pgvector.
2. **Vector storage & retrieval** — embeddings persist; a query runs a similarity search and returns the top-k relevant chunks.
3. **Grounded generation** — retrieved chunks go into the prompt; the answer is based on them.
4. **Citations** — every answer shows *which source chunks* it used. Non-negotiable — it's how anyone trusts the output.
5. **"I don't know" behavior** — when retrieval finds nothing relevant, the app says so instead of inventing an answer.
6. **Authentication** — sign up / log in; each user sees only their own documents and chats. Passwords hashed.
7. **A real frontend** — chat UI with history, a document manager (upload/list/delete), loading and error states, responsive layout.
8. **Persistence** — documents, embeddings, and chat history survive a restart (real DB).
9. **Runs from the README** — a teammate (or mentor) can follow your setup steps to run the full app — frontend, API, Postgres/pgvector, Ollama — on their own machine. (A one-command Docker setup is an optional stretch; see [Deployment & demo](#10-deployment--demo).)
10. **README** — setup, architecture diagram, model choices, screenshots, team credits.

### AI-specific quality bar
- **No leaked secrets** — config lives in env vars; `.env` is never committed (`.env.example` is).
- **Prompt-injection awareness** — you can explain the risk (a document telling the model to ignore instructions) and show one basic mitigation (clearly delimiting retrieved content in the prompt).
- **An evaluation set** — 8–10 question/expected-answer pairs you use to sanity-check retrieval and answers before the demo (see §7).
- **Sensible resource use** — reasonable chunk sizes and top-k; graceful handling when Ollama is slow or unavailable.

### Stretch goals (only after the MVP runs)
Streaming responses (token-by-token), conversation memory / follow-ups, hybrid search (keyword + vector), re-ranking retrieved chunks, multi-document Q&A, page-level PDF citations, an eval dashboard, a model picker that compares `llama3.1` vs `llama3.2` answers, or **containerizing the whole stack with Docker Compose** so it runs with one command.

### Out of scope
No training or fine-tuning, no agents/tool-use, no multi-modal (images/audio), no payments. (That's for the AI program.)

### Presentation deliverable
10-minute demo + 5-minute Q&A:
- Problem, target user, and chosen corpus (1 slide)
- Live demo: upload a doc, ask questions, **show the citations**, and show it saying "I don't know" on an out-of-scope question
- Architecture diagram (ingestion + query paths)
- One technical challenge (chunking, retrieval quality, prompt design…) and how you solved it
- A look at your evaluation: how you know the answers are decent
- Retro: what went well, what you'd do differently, what you'd build next

---

## 7. Evaluating your AI (the habit that matters most)

"It looked right when I tried it" is not evaluation. The lightweight version that fits three weeks:

- **Build a golden set:** 8–10 questions about your corpus with known good answers, including 2–3 the docs *don't* cover (to test "I don't know"). Keep it in `docs/eval.md`.
- **Check retrieval separately from generation:** for each question, did the right chunk come back in the top-k? Bad retrieval is usually the culprit, not the model.
- **Score answers simply:** correct / partial / wrong / hallucinated, plus "did it cite the right source." A table is fine.
- **Re-run after changes:** when you tweak chunk size, top-k, or the prompt, re-run the set and see if the score moved. This is the loop the whole AI field runs on.

Showing this in your presentation — *"we changed chunk size from 1000 to 400 tokens and accuracy went from 6/10 to 9/10"* — is what separates a real AI project from a demo that happened to work once.

---

## 8. Three-week timeline

~15 working days, with a hard **"get it running end-to-end early"** bias so the AI pipeline is debugged while it's cheap. Full Git ceremonies are in [`CONTRIBUTING.md`](./CONTRIBUTING.md).

### Week 1 — Foundations & a "walking skeleton" RAG
*Get an end-to-end slice working on one seeded document.*

| Day | Focus | Outcome |
|-----|-------|---------|
| 1 | Kickoff, team norms, pick project/corpus, problem statement; learn the RAG flow | Repo + README started, roles assigned |
| 2 | Design the ingestion pipeline & API; data modeling (docs, chunks, embeddings); write issues | ERD + ~15 issues on the board |
| 3 | Scaffold Flask + React + local Postgres/pgvector; get **one document chunked and embedded** into pgvector via Ollama | Vectors land in the DB |
| 4 | First retrieval + LLM call: question in → top-k chunks → grounded answer from Ollama | End-to-end answer works locally |
| 5 | Wire it to a minimal React chat UI; **the app runs end-to-end locally** from the README steps; CI green | App answers a question over one seeded doc, with a citation |

> **The most important milestone is end of Week 1: an app that answers one question over one document, with a citation.** Teams that defer the Ollama/vector plumbing to week three lose the demo to a config error at 11pm.

### Week 2 — Core feature build
*Real ingestion, real UI, grounding you can trust.*

| Day | Focus | Outcome |
|-----|-------|---------|
| 6 | Authentica