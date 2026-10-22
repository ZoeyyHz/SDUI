# MVP 范围与验收

## 做
免登录运营台、拖拽排序/属性编辑、预览、草稿与发布；AI Copilot 受限操作、变更预览与确认；FastAPI/SQLite；React Native + Expo 白名单渲染；示例 JSON、线上作品、视频与商业计划书。

## 组件与动作
组件：Column、Text、Banner、ProductList、Button。动作：toast、站内 navigate。

## 不做
登录/注册、权限、多人协作、支付、真实商品 API、自由代码生成、远程 JS/原生代码执行、Figma 同步、多页面 CMS。

## 验收
- 运营台无需注册/登录即可打开。
- 拖拽场景保存草稿，发布后 RN 刷新显示新顺序/文案。
- Copilot 使用真实模型；输出受限操作，先预览和人工确认，再保存/发布。
- 未知组件、非法属性/URL/AI 操作被拒绝；请求失败客户端回退到默认或上次成功页面。