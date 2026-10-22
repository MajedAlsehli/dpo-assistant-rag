# DPO Assistant | مساعد مسؤول حماية البيانات

An Arabic question-answering app for the document **«مشروع قواعد تعيين مسؤول حماية البيانات الشخصية»**. Ask about appointing a Personal Data Protection Officer (DPO), and the app returns an answer with a supporting excerpt and PDF page number.

This is a **Group 5 training project** built for the **Generative AI Solutions Development Program (تطوير حلول الذكاء الاصطناعي التوليدي)** at [SDAIA Academy](https://github.com/SDAIAAcademy). It is an educational prototype, not an official SDAIA service or a source of legal advice. The included document is the project's fixed knowledge source; the app does not check whether newer rules have been published.

## What you can do

- Ask questions in Arabic through a right-to-left chat interface.
- See the PDF page and excerpt used to support an answer.
- Try four suggested questions about DPO qualifications, required appointments, external contractors, and duties.
- Inspect the API through FastAPI's interactive documentation.

The app searches **one included, 11-page PDF**. Questions outside that document's scope should receive an abstention response.

## How it works

```text
PDF → Arabic text cleanup → page-based chunks → OpenAI embeddings
                                                   ↓
Question → OpenAI embedding → cosine-similarity search → relevant chunks
                                                   ↓
                                OpenAI answer + source excerpts and pages
```

The backend uses **FastAPI**; the frontend is plain **HTML, CSS, and JavaScript**. PyMuPDF extracts the PDF text, NumPy ranks chunks by cosine similarity, and OpenAI supplies embeddings and answer generation. Document embeddings are cached in `data/embeddings_cache.json` and rebuilt when the PDF, chunking version, or embedding model changes. The cache is ignored by Git.

## Run locally

You need Python 3.10+ and an OpenAI API key with API access. OpenAI usage may incur charges. Run these commands from the project folder.

**Windows PowerShell**

```powershell
git clone https://github.com/MajedAlsehli/dpo-assistant-rag.git
cd dpo-assistant-rag
python -m venv .venv
.\.venv\Scripts\python.exe -m pip install -r requirements.txt
Copy-Item .env.example .env
```

Open `.env` and replace `your_openai_api_key_here` with your own key. Keep the file private; `.gitignore` excludes it. The defaults in `.env.example` use `gpt-4o-mini` for answers and `text-embedding-3-small` for retrieval.

```powershell
.\.venv\Scripts\python.exe -m uvicorn backend.main:app --host 127.0.0.1 --port 8000
```

**macOS / Linux**

```bash
git clone https://github.com/MajedAlsehli/dpo-assistant-rag.git
cd dpo-assistant-rag
python3 -m venv .venv
.venv/bin/python -m pip install -r requirements.txt
cp .env.example .env
# Add your OpenAI API key to .env, then:
.venv/bin/python -m uvicorn backend.main:app --host 127.0.0.1 --port 8000
```

Open [http://127.0.0.1:8000](http://127.0.0.1:8000). The first start creates document embeddings and can take longer than later starts. The status at the top of the page shows whether the document and API key are ready. If you edit `.env` while the server is running, restart the server.

## Try it

Select a suggested question in the interface, or ask:

> متى يجب تعيين مسؤول حماية البيانات الشخصية؟

The answer should refer to the appointment circumstances in **PDF page 6**. You can also ask an unrelated question, such as `ما هي عاصمة فرنسا؟`, to check that the assistant declines questions outside the source document.

API endpoints:

| Endpoint | Purpose |
| --- | --- |
| `GET /` | Arabic chat interface |
| `GET /api/health` | Indexed chunk count and configuration status |
| `POST /api/chat` | Answer a question; send `{"question":"..."}` |
| `GET /docs` | Interactive API documentation |

`POST /api/chat` returns an `answer`, a `sources` list with `chunk_id`, `page_number`, and `excerpt`, and a training disclaimer. The API rejects empty or overlong questions. It returns a configuration error when the OpenAI key is missing.

## Project files

| Path | Purpose |
| --- | --- |
| `backend/main.py` | API routes and frontend serving |
| `backend/ingest.py` | PDF extraction, text cleanup, chunking, and embedding cache |
| `backend/rag.py` | Retrieval, answer generation, and source selection |
| `backend/config.py` | Paths, models, and environment settings |
| `frontend/` | Arabic chat interface |
| `data/` | Source PDF; generated embedding cache stays local |
| `docs/TECHNICAL.md` | Implementation details |
| `docs/EVALUATION.md` | Evaluation questions and expected pages |

## Limits and data handling

Answers depend on the extracted PDF text, retrieval results, and the language model. **Check the displayed excerpt against the PDF before relying on an answer.** The app checks that cited chunk IDs came from retrieved passages, but it cannot prove that every statement is supported. Its relevance threshold and instruction to abstain reduce off-topic answers; they do not guarantee perfect accuracy.

Questions and retrieved excerpts are sent to OpenAI for processing. Do not enter confidential information in the chat. The OpenAI key remains in the local `.env` file and is not needed in the browser.

## Team and acknowledgement

**Group 5:** Majed Alsehli, Abdullah Alotaibi, Afnan Alharbi, Leena Aljuwaysir, Aljawharah Almutairi, and Raid Almousa.

Developed as a training project within the **Generative AI Solutions Development Program** at [SDAIA Academy](https://github.com/SDAIAAcademy). The project is independently developed for learning and is not endorsed by SDAIA.
