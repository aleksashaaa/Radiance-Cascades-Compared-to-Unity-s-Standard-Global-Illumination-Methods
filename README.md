# Radiance Cascades vs. Unity's Standard Global Illumination Methods

Performance data from a comparison of real-time global illumination (GI) techniques in **Unity 6 (URP)**, collected for a DIGRA conference paper:

> **"Comparative Evaluation of Real-Time Global Illumination Methods in Unity URP"**

## What this project is

Global illumination is the indirect lighting that makes a scene look believable: light bouncing off walls, colour bleeding from a red floor onto a white wall, soft shadowed corners. Doing this in real time is expensive, and engines offer several approaches with different cost and quality.

This project measures what they **cost in frame time**. It compares **Radiance Cascades (RC)**, a newer GI technique, against other GI methods available in Unity, in three different scenes.

The repository holds the **measured data only**.

### Methods compared

| Label in data files | Method |
|---|---|
| `NoGI` | Baseline with no global illumination |
| `RC` | Radiance Cascades |
| `SSGI` | Screen-Space Global Illumination |
| `RTLM` | A Unity GI method |

### Test scenes

| Folder | Scene |
|---|---|
| `Isometric/` | Isometric-camera scene |
| `Realistic FPS/` | Realistic first-person scene |
| `Stylized Fps/` | Stylized first-person scene |

Each scene was captured once per method, giving 12 runs in total, all on 2026-02-19.

## Headline results

Average frame time over the steady-state part of each ~60 s run (the first 5 s are excluded because they contain slow warm-up frames). Lower is better. FPS is `1000 / ms`.

| Scene | NoGI | RC | RTLM | SSGI |
|---|---|---|---|---|
| Isometric | 2.99 ms (335 fps) | 4.48 ms (223 fps) | 4.72 ms (212 fps) | 9.62 ms (104 fps) |
| Realistic FPS | 2.60 ms (385 fps) | 4.47 ms (224 fps) | 4.87 ms (205 fps) | 7.09 ms (141 fps) |
| Stylized FPS | 2.49 ms (402 fps) | 3.78 ms (265 fps) | 3.59 ms (278 fps) | 6.99 ms (143 fps) |

What the numbers show:

- **SSGI is consistently the most expensive**, at roughly 2.7 to 3.2 times the frame time of the no-GI baseline.
- **RC and RTLM are close to each other** in every scene, both adding roughly 1 to 2.5 ms over the baseline. Which one is cheaper depends on the scene.
- These are **performance numbers only**. Visual quality is judged separately, using the comparison videos below.

Each run is a single 60-second capture, so small differences (for example RC vs. RTLM) should not be over-interpreted without repeated runs.

## Video comparison

Rendered videos of each method are in this YouTube playlist: [Watch the visual comparison](https://www.youtube.com/watch?v=daDjPJyB86o&list=PLaHv1eGKS6CyZF2IG05RV7KAHvvU5-xIQ)

## Repository layout

```
Measured Data/
├── Isometric/
│   ├── gi_run_<MODE>_<date>_<time>.csv       one raw capture per method
│   └── Performance_comparison_Isometric.xlsx  summary table and chart
├── Realistic FPS/                             same structure
└── Stylized Fps/                              same structure
```

## Data format

Each `gi_run_*.csv` is a roughly 60-second capture sampled about every 0.2 s (around 280 rows).

| Column | Meaning |
|---|---|
| `time_sec` | Seconds since the capture started |
| `ms_per_frame` | Frame time in milliseconds |
| `fps` | Frames per second |
| `gi_mode` | Method being measured |

Each file ends with a `SUMMARY` block (`avg_ms`, `min_ms`, `max_ms`, `p95_ms`).

**Watch out when parsing.** The files are comma-delimited **and** use a comma as the decimal separator, so `2,749,363,83` means `2.749` ms and `363.83` fps. A normal CSV reader will split the numbers apart. The `max_ms` value in the summary (about 300 ms) comes from the first few warm-up frames and is not typical of the run.

Loading a run in Python:

```python
import re
import pandas as pd

def load_run(path):
    pat = re.compile(r"^(\d+,\d+),(\d+,\d+),(\d+,\d+),(\w+)\s*$")
    rows = []
    with open(path, encoding="utf-8-sig") as f:
        for line in f:
            m = pat.match(line)
            if m:
                t, ms, fps = (float(x.replace(",", ".")) for x in m.groups()[:3])
                rows.append((t, ms, fps, m.group(4)))
    return pd.DataFrame(rows, columns=["time_sec", "ms_per_frame", "fps", "gi_mode"])

df = load_run("Measured Data/Realistic FPS/gi_run_RC_2026-02-19_15-09-13.csv")
steady = df[df.time_sec >= 5]          # drop warm-up
print(steady.ms_per_frame.mean())      # about 4.47 ms
```

## Test setup

| | |
|---|---|
| Engine | Unity 6000.2.12f1, Universal Render Pipeline (URP) |
| GPU | NVIDIA GeForce RTX 4060 Laptop GPU |
| CPU | Intel Core i7-13620H |
| Capture length | About 60 s per run |


## Citation

```bibtex
@inproceedings{TODO,
  title  = {Comparative Evaluation of Real-Time Global Illumination Methods in Unity URP},
  author = {TODO},
  booktitle = {DiGRA},
  year   = {TODO}
}
```

## License

MIT. See [LICENSE](LICENSE).
