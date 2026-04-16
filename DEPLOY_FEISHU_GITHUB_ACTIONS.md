# TrendRadar 飞书部署说明

这份说明对应仓库 README 里的“方案二：GitHub Actions 部署”，并且使用“飞书方案二：机器人应用 + 流程设计 + 发送飞书消息”。

## 这份仓库里我已经帮你做的调整

- 已把 `config/config.yaml` 的 `filter.method` 改成 `keyword`
- 已关闭 `ai_analysis.enabled`
- 已关闭 `ai_translation.enabled`
- 已关闭 `display.regions.ai_analysis`

这样做的目的很简单：你先只配飞书 Webhook，也能把 GitHub Actions 跑起来，不会因为没配 `AI_API_KEY` 在第一次执行时直接失败。

如果你后面想加 AI 分析，再把下面这些打开即可：

- `config/config.yaml` 里的 `filter.method: "ai"`
- `ai_analysis.enabled: true`
- `ai_translation.enabled: true`（可选）
- GitHub Secrets 里的 `AI_API_KEY`
- GitHub Secrets 里的 `AI_MODEL`

## 一、在飞书里配置“方案二”

1. 打开 `https://botbuilder.feishu.cn/home/my-app`
2. 点击“新建机器人应用”
3. 进入应用后，点击“流程设计”
4. 点击“创建流程”
5. 选择触发器“Webhook 触发”
6. 复制飞书生成的 Webhook 地址
7. 点击“选择操作”
8. 选择“发送飞书消息”
9. 勾选“群消息”
10. 在“我管理的群组”里选择你要接收推送的飞书群
11. 消息标题填 `TrendRadar 热点监控`
12. 发布流程

你最后真正要用到的，就是第 6 步复制出来的 Webhook。

## 二、Fork 并开启 GitHub Actions

1. 打开原仓库：`https://github.com/sansan0/TrendRadar`
2. 点击右上角 `Fork`
3. 进入你自己的 Fork 仓库
4. 打开 `Actions`
5. 启用工作流

仓库里的主工作流是：

- `.github/workflows/crawler.yml`

它默认会按 `cron: "33 * * * *"` 运行，也就是每小时第 33 分钟跑一次。

## 三、配置 GitHub Secrets

在你自己的 Fork 仓库里进入：

- `Settings`
- `Secrets and variables`
- `Actions`
- `New repository secret`

至少添加这一个：

- Name: `FEISHU_WEBHOOK_URL`
- Secret: 你在飞书“方案二”里复制出来的 Webhook

如果你想同时推送到多个飞书群，可以这样填：

```text
https://xxx1;https://xxx2;https://xxx3
```

## 四、需要不要配远程存储

先说结论：

- 只想先把飞书通知跑通：可以先不配
- 想长期稳定保留历史数据和 HTML 报告：建议再配 S3 兼容存储

如果后面你要补远程存储，再加这些 Secrets：

- `S3_BUCKET_NAME`
- `S3_ACCESS_KEY_ID`
- `S3_SECRET_ACCESS_KEY`
- `S3_ENDPOINT_URL`
- `S3_REGION`（部分服务商需要）

## 五、手动测试一次

配置好 Secret 后，在仓库里执行：

1. 打开 `Actions`
2. 选择 `Get Hot News`
3. 点击 `Run workflow`

正常情况下，几分钟后你选中的飞书群里就会收到一条 TrendRadar 推送。

## 六、如果你想改推送频率

改这个文件：

- `.github/workflows/crawler.yml`

默认值：

```yaml
- cron: "33 * * * *"
```

常用示例：

```yaml
- cron: "0 */2 * * *"
```

这表示每 2 小时运行一次。

## 七、如果你后面要开启 AI

建议最小配置如下：

- Secret: `AI_API_KEY`
- Secret: `AI_MODEL`

例如：

- `AI_API_KEY=你的 key`
- `AI_MODEL=deepseek/deepseek-chat`

然后把 `config/config.yaml` 这些项打开：

```yaml
filter:
  method: "ai"

ai_analysis:
  enabled: true

display:
  regions:
    ai_analysis: true
```

## 八、这套方案的边界

这次我能替你做的是把仓库收成“可直接 fork 使用”的状态，并把飞书方案二的接法整理成最短路径。

我不能直接替你完成下面两步，因为它们需要你的账号权限：

- 在你的飞书里创建机器人应用和流程
- 在你的 GitHub Fork 里新增 `FEISHU_WEBHOOK_URL` Secret

如果你愿意，我下一步可以继续帮你做两件事之一：

1. 把这个仓库再改成“只推送你关心的关键词”
2. 给你补一版“带 DeepSeek AI 分析”的 GitHub Actions 配置清单
