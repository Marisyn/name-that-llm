# Kaggle 榜单数据接口 使用说明

通过这套接口可以读取 Kaggle 比赛的榜单数据，分析某支队伍的上分过程，并根据时间规律推测它的策略。
接口只用来查数据，不会修改任何内容。

- 地址：`https://lb-kaggle.nyrxa.com/api/agent/v1`
- 所有时间都是 **UTC（世界标准时间）**，格式如 `2026-11-19T08:50:08Z`。

---

## 1. 访问密钥

使用前需要向管理员申请一个访问密钥。每次请求都要在请求头里带上它：

```
Authorization: Bearer <你的密钥>
```

例如：

```bash
curl -s -H "Authorization: Bearer <你的密钥>" https://lb-kaggle.nyrxa.com/api/agent/v1/
```

- 没带密钥或密钥错误，会收到 `401`。
- 请妥善保管密钥，泄露后请联系管理员作废并重新申请。

---

## 2. 接口一览

下面的地址都接在 `https://lb-kaggle.nyrxa.com/api/agent/v1` 后面。`{cid}` 是比赛编号（如 `170484`），`{tid}` 是队伍编号。

| 接口 | 可选参数 | 用途 |
|------|----------|------|
| `/` | – | 接口清单和字段说明，建议先看这个 |
| `/competitions` | – | 可以查看的比赛 |
| `/competitions/{cid}/leaderboard` | `offset` 起始位置（默认 0），`limit` 条数（默认 50，最多 500） | 当前榜单 |
| `/competitions/{cid}/teams/search` | `q` 关键词（必填），`limit` | 按队名、成员用户名或队伍编号找队伍 |
| `/competitions/{cid}/teams/{tid}/timeline` | – | 这支队伍的每一次出分记录、组队和改名情况 |
| `/competitions/{cid}/teams/{tid}/features` | – | 这支队伍的行为特征汇总（**最常用**） |
| `/competitions/{cid}/compare` | `teams` 用逗号分隔的队伍编号（最多 10 个） | 多支队伍横向对比 |
| `/competitions/{cid}/events` | `since` 开始时间，`until` 结束时间，`types` 事件类型，`team_id` 队伍，`limit` | 全场的名次大跳和组队事件 |
| `/competitions/{cid}/improvements` | `since`，`until`，`score` 指定分数，`max_rank` 名次上限，`limit` | 全场的上分记录：谁在什么时候上到了多少分 |
| `/competitions/{cid}/score-clusters` | `window_hours` 时间窗口（默认 12 小时），`min_teams` 最少队伍数（默认 3），`limit` | 短时间内被很多队伍同时达到的分数 |

时间参数的写法如 `since=2026-11-19T00:00:00Z`。

---

## 3. 返回内容说明

### 3.1 比赛列表 `/competitions`

```json
[{"id": "170484", "name": "Name That LLM", "snapshot_time": "2026-11-20T07:45:12Z",
  "teams": 6511, "higher_is_better": true,
  "medal_counts": {"gold": 23, "silver": 302, "bronze": 326}}]
```

| 字段 | 含义 |
|------|------|
| `snapshot_time` | 榜单最近一次更新的时间 |
| `teams` | 队伍数 |
| `higher_is_better` | `true` 表示分数越高越好，`false` 表示越低越好 |
| `medal_counts` | 目前金、银、铜牌区各有多少队 |

### 3.2 队伍信息（榜单和搜索结果）

| 字段 | 含义 |
|------|------|
| `rank` | 当前名次 |
| `team_id`、`name`、`members` | 队伍编号、队名、成员用户名 |
| `score` | 当前公榜最好成绩 |
| `submissions` | 总提交次数 |
| `last_submission` | 最近一次提交的时间 |
| `medal` | 按当前名次所在的奖牌区：`gold` / `silver` / `bronze`，不在奖牌区为 `null` |

### 3.3 出分记录 `/timeline`

`submissions` 里每一行是一次在榜单上看到的变化：

| 字段 | 含义 |
|------|------|
| `idx` | 这是该队的第几次提交 |
| `event` | `init` / `Appeared`：开始记录时的初始状态；`submission_scored`：有新提交出分；`submission_scored_after_merge`：组队后出分；`score_adjustment`：官方调整了分数 |
| `submitted_at` | 当时该队**最近一次**提交的时间。如果一次提交了好几个，多行会显示同一个时间 |
| `scored_at` | 榜单上出现这次变化的时间 |
| `runtime_s` | 从提交到出分大约用了多少秒。代码类比赛中可以近似看作程序运行时长（实际只会更短） |
| `n` | 这一行包含几次提交。很短时间内连续出分的会合并成一行 |
| `score_before` / `score` | 变化前后的最好成绩 |
| `delta` | 分数变化量 |
| `rank_before` / `rank_after` | 变化前后的名次 |
| `class` | `breakthrough` 名次提升一半以上；`leap` 名次提升两成以上；`score_up` 有提升；`effective` 出分了但没提升；`adjustment` 官方改分 |

`team_changes` 是这支队伍的变动记录：改名、成员变化、吸纳了别的队、并入了别的队、从榜单上消失等。

`merged_subteams` 是后来并入这支队的其他队伍，包括并入时间和并入前的分数变化。

### 3.4 行为特征 `/features`

**观测范围 `coverage`**
- `observed_from` / `observed_to` / `observed_hours`：数据覆盖的起止时间和总时长。下面所有特征都只基于这段时间。时间越短，结论越不可靠。

**提交习惯 `activity`**
- `scored_submissions_observed`：这段时间内出分的次数。
- `distinct_submit_actions`：提交的次数。同一时刻提交多个只算一次。
- `submit_hour_hist_utc`：24 个数字，分别是 UTC 0 点到 23 点各个小时的提交次数，可以看出作息。
- `daily_submit_actions`、`max_submit_actions_per_day`：每天提交几次、最多的一天提交几次，可以看出是否用满了每日的提交次数。
- `batch_submits`：同一时刻一次提交多个的情况，通常意味着在同时试多组参数或多个模型。
- `timezone_guess`：所在时区的推测。提交次数达到 15 次才会给出，否则为空。原理是找出一天中最长的不提交时段，假设那是当地凌晨 4 点前后的睡觉时间。
- `sessions`：把间隔不超过 3 小时的提交算作同一轮工作，给出共几轮、每轮一般提交几次，以及最近 10 轮的时间。

**运行时长 `runtime_s`**
- 最短、中位、最长的运行时长（秒）。时长突然变长或变短，往往说明换了模型或方法。

**上分情况 `progress`**
- `score_at_observation_start`：开始记录时的分数。
- `improvements`：上分次数。
- `submissions_per_improvement`：平均提交几次才上一次分。数值小，多半是大改方案；数值大，多半是在反复微调。
- `hours_between_improvements`、`longest_plateau_hours`、`hours_since_last_improvement`：每两次上分之间隔了多久、最长多久没上分、距离上一次上分已经多久。
- `steps`：每一次上分的详细情况，包括时间、分数、名次变化、运行时长。

### 3.5 全场事件 `/events`

按时间顺序列出两类事件：
- **名次大跳**：`breakthrough`、`leap`，带变化前后的名次、分数和提交时间。
- **队伍变动**：`name_change` 改名、`members_change` 成员变化、`absorbed` 吸纳别的队、`join` 并入别的队、`merge_update` 组队更新、`disappear` 从榜单消失，带中文说明。

例如 `types=breakthrough,leap` 只看名次大跳，`team_id=...` 只看某一支队。

### 3.6 全场上分记录 `/improvements`

列出所有分数发生变化的记录：哪支队、什么时候提交、什么时候出分、从多少分到多少分、名次怎么变。
- `score=0.703`：只看上到 0.703 的队伍，可以看出谁先到、谁后到。
- `max_rank=50`：只看上分后进入前 50 名的记录。

### 3.7 同分扎堆 `/score-clusters`

找出在一段时间内（`window_hours`）被很多支队伍同时达到的分数：

```json
{"score": "0.702", "teams": 89, "teams_total_reaching_score": 232,
 "window_start": "...", "window_end": "...", "sample": [ ... ]}
```

意思是：有 89 支队伍在 `window_start` 到 `window_end` 之间达到了 0.702，历史上一共有 232 支队伍到过这个分；`sample` 列出了其中一部分队伍。
很多队伍从不同的分数在短时间内跳到**同一个分数**，通常说明当时有人公开了方案或代码。

---

## 4. 推荐的分析步骤

1. 先看 `/` 和 `/competitions`，确认比赛编号，以及分数是越高越好还是越低越好。
2. 用 `/leaderboard?limit=30` 看前排队伍，或者用 `/teams/search?q=队名` 找到想分析的队伍。
3. 查看这支队伍的 `/features`：
   - 先看观测了多长时间，时间太短就不要下太肯定的结论；
   - 提交时间分布和时区推测：作息规律、可能在哪个地区；
   - 每天提交次数、一次提交多个：是否在用满提交次数大量尝试；
   - 运行时长的变化：是否换了更大的模型或更复杂的流程；
   - 平均几次提交上一次分、每次上分的幅度：是大改方案还是细调。
4. 对它的某一次上分，用 `/improvements?score=上分后的分数` 看同一时间有没有很多队伍到达同一个分，再用 `/score-clusters` 对照，判断这次上分是不是来自公开方案。
5. 如果有组队记录，看组队前后分数有没有明显跳变。
6. 需要逐条核对时看 `/timeline`；想比较几支前排队伍的节奏时用 `/compare`。

示例：

```bash
curl -s -H "Authorization: Bearer <你的密钥>" "https://lb-kaggle.nyrxa.com/api/agent/v1/competitions/170484/teams/17135987/features"
```

```bash
curl -s -H "Authorization: Bearer <你的密钥>" "https://lb-kaggle.nyrxa.com/api/agent/v1/competitions/170484/score-clusters?window_hours=6&min_teams=5"
```

---

## 5. 注意事项

- **只能看到公榜上的最好成绩**。没能上分的提交，只能看到提交次数加了 1，看不到它具体得了多少分。
- **只有开始记录之后的数据**。本站从 2026-11-18 16:51（UTC）开始记录，在此之前每支队伍只有一个初始状态。
- `submitted_at` 是"当时最近一次提交的时间"，不一定正好对应这一行的那次提交，所以运行时长只是一个大概的上限。
- 榜单每分钟更新一次，同一分钟内出分的多次提交会合并成一条。
- 时区推测只是参考：它假设作息规律、凌晨 4 点前后在睡觉。成员分布在不同时区的队伍不适用。
- 全场类的查询（`events`、`improvements`、`score-clusters`）在数据刚更新后的第一次请求可能要等几秒。
