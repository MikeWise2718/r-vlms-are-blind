# Infographic Generation Prompt

Create a detailed, visually polished infographic titled **"Vision-First Small VLMs: Brainstormed Approaches"** based on the content below. Follow these layout and accuracy instructions carefully.

---

## Overall Design

- **Style:** Use two distinct background tones to separate framing content from ideas:
  - **Title banner, Motivation section, and Assessment panel:** Darker background (deep navy or black) — these are the framing/context panels that apply to all ideas.
  - **The seven idea panels (#1-#7):** Slightly lighter background (dark charcoal or dark slate) so they visually pop as the actionable content within the frame.
  - Use a unique bright accent color per idea (e.g., teal for #1, gold for #2, coral for #3, etc.) for borders, headings, and diagram highlights within each idea panel.
- **Layout:** Title banner at top, then a "Motivation" section, then a 2-column grid for ideas 1-6 (3 rows x 2 columns), then idea 7 and the Assessment panel side by side at the bottom.
- **Typography:** Use clean, readable sans-serif fonts. All caption text must be proofread and grammatically correct — no garbled or truncated text.
- **Icons/Illustrations:** Use simple, schematic diagrams rather than photorealistic images. Flowcharts and block diagrams are preferred.

---

## Section 1: Title Banner (top)

**Title:** "VISION-FIRST SMALL VLMs: BRAINSTORMED APPROACHES"
**Subtitle:** "Seven ideas for building small models with better low-level spatial abilities"

---

## Section 2: Motivation Banner (below title)

**Heading:** "THE EVOLUTIONARY INVERSION"

Show two parallel pipelines side by side:

**Left — Human Brain:**
- Bottom layer (large, labeled "FOUNDATION"): "Hundreds of millions of years of visual/spatial processing"
- Top layer (small, labeled "BOLT-ON"): "Language added ~100K years ago"
- Caption: "Vision-first. Language describes what vision already understands."

**Right — Current VLMs:**
- Bottom layer (large, labeled "FOUNDATION"): "Trillions of text tokens"
- Top layer (small, labeled "BOLT-ON"): "Vision added via projection layer + fine-tuning"
- Caption: "Language-first. Vision must express itself through language machinery."

**Center label between the two:** "These are mirror-image architectures with complementary failure modes."

Draw a simple table below:

|                | Strong Foundation              | Weak Bolt-On                        |
|----------------|-------------------------------|-------------------------------------|
| **Humans**     | Spatial/visual reasoning       | Precisely verbalizing spatial info   |
| **VLMs**       | Language, reasoning, knowledge | Precise spatial/geometric perception |

---

## Section 3: The Seven Ideas (main body)

### Row 1, Left — Idea #1: "INVERT THE TRAINING ORDER"

Diagram: A horizontal arrow with two phases:
- Phase 1 (large block): "PRETRAIN ON VISUAL TASKS" — list below: edge detection, spatial relationships, counting, path tracing, geometric primitives
- Phase 2 (smaller block): "FINE-TUNE LANGUAGE ON TOP"
- Arrow labeled: "Circuits wired for spatial reasoning BEFORE seeing text"

Caption: "Language must accommodate existing visual circuits, not the reverse. Uses synthetic geometric datasets. Model size: 1-7B."

### Row 1, Right — Idea #2: "THE TODDLER BRAIN CURRICULUM"

Diagram: A pyramid/staircase built from bottom to top, with training proceeding upward (show an upward arrow on the left side labeled "Training order"):
- Step 1 (bottom, widest): "LOW-LEVEL — edges, figure-ground, spatial frequency"
- Step 2: "MID-LEVEL — shape completion, occlusion, Gestalt grouping"
- Step 3: "HIGH-LEVEL — object recognition, scene understanding"
- Step 4 (top, narrowest): "LANGUAGE — describe what you've already learned to see"

Caption: "Each level scaffolds the next, mimicking human visual development. Current VLMs skip straight to high-level."

IMPORTANT: The arrow must go UPWARD from low-level to language. Training starts at the bottom.

### Row 2, Left — Idea #3: "DUAL-ENCODER: VENTRAL + DORSAL STREAMS"

Diagram: An image feeds into two parallel encoder boxes:
- Top encoder: "SEMANTIC ENCODER (CLIP-like)" labeled "WHAT" — colored one way
- Bottom encoder: "SPATIAL ENCODER (geometric tasks)" labeled "WHERE/HOW" — colored differently
- Both feed via arrows labeled "Cross-Attention Streams" into a box labeled "LLM Backbone"
- The spatial encoder arrow should be thicker or highlighted, with a note: "Architectural priority (processed first)"

Caption: "Mirrors human ventral ('what') and dorsal ('where') visual streams. Current VLMs only have the ventral stream. The dorsal stream handles spatial precision."

### Row 2, Right — Idea #4: "GEOMETRIC PRIMITIVES AS NATIVE TOKENS"

Diagram: Show three token streams feeding into a central "VLM" box:
- Stream 1: "TEXT TOKENS" (e.g., showing "How many circles overlap?")
- Stream 2: "IMAGE PATCH TOKENS" (show a small grid of patches)
- Stream 3 (highlighted as new): "GEOMETRIC PRIMITIVE TOKENS" (show icons: line, circle, intersection point, with coordinate labels)

Caption: "Gives the model a 'spatial language' alongside natural language. Geometric extraction uses classical CV (Hough transforms, contour detection). Sidesteps the 'LLM can't decode spatial info from patches' problem."

### Row 3, Left — Idea #5: "ACTIVE VISION: LET THE MODEL LOOK AGAIN"

Diagram: A circular/iterative loop:
1. "COARSE PASS" — full image processed, produces initial understanding
2. "GENERATE QUERY" — model asks "Where should I look more carefully?"
3. "CROP & ZOOM" — model selects and enlarges a region of interest
4. "DETAILED PASS" — zoomed region processed for fine-grained spatial info
5. Arrow loops back to step 2 if needed, or exits to "ANSWER"

Caption: "Mimics human saccadic eye movement — serial attention to details. The binding problem paper (NeurIPS 2024) identifies lack of serial attention as the core VLM issue. OpenAI's o3 uses this via external tools; this would internalize it."

IMPORTANT: This idea is about iterative looking, NOT about geometric tokenization. Do not include geometric primitive tokens in this diagram.

### Row 3, Right — Idea #6: "CONTRASTIVE SPATIAL PRE-TRAINING"

Diagram: Show two training approaches side by side with "vs." between them:

**Left (current — CLIP):**
- Label: "Semantic Contrastive (CLIP)"
- Positive pair: two images of dogs in different positions, labeled "Same object = similar"
- Result: "Optimizes for WHAT, ignores WHERE"

**Right (proposed):**
- Label: "Spatial Contrastive (new)"
- Positive pair: a circle above a square + a triangle above a diamond, labeled "Same spatial relationship, different objects"
- Negative pair: a circle above a square + a circle below a square, labeled "Different spatial relationship, same objects"
- Result: "Optimizes for WHERE over WHAT"

Caption: "CLIP creates 'CLIP-blind pairs' — visually different images it sees as similar (Eyes Wide Shut, CVPR 2024). A spatial contrastive objective wires the encoder for spatial precision. Synthetic pair generation is cheap."

---

## Section 4: Bottom Row

### Bottom Left — Idea #7: "THE ANNOTATED SKETCHPAD"

Diagram: A left-to-right pipeline:
1. "INPUT IMAGE" (show simple geometric scene — e.g., two overlapping circles)
2. Arrow to "SPATIAL REPRESENTATION" (show SVG-like text: `<circle cx=120 cy=100 r=50/> <circle cx=160 cy=100 r=50/> <intersection at=(140,80)...>`)
3. Arrow to "LANGUAGE REASONING" (model reads the spatial representation as text and answers)
4. Arrow to "ANSWER: Yes, circles overlap"

Caption: "Forces explicit spatial translation before answering. Like humans sketching diagrams to reason. The intermediate representation is inspectable and debuggable. Training data: render geometric scenes + pair with SVG descriptions."

### Bottom Right — Assessment Panel

**Heading:** "MOST PROMISING FOR MODEST RESOURCES"

Show ideas #7, #4, #6 highlighted (e.g., with star icons or a podium):
- #7 Annotated Sketchpad
- #4 Geometric Primitive Tokens
- #6 Contrastive Spatial Pre-training

Four bullet points:
- Rely on synthetic data (cheap to generate at scale)
- Work with small models (1-7B; insight is architectural, not scale-dependent)
- Produce testable hypotheses against BlindTest benchmark
- Don't require training a full LLM from scratch

---

## Critical Accuracy Notes

1. **Idea #5 is about iterative crop/zoom/re-attend.** It is NOT about geometric primitive tokenization. Do not conflate it with Idea #4.
2. **The Toddler Brain pyramid (#2) must show training going UPWARD** from low-level to language. Low-level is the foundation, language is the capstone.
3. **All text labels and captions must be complete, grammatically correct sentences.** Do not truncate or garble any text.
4. **The dual-encoder diagram (#3) must show TWO separate encoders**, not one encoder with two outputs.
5. **The contrastive spatial pre-training (#6) must clearly distinguish positive pairs (same relationship, different objects) from negative pairs (different relationship, same objects).** Do not swap these.
