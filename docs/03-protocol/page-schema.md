# 页面 JSON 协议

状态：草案。唯一字段定义需同步到 `protocol/sdui.schema.json`。

```json
{
  "schemaVersion": "1.0.0",
  "pageKey": "home",
  "title": "商城首页",
  "tokens": {"color": {"bg": "#FFF8F0", "accent": "#C45C26"}},
  "actions": {"showSale": {"type": "toast", "payload": {"message": "进入会场"}}},
  "root": {
    "id": "root", "type": "Column", "props": {"gap": 16},
    "children": [
      {"id": "hero", "type": "Banner", "props": {"title": "今日特惠", "actionId": "showSale"}, "children": []},
      {"id": "products", "type": "ProductList", "props": {"items": []}, "children": []}
    ]
  }
}
```

每个 id 唯一；根节点为 Column；类型/属性/动作均限白名单。配置不带 JS、HTML、表达式或任意 CSS。草稿版本与已发布版本分开。