# TTS Quality Evaluation Pipeline / TTS 质量评估流水线

## English

This project implements an end-to-end benchmark pipeline for TTS quality across multiple providers and configurations. The same source scripts are synthesized, then evaluated with an LLM-as-a-Judge rubric.

It compares:
- provider differences (OpenAI, ElevenLabs, Fish Audio, Minimax, Doubao)
- model / voice / speed settings
- objective speech metrics and rubric-based subjective dimensions

The workflow is fully reproducible and can run offline checks when API keys are unavailable.

### Goals

Answer practical questions such as:
- How much difference exists between `tts-1` and `tts-1-hd`?
- What is the quality cost of changing voice or speed (for example 1.5x)?

The pipeline answers these through a single command and produces a structured comparison report.

### Evaluation dimensions

Per synthesized audio, both objective and judged dimensions are recorded:
- Clarity: transcription consistency with source text
- Naturalness: speaking rate vs target range
- Pause/Rhythm: tempo appropriateness based on speech length
- Overall score: holistic quality

CER-based objective metrics are computed with normalized transcript comparison.

### Provider support

- TTS synthesis is implemented for multiple providers (OpenAI via SDK, others via REST).
- Default run covers 4 OpenAI configurations with only `OPENAI_API_KEY`.
- `--providers` enables cross-provider comparisons.
- Missing key -> that provider is skipped; the benchmark continues.

### Judge/backed details

- Default rubric path uses OpenAI: Whisper (`whisper-1`) for transcript + `gpt-5.6-luna` for scoring.
- OpenRouter fallback is supported only for rubric chat judging (`gpt-*` mapping handled).
- Optional `--gemini` enables multimodal direct scoring from Gemini if `GEMINI_API_KEY` is provided.

### Files

| File | Purpose |
|---|---|
| `config.py` | providers, model pricing, configs, corpus |
| `pipeline.py` | synthesis, ffprobe duration, transcription, CER, rubric scoring |
| `demo.py` | command entry, run grid, output summaries |
| `requirements.txt` / `env.example` | dependencies and env template |

### Run

```bash
pip install -r requirements.txt
brew install ffmpeg
export OPENAI_API_KEY=sk-...

python demo.py
python demo.py --quick
python demo.py --extra
python demo.py --gemini
python demo.py --fresh
python demo.py --providers openai,minimax,elevenlabs
python demo.py --text "2026年营收增长37.5%"
python demo.py --judge-model gpt-5.6-luna
python demo.py --output ./runs/exp1
python demo.py --list-providers
python demo.py --dump-rubric
```

Outputs are under `output/` (audio) and `output/results.json` (structured results).

### Robustness notes

- Missing key checks and ffprobe checks fail fast with clear instructions.
- A single failed (provider, text) cell does not stop the full run.
- OpenAI SDK is configured with retries.

### Limitations

- Default rubric path does not directly hear audio, so tonal/voice authenticity is partly inferred.
- CER depends on Whisper accuracy.
- Scores are relative, not absolute quality certification.

---

## 中文

本项目构建了**全流程 TTS 质量评估流水线**：在同一组文本上，对多个 provider / 配置合成语音，再按 Rubric 维度打分并汇总。

可比较维度包括不同 provider、模型、voice、语速，并给出可复现实验结果表。

### 目标

回答工程中经常出现的问题：
- `tts-1` 与 `tts-1-hd` 差异多大？
- 改 voice 或语速到 1.5x 会损失多少质量？

### 评估维度

- 清晰度：转写与原文一致性
- 自然度：语速接近真实朗读的程度
- 停顿节奏：整体节奏是否合理
- 总体：综合印象

CER 为目标文本与 Whisper 回译文本的字符级编辑距离归一化结果。

### Provider 说明

- 当前默认至少可无额外配置跑通 OpenAI（4 种配置）。
- `--providers` 可开启多 provider 横向对比。
- 缺少某 provider 的 key 时自动跳过该 provider，整批仍可继续。

### 评审模型

- 默认：Whisper 回译 + `gpt-5.6-luna` Rubric 评审。
- OpenRouter 用于聊天评审回退；OpenAI 直连音频模型受实名认证限制，故 gpt-5.x 常优先走 OpenRouter。
- `--gemini` 可改为 Gemini 直接听音频评分。

### 文件 / 运行 / 注意事项

中文含义对应英文表格与命令。主要入口为 `demo.py`，可通过 `--help` 查看完整参数。

### 结果与限制

- 缺 key/ffprobe 不会导致静默失败；会给出清晰报错。
- 任一单元失败会被计为该格失败，不会中断全表。
- 默认评审看不到音频本身（除 `--gemini`），因此“音色/情感”等维度是保守估计。
- Rubric 由 LLM 打分，适合相对对比，不适合当作绝对标准。
