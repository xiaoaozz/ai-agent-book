# Public-health Reporting Evaluation / 公共卫生报告评估示例

A small reproducible project in chapter 6 for evaluating an agent on **synthetic aggregate malaria-reporting data**.

## English

This project uses synthetic DHIS2-style aggregate reports to test tool use, deterministic scoring, and evidence-grounded claims. It is intentionally small and educational.

### What is evaluated

It includes five deterministic tasks:
1. Test positivity
2. Reporting completeness
3. Period-to-period trend comparison
4. Aggregate data-quality checks
5. Commodity stock-out review

Each prediction is a JSON trace with: selected tool, arguments, result, source-row evidence, and claims.
Each task is scored with 6 points:

- Tool selection: 1
- Arguments: 1
- Answer correctness: 2
- Evidence matching: 1
- Grounding/safety: 1

### Files

| File | Purpose |
|---|---|
| `data/synthetic_reports.csv` | Nine synthetic monthly aggregate rows |
| `tasks.json` | Prompts + deterministic plans |
| `expected_answers.json` | Ground truth answers and allowed claims |
| `reporting_tools.py` | Five auditable tools |
| `agent.py` | Deterministic reference agent |
| `evaluator.py` | 6-point objective scoring |
| `demo.py` | CLI for reference / external predictions |
| `test_offline.py` | Offline regression tests |

### Run offline

```bash
cd chapter6/public-health-reporting-eval
python demo.py
```

Expected output includes one line per task and a 30/30 total for the reference implementation.

### Offline tests

```bash
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
pytest -q
```

### Evaluate another agent

Provide predictions in the same JSON shape as this project and run:

```bash
python demo.py --predictions my_predictions.json --output evaluation.json
```

Any system can be evaluated as long as it emits the same trace format.

### Interpretation and limitations

- The benchmark is intentionally constrained and synthetic.
- Source-row evidence supports auditability but is not a production provenance system.
- Tool/argument matching is strict by design.
- Quality rules are illustrative.
- Positivity metrics are descriptive aggregate indicators, not clinical/forecast signals.

## 中文

这是一个面向《深入理解 AI Agent》第6章的小型可复现实践：在**合成 DHIS2 风格的疟疾上报聚合数据**上做 Agent 评测。

### 评测内容

包含 5 个确定性任务：
1. 阳性检出率
2. 报告完整性
3. 月度趋势比较
4. 聚合质量检查
5. 药品断货复核

每条预测输出为一段 JSON 结构，包含所选工具、参数、返回结果、证据行 ID、claim。评分为 6 分制：
- 工具选择（1）
- 参数匹配（1）
- 答案正确性（2）
- 证据可追溯（1）
- grounding/safety（1）

### 文件说明

同上英文表。

### 直接离线运行

```bash
cd chapter6/public-health-reporting-eval
python demo.py
```

### 运行测试

```bash
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
pytest -q
```

### 评测外部 Agent

把外部模型或 agent 的预测导出为同样结构的 JSON，再运行：

```bash
python demo.py --predictions my_predictions.json --output evaluation.json
```

### 使用边界与局限

- 评测面向受控合成环境，不代表真实系统可上线。
- source-row 证据便于审计，但不替代生产级数据血缘与权限体系。
- 工具和参数打分采用严格匹配。
- 质量规则是示例性规则，不可等同真实质量体系。
- 阳性率仅为聚合描述指标，不用于诊断或预测。
