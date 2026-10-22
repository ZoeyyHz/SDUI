# API 契约

基础路径 /api/v1；演示环境免登录。

| 方法 | 路径 | 用途 |
|---|---|---|
| GET | /health | 健康状态 |
| GET | /schema | JSON Schema |
| GET | /published/{pageKey} | 发布文档及版本 |
| GET | /draft/{pageKey} | 当前草稿 |
| PUT | /draft/{pageKey} | 校验并保存草稿 |
| POST | /ai/transform | 返回候选配置/差异，不保存 |
| POST | /publish/{pageKey} | 发布草稿快照 |
| POST | /demo/reset | 恢复示例配置 |

错误 JSON 含 code、message、details；Schema 错误返回 422 和字段路径。AI transform 仅返回 candidateDocument、changes，不能直接保存或发布。