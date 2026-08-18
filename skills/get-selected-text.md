# Skill: get-selected-text 获取页面选中文本

## 基础信息

| 字段 | 内容 |
| ---- | ---- |
| skill_id | get-selected-text |
| display_name | 获取当前页面选中的文本内容 |
| version | 1.0.0 |
| author | YourName |
| category | frontend-browser,dom |
| description | 获取用户在当前网页中鼠标选中的纯文本内容，常用于选中后翻译、解释、搜索、复制等场景；仅浏览器环境可用，不支持跨 iframe 获取选中内容。 |
| tags | browser,selection,选中文本,文本获取,页面交互 |
| permission | 无需额外浏览器权限，原生页面能力 |

## 能力说明

1. 读取当前页面主文档中用户选中的所有文本内容
2. 支持自动去除首尾空白字符与多余换行
3. 未选中任何文本时返回空结果与明确提示
4. 自动过滤不可见元素、隐藏节点中的选中内容
5. 返回选中长度与是否存在选中文本的标识位

## 入参定义（JSON Schema）

```json
{
  "type": "object",
  "properties": {
    "trim": {
      "type": "boolean",
      "description": "是否自动去除选中内容首尾的空白字符与换行",
      "default": true
    }
  }
}
```

## 出参结构

```json
{
  "success": boolean,
  "code": number,
  "msg": string,
  "data": {
    "selectedText": string,
    "hasSelection": boolean,
    "length": number
  }
}
```

状态码说明

- 200：获取成功（未选中文本也返回 200，hasSelection 为 false）
- 403：非浏览器环境，无法获取选中内容

## 前端可执行JS代码

```javascript
async function run(params) {
  const { trim = true } = params;

  try {
    if (typeof window === "undefined" || typeof document === "undefined") {
      return {
        success: false,
        code: 403,
        msg: "非浏览器环境，无法获取页面选中文本",
        data: null
      };
    }

    const selection = window.getSelection();
    let text = selection ? selection.toString() : "";

    if (trim) {
      text = text.trim();
    }

    const hasSelection = text.length > 0;

    return {
      success: true,
      code: 200,
      msg: hasSelection ? "选中文本获取成功" : "当前页面未选中任何文本",
      data: {
        selectedText: text,
        hasSelection,
        length: text.length
      }
    };
  } catch (err) {
    return {
      success: false,
      code: 500,
      msg: `获取选中文本异常：${err.message}`,
      data: null
    };
  }
}
```

## AI调用约束规则

1. 用户明确要求「读取选中的文字」「翻译选中内容」「解释这段文字」时调用
2. 禁止静默、循环、高频获取用户选中内容，避免侵犯浏览隐私
3. 未选中文本时，需提示用户先用鼠标选中目标内容再执行操作
4. 禁止用于获取密码输入框、隐私输入框中的选中内容
5. 选中内容涉及隐私信息时，不得存储、转发，仅当场处理

## 使用示例

### 调用入参示例

```json
{
  "trim": true
}
```

### 选中内容时返回示例

```json
{
  "success": true,
  "code": 200,
  "msg": "选中文本获取成功",
  "data": {
    "selectedText": "前端 Agent Skill 开发规范",
    "hasSelection": true,
    "length": 15
  }
}
```

### 未选中文本时返回示例

```json
{
  "success": true,
  "code": 200,
  "msg": "当前页面未选中任何文本",
  "data": {
    "selectedText": "",
    "hasSelection": false,
    "length": 0
  }
}
```
