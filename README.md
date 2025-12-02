# X4C preorder exception dashboard

A Vertex AI / Gemini-powered analytics agent that answers sales questions, queries BigQuery live over Model Context Protocol (MCP), and delivers polished reports with a persistent history.

## Features

- Conversational sales analysis backed by Google Gemini or Vertex AI and FastAPI
- Natural language to SQL conversion with live BigQuery execution via MCP
- Next.js chat UI with collapsible/resizable history sidebar, swipe-to-delete entries, and chart/table views
- Per-response report downloads in PDF or CSV with consistent styling
- Router layer that separates small-talk from analytical questions for faster responses
- Scheduler-backed daily report delivery (configurable cron trigger + SMTP alerts)

## Setup

### 1. Install Dependencies

```powershell
pip install -r requirements.txt
```

### 2. Configure Environment Variables

Create a new `.env` file in the project root (or edit the existing one) and add your credentials. Examples for Vertex AI API key usage:
```
VERTEX_AI_API_KEY=your_vertex_ai_key
GOOGLE_API_KEY=your_vertex_ai_key
GOOGLE_CLOUD_PROJECT=your_project_id
GOOGLE_CLOUD_LOCATION=us-central1
GOOGLE_GENAI_USE_VERTEXAI=True
GENAI_MODEL_NAME=models/gemini-2.5-flash
BIGQUERY_PROJECT=your_project_id
BIGQUERY_LOCATION=us-central1
DATASET_NAME=test_data
TABLE_NAME=orders
```
If you prefer Google AI Studio (Makersuite) instead, set `GEMINI_API_KEY` and omit the Vertex-specific flags.

### 3. Get a Gemini API Key

1. Go to [Google AI Studio](https://makersuite.google.com/app/apikey)
2. Sign in with your Google account
3. Create a new API key
4. Copy it to your `.env` file

## Usage

### Run the Services

1. **Start the FastAPI router and chat backend**

    ```powershell
    python -m uvicorn backend.app:create_app --factory --reload
    ```

    - `POST /chat-router` classifies prompts via Gemini and decides whether BigQuery is required.
    - `POST /chat` streams analytical requests to `BigQuerySalesAgent` for SQL generation and execution.
    - `GET|POST /report` regenerates the latest analysis and streams PDF or CSV outputs.
    - When using Vertex AI, the agent talks to the REST endpoint at `<location>-aiplatform.googleapis.com` with model IDs like `models/gemini-2.5-flash`.

2. **Launch the Next.js chat UI**

    # X4C preorder exception dashboard

    A FastAPI + Next.js project that uses a Gemini/Vertex-AI driven agent to convert natural-language sales questions into read-only BigQuery SQL (via MCP), run live queries, and return narrative answers plus downloadable reports.

    ## Key Features

    - Conversational sales analysis (Gemini / Vertex AI)
    - Natural language → SQL with live BigQuery execution over Model Context Protocol (MCP)
    - Next.js chat UI with persistent history, resizable/collapsible sidebar, and per-response downloads
    - Export reports as PDF/DOCX/XLSX (backend uses `reportlab`, `python-docx`, `openpyxl`)
    - Router layer that classifies small-talk vs analytical intents for faster replies

    ## Quickstart (local)

    Prerequisites:

    - Python 3.10+ and `pip`
    - Node.js + `npm` (for the Next.js UI)
    - `toolbox.exe` (MCP) available on PATH if you want live BigQuery execution

    1) Install backend dependencies

    ```powershell
    cd C:\test123
    pip install -r requirements.txt
    ```

    2) Configure environment variables

    Create a `.env` file at the project root. Common variables:

    ```text
    GEMINI_API_KEY=your_gemini_or_vertex_key
    VERTEX_AI_API_KEY=your_vertex_key            # optional, for Vertex usage
    GOOGLE_CLOUD_PROJECT=your_project_id
    BIGQUERY_PROJECT=your_project_id
    BIGQUERY_LOCATION=us-central1
    DATASET_NAME=test_data
    TABLE_NAME=orders
    AUTH_MONGO_URI=mongodb+srv://...             # optional, enable MongoDB Atlas
    ```

    Notes:
    - If using Vertex AI, set `VERTEX_AI_API_KEY` and `GOOGLE_GENAI_USE_VERTEXAI=True` (see examples in `backend/ATLAS_README.md`).
    - If you prefer AI Studio/Gemini keys, set `GEMINI_API_KEY` instead.

    Optional validation URL:

    ```text
    API_KEY_VALIDATE_URL=https://api.example.com/v1/validate-key
    ```

    If provided, the health endpoint will call this URL with `Authorization: Bearer <key>` to verify the API key's validity. The health payload will include detailed API key status (valid, invalid, rate_limited, or validation error).

    3) Run the backend (development)

    ```powershell
    cd C:\test123
    python -m uvicorn backend.app:create_app --factory --reload
    ```

    This boots the FastAPI app created by `backend.app:create_app`. Endpoints of interest:

    - `POST /chat-router` — intent classification and quick replies
    - `POST /chat` — analytical requests routed to the agent
    - `GET|POST /report` — generate and stream report bytes
    - `GET /health` — health/readiness check

    4) Run the frontend (development)

    ```powershell
    cd C:\test123\chat-ui
    npm install
    npm run dev
    ```

    Open `http://localhost:3000` and chat. The UI calls the backend router (default `http://127.0.0.1:8000/chat-router`).

    ## Environment & Optional Integrations

    - MongoDB Atlas: optional user persistence — set `AUTH_MONGO_URI` to enable (see `backend/ATLAS_README.md`). The code falls back to `backend/data/users.json` if MongoDB is not configured.
    - MCP: The agent expects `toolbox.exe --prebuilt bigquery` for safe, read-only SQL execution. Ensure `toolbox.exe` is available in PATH.

## Scheduling Daily Reports

- The backend spins up `SchedulerService` during FastAPI startup. It keeps an APScheduler job running that can generate the same daily summary reports used by the UI and email them out via `EmailService`.
- Store SMTP credentials so the scheduler can deliver attachments:

```
SMTP_SERVER=smtp.gmail.com
SMTP_PORT=587
SMTP_USERNAME=your@email
SMTP_PASSWORD=app_specific_password
SMTP_FROM_EMAIL=optional+from@example.com
```

- The scheduler persists settings under `backend/data/schedule_config.json` (hour/minute/enabled) and `backend/data/recipients.json` (recipient groups + defaults). Those files are created automatically if absent.
- API paths to manage scheduling:

    - `GET /reports/config` / `POST /reports/config` to read/update the cron trigger (hour, minute, enabled flag). Updates immediately reconfigure the APScheduler job.
    - `GET /reports/recipients` / `POST /reports/recipients` / `PUT /reports/recipients/{index}` / `PUT /reports/recipients/{index}/set-default` / `DELETE /reports/recipients/{index}` manage who receives the automated emails.
    - `POST /reports/trigger` for manually firing report generation/email delivery (handy for smoke tests before scheduling a job).

- Use `verify_report_backend.py` to exercise the scheduler/agent combo without running the whole FastAPI server: it constructs the agent, scheduler, and email service, then optionally triggers `generate_and_send_report()` for a sanity check.

    ## Project Layout (important files)

    - `backend/` — FastAPI app factory and API routes (`backend/app.py`, `backend/api/routes`)
    - `backend/agents/` — agent implementations (e.g., `bigquery_sales_agent.py`)
    - `backend/services/` — auth, history, memory, and reporting helpers
    - `backend/data/` — sample users, saved history, memory fixtures
    - `chat-ui/` — Next.js application (page `app/page.tsx`, components under `components/chat`)
    - `requirements.txt` — Python dependencies
    - `chat-ui/package.json` — frontend dependencies and scripts

    ## Architecture Overview

-```mermaid
graph LR
    subgraph Frontend
        UI["Next.js chat UI"]
    end
    subgraph Backend
        Router["FastAPI router layer"] --> Health["/health endpoint + caching"]
        Router --> Chat["/chat router + agent"]
        Router --> Reports["/reports routes (scheduler)" ]
        Router --> Auth["/auth + session state"]
    end
    subgraph Services
        Agent["BigQuerySalesAgent"]
        Scheduler["SchedulerService (APScheduler) + EmailService"]
        Storage["backend/data/*.json" ]
    end
    UI --> Router
    Chat --> Agent
    Reports --> Scheduler
    Scheduler --> Agent
    Scheduler --> Email["SMTP / configured recipients"]
    Scheduler --> Storage
    Agent --> Data["BigQuery via MCP"]
    Health --> Storage
-```

    ## Development Notes

    - Backend creates the agent on startup via `create_app()` in `backend/app.py` and registers routes under `backend/api/routes`.
    - The router layer classifies intents (small-talk vs analytics) so short replies can be returned without BigQuery calls.
    - Reports are generated server-side and streamed to the browser — available formats may include `pdf`, `docx`, `xlsx` depending on installed packages.

    ## Troubleshooting

    - If you see API-key errors, double-check `.env` and restart the backend.
    - If BigQuery calls fail, verify `toolbox.exe` availability and `BIGQUERY_PROJECT`/permissions.
    - To test without BigQuery, run the app without `toolbox.exe` and exercise small-talk or mock agent responses.

    ## Useful Commands

    ```powershell
    # Install backend deps
    pip install -r requirements.txt

    # Start backend (dev)
    python -m uvicorn backend.app:create_app --factory --reload

    # Start frontend (dev)
    cd chat-ui
    npm install
    npm run dev
    ```

    ## References

    - `backend/ATLAS_README.md` — notes for MongoDB Atlas
    - `working.md` — higher-level architecture and end-to-end flow

    ----

    If you'd like, I can also:

    - add a short architecture diagram to `working.md` and `README.md`, or
    - add npm/pip scripts to simplify running both services together.
