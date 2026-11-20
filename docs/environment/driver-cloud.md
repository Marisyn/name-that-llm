# 驱动云

连接：`codex_bridge` 是专门用于驱动云的 MCP，驱动云上的所有操作都经由它进行；它只用于驱动云。GPU 数量由用户指定。

`codex_bridge` 适合传命令、短脚本和小配置；大文件用驱动云自身的传输方式放到 `assets/transmission/`，不经 bridge 用 Base64 分片转运。

项目环境脚本与正式 Python 路径：待补（首次连接时确认后写入）。

## 存储

只有两个目录可用，其他目录严格不可见（无论读写），按 I/O 特性分工：

| 目录 | 性质 | 存放内容 |
| --- | --- | --- |
| `/gemini/space/ckpts/kaggle/name-that-llm` | 网络盘：持久保存，I/O 较慢 | `WORKSPACE_DIR`。脚本、配置、命令文档、`analysis/`、日志、需保留的 checkpoint 和最终产物、环境构建依据（requirements、wheelhouse 等） |
| `/tmp` | 本地盘：读写快，但崩溃、重启或实例结束都会清空 | 运行中频繁读写的内容：数据集工作副本、解压数据、加载用的权重副本、各类缓存（pip、HF、编译）、venv 或安装目录、中间 checkpoint、大批量中间输出 |

- 高频读写在 `/tmp/name-that-llm/` 下进行，沿用与网络盘相同的 `runs/lineNN/expNNN/runNNN/` 层级。
- 运行开始时把需要的数据和权重从网络盘复制到 `/tmp` 再用。不在网络盘上做大量小文件读写，也不从网络盘直接加载大权重。
- 日志和指标直接写网络盘的 `analysis/`：数据量小，实例中断后仍能查看失败原因。
- 往 `/tmp` 放大文件前先 `df -h /tmp` 确认剩余空间。

## `/tmp` 随时可能被清空

系统崩溃、重启和实例结束都会让它的内容全部丢失，且没有预警。因此：

- 代码和配置先写在网络盘，再复制到 `/tmp`，不在 `/tmp` 里直接修改。
- 需保留的 checkpoint 和结果定期从 `/tmp` 复制到网络盘对应 run 的 `artifacts/`。间隔按"丢失这段进度能否接受"来定，不要只在结束时复制一次。
- 长训练要能从网络盘上最近的 checkpoint 续跑。
- 崩溃或重启后先检查 `/tmp` 实际状态，再从网络盘重新准备，不假设之前的内容还在。
- 报告中不把只存在于 `/tmp` 的文件说成"已保存"。

## 不删除，只移入 trash

在驱动云上不执行任何删除，由用户本人删除。

- 两个目录各有一个 trash：`/tmp/trash` 和 `/gemini/space/ckpts/kaggle/name-that-llm/trash`，不存在时 `mkdir -p` 创建。
- 要删除的内容用 `mv` 移到**同一磁盘**的 trash：`/tmp` 下的移到 `/tmp/trash`，网络盘下的移到网络盘 trash，不跨盘移动。
- 放在 `trash/<时间戳>/` 下，保留原相对路径，避免重名覆盖。
- `/tmp/trash` 同样会在重启时丢失，只放确定不再需要的内容。
- 移动后告诉用户移了哪些内容、各自原路径和大致大小。
- 禁止：`rm`、`rmdir`、`unlink`、`find -delete`、`rsync --delete`，代码中的 `os.remove`、`shutil.rmtree`、`Path.unlink` 等。运行的代码也不能自动删文件，例如 HF Trainer 的 `save_total_limit` 要关闭或改为移入 trash。
- **覆盖的边界**：数据、checkpoint、run 结果和 `artifacts/` 不覆盖，新结果写新路径；需要替换时先把旧的移入 trash。自己的脚本、配置、命令文档和 `docs/` 可以原地修改；pip、HF、编译器等工具在 `/tmp` 下自管理的缓存和临时文件不受此限。
- trash 不参与扫描、部署和 Git。
