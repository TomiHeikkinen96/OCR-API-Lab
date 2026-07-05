# Agent Guide

This repository is a learning-first, portfolio-quality OCR API project. Future agents should preserve that intent: build useful software, but keep the pipeline explainable, testable, and easy for the human owner to understand phase by phase.

## Product Intent

The project is a local-first OCR API that accepts an image and returns structured OCR results plus visual overlays. It should eventually support local GPU inference on an RTX 4070 Super through Docker, while keeping CPU fallback and small-footprint experiments possible.

This must not become a thin wrapper around one OCR library. Use strong tools where they make sense, but expose the stages: decode, preprocess, detect, crop, normalize, recognize, reconstruct, render, and evaluate.

## Documentation Contract

- Keep `README.md` short, polished, and portfolio-friendly.
- Keep the current project phase in `README.md`.
- Keep detailed roadmap material in `docs/PLAN.md`.
- Do not put "current phase" status in `docs/PLAN.md`; it should remain a passive roadmap.
- When implementation changes the architecture, update the docs in the same change.
- Prefer diagrams and concrete examples over vague claims.

## Implementation Principles

- Build in phases. Each phase should produce something runnable or verifiable.
- Keep modules small and named after pipeline stages.
- Use typed internal models for OCR regions, images, timings, and engine output.
- Wrap third-party OCR engines behind adapters.
- Do not let PaddleOCR, docTR, EasyOCR, or Tesseract-specific response shapes leak through the application.
- Return visual evidence wherever possible: overlays, crops, region IDs, confidence scores, timings, and preprocessing metadata.
- Prefer deterministic, inspectable image processing before adding complex ML customization.
- Add abstractions only when they enable engine swapping, evaluation, or clearer learning.

## Preferred Initial Stack

- Python
- FastAPI
- Pydantic
- PaddleOCR as the first OCR engine
- OpenCV, Pillow, and NumPy for image processing
- Docker and Docker Compose
- NVIDIA GPU support once the baseline works
- Pytest for tests

Optional later tools:

- docTR for a clear two-stage OCR learning path
- EasyOCR for a simple comparative baseline
- Tesseract for classic OCR comparison
- ONNX Runtime for optimization experiments
- Custom detector models only after evaluation infrastructure exists

## Suggested Project Shape

```text
ocr_api/
  main.py
  api/
    routes.py
    schemas.py
  core/
    pipeline.py
    models.py
    config.py
  engines/
    base.py
    paddle_engine.py
  image/
    decode.py
    preprocess.py
    crop.py
    render.py
  eval/
    datasets.py
    metrics.py
tests/
docs/
  PLAN.md
docker-compose.yml
Dockerfile
README.md
AGENTS.md
```

This structure is a guide, not a cage. Match actual code to the phase being implemented.

## API Direction

The first API should be simple:

```http
POST /v1/ocr
```

Expected input:

- Multipart image upload.
- Optional engine selection.
- Optional preprocessing profile.
- Optional debug flag.

Expected output:

- Full reconstructed text.
- Image dimensions.
- OCR regions with IDs, polygons, bounding boxes, text, confidence, and metadata.
- Annotated overlay image or path.
- Timing breakdown.

## Testing and Evaluation

Every phase should include a test or evaluation gate.

Early phases can use:

- Schema tests.
- Geometry tests.
- Smoke tests against a sample image.
- Snapshot-like checks for response shape.

Later phases should add:

- Character error rate.
- Word error rate.
- Per-image benchmark reports.
- CPU versus GPU timing reports.
- Engine comparison reports.

Do not claim an OCR change is better without either a visual debug artifact or a metric.

## Learning Notes

When adding a meaningful feature, include short notes in docs or comments explaining what the feature teaches. The project owner wants to understand OCR beyond "load tool, send image, get result."

Good learning-oriented additions:

- Explain why polygons are used instead of only rectangles.
- Explain why preprocessing profiles can help or hurt.
- Explain how confidence is used and where it can mislead.
- Explain how OCR differs from document understanding.
- Explain CPU/GPU tradeoffs with actual measurements.

## Portfolio Bias

Favor choices that make the project demonstrable:

- Clear API examples.
- Before/after overlays.
- Debug crops.
- Benchmark tables.
- Honest failure cases.
- Small, named milestones.

The best version of this project shows thoughtfulness, not just model accuracy.

