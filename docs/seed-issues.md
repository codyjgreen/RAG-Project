# Seed issues

Starter backlog so you're not staring at a blank board on Day 2. Copy the ones you need into your team's GitHub Issues, split or merge them to fit your chosen project, and **add your own** — these cover the walking-skeleton path, not the whole project. Each has rough acceptance criteria; refine them as a team.

> Tip: keep issues small enough to finish in a day or less. If an issue needs the word "and," it's probably two issues.

1. **Scaffold the Flask API skeleton**
   - A `server/` app runs with `flask run` and serves a `GET /health` returning `{"status": "ok"}`.
   - `requirements.txt` and a `.env`-driven config are in place.

2. **Scaffold the React app (Vite)**
   - `client/` runs with `npm run dev` and renders a placeholder chat screen.
   - It can call `GET /health` and display the result (proves frontend↔backend wiring).

3. **Set up Postgres + pgvector and the schema**
   - `documents`, `chunks`, and an embedding column (`vector(768)`) exist via a migration or SQL script.
   - `CREATE EXTENSION vector` is documented and runs cleanly on a fresh DB.

4. **Implement the chunking function**
   - A function splits text into chunks with a configurable size/overlap (from `.env`).
   - Unit test covers a short and a long input.

5. **Implement `embed()` via Ollama**
   - A single `embed(text)` function returns a 768-dim vector from `nomic-embed-text`.
   - One seed document is chunked, embedded, and stored in pgvector.

6. **Implement retrieval (top-k)**
   - `retrieve(question, k)` embeds the question and returns the k most similar chunks (with similarity scores).
   - Unit test uses a stubbed vector store.

7. **Implement `generate()` with a grounded prompt + citations**
   - Builds a prompt from the question + retrieved chunks, calls the Ollama chat model, and returns an answer plus the source chunk ids.
   - Returns "I don't know based on the provided documents" when no chunk is relevant.

8. **Build the minimal chat UI**
   - A user can type a question, see the answer, and see the cited source chunks.
   - Loading and error states are handled.

9. **Document upload + ingestion endpoint**
   - `POST /documents` accepts a `.txt`/`.md` file, runs the ingestion pipeline, and persists it.
   - A document manager lists and deletes the user's documents.

10. **Authentication + per-user scoping**
    - Signup / login with hashed passwords; protected endpoints reject unauthenticated requests.
    - A user only sees their own documents and chats.

11. **Set up CI (GitHub Actions)**
    - On every PR: install deps, lint, run tests. A red build blocks merge.
    - Ollama calls are mocked so CI doesn't need a running model.
