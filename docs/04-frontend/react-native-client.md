# React Native 客户端

技术：React Native + Expo + TypeScript。

启动时先绘制内置默认页，再请求 GET /published/home。校验通过后渲染并缓存文档/版本。提供手动刷新；失败保留默认页/上次成功配置。

Registry 固定映射 Column、Text、Banner、ProductList、Button。动作由宿主 dispatcher 执行。禁止远程动态模块、eval 或任意 JS 执行。Expo Go 早期联调；RN 若 10/24 前仍无法跑通，备选 iOS SwiftUI，本次不做 Android 原生客户端。