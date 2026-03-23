# Vision-First Small VLMs: Brainstormed Approaches

Ideas for building smaller VLMs with better low-level spatial abilities, motivated by the "evolutionary inversion" insight: human brains are vision-first with language bolted on, while VLMs are language-first with vision bolted on. What would a vision-first small model look like?

## 1. Invert the Training Order

Instead of pretraining on text then adding vision, pretrain a small transformer (1-7B) primarily on **visual tasks first** — edge detection, spatial relationship classification, geometric primitive identification, counting, path tracing — then fine-tune language capabilities on top.

The model's circuits would be wired for spatial reasoning before it ever sees text. Language would have to accommodate existing visual circuits rather than the reverse.

**Feasibility:** Synthetic dataset generators exist (the BlindTest paper has code for geometric tasks). Language fine-tuning could use a small instruction dataset. Modest GPU requirements since the model is small.

## 2. The "Toddler Brain" Curriculum

Mimic the developmental sequence of human vision. Train in stages:

1. **Low-level:** Edge detection, figure-ground separation, spatial frequency processing
2. **Mid-level:** Shape completion, occlusion reasoning, grouping (Gestalt principles)
3. **High-level:** Object recognition, scene understanding
4. **Language:** Only now add text describing what the model has already learned to see

Human visual development follows this hierarchy for good evolutionary reasons — each level provides scaffolding for the next. Current VLMs skip straight to high-level because their training data (internet images + captions) is all high-level.

**Feasibility:** Synthetic data for stages 1-3 is cheap to generate. Stage 4 could use existing caption datasets. The curriculum adds training complexity but not necessarily compute.

## 3. Dual-Encoder with a Spatial Bottleneck

Train two small encoders in parallel:

- A **semantic encoder** (CLIP-like, trained on image-text pairs) for "what"
- A **spatial encoder** (trained on geometric/spatial tasks) for "where/how"

Force the LLM backbone to attend to both via separate cross-attention streams, with the spatial encoder's representations given architectural priority (processed first, or with dedicated attention heads).

This mirrors the human ventral ("what") and dorsal ("where") visual streams. Current VLMs only have the ventral stream equivalent (CLIP). The dorsal stream is what handles the spatial precision that BlindTest tests.

**Feasibility:** Both encoders can be small. The spatial encoder could be trained entirely on synthetic geometric data. The interesting research question is the fusion mechanism.

## 4. Geometric Primitives as a Native Token Type

Instead of only having text tokens and image patch tokens, add a third token type: **geometric primitive tokens** — explicit representations of lines, circles, edges, intersections, and their spatial relationships. Train a small tokenizer to extract these from images, and train the model to reason over them directly.

This sidesteps the "LLM can't decode spatial info from patch embeddings" problem entirely. The spatial information arrives pre-structured in a form the language model can work with — essentially giving it a "spatial language" alongside natural language.

**Feasibility:** Extracting geometric primitives from images is classical CV (Hough transforms, contour detection). The novel part is training a model that consumes these tokens alongside text and image tokens.

## 5. Active Vision: Let the Model "Look Again"

Instead of single-pass feedforward processing, give a small model the ability to **iteratively re-attend** to image regions. On first pass, get a coarse understanding. Then generate "where should I look more carefully?" queries, crop/zoom those regions, and process again. Repeat.

This mimics human saccadic eye movement — we don't process the whole visual field at once, we serially attend to details. The binding problem paper (NeurIPS 2024) explicitly identifies the lack of serial attention as the core issue. OpenAI's o3 tool-use approach (crop/zoom) already shows this helps, but it's external scaffolding — this would be internalized.

**Feasibility:** Could be implemented as a small VLM + a lightweight attention policy network. The VISER paper (NeurIPS 2025) is a step in this direction.

## 6. Contrastive Spatial Pre-training

Before any language training, pretrain the vision encoder with a **spatial contrastive objective** instead of (or in addition to) CLIP's semantic contrastive objective. Positive pairs: same spatial relationship, different objects. Negative pairs: different spatial relationship, same objects. Force the encoder to prioritize "where" over "what."

CLIP's training objective creates "CLIP-blind pairs" (Eyes Wide Shut paper, CVPR 2024) because it optimizes for semantic similarity, not spatial precision. A spatial contrastive objective would wire the encoder differently.

**Feasibility:** Synthetic pair generation is cheap. Could be combined with a small existing vision encoder as a fine-tuning stage.

## 7. The "Annotated Sketchpad" Approach

Train a small model to first **draw** or annotate what it sees before answering questions — convert the image into an explicit spatial representation (SVG-like description, coordinate list, or simple sketch), then reason over that intermediate representation in text space.

This forces the model to translate visual information into an explicit spatial format *before* answering. Analogous to how humans sketch diagrams to reason about spatial problems. The intermediate representation is inspectable and debuggable.

**Feasibility:** Training data could be generated by rendering known geometric scenes and pairing them with their SVG/coordinate descriptions. Small model, small data, fully synthetic.

## Assessment: Most Promising for Modest Resources

**#7 (Annotated Sketchpad)**, **#4 (Geometric Primitive Tokens)**, and **#6 (Contrastive Spatial Pre-training)** rank highest because:

- They rely on **synthetic data** (cheap to generate at scale)
- They work with **small models** (the insight is architectural, not scale-dependent)
- They produce **testable hypotheses** against BlindTest (clear evaluation)
- They don't require training a full LLM from scratch

---

*Last updated: 2026-03-19*
