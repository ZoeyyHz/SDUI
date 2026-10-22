# 本地开发与启动

前置：Python 3.11+、Node LTS/npm、Expo Go 或 iOS Simulator、兼容模型 API Key。

1. backend：建 venv、安装 requirements，配置模型服务环境变量，运行 uvicorn app:app --reload --port 8000。
2. editor：启动静态服务器，设置 API base URL。
3. client：安装依赖，配置 EXPO_PUBLIC_API_BASE_URL，运行 npx expo start 并用 Expo Go 扫码。
4. 确认 health、默认页、保存发布与客户端刷新链路。

真机访问本机服务时用电脑局域网 IP，不使用 localhost。