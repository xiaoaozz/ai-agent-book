# Agent End-to-End Cost Analysis / Agent 任务端到端成本分析（实验 6-7）

## English

This project performs a full cost decomposition for a multi-turn agent workflow (refund handling), including input/cache/output tokens, latency, and cost distribution. It enables practical measurement of where costs come from and how optimization strategies affect total spend.

### What it does

The benchmark runs a fixed 8-turn customer refund scenario and records every LLM call with a lightweight tracing layer:
- token usage (prompt, cached prompt, output)
- latency
- model cost by pricing table

It then reports:
- per-step cost breakdown
- cost component breakdown (non-cached input / cached input / output)
- p50/p95/p99 for per-step cost
- full 2×2 A/B comparison between optimization levers

The two levers are:
- **KV-cache friendliness**: keep a stable prefix to maximize cache hits
- **Context compression**: summarize long tool outputs for earlier turns

### Default model and API behavior

- Default model: `gpt-5.6-luna`.
- Preferred credentials: `OPENAI_API_KEY`, fallback to `OPENROUTER_API_KEY` (`gpt-*` remapped to `openai/*`).
- If OpenRouter keys exist for `gpt-5.x`, it is preferred due to authentication requirements.
- Offline mode is supported via `sample_trace.json` so all tables can be recomputed without API calls.

### Files

| File | Purpose |
|---|---|
| `config.py` | pricing model definitions and pricing presets |
| `tracer.py` | tracing helper and cost decomposition/aggregation |
| `agent.py` | 8-turn refund agent with `run_scenario(kv_cache, compress)` |
| `demo.py` | CLI entry for online/offline runs |
| `sample_trace.json` | captured 2×2 scenario token records for offline recomputation |
| `requirements.txt` / `env.example` | dependencies and environment templates |

### Run

```bash
pip install -r requirements.txt

export OPENAI_API_KEY=sk-...   # or OPENROUTER_API_KEY=sk-or-...
python demo.py
python demo.py --offline --scenario all
```

### CLI options

| Argument | Meaning |
|---|---|
| `--live` / `--offline` | call real model (default) / recompute from trace |
| `--scenario` | `ab` (naive+both), `all` (four scenarios), or subset list |
| `--trace` | trace file for offline mode |
| `--save-trace` | persist observed token usage from online runs |
| `--model` | model name for price preset |
| `--price-input` / `--price-cached` / `--price-output` | override per-million-token prices |
| `--no-warmup` | disable prefix warmup for KV-cache scenario |
| `--output` | export full result JSON |

### A/B scenarios (2×2)

| Scenario | KV-cache | Compression | Context design |
|---|---|---|---|
| `naive` | no | no | random session header + full tool returns |
| `kv` | yes | no | stable long prefix |
| `compress` | no | yes | only keep last 2 turns full; older turns summarized |
| `both` | yes | yes | stable prefix + compressed history |

The task logic is identical across scenarios so differences isolate optimization effects.

### Interpretation

Empirical results show:
- KV-cache can produce large improvements when prefixes are stable.
- Compression lowers prompt growth while keeping functional behavior.
- Joint optimization usually gives best total cost, though cache gains and compression gains are not simply additive.

### Offline recomputation

Offline mode reads `sample_trace.json` and re-runs only cost arithmetic, enabling:
- quick replication without keys
- quick “what-if” with different model prices

### Notes

- Observed numbers can vary due to real API behavior and cache timing.
- Prompt cache is best-effort and may miss in some turns.
- Tool-return token estimate uses tokenizer counts against current model encoder.
- Key precedence: prefer `OPENAI_API_KEY`; fallback is automatic via OpenRouter for supported paths.

---

## 中文

该项目对一个 8 轮客服退款任务做**完整端到端成本拆解**：逐步记录 LLM 的输入/缓存/输出 token、时延和价格，输出“哪一步最贵”和“成本从哪来”。

核心是验证书中问题：
- KV-cache 在真实 API 下是否真的便宜？
- 上下文压缩是否有效降低输入规模？
- 两者叠加是否产生协同收益？

### 关键特性

- 默认模型 `gpt-5.6-luna`，无 key 时走 OpenRouter 回退。
- 提供离线复算模式，使用 `sample_trace.json` 的真实 token 记录，不联网也能复现报表。
- 输出包括：
  - 按步骤的成本明细
  - 成本构成（未缓存输入 / 缓存输入 / 输出）
  - 单步成本分布（p50/p95/p99）
  - 2×2 A/B 对比表

### 文件

- `config.py`：价格定义与换价逻辑。
- `tracer.py`：轻量 tracing 与成本聚合。
- `agent.py`：退款任务实现与 `run_scenario(kv_cache, compress)`。
- `demo.py`：CLI 入口。
- `sample_trace.json`：离线复算的真实 token 用量样本。

### 快速运行

```bash
pip install -r requirements.txt
export OPENAI_API_KEY=sk-...
python demo.py
python demo.py --offline --scenario all
```

### 参数示例

- `--scenario ab`：默认朴素与优化
- `--scenario all`：四组场景（naive / kv / compress / both）
- `--model gpt-4o`：按其他模型定价复算
- `--no-warmup`：关闭 KV-cache 组预热
- `--output`：导出完整结果 JSON

### 结果解读

- `kv` 场景主要节省缓存单元价格，适合前缀稳定任务；
- `compress` 场景控制历史膨胀，降低未缓存输入；
- `both` 通常是综合最优，但缓存量和压缩收益并非线性相加。

> 例如：B 优化（kv+压缩）比朴素组显著降本，但缓存 token 也可能小于“仅 kv”场景，因为压缩减少了可复用历史长度。
