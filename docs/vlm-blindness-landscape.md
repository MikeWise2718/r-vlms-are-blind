# VLM Blindness: Research Landscape and Follow-Up Work

A survey of the research landscape around visual perception failures in Vision Language Models, anchored by the paper "Vision Language Models Are Blind" (Rahmanzadehgervi et al., 2024).

## The Core Finding

The VLMs-are-blind paper (arXiv:2407.06581, ACCV 2024) showed that SOTA VLMs (GPT-4o, Gemini-1.5 Pro, Claude-3 Sonnet, Claude-3.5 Sonnet) average only 58.07% accuracy on BlindTest — a suite of 7 trivially easy visual tasks involving geometric primitives (overlapping circles, intersecting lines, circled letters, counting shapes, grids, and paths). Claude 3.5 Sonnet was best at 77.84%, still far from the expected 100%.

The critical mechanistic finding: **linear probing of frozen vision encoder features achieves >=99.47% accuracy** on these tasks. The visual information is present — the LLM fails to decode it into correct language outputs. The paper identifies the **late-fusion architecture** (vision encoder extracts features without seeing the question, then the LLM processes them) as the root cause.

## Direct Responses and Replications

### OpenAI "Thinking with Images" (April 2025)
OpenAI's o3 and o4-mini claim ~90% on BlindTest, but via **tool use** — cropping, zooming, rotating images during chain-of-thought. Without tools, o1 drops to ~57%. This sidesteps the perceptual problem rather than solving it; the authors liken it to "a patient using rulers and a zoom lens at an eye exam."
- https://openai.com/index/thinking-with-images/

### Taesiri Continued Testing (March 2025)
The original authors tested o1-Pro on counting fingers, detecting unusual body postures, and reading analog clocks — it failed all three. The fundamental blindness persists in reasoning models.
- https://www.threads.com/@taesiri/post/DHcx98ogYXh/

## Papers Extending the Blindness Thesis

### "Visual Language Models Show Widespread Visual Deficits on Neuropsychological Tests" (Nature Machine Intelligence, 2026)
Tangtartharakul & Storrs used 51 tests from six clinical/experimental neuropsychology batteries, evaluating three SOTA VLMs against normative healthy-adult performance. VLMs excel at high-level object recognition but show deficits in low- and mid-level visual abilities — exactly the inverse of what you'd see with a brain lesion affecting high-level visual cortex.
- https://www.nature.com/articles/s42256-026-01179-y

### "Hidden in Plain Sight: VLMs Overlook Their Visual Representations" (UC Berkeley, June 2025)
Confirms the VLMs-are-blind linear probing result at scale: VLMs perform **worse** on vision-centric tasks (depth estimation, correspondence) than their own vision encoders in isolation. The LLM doesn't use the visual information available to it, falling back on language priors instead. The bottleneck is integration, not perception.
- https://arxiv.org/abs/2506.08008

### MeasureBench (Oct 2025)
Tests VLMs on reading 26 types of measuring instruments (thermometers, gauges, rulers). Best model (Gemini 2.5 Pro) achieves only 30.3% accuracy. Key failure mode: models read digits/labels but misidentify pointer positions and alignments — the same spatial precision problem as BlindTest.
- https://arxiv.org/abs/2510.26865

### CAPTURE (ICCV 2025)
Tests VLMs on counting objects in patterns when some are occluded. Even GPT-4o fails; humans have near-zero error. Extends the blindness finding to amodal completion — VLMs can't reason about spatial patterns continuing behind occluders.
- https://arxiv.org/abs/2504.15485

### "Eyes Wide Shut? Exploring the Visual Shortcomings of Multimodal LLMs" (CVPR 2024)
Traces VLM visual failures to CLIP's training objective. Identifies "CLIP-blind pairs" — visually different images CLIP sees as similar. Proposes Mixture of Features (MoF) using self-supervised vision features. Cited in the VLMs-are-blind paper as related prior work.
- https://arxiv.org/abs/2401.06209

## Theoretical Frameworks

### The Binding Problem Lens (NeurIPS 2024)
"Understanding the Limits of Vision Language Models Through the Lens of the Binding Problem" frames VLM failures via the cognitive science **binding problem**: when shared representational resources must represent distinct entities, serial processing is needed to avoid interference. The paper argues VLMs are stuck in a permanently "feedforward" mode — they process visual features in parallel but lack serial, spatially-grounded attention. This mirrors human rapid feedforward processing failures, but humans have a serial attention pathway to fall back on; VLMs do not.
- https://openreview.net/forum?id=Q5RYn6jagC

### VISER: Addressing the Binding Problem (NeurIPS 2025)
Follow-up proposing Visual Input Structure for Enhanced Reasoning — augments visual inputs with low-level spatial structures and sequential parsing prompts. Key insight: the fix requires giving VLMs something analogous to human serial attention.
- https://arxiv.org/html/2506.22146v3

### "Deconstructing Spatial Intelligence in Vision-Language Models" (TechRxiv, 2025/2026)
The most comprehensive survey. Uses a framework from human spatial cognition with three levels: Perception (seeing objects), Understanding (reasoning about relationships), Extrapolation (inferring unobserved states). Systematically traces root causes and categorizes proposed fixes: training-free prompting, model-centric enhancements, explicit 2D/3D information injection, and data-centric approaches.
- https://www.techrxiv.org/users/992599/articles/1354538

### VLM Spatial Abilities via Psychometrics (Feb 2025)
Applies psychometric methods to define and evaluate basic spatial abilities in VLMs, bringing standardized human cognitive testing frameworks to VLM evaluation.
- https://arxiv.org/html/2502.11859v2

## The Evolutionary Inversion Insight

An observation that runs through this literature but is never stated explicitly as such:

**Human brains** evolved hundreds of millions of years of visual/spatial processing (navigation, predator detection, object manipulation) before language was layered on top ~100K years ago. Language is the late add-on that learned to *describe* what vision already understood. Humans never struggle to see overlapping circles — that's ancient neural machinery. We sometimes struggle to *verbalize* complex spatial relationships.

**VLMs** were trained on trillions of text tokens before vision was bolted on via projection layers and fine-tuning. Language is the deep foundation. Vision is the late add-on that must express itself *through* the language machinery.

The architectures are **inverted** relative to each other. This maps directly onto where each system fails:

| | Strong foundation | Weak bolt-on |
|---|---|---|
| **Humans** | Spatial/visual reasoning | Precisely verbalizing spatial relationships |
| **VLMs** | Language, reasoning, world knowledge | Precise spatial/geometric perception |

The binding problem paper comes closest to this framing — noting VLMs fail like human fast feedforward processing. The neuropsychology paper treats VLMs as "patients" with specific lesion-like profiles. The spatial intelligence survey uses human cognition as its organizing framework. But the explicit statement that these are **mirror-image architectures** — vision-first vs. language-first — with predictably complementary failure modes appears to be a novel synthesis not yet articulated in the literature.

## Architectural Responses

### Early Fusion: Chameleon (Meta FAIR, May 2024)
Tokenizes everything (text + images) into a single discrete token space and trains a single transformer from scratch. Avoids the late-fusion bottleneck entirely. However, image tokenization (512x512 -> 1024 tokens, codebook of 8192) is lossy compression. The repo was archived Sep 2025; Meta moved to Transfusion.
- https://arxiv.org/abs/2405.09818

### Hybrid AR + Diffusion: Transfusion (Meta FAIR, Aug 2024; ICLR 2025)
Meta's successor to Chameleon. Uses next-token prediction for text but **diffusion for images** in the same transformer. Outperformed Chameleon with <1/3 the compute. Continuous latent representations (VAE patches) may preserve more spatial detail than discrete tokenization.
- https://arxiv.org/abs/2408.11039

### Emu3 / Emu3.5 (BAAI, 2024-2025; Nature 2025)
Pure autoregressive approach taken further — everything is next-token prediction including video and robotic manipulation. Published in Nature. Emu3.5 rivals closed-source diffusion models.
- https://www.nature.com/articles/s41586-025-10041-x

### Show-o / Show-o2 (ICLR 2025)
Hybrid autoregressive + discrete diffusion. Causal attention for text, bidirectional for images. ~20x fewer sampling steps than pure autoregressive image generation.
- https://openreview.net/forum?id=o6Ynz6OIQ6

### G2VLM: Geometry Grounded VLM (CVPR 2026)
Unifies 3D reconstruction with VLM reasoning, attempting to give VLMs genuine geometric understanding rather than linguistic approximations.
- https://arxiv.org/html/2511.21688

## Open Question

None of these architectural responses have been explicitly tested against BlindTest-style tasks. It remains unknown whether early fusion, hybrid AR+diffusion, or geometry grounding actually fixes the low-level spatial perception failures that the VLMs-are-blind paper identified. This seems like a significant gap in the literature.

---

*Last updated: 2026-03-19*
