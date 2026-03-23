# Spec: Hex Pattern Recognition Benchmark

A benchmark that builds from BlindTest-style geometric primitives up to recognizing noisy hexagonal grid patterns in SEM-like images. Two versions: a brief statistically significant run (10-30 min) and a comprehensive version (few hours).

## Hardware

- **Primary:** ASUS GX10 (Blackwell GPU, 128 GB VRAM)
- **Secondary:** Machine with RTX 4090, 64 GB RAM

## Task Hierarchy

### Level 1 — BlindTest Baseline (replicate core tasks)

Replicates a subset of the original BlindTest to establish baseline VLM performance and validate our evaluation pipeline matches published results.

| Task | Description | Images | Metric |
|------|-------------|--------|--------|
| 1a. Line intersections | Count intersections (0, 1, 2) of two 2-segment piecewise-linear functions | Vary line width, image size, distance between lines | Accuracy (3-class) |
| 1b. Circle overlap | Are two circles overlapping? Yes/No | Vary distance, diameter, orientation | Accuracy (binary) |
| 1c. Circled letter | Which letter is circled in a word? | Vary font, padding, oval thickness | Accuracy (exact match) |
| 1d. Shape counting | Count overlapping circles (5-9) | Vary count, line width, image size | Accuracy (exact count) |
| 1e. Grid counting | Count rows and columns of a grid | Vary grid size, empty vs. text-filled | Accuracy (both correct) |

### Level 2 — Regular Geometric Patterns (new tasks)

Tests whether VLMs can identify spatial arrangement types — the prerequisite for recognizing hex patterns.

| Task | Description | Images | Metric |
|------|-------------|--------|--------|
| 2a. Grid vs. random | Are these dots arranged in a grid or randomly? | Dots on white background, N=20-100 | Accuracy (binary) |
| 2b. Grid type ID | Is this a square, hexagonal, or triangular grid? | Perfect grids, vary spacing and rotation | Accuracy (3-class) |
| 2c. Dot grid counting | How many rows/columns in this dot grid? | Square and hex grids, 3-10 rows | Accuracy (exact count) |
| 2d. Regularity scoring | Rate the regularity of this dot pattern: perfect / slightly irregular / very irregular / random | Four levels of positional noise (σ = 0, 0.1d, 0.3d, 0.5d where d = spacing) | Accuracy (4-class) |
| 2e. Spacing estimation | Which grid has larger spacing, A or B? | Side-by-side images with 10-50% spacing difference | Accuracy (binary) |

### Level 3 — Degraded/Noisy Patterns (new tasks)

Progressive degradation toward SEM-realistic conditions.

| Task | Description | Images | Metric |
|------|-------------|--------|--------|
| 3a. Noisy hex detection | Is there a hexagonal pattern present? Yes/No | Hex grid + Gaussian noise on positions (σ = 0 to 0.5d), matched with random dot controls | Accuracy (binary), measured per noise level |
| 3b. Vacancy detection | How many nodes are missing from this hex grid? | Perfect hex grid with 0-5 vacancies | Accuracy (exact count) |
| 3c. Gradient contrast | Is there a hexagonal pattern present? | Hex dots with brightness fading across image (simulating SEM uneven illumination) | Accuracy (binary) |
| 3d. Dual pattern | How many distinct grid patterns are in this image? (1 or 2) | Single grid vs. two superimposed grids at different scales/orientations | Accuracy (binary) |
| 3e. Partial pattern | What fraction of the image contains a hexagonal pattern? (~25%, ~50%, ~75%, ~100%) | Hex grid that fades/stops partway across | Accuracy (4-class) |

### Level 4 — SEM-Realistic (new tasks)

Synthetic SEM-like rendering plus real SEM images.

| Task | Description | Images | Metric |
|------|-------------|--------|--------|
| 4a. Synthetic SEM hex | Is there a hexagonal bump pattern in this SEM image? | 3D-rendered hex bump arrays with directional lighting, grayscale, surface curvature | Accuracy (binary) |
| 4b. Synthetic SEM regularity | Rate the regularity of the bump pattern: well-ordered / partially ordered / disordered | Three noise levels rendered as SEM-like bumps | Accuracy (3-class) |
| 4c. SEM noise robustness | Is there a hexagonal pattern? | Same as 4a but with added shot noise, scan line artifacts, charging artifacts | Accuracy (binary) |
| 4d. Scale invariance | Which image shows a more regular hexagonal pattern, A or B? | Pairs at different magnifications (vary bump count and apparent size) | Accuracy (binary) |
| 4e. Real SEM (if available) | Is there a hexagonal bump pattern? Describe its regularity. | Real fish scale SEM images with manual annotations | Accuracy + qualitative assessment |

## Image Generation

All images for Levels 1-4d are **fully synthetic**, generated via Python scripts.

### Dependencies
- `PIL` / `Pillow` for basic 2D rendering (Levels 1-3)
- `matplotlib` for plots and grids
- `numpy` for noise generation and point placement
- `trimesh` or `pyvista` for 3D SEM-like bump rendering (Level 4a-d)
- Optionally `blender` via scripting for higher-quality SEM rendering

### Hex Grid Generation Parameters
- **Spacing (d):** 20-60 px
- **Positional noise (σ):** 0, 0.1d, 0.2d, 0.3d, 0.5d
- **Dot/bump radius:** 0.2d - 0.4d
- **Image size:** 512x512 and 1024x1024
- **Rotation:** 0°, 15°, 30° (hex grids look different at different orientations)
- **Vacancy rate:** 0%, 5%, 10%, 20%
- **Contrast gradient:** none, mild (20%), strong (50%)

## Benchmark Versions

### Brief Version (10-30 minutes)

Target: **statistically significant** results with minimal image count. For binary/few-class tasks, ~100 images per task gives ±10% confidence intervals at 95% confidence. We use 50 images per sub-task condition.

| Level | Tasks | Images per task | Conditions | Total images |
|-------|-------|----------------|------------|-------------|
| 1 | 5 tasks | 50 | 2 (easy/hard) | 500 |
| 2 | 5 tasks | 50 | 2 (easy/hard) | 500 |
| 3 | 5 tasks | 50 | 3 (noise levels) | 750 |
| 4 | 4 tasks (no real) | 50 | 2 (clean/noisy) | 400 |
| **Total** | | | | **2,150 images** |

**Estimated runtime at ~1 image/sec API throughput:** ~35 min per model.
**For local inference on GX10:** Faster, depending on model. A 7B model should do ~2-5 images/sec, so ~7-18 min.

Each task uses two standardized prompts (to control for prompt sensitivity), answers extracted programmatically.

### Comprehensive Version (few hours)

Full parameter sweeps for publishable results.

| Level | Tasks | Images per task | Conditions | Total images |
|-------|-------|----------------|------------|-------------|
| 1 | 5 tasks | 200 | 5 (image size × line width) | 5,000 |
| 2 | 5 tasks | 200 | 5 (grid size × rotation × spacing) | 5,000 |
| 3 | 5 tasks | 200 | 6 (noise levels × contrast) | 6,000 |
| 4 | 5 tasks (incl. real) | 200 | 4 (render quality × noise) | 4,000 |
| **Total** | | | | **20,000 images** |

**Estimated runtime:** ~3-5 hours per model on GX10 with local 7B inference. Longer for API-based models.

Two prompts per task, results reported per-condition for fine-grained analysis.

## Models to Test

### Local (GX10 / 4090)
- LLaVA-OneVision 7B (open-weight, common baseline)
- LLaVA-OneVision 72B (GX10 only — needs ~50 GB VRAM in 4-bit)
- Phi-3.5-vision (4.2B, small but tested in original paper)
- Qwen2-VL 7B and 72B

### API
- GPT-4o (or latest)
- Claude Sonnet (latest)
- Gemini Pro (latest)

## Evaluation Pipeline

```
generate_images.py    — renders all synthetic images + ground truth labels
run_benchmark.py      — feeds images to models, collects responses
parse_responses.py    — extracts structured answers from model outputs
score_results.py      — computes accuracy per task/condition, confidence intervals
report.py             — generates summary tables + accuracy-vs-noise-level plots
```

### Key Metrics
- **Accuracy** per task, per condition (noise level, grid size, etc.)
- **Accuracy vs. noise curve** for Level 3-4 (the degradation profile)
- **Confusion matrices** for multi-class tasks (grid type ID, regularity scoring)
- **95% confidence intervals** via bootstrap or Wilson score

### Output Format
- Per-model CSV with columns: `task, condition, image_id, ground_truth, model_response, parsed_answer, correct`
- Summary tables in markdown
- Plots as PNG (accuracy vs. noise, per-level radar charts)

## Task Status

| Step | Status |
|------|--------|
| Write spec | Done |
| Implement image generators (Level 1) | Not started |
| Implement image generators (Level 2) | Not started |
| Implement image generators (Level 3) | Not started |
| Implement image generators (Level 4) | Not started |
| Implement evaluation pipeline | Not started |
| Run brief benchmark | Not started |
| Run comprehensive benchmark | Not started |
| Analysis and write-up | Not started |

---

*Created: 2026-03-23*
