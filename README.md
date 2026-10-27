# ClearMark OMR — Student Portal and Teacher Grading Desk

ClearMark is a React/Vite and FastAPI/OpenCV app for the A4 Kathmandu Metropolitan City Class 11 Scholarship Examination answer sheet included in `demo/`. It provides a public student dashboard and upload portal, plus a separate teacher/admin dashboard protected by HTTP Basic sign-in. Teachers create answer keys; students can see published test names and submit scans without creating an account.

## What is included

```text
ClearMark-OMR/
├── README.md                 Local setup, use, roles, security, and deployment guide
├── Dockerfile                Builds and serves frontend plus API as one service
├── .gitignore                Ignores dependencies, generated builds, keys, and secrets
├── backend/
│   ├── main.py               FastAPI API, teacher authentication, OpenCV scan and grading
│   ├── requirements.txt      Python dependencies
│   └── .gitignore            Backend generated files and answer-key data
├── frontend/
│   ├── index.html            Browser entry point
│   ├── package.json          npm scripts and dependencies
│   ├── vite.config.js        Vite build/dev server settings
│   ├── tailwind.config.js    Tailwind source configuration
│   ├── postcss.config.js     Tailwind CSS processing
│   ├── .env.example          Optional API URL
│   └── src/
│       ├── main.jsx          Starts the React app
│       ├── App.jsx           Student portal, teacher/admin sign-in, setup, upload, results
│       └── style.css         Responsive layout and visual design
└── demo/
    └── omr-demo-filled.png   Sample sheet with 100 marks; key repeats A, B, C, D
```

## Requirements

- Python 3.10 or newer
- Node.js 20 or newer and npm
- Docker Desktop only if using the Docker deployment option

## Run locally on Windows

Open PowerShell in the unzipped `ClearMark-OMR` folder.

### 1. Start the API

```powershell
cd backend
python -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install -r requirements.txt
$env:TEACHER_USERNAME = "teacher"
$env:TEACHER_PASSWORD = "choose-a-private-password"
uvicorn main:app --reload --host 127.0.0.1 --port 8000
```

Keep that terminal open. The API health check is `http://localhost:8000/api/health`; interactive API docs are at `http://localhost:8000/docs`. Test answer keys are saved to `backend/data/` (or to the directory set by `OMR_DATA_DIR`).

### 2. Start the website

Open a second PowerShell window in the project folder:

```powershell
cd frontend
npm install
npm run dev
```

Open the local URL printed by Vite, usually `http://localhost:5173`.

### macOS or Linux

Use `python3 -m venv .venv`, activate with `source .venv/bin/activate`, and install with `python -m pip install -r requirements.txt`. Export teacher credentials before starting Uvicorn:

```bash
export TEACHER_USERNAME=teacher
export TEACHER_PASSWORD='choose-a-private-password'
uvicorn main:app --reload --host 127.0.0.1 --port 8000
```

In another terminal, run `cd frontend && npm install && npm run dev`.

## How to use it

### Teacher / Admin

Choose **Teacher / Admin sign-in** in the left sidebar and use the credentials configured above. Teacher and admin share the same role. From the teacher dashboard, create a test (20, 40, 60, 80, or 100 questions), choose one A/B/C/D answer per question, and save. Test creation and direct answer-key reads require teacher credentials.

### Student

The Student Portal opens without an account. Choose an available assessment, upload a JPG, PNG, or WEBP photo/scan (12 MB maximum), confirm the OMR checklist, and run **Scan & grade**. The result shows correct, incorrect, unclear/unanswered, and a question-by-question review. The review uses the same five vertical groups as the paper: 1–20, 21–40, 41–60, 61–80, and 81–100.

## OMR rules shown during upload

The upload screen reminds students to follow the sheet’s printed rules:

1. Write the roll number in the boxes and fill the matching digit bubbles.
2. Use a black pen.
3. Fill the selected bubble completely.
4. Mark only one answer per question. Multiple dark bubbles are reported as ambiguous/unanswered.
5. Do not erase or alter answers and do not add stray marks or rough work.
6. Fill in the Question Set correctly when the sheet asks for it.
7. Photograph the full page upright, flat, in focus, evenly lit, and with all corners visible.

The scanner is calibrated to the included Kathmandu A4 five-column sheet. It rejects unsupported file types, empty/corrupt images, files above 12 MB, and images smaller than 700 pixels on a side. It corrects perspective when it can find the paper edges. Human review is still required for unclear marks and before recording official grades. Uploaded images are processed in memory and are not saved by the app.

## Deploy for free (demo / classroom preview)

The included Dockerfile builds the React site and serves it with FastAPI from one service. Render currently offers free web services suitable for demos. Free instances can sleep after 15 minutes without traffic, take about a minute to wake, and have an ephemeral filesystem; answer-key files can disappear after restarts or deploys. Render says free services are for testing and hobby projects, not production. See [Render's free-service limits](https://render.com/docs/free) and [filesystem FAQ](https://render.com/docs/faq).

1. Put this folder in a GitHub repository (the Dockerfile must be at the repository root).
2. In Render, create a **Web Service** from that repository and select the **Free** instance type with Docker runtime.
3. Set `TEACHER_USERNAME` and `TEACHER_PASSWORD` in the service environment. Set `OMR_DATA_DIR=/data` and `CORS_ORIGINS` to the deployed site origin if the site and API are split across domains.
4. Deploy and open the service URL. Confirm `/api/health` responds with `{"status":"ok"}`.

For a no-cost preview, accept that free-host test data may be lost and the first visit after idle may be slow. Do not use this file-based store for permanent school records. For dependable grading, use a persistent database and storage plan, HTTPS, backups, and your institution's required student-data controls. Do not put real teacher passwords in source files or commit `.env` files.

## Deploy with Docker

From this project folder:

```bash
docker build -t clearmark-omr .
docker run --rm -p 8000:8000 -e TEACHER_USERNAME=teacher -e TEACHER_PASSWORD='choose-a-private-password' -v clearmark-data:/data clearmark-omr
```

Open `http://localhost:8000`. Docker serves both the React build and API on the same origin. A Docker volume preserves answer keys between local container restarts. Production hosts should provide HTTPS and a persistent volume or database.

## Production environment svettings

- `TEACHER_USERNAME`, `TEACHER_PASSWORD`: required to enable teacher/admin operations.
- `OMR_DATA_DIR`: writable persistent directory for answer-key JSON files.
- `CORS_ORIGINS`: comma-separated exact frontend origins when frontend and API have different domains. Default: `http://localhost:5173`.
- `VITE_API_URL`: frontend API origin at build time when deploying the frontend separately. Leave unset for the included same-origin Docker deployment.

The JSON store supports a single backend instance. Before scaling to multiple workers, move test storage to a shared database. Basic authentication must only be used over HTTPS in deployment.
