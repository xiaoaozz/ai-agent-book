# Elo Rating Leaderboard from Pairwise Comparisons / 配对对比得分榜

**Experiment 6-6 / 实验 6-6**: Building Model Leaderboard from Pairwise Comparison Data / 从两两对比数据构建模型排行榜

## English

This project implements an Elo-based evaluation pipeline and applies it to model ranking data (in particular public Chatbot Arena pairwise voting records). It demonstrates how model performance can be extracted from comparison data and visualized as a stable ranking system.

## Overview

The Elo rating system estimates relative skill from outcomes in a pairwise setting. It was first introduced for chess and is now commonly used to rank language models based on win/loss comparisons.

### Key features

- **High-performance implementation**: NumPy + Numba JIT + parallel processing for speed.
- **Real voting data analysis**: Uses large-scale public comparison data for model ranking.
- **Win rate prediction**: Estimates pairwise win probabilities for any two models.
- **Historical tracking**: Builds time-series snapshots to show ranking evolution.
- **Interactive visualizations**: Includes static charts and animation-style leaderboard views.
- **Scalable**: Handles large datasets efficiently.

## Mathematical foundation

Bradley-Terry model:

```
P(A beats B) = 1 / (1 + 10^((R_B - R_A) / 400))
```

After each match:

```
R_A_new = R_A + K * (S_A - E_A)
```

where:
- `R_A`: current rating of A
- `K`: learning rate (K-factor)
- `S_A`: observed score (`1`, `0`, or `0.5`)
- `E_A`: expected score (predicted probability)

## Requirements

- **Disk**: 3GB+ recommended (2GB data + 1GB processing)
- **RAM**: 4GB+ for full dataset
- **Internet**: stable network for about 2GB download
- **Python**: 3.8+

## Installation

```bash
cd projects/week6/elo-leaderboard
pip install -r requirements.txt
```

## Command-line interface (`cli.py`)

`cli.py` exposes unified subcommands: `battle -> elo -> leaderboard`, and a one-shot `pipeline`.

```bash
python cli.py --help
python cli.py battle --help
python cli.py  # defaults to `pipeline`
```

### Subcommands

| Command | Purpose | Important args |
|---|---|---|
| `battle` | Generate pairwise results | `--source {simulate,arena,llm}`, `--num-battles`, `--tie-prob`, `--seed`, `--sample`, `--output` |
| `elo` | Compute ratings | `--method {online-elo,bradley-terry}`, `--k`, `--bootstrap`, `--input`, `--output` |
| `leaderboard` | Render leaderboard table | `--input`, `--method`, `--bootstrap`, `--top-n` |
| `pipeline` | Run end-to-end | combined parameters |

### Battle sources (`--source`)

- `simulate` (default): synthetic battles against hidden latent strengths; supports validation and tie modeling via `--tie-prob`.
- `arena` (offline): loads public Chatbot Arena data (default `arena_data.json`, ~2GB), supports `--sample`.
- `llm` (requires API): judges pairs with an LLM, with position-bias mitigation.

Judging backends (`--judge-backend {anthropic,openrouter,auto}`, default `auto`):
- `anthropic`: official Anthropic SDK via `ANTHROPIC_API_KEY`
- `openrouter`: OpenRouter-compatible endpoint via `OPENROUTER_API_KEY`, with automatic model id remap
- `auto`: prefer Anthropic key if present, otherwise fallback to OpenRouter

### Example workflow

```bash
python cli.py battle --source simulate --num-battles 5000 --tie-prob 0.1 --output battles.json
python cli.py elo --input battles.json --method bradley-terry --bootstrap 100
python cli.py leaderboard --input battles.json --top-n 20
python cli.py pipeline --source arena --arena-file arena_data.json --sample 50000 --method bradley-terry --bootstrap 100

export ANTHROPIC_API_KEY=sk-...
python cli.py battle --source llm --candidate-models claude-opus-4-8 claude-haiku-4-5

export OPENROUTER_API_KEY=sk-or-...
python cli.py battle --source llm --judge-backend openrouter \
  --judge-model claude-opus-4-8 \
  --candidate-models anthropic/claude-haiku-4.5 openai/gpt-5.6-luna
```

## Quick start methods

The project includes two official-compatible methods:

1) **Bradley-Terry** (recommended)
2) **Online Elo** (K=4)

## Project structure

```text
elo-leaderboard/
├── cli.py
├── battle_simulator.py
├── llm_judge.py
├── main.py
├── optimized_elo.py
├── parallel_processing.py
├── data_loader.py
├── leaderboard.py
├── visualization.py
├── animation.py
├── benchmark.py
├── quickstart.py
├── elo_rating.py
├── test_elo.py
├── requirements.txt
└── README.md
```

## Usage examples

### Build leaderboard

```python
from optimized_elo import build_leaderboard_optimized
elo = build_leaderboard_optimized(df)
leaderboard = elo.get_leaderboard()
```

### Load and filter data

```python
from data_loader import load_arena_data, filter_data
df = load_arena_data("arena_data.json")
df_filtered = filter_data(df, anony_only=True, language="English", min_turn=1)
```

---

## 中文

该项目围绕**配对比较（pairwise）数据**构建 Elo/Bradley-Terry 排名流程，目标是用公开的模型对战投票数据（重点是 Chatbot Arena）形成可复现的模型排行榜与可视化分析。

### 实验导向背景

Elo 本质上用于“成对对局中的胜率”学习相对能力，最初用于棋类，现被广泛用于语言模型两两对比的排序。

### 关键特性

- 高性能实现：NumPy + Numba JIT + 并行处理。
- 真实数据分析：接入大规模公开投票数据。
- 胜率推断：可预测任意两个模型的胜率。
- 历史追踪：可输出时间序列排行快照。
- 交互可视化：支持静态图与动态动画。
- 可扩展：可承接较大规模比赛集合。

### 数学原理

与 AndroidWorld 风格一致，评分来自 Bradley-Terry：

```
P(A 胜过 B) = 1 / (1 + 10^((R_B - R_A) / 400))
```

单步更新：

```
R_A_new = R_A + K * (S_A - E_A)
```

### 安装与运行

```bash
cd projects/week6/elo-leaderboard
pip install -r requirements.txt
```

### 命令行（`cli.py`）

`cli.py` 是统一入口：
- `battle`：生成/采集两两对战
- `elo`：计算评级
- `leaderboard`：出榜
- `pipeline`：端到端一条龙

```bash
python cli.py --help
python cli.py battle --help
python cli.py   # 等价于 python cli.py pipeline
```

### 三类对战源

- `simulate`：合成对战（有真值），用于验证是否恢复出正确排序。
- `arena`：离线加载 `arena_data.json`（约 2GB）；可用 `--sample` 抽样。
- `llm`：调用 LLM 判断对战，带位置偏差消除；支持 `anthropic`、`openrouter` 和 `auto`。

`auto` 会优先使用 Anthropic key，失败时回退 OpenRouter；位置消偏策略与 A/B/tie 判定和后端无关。

### 两种核心评分方法

- Bradley-Terry（推荐）：更稳定，适合正式排行。
- Online Elo：更贴近课程里的机制讲解，速度快但对顺序敏感。

### 项目结构

同上方英文学段落中的文件列表。

### 使用示例

核心示例同上英文学：
- `python cli.py battle ...`
- `python cli.py elo ...`
- `python cli.py leaderboard ...`
- `python demo.py` / `python benchmark.py`

### 注意

- `--sample`、`--pipeline`、`--top-n` 等参数见命令行帮助。
- 建议先看 CLI 输出再对照 `leaderboard` 与可视化文件确认理解。
