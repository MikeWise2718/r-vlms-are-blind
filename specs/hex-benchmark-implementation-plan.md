# Implementation Plan: Hex Pattern Benchmark

Step-by-step plan for building and running the benchmark defined in `specs/hex-pattern-benchmark.md`.

## Phase 1: Project Setup

### 1.1 Repository and Environment
- Initialize `uv` project with virtual environment in this repo
- Core dependencies: `pillow`, `numpy`, `matplotlib`, `rich`, `rich-argparse`, `pandas`
- Level 4 rendering: `pyvista` or `trimesh` (add later when needed)
- Evaluation: `scipy` (confidence intervals), `seaborn` (plots)
- Model inference: `transformers`, `torch`, `vllm` (for fast local inference on GX10)

### 1.2 GX10 Setup
- SSH into GX10, clone repo
- Install Claude Code on GX10 for overnight runs
- Install `uv`, create matching virtual environment
- Install CUDA-compatible PyTorch + vLLM for Blackwell
- Download model weights: LLaVA-OneV 7B, LLaVA-OneV 72B, Qwen2-VL 7B/72B, Phi-3.5-vision
- Verify GPU is recognized and inference works with a smoke test

### 1.3 Directory Structure
```
src/
  generators/         # Image generation scripts
    level1.py         # BlindTest-style tasks
    level2.py         # Regular geometric patterns
    level3.py         # Degraded/noisy patterns
    level4.py         # SEM-realistic rendering
    common.py         # Shared utilities (hex grid math, noise, rendering helpers)
  evaluation/         # Benchmark execution and scoring
    run_benchmark.py  # Main runner — feeds images to models, collects responses
    parse.py          # Extract structured answers from model outputs
    score.py          # Compute accuracy, CIs, confusion matrices
    report.py         # Generate markdown tables + plots
  prompts/            # Standardized prompt templates per task
data/
  images/             # Generated benchmark images (gitignored — too large)
  labels/             # Ground truth CSV files (tracked in git)
results/              # Model outputs and analysis (gitignored)
```

Add `data/images/` and `results/` to `.gitignore`. Track `data/labels/`, `src/`, and `specs/`.

## Phase 2: Image Generators (do on this machine)

Build and test locally, visually inspect samples before committing.

### 2.1 Common Utilities (`src/generators/common.py`)
- `generate_hex_grid(spacing, noise_sigma, rotation, vacancies) -> list[Point]`
- `generate_square_grid(...)` and `generate_triangular_grid(...)`
- `generate_random_dots(n, bounds) -> list[Point]`
- `render_dots(points, image_size, dot_radius, ...) -> PIL.Image`
- `add_gradient_contrast(image, direction, strength)`
- `add_sem_noise(image, shot_noise, scan_lines, charging)`
- Utility to render SEM-like 3D bumps from a point grid (Phase 2.4)

### 2.2 Level 1 Generator (`src/generators/level1.py`)
Replicate BlindTest image generation. The original code is public at:
https://github.com/anguyen8/vision-llms-are-blind

Tasks: line intersections, circle overlap, circled letter, shape counting, grid counting.

Borrow generation logic where possible, adapt to our image count requirements. Output: images + `labels/level1.csv` with columns `image_id, task, condition, ground_truth`.

### 2.3 Level 2 Generator (`src/generators/level2.py`)
New tasks using `common.py` grid/dot utilities.
- 2a: render hex/square/tri grids vs. random dots, label grid/random
- 2b: render perfect grids of each type, label type
- 2c: render dot grids, label row/column count
- 2d: render hex grids at four noise levels, label regularity class
- 2e: render image pairs with different spacing, label which is larger

### 2.4 Level 3 Generator (`src/generators/level3.py`)
Progressive degradation using `common.py` noise utilities.
- 3a: hex grids + positional noise sweep
- 3b: hex grids with vacancies
- 3c: hex grids with brightness gradients
- 3d: single vs. dual superimposed grids
- 3e: hex grids covering partial image area

### 2.5 Level 4 Generator (`src/generators/level4.py`)
SEM-realistic rendering. Options:
- **Quick version:** Use `matplotlib` 3D surface plots with bump height maps + grayscale colormap + directional lighting. Good enough for a benchmark.
- **Better version:** Use `pyvista` to render a mesh of spherical bumps with proper lighting and camera angle.
- Add SEM-specific artifacts: shot noise, horizontal scan lines, uneven brightness.

### 2.6 Validation
- Generate 10 sample images per task, visually inspect
- Verify ground truth labels match images
- Run one model (e.g., Phi-3.5 locally on 4090) on samples to confirm pipeline works end-to-end

## Phase 3: Evaluation Pipeline (do on this machine, run on GX10)

### 3.1 Prompt Templates (`src/prompts/`)
One YAML or JSON file per task with:
- Two prompt variants per task (to measure prompt sensitivity)
- Expected answer format (e.g., `{number}`, `Yes/No`, `{letter}`)
- Parsing regex for extracting answers

### 3.2 Benchmark Runner (`src/evaluation/run_benchmark.py`)
CLI with `rich-argparse`:
```
python run_benchmark.py --model llava-onevision-7b \
                        --version brief \
                        --levels 1,2,3,4 \
                        --output results/llava7b_brief.csv \
                        -v
```
- Loads images and prompts
- Runs inference (local via vLLM, or API)
- Saves raw model responses to CSV: `image_id, task, condition, prompt_variant, raw_response`
- Supports `--resume` for interrupted runs
- Progress bar via `rich`

### 3.3 Model Backends
Abstract interface with two implementations:
- **LocalModel:** Uses vLLM or HuggingFace transformers for local inference
- **APIModel:** Calls OpenAI/Anthropic/Google APIs with rate limiting and retry

### 3.4 Parser (`src/evaluation/parse.py`)
- Extracts formatted answers from raw responses using per-task regexes
- Handles common VLM response quirks (extra text, hedging, multiple answers)
- Outputs: `image_id, task, condition, ground_truth, parsed_answer, correct`

### 3.5 Scorer (`src/evaluation/score.py`)
- Accuracy per task, per condition, per model
- 95% confidence intervals (Wilson score for proportions)
- Confusion matrices for multi-class tasks
- Accuracy-vs-noise-level data for plotting

### 3.6 Report Generator (`src/evaluation/report.py`)
- Markdown summary tables
- PNG plots: accuracy vs. noise level (Level 3-4 degradation curves), per-level radar charts, model comparison bar charts
- Output to `results/reports/`

## Phase 4: Run Benchmarks (on GX10)

### 4.1 Brief Version First
- Push code + generated images to repo (or rsync to GX10)
- SSH into GX10, run brief benchmark on all local models
- Review results, fix any pipeline issues
- Run API models from this machine (or GX10 if it has network access)

### 4.2 Comprehensive Version (overnight)
- Set up Claude Code on GX10 with instructions to run the comprehensive benchmark
- Or: write a shell script that runs all models sequentially, launch via SSH with `nohup`
- Estimated wall time: ~3-5 hours per model × 5-7 models = ~15-35 hours total
- Could parallelize: 72B models on GX10, 7B models on 4090 machine simultaneously

### 4.3 Collect and Analyze
- Pull results back to this machine
- Run report generator
- Compare results across models and levels
- Write up findings

## Phase 5: Real SEM Images (stretch goal)

- Manually annotate the three fish scale images with:
  - Region masks (hex pattern present/absent)
  - Approximate grid parameters (spacing, orientation)
  - Regularity rating
- Add as Level 4e test set (small N, qualitative comparison only)
- If more SEM images available, expand

## Task Status

| Step | Status |
|------|--------|
| 1.1 Project setup (local) | Not started |
| 1.2 GX10 setup | Not started |
| 1.3 Directory structure | Not started |
| 2.1 Common utilities | Not started |
| 2.2 Level 1 generator | Not started |
| 2.3 Level 2 generator | Not started |
| 2.4 Level 3 generator | Not started |
| 2.5 Level 4 generator | Not started |
| 2.6 Validation | Not started |
| 3.1 Prompt templates | Not started |
| 3.2 Benchmark runner | Not started |
| 3.3 Model backends | Not started |
| 3.4 Parser | Not started |
| 3.5 Scorer | Not started |
| 3.6 Report generator | Not started |
| 4.1 Brief benchmark run | Not started |
| 4.2 Comprehensive benchmark run | Not started |
| 4.3 Analysis and write-up | Not started |
| 5. Real SEM images | Not started |

## Estimated Timeline

| Phase | Effort | Where |
|-------|--------|-------|
| Phase 1: Setup | Half day | Local + GX10 |
| Phase 2: Image generators | 1-2 days | Local |
| Phase 3: Evaluation pipeline | 1 day | Local |
| Phase 4: Brief run + debug | Half day | GX10 |
| Phase 4: Comprehensive run | Overnight | GX10 (unattended) |
| Phase 5: Analysis | Half day | Local |

**Total: ~4-5 days of active work + 1 overnight run.**

---

*Created: 2026-03-23*
