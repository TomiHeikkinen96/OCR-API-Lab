# Project Plan

This plan is written as a learning roadmap. Each phase should produce something testable, explainable, and useful on its own. The project should grow from a transparent baseline into a richer OCR system without becoming a black box.

## Phase 0: Architecture and Documentation

Purpose:

Establish the project direction, vocabulary, structure, and implementation rules before writing the first service code.

Deliverables:

- README with project positioning, architecture, and public-facing story.
- AGENTS.md with implementation guidance for future coding agents.
- Initial phased plan.
- Agreed internal model for OCR regions, crops, overlays, and metrics.

Learning goals:

- Understand the difference between OCR as a product feature and OCR as a pipeline.
- Separate detection, preprocessing, recognition, rendering, and evaluation conceptually.
- Decide what should be observable before building anything.

Test/evaluation gate:

- Documentation clearly explains what the project is, what the first implementation should do, and how later phases should avoid turning into a single opaque tool wrapper.

## Phase 1: Minimal Local API Baseline

Purpose:

Create the first working API that accepts an image and returns OCR results with visual evidence.

Deliverables:

- FastAPI app with `POST /v1/ocr`.
- Multipart image upload.
- Image decode and validation.
- PaddleOCR baseline engine.
- JSON response with text, boxes/polygons, confidence, image dimensions, and timing.
- Annotated overlay image returned as either base64 or a generated file path.
- Basic Dockerfile and docker-compose setup.

Learning goals:

- Learn how image uploads move through an API.
- Understand the shape of OCR model outputs.
- See how bounding boxes, polygons, confidence, and recognized text relate to the original image.
- Learn the difference between API latency, model latency, and rendering time.

Test/evaluation gate:

- A sample image can be sent to the API.
- The response includes structured OCR regions.
- The overlay visually matches the detected text.
- The same sample produces repeatable output on repeated runs.

## Phase 2: Internal Pipeline Model

Purpose:

Refactor the baseline into clear pipeline stages and stable internal data structures.

Deliverables:

- Engine-neutral `OCRRegion`, `OCRResult`, `ImageInfo`, and `PipelineTiming` models.
- Engine adapter interface.
- PaddleOCR implementation behind the adapter.
- Separate modules for decode, preprocessing, crop extraction, recognition, rendering, and response mapping.
- Unit tests for geometry and response schemas.

Learning goals:

- Understand why ML systems need stable internal contracts.
- Learn how to isolate third-party library output from the rest of the application.
- Make later engine comparisons possible.

Test/evaluation gate:

- The API response remains stable while the internals become modular.
- PaddleOCR-specific data does not leak through the whole codebase.
- Geometry tests cover bounding boxes, polygons, and crop coordinates.

## Phase 3: Visual Debugging and Crop Inspection

Purpose:

Make the OCR process inspectable by exposing intermediate artifacts.

Deliverables:

- Optional `debug=true` request mode.
- Saved or returned crop images for each text region.
- Region IDs that connect JSON entries, crops, and overlay labels.
- Overlay variants: boxes only, confidence heatmap, reading order, and low-confidence highlights.
- Per-region timing where practical.

Learning goals:

- Learn to debug OCR failures visually.
- Understand how crop quality affects recognition.
- Build intuition around false positives, missed detections, skew, blur, glare, and low contrast.

Test/evaluation gate:

- Every recognized region can be traced back to a visible crop.
- Low-confidence regions are easy to find.
- Debug output helps explain at least one OCR failure case.

## Phase 4: Preprocessing Experiments

Purpose:

Introduce controlled image normalization without pretending there is one universal best preprocessing method.

Deliverables:

- Named preprocessing profiles: `default`, `document`, `photo`, and `aggressive`.
- Whole-image preprocessing options.
- Per-crop preprocessing options.
- Perspective rectification for polygon crops.
- Confidence-based retry strategy for low-confidence crops.
- Metadata describing which preprocessing path produced each result.

Learning goals:

- Learn common OCR preprocessing techniques: grayscale conversion, denoising, deskewing, thresholding, morphology, contrast normalization, and sharpening.
- Understand when preprocessing helps and when it destroys signal.
- Compare deterministic image processing with model-based OCR behavior.

Test/evaluation gate:

- The same image can be processed with multiple profiles.
- The response records which preprocessing profile was used.
- At least one documented sample shows improved results from preprocessing.
- At least one documented sample shows preprocessing making results worse.

## Phase 5: Evaluation Harness

Purpose:

Move from "it looks better" to measurable OCR quality.

Deliverables:

- `samples/` dataset structure for images and expected text.
- Evaluation CLI or script.
- Character error rate and word error rate metrics.
- Per-image and aggregate reports.
- Regression test set for known examples.
- Markdown evaluation reports suitable for portfolio updates.

Learning goals:

- Learn how OCR quality is measured.
- Understand accuracy, confidence, precision, recall, false positives, and false negatives.
- Build the habit of measuring changes instead of trusting visual impressions.

Test/evaluation gate:

- Evaluation can run locally with one command.
- Reports show accuracy before and after preprocessing changes.
- A future engine can be compared against the baseline.

## Phase 6: Multi-Engine Comparison

Purpose:

Make the project engine-agnostic and compare real OCR tradeoffs.

Deliverables:

- Additional adapters for docTR, EasyOCR, and Tesseract as optional engines.
- Configurable engine selection.
- Common response model across all engines.
- Side-by-side benchmark report.
- Notes on setup complexity, speed, accuracy, output shape, and GPU support.

Learning goals:

- Understand how OCR engines differ internally and operationally.
- Learn what makes one engine better for documents, another better for scene text, and another useful as a baseline.
- Practice building fair comparisons.

Test/evaluation gate:

- At least two engines can process the same image through the same API shape.
- Evaluation reports compare engine outputs using the same metrics.
- Documentation explains the tradeoffs in practical language.

## Phase 7: Local GPU Inference and Performance

Purpose:

Optimize local inference for the target workstation while keeping CPU fallback available.

Deliverables:

- Docker Compose GPU profile.
- Clear local setup notes for NVIDIA GPU usage.
- Model cache volume.
- Warmup endpoint or startup model loading.
- Timing breakdown for CPU versus GPU runs.
- Optional ONNX Runtime or engine-specific acceleration experiments.

Learning goals:

- Learn how local GPU inference is exposed to containers.
- Understand model loading, warm starts, memory usage, batch size, and latency.
- Compare developer convenience against runtime performance.

Test/evaluation gate:

- API can run locally in Docker.
- GPU availability can be detected and reported.
- Benchmark report compares CPU and GPU behavior on the same image set.

## Phase 8: Advanced Document Understanding

Purpose:

Extend from raw OCR into document-aware structure.

Deliverables:

- Reading order reconstruction improvements.
- Line and paragraph grouping.
- Optional layout labels such as title, table, footer, or body text.
- Markdown export.
- JSON export suitable for downstream LLM/RAG experiments.

Learning goals:

- Understand the difference between OCR and document understanding.
- Learn why layout, reading order, and grouping matter.
- Build outputs that are useful beyond plain text extraction.

Test/evaluation gate:

- Multi-column or document-like examples produce readable ordered text.
- Markdown export is useful enough to inspect manually.
- The project can explain where OCR ends and document parsing begins.

## Phase 9: Portfolio Polish

Purpose:

Package the project so it communicates well to engineers, recruiters, and future collaborators.

Deliverables:

- Screenshots or GIFs of upload, overlay, and debug views.
- Example API responses.
- Benchmark table.
- Architecture diagram.
- LinkedIn-friendly project summary.
- Clear "what I learned" notes.

Learning goals:

- Practice presenting ML engineering work clearly.
- Show tradeoffs, failure cases, and iteration instead of only final results.
- Turn technical exploration into a credible public project.

Test/evaluation gate:

- A reader can understand what the project does in under one minute.
- A technical reader can understand the architecture in under five minutes.
- The repository demonstrates both implementation skill and learning process.

