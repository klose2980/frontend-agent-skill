# Skill: dom-input-fill 表单输入框填充

## 基础信息

| 字段 | 内容 |
| ---- | ---- |
| skill_id | dom-input-fill |
| display_name | 向页面输入框填入文本内容 |
| version | 1.0.0 |
| author | YourName |
| category | frontend-browser,dom,form |
| description | 通过CSS选择器定位 input / textarea 输入控件，模拟人工填入内容；自动触发输入事件，适配Vue/React受控组件。仅浏览器环境可用。 |
| tags | browser,input,textarea,表单填充,dom,受控组件 |
| permission | 普通页面DOM访问权限，无需额外授权 |

## 能力说明

1. 支持 `<input>`、`<textarea>` 元素填充
2. 主动触发 `input`、`change` 事件，适配前端框架受控表单
3. 可选先清空原有内容再填入
4. 支持等待元素加载
5. 区分只读、禁用输入框并返回提示

## 入参定义（JSON Schema）

```json
{
  "type": "object",
  "required": ["selector", "text"],
  "properties": {
    "selector": {
      "type": "string",
      "description": "输入框CSS选择器，例如 #username, .search-input"
    },
    "text": {
      "type": "string",
      "description": "需要填入输入框的文本内容"
    },
    "waitMs": {
      "type": "number",
      "description": "等待元素出现最大毫秒数，0=不等待",
      "default": 0
    },
    "clearFirst": {
      "type": "boolean",
      "description": "是否先清空输入框原有内容",
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
    "selector": string,
    "finalValue": string
  }
}
```

状态码说明
- 200：填充成功
- 400：选择器语法错误
- 404：超时未找到输入元素
- 403：非浏览器环境
- 405：找到元素，但不是输入框/textarea
- 406：输入框被禁用或只读，无法写入

## 前端可执行JS代码

```js
async function run(params) {
  const { selector, text, waitMs = 0, clearFirst = true } = params;
  try {
    if (typeof window === 'undefined') {
      return {
        success: false,
        code: 403,
        msg: "非浏览器环境，无法操作DOM",
        data: null
      }
    }

    let el = null;
    const start = Date.now();
    while (Date.now() - start < waitMs) {
      el = document.querySelector(selector);
      if (el) break;
      await new Promise(r => setTimeout(r, 100));
    }

    if (!el) {
      return {
        success: false,
        code: 404,
        msg: `等待${waitMs}ms未找到元素：${selector}`,
        data: { selector, finalValue: "" }
      }
    }

    const tagName = el.tagName.toLowerCase();
    if (tagName !== "input" && tagName !== "textarea") {
      return {
        success: false,
        code: 405,
        msg: "匹配到的元素不是输入框或文本域",
        data: { selector, finalValue: "" }
      }
    }

    if (el.disabled || el.readOnly) {
      return {
        success: false,
        code: 406,
        msg: "输入框已禁用或只读，无法填充",
        data: { selector, finalValue: el.value }
      }
    }

    if (clearFirst) el.value = "";
    el.value = text;

    // 触发框架监听事件（适配React/Vue受控组件）
    el.dispatchEvent(new Event('input', { bubbles: true }));
    el.dispatchEvent(new Event('change', { bubbles: true }));

    return {
      success: true,
      code: 200,
      msg: "文本填充完成",
      data: { selector, finalValue: el.value }
    }
  } catch (err) {
    return {
      success: false,
      code: 500,
      msg: `填充异常：${err.message}`,
      data: { selector, finalValue: "" }
    }
  }
}
```

## AI调用约束规则

1. 用户要求填写搜索框、表单输入内容时调用
2. **禁止自动填充密码、敏感隐私信息**
3. 高危表单（支付、注册）填充前需要确认用户
4. 无法操作iframe内的输入控件

## 使用示例

### 调用入参示例

```json
{
  "selector": "#search-input",
  "text": "前端Agent Skill开发",
  "waitMs": 1500,
  "clearFirst": true
}
```

### 成功返回示例

```json
{
  "success": true,
  "code": 200,
  "msg": "文本填充完成",
  "data": {
    "selector": "#search-input",
    "finalValue": "前端Agent Skill开发"
  }
}
```
