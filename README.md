# Virtual Fashion Try-On

A local virtual fitting room app. The frontend lets a user upload a person photo,
upload a garment photo, choose the garment category, and preview the generated
try-on result. The backend exposes a FastAPI API that loads the FASHN virtual
try-on pipeline and returns a PNG image.

## Project Structure

```text
.
+-- backend/   FastAPI service for model loading and try-on inference
`-- frontend/  Next.js single-page upload and preview UI
```

## Prerequisites

- Python `3.12.11` or newer
- `uv` for Python dependency management
- Node.js and `pnpm` for the frontend
- FASHN model weights available on the machine
- A CUDA-capable GPU is recommended for practical inference speed

## Model Files

By default, the backend looks for model files in `backend/models`.

Expected layout:

```text
backend/models/
+-- model.safetensors
`-- dwpose/
    +-- yolox_l.onnx
    `-- dw-ll_ucoco_384.onnx
```

To use another model directory, set `FASHN_WEIGHTS_DIR` before starting the
backend.

## Backend Setup

```bash
cd backend
uv sync
```

Optional backend environment variables:

```bash
FASHN_WEIGHTS_DIR=./models
FASHN_DEVICE=cuda
CORS_ORIGINS=http://localhost:3000,http://127.0.0.1:3000
HOST=0.0.0.0
PORT=8000
RELOAD=false
```

Start the backend:

```bash
uv run python run.py
```

The API will be available at `http://localhost:8000`.

Useful backend URLs:

- Health check: `http://localhost:8000/health`
- API docs: `http://localhost:8000/docs`

## Frontend Setup

In a second terminal:

```bash
cd frontend
pnpm install
pnpm dev
```

Open `http://localhost:3000`.

The frontend calls `http://localhost:8000` by default. To point it at another
backend, create `frontend/.env.local`:

```bash
NEXT_PUBLIC_API_BASE_URL=http://localhost:8000
```

If the frontend runs on a different origin, also update the backend
`CORS_ORIGINS` value.

## How To Use

1. Start the backend and wait for the model pipeline to load.
2. Start the frontend.
3. Upload a person image.
4. Upload a garment image.
5. Select one of `tops`, `bottoms`, or `one-pieces`.
6. Click `Generate`.
7. The generated PNG appears in the result panel.

## API Contract

`POST /try-on`

Request type: `multipart/form-data`

Fields:

```text
person_image=<image file>
garment_image=<image file>
category=tops | bottoms | one-pieces
```

Successful response:

```text
Content-Type: image/png
```

Example:

```bash
curl -X POST http://localhost:8000/try-on \
  -F person_image=@/path/to/person.png \
  -F garment_image=@/path/to/garment.png \
  -F category=tops \
  --output output.png
```

## Checks

Backend tests:

```bash
cd backend
uv run python -m unittest discover -s tests -v
```

Frontend checks:

```bash
cd frontend
pnpm lint
pnpm build
```

## Troubleshooting

If `/health` reports `model_loaded: false` or the backend fails during startup,
check that `FASHN_WEIGHTS_DIR` points to the directory containing the model
files shown above.

If the frontend cannot call the backend, confirm the backend is running on the
same URL as `NEXT_PUBLIC_API_BASE_URL` and that `CORS_ORIGINS` includes the
frontend origin.

If inference is slow, confirm the machine has a supported GPU and start the
backend with `FASHN_DEVICE=cuda`.
