# Kaggle

连接：Kaggle MCP。比赛 Name That LLM，编号 170484。原文见 `assets/kaggle-sync-name-that-llm/overview.md`。

## 提交约束

- 只能通过 Notebook 提交；CPU / GPU Notebook 运行 ≤ 9 小时；禁止联网。
- 允许免费公开的外部数据和预训练模型，以 Kaggle 数据集形式挂载。
- 输出文件名 `submission.parquet`，格式见 `assets/data/sample_submission.parquet`。
- 每日最多 5 次提交；最终截止 2027-01-13 23:59 UTC（所有截止均为当日 23:59 UTC）。
- 正式提交前须用户确认；每次记下 Notebook 版本和回执，结果不明时先查询，不重复提交。

## 指标与 LB

- 指标 balanced accuracy，越高越好。测试约 4000 段，public / private 类别构成不同。
- 同一代码重复提交分数波动 0.006–0.012；public 与 private 的抽样噪声标准差约 0.015。LB 只作粗检，选提交以 OOF 为准。详见 `docs/research/competition-insight.md`。

## 榜单监控

外部只读接口，说明见 `tools/kaggleLB_api.md`。密钥放环境变量 `KAGGLE_LB_API_KEY`，不写入文件。
