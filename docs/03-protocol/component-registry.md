# 组件 Registry

| 类型 | 用途 | MVP 属性 | 子节点 |
|---|---|---|---|
| Column | 垂直布局 | gap、padding、align | 是 |
| Text | 标题/正文 | text、variant | 否 |
| Banner | 活动运营位 | title、image、actionId | 否 |
| ProductList | 商品卡片 | items[] | 否 |
| Button | 业务按钮 | text、variant、actionId | 否 |

动作仅 toast 与站内 navigate。服务端拒绝未知类型/属性；客户端遇到未知类型时跳过节点并记录错误。