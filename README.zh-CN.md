<sup>
<p align="right">
<a href="README.md">English</a>
| <b>(📍简体中文)</b>
| <a href="README.llm.md">LLM</a>
</p>
</sup>

<a href="https://audio-meet.localwu.com/?f=ghrdm" style="--ver-lang-en:0;">
<img width="3600" alt="audio-meet readme header" src="https://raw.githubusercontent.com/DashCal/DashCal/main/docs/asset/project/audio-meet/readme/header.svg#zh" />
</a>

<!-- <p>
<a href="https://audio-meet.localwu.com">Website</a>
| <a href="usage.human.md">Usage (Human)</a>
| <a href="usage.llm.md">Usage (LLM)</a>
</p> -->

多人对话音频进入剪辑软件之前的**静音初筛**工具。

把「所有人同时都没在说话」的空白找出来、剪短（不是剪没），每条轨道分别输出，各轨之间仍保持对齐，直接进剪辑软件继续精修。

> 版本：v0.2.6

---

## 原理

每条轨道先各自找出自己的安静段，**所有轨道安静段的重叠**才是真的没人说话。任何一条轨上有声音（咳嗽、笑声、键盘声），这段就不剪。

宁可少剪，不能剪错。

---

## 环境要求

- Python 3.10+（需 `numpy`）
- ffmpeg（必需）、ffprobe（可选）——放在 `tools/ffmpeg/bin/` 或加入 `PATH`

---

## 快速开始

在 `audio-meet/` 目录下执行，`--tracks` 的顺序有意义。

```bash
# 1) 只分析，不动音频：先看看会剪哪里
python -m audiomeet analyze --tracks 轨道A.mp3 轨道B.mp3 --out out/项目名

# 2) 一步跑完：分析 + 剪辑 + 输出
python -m audiomeet run --tracks 轨道A.mp3 轨道B.mp3 \
  --config presets/对话初筛-均衡.json --out out/项目名

# 3) 交付前验证（安全性 / 账目 / 输出参数 / 漂移，四项必须全过）
python tests/verify_run.py out/项目名
```

---

## 预设

| 预设 | 风格 |
|---|---|
| `presets/对话初筛-保守.json` | 少剪多留，第一次处理新素材用 |
| `presets/对话初筛-均衡.json` | 出厂默认，通用初剪 |
| `presets/对话初筛-利落.json` | 动态留长，剪得狠、节奏快 |

常用参数：`--protect-pauses 1200`（保住说话间停顿）、`--keep 300` / `150`（空白留长 / 留短）、`--shrink-mode compress`（快进压短空白，不删内容）。完整参数见同版本 `README` 详表或文档 §7。

---

## 输出

`--out` 目录包含：

- `trimmed/*.trimmed.m4a` — 每轨输出音频（编码参数照抄源文件）
- `cutlist.json` — 修剪清单，可手改后 `render`
- `summary.txt` / `report.json` — 报告：响度统计、音量线建议、剪辑点清单
- `_work/` — 中间文件，可删

---

## 限制

- 只做静音初筛，不做精剪；
- m4a 输出为主，m4a/mp3 → m4a 会有一次再压缩；
- `compress` 剪法对含呼吸、笑声的空白有轻微涂抹感，拿不准用默认 `center`；
- 某轨结束后该轨「永远算安静」，只剩一人时其独白停顿也会被剪短。

---

## 文档

- 参数语义以同版本 `README.md` 为准
- LLM / agent 操作手册见 `audio-meet — LLM 使用说明（skill 文档）`
