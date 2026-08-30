# Multiview2BIM

This repository is the official implementation of the paper below, which converts a set
of 2D architectural drawings — a floor plan and its elevation drawings — into structured
3D building data (BIM). It combines YOLO11-seg object detection, geometric
post-processing, and an LMM-based scale reader to recover wall/window/door geometry with
real-world dimensions.

- **Paper:** Lee, H., Jang, S., Lee, J., Jeong, H. D., & Lee, G. (2026). *Automated BIM
  generation from inconsistent multi-view raster architectural drawings with missing
  dimensions.* Automation in Construction, 187, 106971.
- **Authors:** Hyunjun Lee et al. (Building Informatics Group, Yonsei University)

## Introduction

The notebook [`Multiview2BIM.ipynb`](Multiview2BIM.ipynb) implements the full pipeline,
organized into five stages that run top to bottom:

- [x] **Floor plan parsing** — detect Wall/Window/Door objects (YOLO11-seg + SAHI), find
      the outermost slab contour, and assign a compass direction (N/S/E/W) to each
      floor's outermost openings.
- [x] **Elevation recognition** — detect objects in elevation drawings, cluster them into
      floors, and convert to the building's coordinate system.
- [x] **Multi-view correspondence matching** — match floor-plan objects to elevation
      objects (same floor, same direction) via the Hungarian algorithm to recover full 3D
      positions.
- [x] **Post-processing** — skeletonize walls into centerlines and corner points, then
      host windows/doors onto their nearest wall.
- [x] **Dimensional recognition & calibration** — read the drawing's paper size/scale
      with GPT-4o-mini and convert every coordinate from pixels to millimeters.

Each stage reads/writes JSON annotation files under `input/` and `output/` (created at
runtime, not tracked in this repo — see **Data layout** below).

## Updates

- **`2026/08/29`**: Initial public release — full pipeline notebook and documentation.

## Setup

```bash
pip install -r requirements.txt
```

Stage 5 calls the OpenAI API, so set an API key before running that section:

```bash
export OPENAI_API_KEY="sk-..."
```

## Pretrained weights

Pre-trained model weights are **not included** in this repository. Stage 1 and Stage 2
each need a YOLO11-seg checkpoint fine-tuned to detect `Wall`, `Window`, and `Door`:

- `MODEL_PATH` in Stage 1 — a floor plan object detector
- `model_path` in Stage 2 — an elevation object detector

Train your own with [Ultralytics](https://github.com/ultralytics/ultralytics) on your own
annotated floor plan / elevation data, then point these variables at your `.pt` files.

**Ultralytics/YOLO license notice:** this project uses the [Ultralytics](https://www.ultralytics.com/)
YOLO11 implementation, which is licensed under **AGPL-3.0**. If you train a model with it
and intend to use the code or the resulting weights in a closed-source or commercial
product, review the [Ultralytics licensing terms](https://www.ultralytics.com/license) —
a separate Enterprise License from Ultralytics may be required independent of this
repository's own license.

## Data layout

The notebook reads source drawings from `input/` and writes every intermediate and final
result to `output/` (both created at runtime, not tracked in this repo):

```
input/
├── FP/                     # floor plan images, named <name>_<floor>.<ext>, e.g. plan_1.png
└── ELEV/                   # elevation images, filename must contain a direction keyword
                             # (North / South / East / West), e.g. house_east.png

output/
├── FP/                     # Stage 1 intermediate results + visualizations
├── ELEV/                   # Stage 2 intermediate results + visualizations
├── Matched/                # Stage 3 multi-view matching results
├── PostProcessing/         # Stage 4 skeletonization / wall-vectorization results
├── Scale/                  # Stage 5 scale-reading results
├── Calibrated/             # Stage 5 dimension-calibrated results
└── Final/                  # final BIM-ready JSON per drawing
```

Everything under `output/` is created automatically as each stage runs — you only need to
populate `input/FP` and `input/ELEV` before starting.

## License

- **Code** (this notebook): [PolyForm Noncommercial License 1.0.0](LICENSE) — free for
  academic and other noncommercial use; commercial use requires a separate agreement.
- **Written content** of this README: [CC BY-NC 4.0](https://creativecommons.org/licenses/by-nc/4.0/).

For commercial licensing inquiries, please open an [issue](../../issues) on this repository.

Note that the AGPL-3.0 dependency described above under **Pretrained weights** applies
regardless of this repository's own license terms.

## Citation

If you use this code, please cite our paper:

```bibtex
@article{Multiview2BIM2026,
  title   = {Automated BIM generation from inconsistent multi-view raster architectural drawings with missing dimensions},
  author  = {Lee, H. and Jang, S. and Lee, J. and Jeong, H. D. and Lee, G.},
  journal = {Automation in Construction},
  volume  = {187},
  pages   = {106971},
  year    = {2026}
}
```

## Acknowledgements

This project builds on [Ultralytics YOLO11](https://github.com/ultralytics/ultralytics)
for instance segmentation, [SAHI](https://github.com/obss/sahi) for sliced inference, and
the OpenAI API for LMM-based scale reading. Thanks to their maintainers for these
open-source tools.
