<sup>
<p align="right">
<b>English</b>
| <a href="README.zh-CN.md">简体中文</a>
| <a href="README.llm.md">LLM</a>
</p>
</sup>

<a href="https://audio-meet.localwu.com/?f=ghrdm">
<!-- ghrepo with rawgh no commit hash OK -->
<img width="3600" alt="audio-meet readme header" src="https://raw.githubusercontent.com/DashCal/DashCal/main/docs/asset/project/audio-meet/readme/header.svg#en" />
<!-- <img width="3600" alt="audio-meet readme header" src="https://camo.githubusercontent.com/f0224ac6930df89d3d82363f5161863783dcca987ec92238c83a70067cc96673/68747470733a2f2f647374312e6c6f63616c77752e746f702f70726f6a6563742f617564696f2d6d6565742f726561646d652f6865616465722e7376673f763d31383265617378757965#en" /> 
-->
<!-- cfpages with self-domain NO -->
<!-- <img width="3600" alt="audio-meet readme header" src="https://dst1.localwu.top/project/audio-meet/readme/header.svg?v=182easxuye#en" /> -->
<!-- ghrepo with direct-url OK -->
<!-- <img width="3600" alt="audio-meet readme header" src="https://github.com/DashCal/gReadmeTest/blob/main/audio-meet-header-repo.svg#en" /> -->
<!-- ghpages with self-domain NO -->
<!-- <img width="3600" alt="audio-meet readme header" src="https://dstghdc1.localwu.top/asset/project/audio-meet/readme/header.svg#en" /> -->
<!-- ghUSERNAMErepo with direct-url OK -->
<!-- <img width="3600" alt="audio-meet readme header" src="https://github.com/DashCal/DashCal/blob/main/docs/asset/project/audio-meet/readme/header.svg#en" /> -->
<!-- ghpages with ghio-domain NO -->
<!-- <img width="3600" alt="audio-meet readme header" src="https://dashcal.github.io/DashCal/asset/project/audio-meet/readme/header.svg#en" /> -->
<!-- ghrepo with rawgh commit hash OK -->
<!-- <img width="3600" alt="audio-meet readme header" src="https://raw.githubusercontent.com/DashCal/gReadmeTest/048d36e16ec8c55b32a151f5cd84d98a34dac4a1/audio-meet-header-repo.svg#en" /> -->
</a>

<!-- <p>
<a href="https://audio-meet.localwu.com">Website</a>
| <a href="usage.human.md">Usage (Human)</a>
| <a href="usage.llm.md">Usage (LLM)</a>
</p> -->

A **silence pre-pass** tool for multi-person dialogue audio before it goes into an editor.

It finds stretches where **everyone is silent at the same time**, shortens them (not removes them), and outputs each track separately while keeping all tracks aligned. The result can go straight into your editing software for fine-tuning.

> Version: v0.2.6

---

## How it works

Each track first finds its own silent regions. Only the **overlap of silent regions across all tracks** counts as true silence. If any track has sound — coughing, laughter, keyboard clicks — that stretch is not cut.

Better to cut too little than to cut wrongly.

---

## Requirements

- Python 3.10+ (with `numpy`)
- ffmpeg (required), ffprobe (optional) — place them in `tools/ffmpeg/bin/` or add them to `PATH`

---

## Quick start

Run from the `audio-meet/` directory. The order of `--tracks` matters.

```bash
# 1) Analyze only — see what would be cut, without touching audio
python -m audiomeet analyze --tracks trackA.mp3 trackB.mp3 --out out/project-name

# 2) Full run — analyze + cut + output
python -m audiomeet run --tracks trackA.mp3 trackB.mp3 \
  --config presets/对话初筛-均衡.json --out out/project-name

# 3) Verify before delivery
#    (safety / sample accounting / output params / drift — all four must pass)
python tests/verify_run.py out/project-name
```

---

## Presets

| Preset | Style |
|---|---|
| `presets/对话初筛-保守.json` | Conservative: cuts less, keeps more. Good for first pass on new material. |
| `presets/对话初筛-均衡.json` | Default: balanced general-purpose pre-pass. |
| `presets/对话初筛-利落.json` | Aggressive: dynamic keep length, tighter pacing. |

Common options: `--protect-pauses 1200` (preserve pauses between speech), `--keep 300` / `150` (keep more / less silence), `--shrink-mode compress` (speed up silence instead of deleting it). See the same-version `README` or documentation §7 for the full parameter table.

---

## Output

The `--out` directory contains:

- `trimmed/*.trimmed.m4a` — one output audio file per track (encoding parameters copied from the source)
- `cutlist.json` — cut list; can be edited manually and re-rendered with `render`
- `summary.txt` / `report.json` — report: loudness stats, threshold suggestions, cut-point list
- `_work/` — intermediate files; safe to delete

---

## Limitations

- Silence pre-pass only — not fine editing.
- Primary output is m4a; m4a/mp3 → m4a involves one re-encode.
- `compress` mode may produce a slight smearing effect on silence containing breathing or laughter; use the default `center` if unsure.
- After a track ends, that track is considered “always silent”; if only one speaker remains, their monologue pauses may also be shortened.

---

## Documentation

- Parameter semantics follow the same-version `README.md`
- LLM / agent operation manual: `audio-meet — LLM Usage（skill Doc）`
