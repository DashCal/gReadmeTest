<sup>
<p align="right">
<b>English</b>
| <a href="README.zh-CN.md">简体中文</a>
| <a href="README.llm.md">LLM</a>
</p>
</sup>

<a href="https://audio-meet.localwu.com/?f=ghrdm" style="--ver-lang-en:0;">
<img width="3600" alt="audio-meet readme header" src="https://dst1.localwu.top/project/audio-meet/readme/header.svg?v=182easxuy#en" />
</a>

<a href="#zh">中文</a>

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
