# Skill: clipboard-copy 剪贴板文本复制

## 基础信息

| 字段 | 内容 |
| ---- | ---- |
| skill_id | clipboard-copy |
| display_name | 浏览器复制文本到剪贴板 |
| version | 1.0.0 |
| author | YourName |
| category | frontend-browser |
| description | 前端环境调用浏览器Clipboard API，将指定文本复制到用户剪贴板；仅浏览器可用，Node.js不可用 |
| tags | browser,clipboard,copy,dom,frontend |
| permission | 需要用户页面剪贴板权限，HTTPS/localhost环境生效 |

## 能力说明
1. 接收任意字符串文本，执行页面复制
2. 复制成功/失败返回明确状态文案
3. 超长文本支持完整复制
4. 可自定义复制成功弹窗提示文案

## 入参定义（JSON Schema，AI严格遵循）

```json
{
  "type": "object",
  "required": ["text"],
  "properties": {
    "text": {
      "type": "string",
      "description": "需要复制到剪贴板的正文内容，支持换行、符号、中文、代码"
    },
    "tipText": {
      "type": "string",
      "description": "可选，复制成功后页面弹窗提示文字，不传默认「复制成功」"
    }
  }
}
