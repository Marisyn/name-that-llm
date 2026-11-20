# 不入库资产的来源

新设备 pull 后按此恢复。这些目录由 `.gitignore` 排除。

| 本地路径 | 内容 | 来源 / 恢复方式 |
| --- | --- | --- |
| `assets/data/` | `train.csv`、`test.csv`、`sample_submission.parquet` | Kaggle 比赛数据（编号 170484），用 Kaggle CLI / MCP 下载 |
| `assets/kaggle-sync-name-that-llm/code/` | 公开 notebook 代码（约 328M） | 用 kaggle-sync 工具重新导出；导出参数见同目录 `manifest.json`。其余文件（overview、discussions、writeups 等）入库 |
| `assets/external/open_pool/` | Open Pool v15（`ntl_open_pool.parquet`，226,191 行）及清洗分析表 | 待补：Kaggle 数据集 slug 与版本 |
| `.venv/` | 本地 Python 3.12 环境 | 待补：依赖冻结文件 |
