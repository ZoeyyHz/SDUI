# ADR 0001：RN 与 JSON 白名单渲染

状态：接受 · 日期：2026-10-22

背景：队伍有 Android 前端，但用户明确要求本项目优先 RN，其次 iOS，不做 Android 原生客户端。

决策：React Native + Expo + TypeScript，固定组件 Registry 渲染 JSON，不加载或执行远程代码。若 2026-10-24 前 RN 仍跑不通，备选 iOS SwiftUI。

后果：协议统一覆盖客户端；需要学习 React/TypeScript；配置更新不等于 JS Bundle 或原生模块热替换。