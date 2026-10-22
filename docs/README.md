# SDUI MVP 项目文档

## 文档目录

```text
docs/
├── README.md
├── technical-plan.md                 # 四人小白可执行版技术方案（推荐先读）
├── 01-product/
│   ├── product-brief.md
│   ├── mvp-scope.md
│   └── business-plan-outline.md
├── 02-architecture/
│   ├── system-architecture.md
│   └── decisions.md
├── 03-protocol/
│   ├── page-schema.md
│   └── component-registry.md
├── 04-frontend/
│   ├── editor.md
│   └── react-native-client.md
├── 05-backend/api-contract.md
├── 06-ai-copilot/copilot-flow.md
├── 07-quality/
│   ├── acceptance.md
│   └── security.md
├── 08-deployment/
│   ├── local-development.md
│   └── demo-runbook.md
├── 09-delivery/
│   ├── schedule-and-owners.md
│   └── hackathon-submission.md
├── adr/0001-rn-and-json-rendering.md
└── meetings/README.md
```

## 阅读顺序

先读 `technical-plan.md`，按启动步骤运行各自模块；再读产品简介与 MVP 范围 → 架构 → 页面协议与 API → 运营台/RN/AI 实施 → 验收 → 本地启动与演示 Runbook。

## 约定

先更新协议/API 文档再改实现。文档状态使用“草案/实施中/已验收”。联调纪要记录到 `meetings/`，行动项要有负责人和日期。演示环境免登录且不得放真实客户数据。模型密钥只保存在服务端环境变量。

## 当前约束

- 两个核心场景必须可演示：拖拽修改并发布；自然语言经 AI 修改、人工确认并发布。
- 首选 React Native + Expo；若 10/24 前跑不通，备选 iOS SwiftUI。本次不做 Android 原生客户端。
- 动态更新只下发白名单 JSON 配置，不执行远程代码或更新 JS bundle/原生模块。
- 开发窗口 10/23–11/09；11/10 仅作缓冲。
- 提交三件套、收件人和标题格式见 `09-delivery/hackathon-submission.md`。
