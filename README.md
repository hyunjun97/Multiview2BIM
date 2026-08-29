# Multiview2BIM

Multiview2BIM converts a set of 2D architectural drawings — a floor plan and its
elevation drawings — into structured 3D building data (BIM). It combines
YOLO11-seg object detection, geometric post-processing, and an LMM-based scale
reader to recover wall/window/door geometry with real-world dimensions.

## Pipeline

The notebook [`Multiview2BIM.ipynb`](Multiview2BIM.ipynb) is organized into five stages, run top to bottom:

1. **Floor plan parsing** — detect Wall/Window/Door objects (YOLO11-seg + SAHI), find the
   outermost slab contour, and assign a compass direction (N/S/E/W) to each floor's
   outermost openings.
2. **Elevation recognition** — detect objects in elevation drawings, cluster them into
   floors, and convert to the building's coordinate system.
3. **Multi-view correspondence matching** — match floor-plan objects to elevation objects
   (same floor, same direction) via the Hungarian algorithm to recover full 3D positions.
4. **Post-processing** — skeletonize walls into centerlines and corner points, then host
   windows/doors onto their nearest wall.
5. **Dimensional recognition & calibration** — read the drawing's paper size/scale with
   GPT-4o-mini and convert every coordinate from pixels to millimeters.

Each stage reads/writes JSON annotation files under a `Drawings/` working directory
(not included in this repository — see **Data layout** below).

## Setup

```bash
pip install -r requirements.txt
```

Stage 5 calls the OpenAI API, so set an API key before running that section:

```bash
export OPENAI_API_KEY="sk-..."
```

## Pretrained weights

Two YOLO11-seg checkpoints are included, both fine-tuned to detect `Wall`, `Window`,
and `Door`:

- `FP_best_v2.pt` — floor plan object detector (used in Stage 1)
- `ED_best.pt` — elevation object detector (used in Stage 2)

Point the notebook's `MODEL_PATH` / `model_path` variables at these files (or your own
compatible weights) before running Stages 1–2.

**Ultralytics/YOLO license notice:** this project uses the [Ultralytics](https://www.ultralytics.com/)
YOLO11 implementation, which is licensed under **AGPL-3.0**. The provided weights were
produced with that library. If you intend to use this code or the provided weights in a
closed-source or commercial product, review the
[Ultralytics licensing terms](https://www.ultralytics.com/license) — a separate
Enterprise License from Ultralytics may be required independent of this repository's own
license.

## Data layout

The notebook expects a working directory named `Drawings/` (created at runtime, not
tracked in this repo) with roughly this structure:

```
Drawings/
├── FP/
│   └── Images/            # floor plan images, named <name>_<floor>.<ext>, e.g. plan_1.png
└── ED/
    └── Images/             # elevation images, filename must contain a direction keyword
                             # (North / South / East / West), e.g. house_east.png
```

Every intermediate/output folder referenced later in the notebook (`result_FP`,
`result2_FP`, `viz_ED`, `PostProcessing/Skeleton*`, `Matched/outputs`, `Calibrated`,
`Final`, ...) is created automatically as each stage runs.

## License

- **Code** (notebook, weights): [PolyForm Noncommercial License 1.0.0](LICENSE) — free for
  academic and other noncommercial use; commercial use requires a separate agreement.
- **Written content** of this README: [CC BY-NC 4.0](https://creativecommons.org/licenses/by-nc/4.0/).

For commercial licensing inquiries, please open an [issue](../../issues) on this repository.

Note that the AGPL-3.0 dependency described above under **Pretrained weights** applies
regardless of this repository's own license terms.

## Citation

If you use this code or the accompanying paper, please cite:

```
<paper citation to be added>
```
