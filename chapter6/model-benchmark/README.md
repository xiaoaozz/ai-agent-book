# Multi-dimensional Model Benchmarking / 多维度模型性能基准 / 实验 6-8

**Experiment 6-8 / 实验 6-8**: Model Performance Benchmark Across Multiple Dimensions.

## English

This directory implements a practical benchmarking harness for comparing multiple OpenAI-compatible LLM providers. One command can produce a table including **TTFT, end-to-end latency, throughput, std, p50/p95/p99, and success rate**.

It supports:
- **Concurrency stress testing**: sweep concurrency to identify rate limits and observe metric curves.
- **Offline mock mode** (`--mock`): synthetic data pipeline for verifying aggregation logic without API keys/network.

The original chapter workflow describes hourly probes, multiple context windows, and threshold checks. This project focuses on the core locally reproducible piece: measure TTFT precisely via streaming, evaluate percentile latency under concurrency, and use success rate as availability signal.

## Metric definitions

| Metric | Meaning | How measured |
|---|---|---|
| Success rate | Availability | failed request count / total |
| TTFT | Time to first token | stream first non-empty chunk - request start |
| End-to-end latency | complete response time | request start -> final chunk |
| Throughput (tokens/s) | generation speed | output token count / (end-to-end - TTFT) |
| p50 / p95 / p99 | latency percentiles | interpolation over successful requests |
| std | standard deviation | per-provider latency dispersion |
| aggregate throughput / RPS | batch throughput metrics | aggregated output token rate and request rate |

If usage usage-completion is unavailable, token count falls back to chunk-count approximation with documented caveat.

## Run

```bash
cd chapter6/model-benchmark
pip install -r requirements.txt

cp env.example .env
# or export OPENAI_API_KEY=... MOONSHOT_API_KEY=... ARK_API_KEY=...

python demo.py
```

### Common parameters

```bash
python demo.py --list
python demo.py --num-requests 20 --concurrency 5
python demo.py --serial
python demo.py --max-tokens 256
```

## Specify custom endpoint/model

Use `--base-url`, `--model`, and `--api-key-env` to test a new provider without changing `DEFAULT_PROVIDERS`.

```bash
python demo.py --base-url https://api.deepseek.com --model deepseek-chat \
  --api-key-env DEEPSEEK_API_KEY --name "DeepSeek官方/deepseek-chat"
```

## Concurrency sweep

```bash
python demo.py --model gpt-5.6-luna --concurrency-sweep 1,2,4,8,16 --num-requests 100
```

As concurrency increases, p95/p99/std generally get worse, and success rate may drop due to rate limits; aggregate throughput/RPS usually rises then plateaus.

## Metrics and export

```bash
python demo.py --metrics ttft,throughput
python demo.py --output result.json
```

## Offline mock validation

```bash
python demo.py --mock
python demo.py --mock --concurrency-sweep 1,2,4,8,16
```

This validates full aggregation logic with synthetic numbers labelled `[SYNTHETIC]`.

## Default providers

`DEFAULT_PROVIDERS` includes the keys that are present in environment:

- OpenAI-compatible entries (gpt-5.6-luna)
- Moonshot / doubao (explicit base_url + key)

OpenRouter fallback behavior:
- If `OPENAI_API_KEY` is missing, OpenAI-style entries can still run via OpenRouter (`OPENROUTER_API_KEY`), with model id mapping.
- For gpt-5.x, OpenRouter is preferred when `OPENROUTER_API_KEY` exists.

## Files

| File | Purpose |
|---|---|
| `benchmark.py` | core benchmark core: provider config, streaming measure, concurrency scheduling, aggregation |
| `demo.py` | CLI, parameter parsing, reporting and mock mode |
| `requirements.txt` | dependencies |
| `env.example` | env templates |

## Limitations

- Default parameters are low-cost defaults (`N=10`, `concurrency=3`, `max_tokens=64`).
- Larger requests increase cost and rate-limit risk.
- TTFT depends heavily on geography/network.
- Offline mock is for method validation only, not production decisions.

---

## 中文

这个目录用于构建**多模型、多提供商**的可复现延迟与成本评测。核心目标是让一条命令输出一张“可落地”的对比表：
TTFT、端到端延迟、吞吐、标准差、p50/p95/p99、成功率。

支持两条主线：
- **并发压测**：逐步提高并发找限流点；
- **离线自检（`--mock`）**：无 key/网络下验证聚合计算逻辑。

### 指标口径

- 成功率：任何异常（超时/限流/报错）都记为失败；不影响整表继续跑。
- TTFT：流式首个有效 chunk 到达时间差。
- 端到端：请求开始到最后一个 chunk 到达。
- 吞吐：输出 token /（端到端 - TTFT）。
- p50/p95/p99/std：基于成功样本的延迟分布。
- 聚合吞吐/RPS：每批次成功 token 总数和墙钟时间。

### 快速运行

```bash
cd chapter6/model-benchmark
pip install -r requirements.txt
cp env.example .env
python demo.py
```

### 常用参数

- `--list`：列出将测试的提供商
- `--num-requests` 与 `--concurrency`：样本与并发
- `--serial`：并发=1 的基线
- `--max-tokens`：拉高输出长度以观察吞吐

### 支持任意兼容端点

`--base-url / --model / --api-key-env` 允许不用改代码就测新模型、新服务商。

### 并发扫描

通过 `--concurrency-sweep` 观察：
- 并发上升时长尾通常变差（p95/p99/std 上升）
- 成功率可能受限流下跌
- 聚合吞吐先升后稳（平台上限出现后趋于平台容量）

### 文件说明

- `benchmark.py`：核心逻辑（provider/调度/聚合）
- `demo.py`：命令行入口
- `requirements.txt`：依赖
- `env.example`：环境变量模板

### 注意

- 默认参数的目的是低成本快速验证（约 `N=10`、并发 3、`max_tokens=64`）。
- 提高规模前先评估调用费用与限流影响。
- TTFT 与部署网络关联强，海外/国内可能量级不同。
