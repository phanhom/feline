## Feline 项目评估
### feline-plugin 不足
类别 问题 影响 架构 whisker 是独立 Python 进程，需额外启动 部署复杂，资源占用高 配置 Profile/API Key 存本地，无法跨设备同步 换设备需重新配置 安全 API Key 明文存储在 feline.config.json 安全风险 Provider 仅支持 OpenAI/Claude/OpenRouter 缺少国内模型（DeepSeek、智谱、Kimi、通义） 功能 无 Token 用量统计、无订阅管理 用户无法了解使用情况 Thinking 不支持 thinking/reasoning 模式 无法使用 DeepSeek-R1 等推理模型 重试 无自动重试和 Provider 切换 单点故障，可靠性低

### feline-server 不足
类别 问题 影响 集成 未与 feline-plugin 对接 两套独立系统 订阅 Subscription 表独立，未与 Usage 关联 无法自动扣费 限制 无 Token 使用限制检查 用户可超额使用 计费 无 Stripe 支付集成 无法商业化 实时 无 WebSocket 支持 无法实时推送消息 安全 API Key 存配置文件，未加密 安全风险 限流 无速率限制 易被滥用 运维 无监控、日志聚合、告警 难以排查问题 迁移 无 Alembic 数据库迁移 表结构变更困难