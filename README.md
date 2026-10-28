# Finance Intelligence Agent — Finwise

A classroom-friendly full-stack demo that turns transaction data into a spending dashboard and uses a bounded, deterministic Python agent to detect subscription price hikes. It is educational software, not financial advice.

## Safety and limitations
- Use fictional or redacted data only. **Do not upload real bank statements or sensitive financial information.**
- Sample data is fictional and denominated in USD.
- Audit sessions live in server memory and disappear when the server restarts. There is no database or file persistence.
- Approving a draft only records a local simulation. The app never sends messages, contacts merchants/banks, calls payment APIs, or moves money. `sent` remains `false`.
- Gemini extraction is intentionally a stub; implement the TODOs in `server/app/ai_service.py` if you want live extraction. Until then, the UI and audit workflow work in offline demo mode.
- A configured API key currently reports as configured, but does not mean the manual Gemini integration has been implemented. The AI planner continues to use the deterministic offline planner.

## Project structure
```text
finance-intelligence-agent/
├── server/
│   ├── app/
│   │   ├── __init__.py
│   │   ├── ai_service.py
│   │   ├── main.py
│   │   ├── models.py
│   │   ├── sample_data.py
│   │   └── tools.py
│   ├── tests/
│   │   ├── test_tools.py
│   │   └── test_api.py
│   ├── .env.example
│   └── requirements.txt
├── client/
│   ├── src/App.jsx
│   ├── src/main.jsx
│   ├── src/styles.css
│   ├── index.html
│   ├── package.json
│   └── vite.config.js
├── README.md
└── .gitignore
```

## Prerequisites
- Python 3.11+ (3.12 recommended)
- Node.js 20+
- Optional: Gemini API key if implementing the manual AI integration

## Run locally

### 1. Backend
```bash
cd server
python -m venv .venv
# Windows PowerShell:
.venv\Scripts\Activate.ps1
# Windows cmd:
# .venv\Scripts\activate.bat
# macOS/Linux:
# source .venv/bin/activate
pip install -r requirements.txt
copy .env.example .env   # Windows; macOS/Linux: cp .env.example .env
uvicorn app.main:app --reload --port 8000
```
Interactive API docs: http://localhost:8000/docs

The app works with `GEMINI_API_KEY=replace_me` and no live AI calls for sample dashboard/audits. Do not add an API key unless you also implement the TODOs in `app/ai_service.py`.

### 2. Frontend (new terminal)
```bash
cd client
npm install
npm run dev
```
Open http://localhost:5173. Optional `.env` in `client/`:
```env
VITE_API_URL=http://localhost:8000
```
If you change the client origin, update `FRONTEND_ORIGIN` in `server/.env` (comma-separated origins are supported).

### 3. Run tests
```bash
cd server
pytest
```
Build client:
```bash
cd client
npm run build
```

## Six-step demo flow
1. Open Finwise; fictional sample transactions load automatically.
2. Review the four metrics, monthly chart, category breakdown, and transaction table.
3. In the import panel, click **Use example** to fill example lines, or choose a file. Extraction requires Gemini integration; without it, the app clearly reports that it is not configured and keeps current data.
4. Scroll to **Run a finance audit** and click **Run agent audit**.
5. Inspect the StreamBox Premium +30% finding, the $588 annualized cost, and the agent trace.
6. Review the draft and choose **Approve simulation** or **Reject**. No message is sent.

## API
| Method | Path | Purpose |
|---|---|---|
| GET | `/api/health` | Backend status and key-configured flag |
| GET | `/api/sample` | Fictional starter transactions |
| POST | `/api/extract` | Multipart `text` and/or `file` → validated transactions |
| POST | `/api/audits` | Start bounded audit |
| GET | `/api/audits/{session_id}` | Read audit session |
| POST | `/api/audits/{session_id}/approval` | Record local approve/reject simulation |

## Teaching notes
- Pydantic validates transactions; unclear dates remain `null`.
- All financial arithmetic is deterministic Python/JavaScript, not delegated to an LLM.
- The planner can only select allowlisted tools, and the loop has a six-step cap.
- Tool arguments for leakage/draft actions are grounded back to detected findings.
- Every meaningful audit step is recorded with UTC timestamp, message and details.
- Uploaded files are held only in request memory and are not persisted or logged. Upload limit is 12 MB and text is capped at 100,000 characters.
- CSV parsing is performed in Python before normalisation. Duplicate transactions are not deduplicated.

## Manual Gemini integration
The requested integration boundary is intentionally stub-only. In `server/app/ai_service.py`, implement `_client()`, `extract_statement()` using Gemini structured output (`response_schema=Statement`, temperature 0), and optionally an LLM-backed `choose_next_step()` returning `AgentDecision`. Keep all AI access in this module, validate outputs, never trust model arguments, and retain the offline planner fallback. Do not add SDK calls in `main.py`.

## Troubleshooting
- **Frontend cannot connect:** ensure server is running at port 8000; set `VITE_API_URL` if needed and restart Vite.
- **Gemini not configured:** this is expected in demo mode; sample data and agent audits work without a key.
- **Gemini model/quota errors:** live extraction is not implemented until you hand-code the TODOs; after implementation, surface provider errors without logging sensitive statement contents.
- **Port in use:** stop the other process or change the server port and set matching `VITE_API_URL`; update `FRONTEND_ORIGIN` if the client port changes.
- **Sessions disappear:** expected; sessions are stored in process memory only.
