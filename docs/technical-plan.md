# SDUI MVP 技术方案（4 人小白可执行版）

> 周期：2026-10-23 至 11-09 开发，11/10 缓冲。四人异步线上协作，每周至少两次联调。
> 约束：只做两个核心场景——画布拖拽调整并发布、AI Copilot 修改并由人确认后发布。不做产品登录页；客户端首选 React Native + Expo，不做 Android 原生。若 RN 到 10/24 仍无法启动，再评估 SwiftUI，不能并行维护两套。
> 参考结构：借鉴《美食探店地图·技术方案（小白可执行版）》的一页速览、铁律、数据契约、文件归属、逐人任务、排期、降级链和验收清单，并按 SDUI 改写。

---

## 0. 一页速览

评审打开浏览器运营画布，把首页 Banner 和商品列表拖动换位，保存并发布；RN 客户端刷新后显示新顺序。接着在 Copilot 输入“把按钮放到 Banner 下方，主题色改为橙色”，查看 AI 候选差异，人工确认并发布，RN 再刷新看到变化。

| 部分 | MVP 做法 |
|---|---|
| 运营画布 | 单页 HTML/CSS/JS，免登录，仅编辑 `home` |
| 客户端 | React Native + Expo + TypeScript，渲染本地白名单组件 |
| 后端 | FastAPI + SQLite，保存草稿和已发布快照，服务端代理模型调用 |
| 动态更新 | 仅拉取经过校验的 JSON；不下发或执行 JS、HTML、原生代码 |
| 交付 | 可试用作品、商业计划书、演示视频；邮件规则见 [提交说明](09-delivery/hackathon-submission.md) |

**不做**：登录与权限、多租户、通用页面生成、任意 CSS、远程 bundle、灰度发布、Figma 插件、组件市场。仅用虚构商城首页数据。

## 1. 三条铁律

1. 先跑通再美化：RN 能渲染内置 JSON，再从 API 读取，最后才加编辑器和 AI。
2. 先竖向闭环：画布改草稿 → 后端校验保存 → 发布版本 → RN 拉取显示。
3. AI 只提议，人来确认：AI 接口只返回候选 JSON 和差异，不保存、不发布。

协议字段由 C 维护；改字段先改协议文档和示例，再通知全组。每个文件只设一位主责人；`.env`、数据库、密钥不进 Git。主分支保持可启动。

## 2. 技术选型

| 层 | 选型 | 原因 |
|---|---|---|
| 运营画布 | 原生 HTML/CSS/JS | 免构建，拖动一个模块列表即可 |
| 客户端 | React Native + Expo + TypeScript | 符合 RN 优先，组件由 App 内置 |
| 后端 | Python 3.11+ + FastAPI | 少量代码起 API，带 /docs |
| 数据库 | SQLite | 一个文件，免数据库服务 |
| 协议 | JSON + JSON Schema | Web、API、RN 共用同一份字段定义 |
| AI | OpenAI 兼容 Chat Completions API | Key 只放后端环境变量 |
| 部署 | 一个可公网访问的演示服务 | 不引入容器编排或微服务 |

m3e-canvas 仅作画布交互参考，本 MVP 不把它加入运行时依赖。先做上移/下移按钮，排序闭环完成后再加拖放。

## 3. 架构与两条路径

```text
运营画布 ──草稿/发布──> FastAPI + SQLite ──发布 JSON/版本──> React Native
   └──自然语言──> /api/v1/ai/transform ──服务端──> 模型服务
                        └──候选配置 + diff 返回画布，人工确认后才保存
```

**拖拽**：读草稿 → 调整顺序 → 预览 → 保存草稿 → 发布（版本号+1）→ RN 手动刷新读取 published。

**AI**：输入指令和当前草稿 → 后端请求模型 → 白名单操作应用到副本并校验 → 返回候选和 diff → 人工应用到草稿 → 保存、发布。取消候选不得改变草稿。

**故障兜底**：API 不通时 RN 显示内置默认页/上次配置；AI 超时或格式错误时草稿不变，改用拖拽。模型服务不可用时可展示预置候选，但对外材料要标明演示模式。

## 4. 先冻结数据契约

唯一协议文件：`protocol/page.schema.json`，C 维护；统一示例为 `protocol/home.demo.json`。

```json
{
  "schemaVersion": "1.0.0",
  "pageKey": "home",
  "title": "商城首页",
  "tokens": { "color": { "background": "#FFF8F0", "primary": "#C45C26" } },
  "root": {
    "id": "root", "type": "Column", "props": { "gap": 12 },
    "children": [
      { "id": "banner", "type": "Banner", "props": { "title": "今日特惠" }, "children": [] },
      { "id": "products", "type": "ProductList", "props": { "title": "热门商品", "items": ["咖啡", "面包"] }, "children": [] },
      { "id": "action", "type": "Button", "props": { "title": "立即选购" }, "children": [] }
    ]
  }
}
```

首版只允许 `Column / Text / Banner / ProductList / Button`；id 唯一；根为 Column；最多 20 个节点、5 层；颜色为 #RRGGBB；组件属性逐项白名单。禁止脚本、表达式、任意 CSS、HTML 和任意 URL 动作。

AI 只返回受限操作，例如 `move_component`、`set_token`、`set_text`，不得返回可执行代码或直接发布。

## 5. 仓库目录与文件归属

```text
protocol/page.schema.json       # C：唯一协议
protocol/home.demo.json         # C：演示数据
backend/app.py                  # B：FastAPI 路由
backend/storage.py              # B：SQLite 草稿/发布
backend/validate.py             # C：白名单和 Schema 校验
backend/ai_transform.py         # B：模型调用、候选操作
editor/index.html               # D：画布结构
editor/app.js                   # D：预览、排序、AI 确认、保存发布
editor/style.css                # D：样式
client/App.tsx                  # A：RN 页面和刷新
client/renderer.tsx             # A：白名单渲染器
client/components.tsx           # A：演示组件
data/                           # 自动生成，加入 .gitignore
docs/                           # 项目文档
```

| 成员 | 主责 | 第一个交付 |
|---|---|---|
| A：前端/RN | `client/` | 10/24 前 Expo 打开内置 JSON 页面 |
| B：后端 | `backend/app.py`、`storage.py` | 10/24 前 /health 和 /docs 可用 |
| C：协议/验收 | `protocol/`、`validate.py`、验收清单 | 冻结字段并验证合法/非法配置 |
| D：运营画布/交付 | `editor/`、演示材料 | 静态画布、排序、AI 确认 UI |

按真实技能可在第一天互换；协议维护人和后端主责人必须唯一。

## 6. 第一天启动命令

先克隆项目并建自己的分支：

```bash
git clone https://github.com/ZoeyyHz/SDUI.git
cd SDUI
git checkout -b feat/your-name-task
```

**B 后端**（在 backend 目录）：

```bash
python -m venv .venv
# macOS/Linux: source .venv/bin/activate
# Windows PowerShell: .venv\\Scripts\\Activate.ps1
pip install fastapi uvicorn
uvicorn app:app --reload --port 8000
```

浏览器访问 `http://127.0.0.1:8000/health`，看到 ok 即第一步通过，`/docs` 应可打开。

**D 画布**（editor 目录，先不安装 npm 包）：

```bash
python -m http.server 5500
```

访问 `http://127.0.0.1:5500`。

**A RN**（先安装 Node LTS、手机 Expo Go；在 client 目录执行）：

```bash
npx create-expo-app@latest . --template blank-typescript
npx expo start
```

手机与电脑连同一 Wi-Fi 扫码。手机访问本机 API 时，不能用 localhost，要用电脑局域网 IP。A 先运行成功后，其他人不要重复生成脚手架。

## 7. API 契约

基础路径 `/api/v1`，画布演示免登录；错误统一含 `code/message/details`。

| 方法 | 路径 | 用途 |
|---|---|---|
| GET | `/health` | 自检 |
| GET | `/api/v1/draft/home` | 读取草稿 |
| PUT | `/api/v1/draft/home` | 校验并保存草稿 |
| POST | `/api/v1/publish/home` | 复制草稿为发布快照、版本号加一 |
| GET | `/api/v1/published/home` | RN 读取发布配置和版本 |
| POST | `/api/v1/ai/transform` | 返回候选配置和 diff，不保存 |
| POST | `/api/v1/demo/reset` | 重置示例页 |

B 先用固定候选模拟 AI，确保端到端 UI 能联调，再接模型。每次改接口同步更新 `docs/05-backend/api-contract.md`。

## 8. 排期与每周验收

| 日期 | 目标 | 验收 |
|---|---|---|
| 10/23 | 分工、协议初版、A/B/D 各自骨架启动 | Expo 静态页、/health、浏览器画布 |
| 10/24 | RN 选型判断、协议冻结 | 全员使用同一个 demo JSON |
| 10/25–28 | RN 白名单渲染、草稿/发布 API、画布接 API | 改标题、保存发布后 RN 刷新见变化 |
| 10/29–11/01 | 拖拽/排序闭环和首次合流 | 从另一台电脑完整演示发布 |
| 11/02–11/04 | AI 候选、diff、人工确认；可公网访问 | 取消不改草稿，确认发布后 RN 更新 |
| 11/05–11/06 | 缺陷修复、功能冻结、计划书初稿 | 新设备照 README 可启动 |
| 11/07–11/09 | 视频、计划书定稿、试用检查、发邮件 | 三件套和提交信息都复核完成 |
| 11/10 | 缓冲 | 仅处理提交意外 |

每天同步“已完成 / 下一步 / 阻塞”；每周至少两次联调。每次都让非作者在另一台电脑从零启动。

## 9. 四人执行卡

**A（RN）**：①空 Expo 显示 SDUI Preview；②用 `switch(node.type)` 渲染五种组件；③请求 published API、显示版本号和刷新按钮。验收：后端关闭时显示默认页，发布后刷新见新配置。

**B（后端）**：①/health、/docs；②先返回固定 JSON；③SQLite 分开存 draft/published；④AI 代理只返回候选。验收：重启后数据仍在，AI 失败不覆盖 published。卡住时先用 JSON 文件存储、预置候选。

**C（协议/质量）**：①准备唯一 demo JSON；②列出组件属性白名单；③验收未知组件、重复 id、非法颜色、额外脚本字段会被拒绝；④负责联调记录，不独立改数据库核心。

**D（画布/交付）**：①静态画布和预览；②先做上移/下移，再换拖放；③AI 输入、diff、应用/丢弃；④准备截图、视频脚本和商业计划书。验收：丢弃不改草稿，排序立刻反映到预览。

## 10. 风险与降级

1. 只保留 1 个页面、5 种组件、3 个模块。
2. 拖放卡住就用上移/下移按钮；保持可演示排序。
3. 模型 API 不通就用预置候选；诚实标注演示模式，人工确认和发布链仍真实运行。
4. Expo 到 10/24 仍无法启动时再评估 SwiftUI；切换后冻结另一套客户端。
5. 最终必须展示拖拽真实配置闭环和 AI 候选→人工确认→发布闭环。

| 问题 | 处理 |
|---|---|
| RN 读不到接口 | 手机和电脑同 Wi-Fi；API 地址用电脑 LAN IP，不用 localhost |
| 浏览器 CORS | 只配置明确的开发来源 |
| RN 显示旧页面 | 检查是否发布、版本是否递增，再手动刷新 |
| AI 返回非 JSON | 后端解析并做白名单校验；失败走预置候选 |
| 草稿影响线上预览 | draft 和 published 分开存；发布复制快照 |
| 未知组件让客户端崩溃 | 渲染前白名单过滤，显示占位卡片 |
| Key 泄露 | Key 仅后端 .env；禁止 EXPO_PUBLIC_ 前缀；检查 Git diff |

## 11. 演示视频脚本（3 分钟）

- 0:00–0:25：说明小改动过去也要改代码、打包、测试。
- 0:25–1:20：画布调整模块顺序、预览、保存发布；RN 刷新后变化。
- 1:20–2:25：输入自然语言；展示候选差异，丢弃一次，再确认、发布；RN 刷新。
- 2:25–2:50：说明五种本地组件白名单、模型不执行代码。
- 2:50–3:00：总结运营可以快速完成微小需求，不必修改客户端代码。

先重置 demo，关闭通知，完整录两遍选稳定的一次。

## 12. 提交前验收清单

**拖拽**：初始页有三个模块；重排后预览立即变化；保存后草稿保留；发布版本号递增；RN 刷新看到发布版；未发布草稿不会出现在 RN。

**AI**：支持改顺序、标题、主题色；先返回候选和 diff；丢弃不改变草稿；应用后可保存发布；失败时有预置候选且不伪称真实模型调用。

**交付**：新成员按 README 启动；演示作品可访问；仓库无密钥和数据库；商业计划书、作品、视频齐备；邮件收件人和标题格式复核。

---

*状态：可执行基线。协议、接口或范围变更时先更新对应文档并通知全员。目标是在截止日前完成两个可验证闭环，而不是做通用低代码平台。*
