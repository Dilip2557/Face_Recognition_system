# VisionPRO — Facial Recognition System

> A full-stack facial recognition application with a premium browser interface, FastAPI backend, MTCNN face detection, FaceNet-style embeddings using InceptionResnetV1/VGGFace2, cosine-similarity matching, duplicate-identity protection, SQLite persistence, live webcam recognition, and a database-backed Evaluation Lab.

---

## Overview

VisionPRO is a web-based facial recognition system designed for interactive demonstrations, experimentation, and internship/project evaluation.

The application combines a browser-based interface with a Python/FastAPI backend. Users can enroll identities using images or webcam captures, recognize faces from uploaded images or a live camera, manage enrolled identities, tune recognition thresholds, and evaluate the system against the currently stored dataset.

### Core pipeline

```text
Browser / Webcam / Image Upload
              │
              ▼
        FastAPI Backend
              │
              ▼
        MTCNN Detection
              │
              ▼
       Face Extraction
              │
              ▼
 InceptionResnetV1 (VGGFace2)
              │
              ▼
      Normalized Embedding
              │
              ▼
     Cosine Similarity
              │
              ▼
       Known / Unknown
              │
              ▼
          SQLite
```

---

## Features

### Recognition
- Upload an image and detect faces.
- Use a browser webcam for live recognition.
- Generate normalized face embeddings.
- Match against enrolled identities using cosine similarity.
- Reject low-confidence matches as **Unknown**.

### Identity Enrollment
- Enroll a person from one or more images.
- Capture enrollment images using the browser camera.
- Validate that each submitted image contains an appropriate face.
- Store reference images and embeddings in SQLite.

### Duplicate Protection
Enrollment uses multiple duplicate checks:

1. **Exact image duplicate** using SHA-256.
2. **Visual duplicate candidate** using perceptual hashing.
3. **Identity duplicate** using face-embedding similarity.

A new name is blocked when the incoming face is sufficiently similar to an already enrolled identity. New reference images can still be added to an existing person.

### People Directory
- View enrolled identities.
- View stored reference counts and metadata.
- Delete identities and associated samples.

### Evaluation Lab
- Evaluate recognition using the current SQLite dataset.
- Use leave-one-out matching for identities with multiple reference samples.
- Avoid treating single-reference identities as artificially perfect test cases.
- Calculate:
  - Accuracy
  - False Accept Rate (FAR)
  - False Reject Rate (FRR)
  - Detection Failure Rate
  - Per-identity results
  - Per-sample decisions
- Persist evaluation runs and results in SQLite.
- Render evaluation metrics in the interface.

### Demo Dataset
- Optional demo-data loading from configured public sources.
- Downloaded images are converted and stored in the runtime SQLite database.
- Source URLs and attribution information are recorded in `demo_sources`.

### Persistence
All application state is stored in SQLite rather than browser `localStorage` or JSON runtime stores.

The database can contain:

- People and identity metadata
- Face samples
- Image hashes
- Face embeddings
- Recognition activity
- Matching settings
- Evaluation runs
- Evaluation results
- Demo-source metadata

---

## Technology Stack

| Layer | Technology |
|---|---|
| Frontend | HTML, CSS, JavaScript |
| Backend | FastAPI |
| Server | Uvicorn |
| Face Detection | MTCNN (`facenet-pytorch`) |
| Face Embeddings | InceptionResnetV1 / VGGFace2 |
| Similarity | Cosine similarity |
| Image Processing | Pillow, OpenCV |
| ML Runtime | PyTorch, TorchVision |
| Database | SQLite |
| Testing | Pytest |
| Deployment | Docker / GitHub Codespaces |

---

## Project Structure

```text
Facial_Recognition_System-main/
│
├── backend/
│   ├── __init__.py
│   ├── main.py              # FastAPI routes and application logic
│   ├── face_engine.py       # Face detection, preprocessing, embeddings, matching
│   ├── database.py          # SQLite persistence and schema initialization
│   └── data/
│       └── face_recognition.db   # Runtime database (created automatically)
│
├── frontend/
│   ├── index.html            # Application UI
│   ├── styles.css            # Premium responsive styling and animations
│   └── app.js                # Frontend interaction and API integration
│
├── database/
│   └── schema.sql            # Database schema reference
│
├── tests/
│   ├── test_duplicate_logic.py
│   ├── test_evaluation_persistence.py
│   ├── test_evaluation_regression.py
│   └── test_ephemeral_config.py
│
├── .devcontainer/
│   └── devcontainer.json     # GitHub Codespaces configuration
│
├── .github/
│   └── workflows/
│       └── python-tests.yml  # Automated test workflow
│
├── Dockerfile
├── requirements.txt
├── GITHUB_DEPLOYMENT.md
├── VERIFICATION.md
├── run_windows.bat
├── run_public_demo.bat
├── .gitignore
└── README.md
```

---

## Requirements

For the pinned dependency stack, use **Python 3.11**.

The project requirements include:

```text
fastapi==0.115.0
uvicorn==0.30.6
torch==2.4.1+cpu
torchvision==0.19.1+cpu
facenet-pytorch==2.5.3
numpy==1.26.4
python-multipart==0.0.9
pillow==10.4.0
opencv-python-headless==4.10.0.84
```

> Python 3.14 is not recommended for this pinned PyTorch/FaceNet stack.

---

# Local Installation — Linux / Kali

## 1. Clone the repository

```bash
git clone https://github.com/Dilip2557/Facial_Recognition_System.git
cd Facial_Recognition_System
```

## 2. Create a Python 3.11 virtual environment

If Python 3.11 is installed:

```bash
python3.11 -m venv .venv
```

Activate it:

```bash
source .venv/bin/activate
```

Verify:

```bash
python --version
```

Expected:

```text
Python 3.11.x
```

If your Linux distribution does not ship Python 3.11, use a Python version manager such as `uv` or use Docker.

## 3. Install dependencies

Using `pip`:

```bash
python -m pip install --upgrade pip
python -m pip install -r requirements.txt
```

Using `uv`:

```bash
uv pip install -r requirements.txt
```

## 4. Start the application

Run from the **project root**:

```bash
python -m uvicorn backend.main:app --host 0.0.0.0 --port 8000
```

Open:

```text
http://127.0.0.1:8000/
```

API documentation:

```text
http://127.0.0.1:8000/docs
```

### Important import rule

Because the backend uses package-relative imports such as:

```python
from . import database
```

start Uvicorn from the project root using:

```bash
python -m uvicorn backend.main:app --host 0.0.0.0 --port 8000
```

Do **not** start it from inside `backend/` with `main:app`.

---

# Windows

From Command Prompt:

```cmd
cd C:\path\to\Facial_Recognition_System
run_windows.bat
```

Then open:

```text
http://127.0.0.1:8000/
```

Swagger/API documentation:

```text
http://127.0.0.1:8000/docs
```

---

# Docker

The included Dockerfile uses Python 3.11.

Build:

```bash
docker build -t visionid .
```

Run:

```bash
docker run --rm -p 8000:8000 visionid
```

Open:

```text
http://127.0.0.1:8000/
```

The Docker configuration defaults to ephemeral demo storage.

---

# GitHub Codespaces

This repository includes a `.devcontainer/devcontainer.json` configured for GitHub Codespaces.

### Steps

1. Push the repository to GitHub.
2. Open the repository on GitHub.
3. Select **Code → Codespaces → Create codespace on main**.
4. Wait for the development container to finish installing dependencies.
5. The application starts on port `8000`.
6. Open the **Ports** panel and use the forwarded URL.
7. For an externally accessible demo, set port `8000` visibility to **Public** where appropriate.

The Codespaces configuration runs the application in ephemeral mode so runtime enrollment data does not become a permanent repository artifact.

See `GITHUB_DEPLOYMENT.md` for the repository-specific deployment notes.

---

# Configuration

The application supports environment variables for deployment behavior.

| Variable | Purpose | Typical value |
|---|---|---|
| `VISIONPRO_EPHEMERAL` | Use temporary runtime storage | `1` or `0` |
| `VISIONPRO_RESET_ON_STARTUP` | Reset runtime DB when the process starts | `1` or `0` |
| `VISIONPRO_IDLE_RESET_MINUTES` | Reset an ephemeral runtime after inactivity | `120` |
| `VISIONPRO_DATA_DIR` | Override the SQLite data directory | Custom path |

### Local persistent mode

Default behavior stores the database under:

```text
backend/data/face_recognition.db
```

### Public demo mode

For temporary demos:

```bash
export VISIONPRO_EPHEMERAL=1
export VISIONPRO_RESET_ON_STARTUP=1
export VISIONPRo_IDLE_RESET_MINUTES=120
python -m uvicorn backend.main:app --host 0.0.0.0 --port 8000
```

In ephemeral mode, the application uses a temporary runtime directory and recreates the database when configured to reset on startup. An idle timeout can also reset the runtime database.

---

# Recognition Settings

The current default values are:

| Setting | Default |
|---|---:|
| Recognition threshold | `0.65` |
| Duplicate identity threshold | `0.80` |
| Activity history limit | `500` events |
| Default idle reset | `120` minutes in ephemeral mode |

The recognition threshold controls Known/Unknown acceptance. The duplicate threshold is intentionally stricter and is used during enrollment to prevent the same identity from being registered under different names.

Thresholds can be viewed and updated through the application settings interface and API.

---

# API Endpoints

## Health and status

```text
GET /api/health
GET /api/stats
GET /api/activity
GET /api/model-info
```

## People

```text
GET    /api/people
POST   /api/people
DELETE /api/people
```

## Recognition

```text
POST /api/recognize
```

## Threshold settings

```text
GET  /api/settings/threshold
POST /api/settings/threshold
```

## Evaluation

```text
GET  /api/evaluation/status
POST /api/evaluate
GET  /api/evaluation/latest
```

## Demo dataset / runtime mode

```text
GET  /api/demo/status
POST /api/demo/load
GET  /api/demo/mode
POST /api/demo/reset
```

Interactive API documentation is available at:

```text
http://127.0.0.1:8000/docs
```

---

# How Recognition Works

1. The browser supplies an image or webcam frame.
2. The backend loads and preprocesses the image.
3. MTCNN detects the face.
4. The face is extracted and prepared for embedding generation.
5. InceptionResnetV1 generates the face representation.
6. The embedding is normalized.
7. Cosine similarity is calculated against enrolled reference embeddings.
8. The highest score is compared with the configured recognition threshold.
9. The result is returned as a known identity or Unknown.
10. Recognition activity can be stored in SQLite.

---

# Enrollment and Duplicate Detection

For an enrollment request, each candidate image goes through validation and duplicate checks before it is stored.

### Exact duplicate

A canonicalized image receives a SHA-256 hash. Duplicate SHA-256 values are rejected.

### Perceptual duplicate

A perceptual hash is used to identify visually similar re-encoded or resized versions of the same image.

### Identity duplicate

The incoming face embedding is compared against stored face samples. A similarity score at or above the duplicate threshold (`0.80`) prevents the same face from being enrolled under another identity.

The duplicate check considers previously stored face samples rather than relying only on the first image of a person.

---

# Evaluation Methodology

The Evaluation Lab reads its dataset from the current SQLite database.

For known identities with multiple reference samples, the evaluator uses **leave-one-out matching**, meaning the test sample is not matched against its own stored embedding.

For identities with only one reference image, the system can mark the sample as not evaluable rather than turning a self-match into an artificial perfect score.

Evaluation data can include unknown samples with no associated `person_id`.

The evaluation subsystem records both summary metrics and individual decisions in SQLite.

---

# Testing

Run the test suite with:

```bash
pytest -q
```

The repository includes regression and persistence tests covering duplicate logic, evaluation behavior, and ephemeral configuration.

GitHub Actions is configured under:

```text
.github/workflows/python-tests.yml
```

---

# Webcam Usage

The application uses the browser's camera APIs.

For local development, use:

```text
http://127.0.0.1:8000/
```

For a hosted deployment, use an **HTTPS** URL so that modern browsers can grant camera access.

When prompted by the browser, allow camera access for the site.

---

# Privacy and Security

Face images and face embeddings are biometric information and should be handled as sensitive data.

### Before public or production use

- Do not commit real face databases to GitHub.
- Do not commit `.env` files containing secrets.
- Do not expose an administrative enrollment interface without access control.
- Add authentication and authorization before production deployment.
- Restrict access to biometric records and stored embeddings.
- Use HTTPS for remote access.
- Establish a data-retention and deletion policy.
- Obtain the appropriate user consent and comply with applicable privacy/data-protection requirements.

The repository's `.gitignore` is intended to keep runtime databases, virtual environments, caches, and common sensitive artifacts out of source control.

---

# Limitations

### Recognition accuracy

Performance can vary with lighting, pose, expression, motion blur, image quality, occlusion, and camera conditions.

### Fixed threshold

The initial recognition threshold is configurable but is not guaranteed to be optimal across every environment.

### No liveness detection

The current system identifies faces but does not determine whether the presented face belongs to a live person or a photo/video presentation.

### CPU inference

CPU-based face detection and embedding generation can increase latency, especially during repeated webcam recognition or larger evaluation runs.

### SQLite scalability

SQLite is appropriate for small-scale demos and local applications, but a server-based database and vector-search system may be more appropriate for large deployments and concurrent users.

### Evaluation dataset size

Evaluation quality depends on the number and diversity of known and unknown samples available in the database.

---

# Future Improvements

Potential next steps include:

- GPU acceleration and optimized inference.
- ONNX/TensorRT or equivalent model optimization.
- Frame skipping and face tracking for smoother webcam recognition.
- Improved face alignment and preprocessing.
- Automated threshold calibration using a validation dataset.
- Liveness / anti-spoofing detection.
- Larger and more diverse evaluation datasets.
- ROC/DET curves and richer evaluation reporting.
- PostgreSQL or another server database for larger deployments.
- FAISS or a vector database for large-scale embedding search.
- Authentication, authorization, audit controls, and role-based access.

---

# Project Demonstration Flow

A typical demonstration can follow this sequence:

```text
1. Open Dashboard
       ↓
2. Enroll Identity
       ↓
3. Capture / Upload Reference Images
       ↓
4. View Identity in People Directory
       ↓
5. Open Recognize
       ↓
6. Upload an image or start Webcam Recognition
       ↓
7. View Known / Unknown result and similarity
       ↓
8. Open Evaluation Lab
       ↓
9. Run Evaluation
       ↓
10. Review Accuracy / FAR / FRR / Detection Failure metrics
```

---

# Development Notes

### Run from the project root

Use:

```bash
cd Facial_Recognition_System-main
python -m uvicorn backend.main:app --host 0.0.0.0 --port 8000
```

Avoid launching `main.py` directly from inside `backend/`, because the backend uses package-relative imports.

### Runtime database

The default database is created automatically. The schema is maintained in the Python backend and documented in:

```text
database/schema.sql
```


# Author

**Dilip**

Facial Recognition System — VisionPRO

---

## Acknowledgements

This project uses open-source libraries and pretrained models including FastAPI, PyTorch, TorchVision, facenet-pytorch, OpenCV, Pillow, SQLite, and related Python tooling.

Please review and comply with the licenses and usage terms of all third-party dependencies and any public image sources used by the optional demo dataset.
