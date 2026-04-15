[跳转到主要内容](https://code.claude.com/docs/zh-CN/scheduled-tasks#content-area)

[Claude Code Docs home page![light logo](https://mintcdn.com/claude-code/c5r9_6tjPMzFdDDT/logo/light.svg?fit=max&auto=format&n=c5r9_6tjPMzFdDDT&q=85&s=78fd01ff4f4340295a4f66e2ea54903c)![dark logo](https://mintcdn.com/claude-code/c5r9_6tjPMzFdDDT/logo/dark.svg?fit=max&auto=format&n=c5r9_6tjPMzFdDDT&q=85&s=1298a0c3b3a1da603b190d0de0e31712)](https://code.claude.com/docs/zh-CN/overview)

![CN](https://d3gk2c5xim1je2.cloudfront.net/flags/CN.svg)

简体中文

搜索...

Ctrl K询问AI

搜索...

Navigation

自动化

按计划运行提示词

[快速开始](https://code.claude.com/docs/zh-CN/overview) [使用 Claude Code 构建](https://code.claude.com/docs/zh-CN/sub-agents) [部署](https://code.claude.com/docs/zh-CN/third-party-integrations) [管理](https://code.claude.com/docs/zh-CN/setup) [配置](https://code.claude.com/docs/zh-CN/settings) [参考](https://code.claude.com/docs/zh-CN/cli-reference) [资源](https://code.claude.com/docs/zh-CN/legal-and-compliance)

在此页面

- [比较调度选项](https://code.claude.com/docs/zh-CN/scheduled-tasks#%E6%AF%94%E8%BE%83%E8%B0%83%E5%BA%A6%E9%80%89%E9%A1%B9)
- [使用 /loop 计划重复提示词](https://code.claude.com/docs/zh-CN/scheduled-tasks#%E4%BD%BF%E7%94%A8-%2Floop-%E8%AE%A1%E5%88%92%E9%87%8D%E5%A4%8D%E6%8F%90%E7%A4%BA%E8%AF%8D)
- [间隔语法](https://code.claude.com/docs/zh-CN/scheduled-tasks#%E9%97%B4%E9%9A%94%E8%AF%AD%E6%B3%95)
- [循环另一个命令](https://code.claude.com/docs/zh-CN/scheduled-tasks#%E5%BE%AA%E7%8E%AF%E5%8F%A6%E4%B8%80%E4%B8%AA%E5%91%BD%E4%BB%A4)
- [设置一次性提醒](https://code.claude.com/docs/zh-CN/scheduled-tasks#%E8%AE%BE%E7%BD%AE%E4%B8%80%E6%AC%A1%E6%80%A7%E6%8F%90%E9%86%92)
- [管理计划任务](https://code.claude.com/docs/zh-CN/scheduled-tasks#%E7%AE%A1%E7%90%86%E8%AE%A1%E5%88%92%E4%BB%BB%E5%8A%A1)
- [计划任务如何运行](https://code.claude.com/docs/zh-CN/scheduled-tasks#%E8%AE%A1%E5%88%92%E4%BB%BB%E5%8A%A1%E5%A6%82%E4%BD%95%E8%BF%90%E8%A1%8C)
- [抖动](https://code.claude.com/docs/zh-CN/scheduled-tasks#%E6%8A%96%E5%8A%A8)
- [七天过期](https://code.claude.com/docs/zh-CN/scheduled-tasks#%E4%B8%83%E5%A4%A9%E8%BF%87%E6%9C%9F)
- [Cron 表达式参考](https://code.claude.com/docs/zh-CN/scheduled-tasks#cron-%E8%A1%A8%E8%BE%BE%E5%BC%8F%E5%8F%82%E8%80%83)
- [禁用计划任务](https://code.claude.com/docs/zh-CN/scheduled-tasks#%E7%A6%81%E7%94%A8%E8%AE%A1%E5%88%92%E4%BB%BB%E5%8A%A1)
- [限制](https://code.claude.com/docs/zh-CN/scheduled-tasks#%E9%99%90%E5%88%B6)

计划任务需要 Claude Code v2.1.72 或更高版本。使用 `claude --version` 检查您的版本。

计划任务让 Claude 按间隔自动重新运行提示词。使用它们来轮询部署、监督 PR、检查长时间运行的构建，或在会话中稍后提醒自己做某事。要对事件进行实时反应而不是轮询，请参阅 [Channels](https://code.claude.com/docs/zh-CN/channels)：您的 CI 可以直接将失败推送到会话中。任务是会话范围的：它们存在于当前 Claude Code 进程中，当您退出时就会消失。对于需要在重启后继续运行的持久调度，请使用 [Cloud](https://code.claude.com/docs/zh-CN/web-scheduled-tasks) 或 [Desktop](https://code.claude.com/docs/zh-CN/desktop#schedule-recurring-tasks) 计划任务，或 [GitHub Actions](https://code.claude.com/docs/zh-CN/github-actions)。

## [​](https://code.claude.com/docs/zh-CN/scheduled-tasks\#%E6%AF%94%E8%BE%83%E8%B0%83%E5%BA%A6%E9%80%89%E9%A1%B9)  比较调度选项

Claude Code offers three ways to schedule recurring work:

|  | [Cloud](https://code.claude.com/docs/en/web-scheduled-tasks) | [Desktop](https://code.claude.com/docs/en/desktop-scheduled-tasks) | [`/loop`](https://code.claude.com/docs/en/scheduled-tasks) |
| --- | --- | --- | --- |
| Runs on | Anthropic cloud | Your machine | Your machine |
| Requires machine on | No | Yes | Yes |
| Requires open session | No | No | Yes |
| Persistent across restarts | Yes | Yes | No (session-scoped) |
| Access to local files | No (fresh clone) | Yes | Yes |
| MCP servers | Connectors configured per task | [Config files](https://code.claude.com/docs/en/mcp) and connectors | Inherits from session |
| Permission prompts | No (runs autonomously) | Configurable per task | Inherits from session |
| Customizable schedule | Via `/schedule` in the CLI | Yes | Yes |
| Minimum interval | 1 hour | 1 minute | 1 minute |

Use **cloud tasks** for work that should run reliably without your machine. Use **Desktop tasks** when you need access to local files and tools. Use **`/loop`** for quick polling during a session.

## [​](https://code.claude.com/docs/zh-CN/scheduled-tasks\#%E4%BD%BF%E7%94%A8-/loop-%E8%AE%A1%E5%88%92%E9%87%8D%E5%A4%8D%E6%8F%90%E7%A4%BA%E8%AF%8D)  使用 /loop 计划重复提示词

`/loop` [捆绑技能](https://code.claude.com/docs/zh-CN/skills#bundled-skills) 是计划重复提示词的最快方式。传递可选的间隔和提示词，Claude 会设置一个在后台运行的 cron 作业，同时会话保持打开。

```
/loop 5m check if the deployment finished and tell me what happened
```

Claude 解析间隔，将其转换为 cron 表达式，计划作业，并确认频率和作业 ID。

### [​](https://code.claude.com/docs/zh-CN/scheduled-tasks\#%E9%97%B4%E9%9A%94%E8%AF%AD%E6%B3%95)  间隔语法

间隔是可选的。您可以在开头使用它们、在末尾使用它们，或完全省略它们。

| 形式 | 示例 | 解析的间隔 |
| --- | --- | --- |
| 前导令牌 | `/loop 30m check the build` | 每 30 分钟 |
| 尾部 `every` 子句 | `/loop check the build every 2 hours` | 每 2 小时 |
| 无间隔 | `/loop check the build` | 默认为每 10 分钟 |

支持的单位是 `s` 表示秒、`m` 表示分钟、`h` 表示小时、`d` 表示天。秒数向上舍入到最近的分钟，因为 cron 的粒度为一分钟。不能均匀分割其单位的间隔，例如 `7m` 或 `90m`，会舍入到最近的整数间隔，Claude 会告诉您它选择了什么。

### [​](https://code.claude.com/docs/zh-CN/scheduled-tasks\#%E5%BE%AA%E7%8E%AF%E5%8F%A6%E4%B8%80%E4%B8%AA%E5%91%BD%E4%BB%A4)  循环另一个命令

计划的提示词本身可以是命令或技能调用。这对于重新运行您已经打包的工作流很有用。

```
/loop 20m /review-pr 1234
```

每次作业触发时，Claude 都会运行 `/review-pr 1234`，就像您输入了它一样。

## [​](https://code.claude.com/docs/zh-CN/scheduled-tasks\#%E8%AE%BE%E7%BD%AE%E4%B8%80%E6%AC%A1%E6%80%A7%E6%8F%90%E9%86%92)  设置一次性提醒

对于一次性提醒，用自然语言描述您想要的内容，而不是使用 `/loop`。Claude 计划一个单次触发的任务，该任务在运行后删除自己。

```
remind me at 3pm to push the release branch
```

```
in 45 minutes, check whether the integration tests passed
```

Claude 使用 cron 表达式将触发时间固定到特定的分钟和小时，并确认何时触发。

## [​](https://code.claude.com/docs/zh-CN/scheduled-tasks\#%E7%AE%A1%E7%90%86%E8%AE%A1%E5%88%92%E4%BB%BB%E5%8A%A1)  管理计划任务

用自然语言要求 Claude 列出或取消任务，或直接引用底层工具。

```
what scheduled tasks do I have?
```

```
cancel the deploy check job
```

在幕后，Claude 使用这些工具：

| 工具 | 目的 |
| --- | --- |
| `CronCreate` | 计划新任务。接受 5 字段 cron 表达式、要运行的提示词以及是否重复或仅触发一次。 |
| `CronList` | 列出所有计划任务及其 ID、计划和提示词。 |
| `CronDelete` | 按 ID 取消任务。 |

每个计划任务都有一个 8 字符的 ID，您可以将其传递给 `CronDelete`。一个会话最多可以同时保存 50 个计划任务。

## [​](https://code.claude.com/docs/zh-CN/scheduled-tasks\#%E8%AE%A1%E5%88%92%E4%BB%BB%E5%8A%A1%E5%A6%82%E4%BD%95%E8%BF%90%E8%A1%8C)  计划任务如何运行

调度程序每秒检查一次到期的任务，并以低优先级将其加入队列。计划的提示词在您的回合之间触发，而不是在 Claude 正在响应时。如果 Claude 在任务到期时忙碌，提示词会等到当前回合结束。所有时间都在您的本地时区中解释。像 `0 9 * * *` 这样的 cron 表达式意味着 9am 在您运行 Claude Code 的任何地方，而不是 UTC。

### [​](https://code.claude.com/docs/zh-CN/scheduled-tasks\#%E6%8A%96%E5%8A%A8)  抖动

为了避免每个会话在同一个挂钟时刻击中 API，调度程序会向触发时间添加一个小的确定性偏移：

- 重复任务最多晚触发其周期的 10%，上限为 15 分钟。每小时的作业可能在 `:00` 到 `:06` 之间的任何时间触发。
- 为小时顶部或底部计划的一次性任务最多提前 90 秒触发。

偏移是从任务 ID 派生的，所以相同的任务总是获得相同的偏移。如果精确的时间很重要，选择不是 `:00` 或 `:30` 的分钟，例如 `3 9 * * *` 而不是 `0 9 * * *`，一次性抖动将不适用。

### [​](https://code.claude.com/docs/zh-CN/scheduled-tasks\#%E4%B8%83%E5%A4%A9%E8%BF%87%E6%9C%9F)  七天过期

重复任务在创建后 7 天自动过期。任务最后触发一次，然后删除自己。这限制了被遗忘的循环可以运行多长时间。如果您需要重复任务持续更长时间，请在过期前取消并重新创建它，或使用 [Cloud 计划任务](https://code.claude.com/docs/zh-CN/web-scheduled-tasks) 或 [Desktop 计划任务](https://code.claude.com/docs/zh-CN/desktop#schedule-recurring-tasks) 进行持久调度。

## [​](https://code.claude.com/docs/zh-CN/scheduled-tasks\#cron-%E8%A1%A8%E8%BE%BE%E5%BC%8F%E5%8F%82%E8%80%83)  Cron 表达式参考

`CronCreate` 接受标准 5 字段 cron 表达式：`minute hour day-of-month month day-of-week`。所有字段都支持通配符 (`*`)、单个值 (`5`)、步长 (`*/15`)、范围 (`1-5`) 和逗号分隔的列表 (`1,15,30`)。

| 示例 | 含义 |
| --- | --- |
| `*/5 * * * *` | 每 5 分钟 |
| `0 * * * *` | 每小时整点 |
| `7 * * * *` | 每小时的第 7 分钟 |
| `0 9 * * *` | 每天本地时间 9am |
| `0 9 * * 1-5` | 工作日本地时间 9am |
| `30 14 15 3 *` | 3 月 15 日本地时间下午 2:30 |

星期几使用 `0` 或 `7` 表示星期日，`6` 表示星期六。不支持扩展语法如 `L`、`W`、`?` 和名称别名如 `MON` 或 `JAN`。当月份日期和星期几都受到限制时，如果任一字段匹配，日期就匹配。这遵循标准的 vixie-cron 语义。

## [​](https://code.claude.com/docs/zh-CN/scheduled-tasks\#%E7%A6%81%E7%94%A8%E8%AE%A1%E5%88%92%E4%BB%BB%E5%8A%A1)  禁用计划任务

在您的环境中设置 `CLAUDE_CODE_DISABLE_CRON=1` 以完全禁用调度程序。cron 工具和 `/loop` 变得不可用，任何已计划的任务都停止触发。有关禁用标志的完整列表，请参阅 [环境变量](https://code.claude.com/docs/zh-CN/env-vars)。

## [​](https://code.claude.com/docs/zh-CN/scheduled-tasks\#%E9%99%90%E5%88%B6)  限制

会话范围的调度有固有的限制：

- 任务仅在 Claude Code 运行且空闲时触发。关闭终端或让会话退出会取消所有内容。
- 没有错过触发的追赶。如果任务的计划时间在 Claude 忙于长时间运行的请求时经过，它会在 Claude 变为空闲时触发一次，而不是每个错过的间隔触发一次。
- 没有跨重启的持久性。重启 Claude Code 会清除所有会话范围的任务。

对于需要无人值守运行的 cron 驱动自动化：

- [Cloud 计划任务](https://code.claude.com/docs/zh-CN/web-scheduled-tasks)：在 Anthropic 管理的基础设施上运行
- [GitHub Actions](https://code.claude.com/docs/zh-CN/github-actions)：在 CI 中使用 `schedule` 触发器
- [Desktop 计划任务](https://code.claude.com/docs/zh-CN/desktop#schedule-recurring-tasks)：在您的机器上本地运行

此页面对您有帮助吗？

是否

[将外部事件推送到 Claude](https://code.claude.com/docs/zh-CN/channels) [编程使用](https://code.claude.com/docs/zh-CN/headless)

Ctrl+I

助手

AI生成的回答可能包含错误。