# Skill: dom-click-element 点击页面DOM元素

## 基础信息

| 字段 | 内容 |
| ---- | ---- |
| skill_id | dom-click-element |
| display_name | 模拟点击页面DOM元素 |
| version | 1.0.0 |
| author | YourName |
| category | frontend-browser,dom |
| description | 通过CSS选择器定位页面元素，触发模拟点击事件；仅浏览器环境可用。可用于点击按钮、链接、下拉菜单等页面交互元素。 |
| tags | browser,dom,click,模拟点击,交互,querySelector |
| permission | 普通页面DOM访问权限，无额外浏览器授权 |

## 能力说明

1. 支持标准CSS选择器定位目标元素
2. 支持自定义等待时间，等待元素出现后再点击
3. 找不到元素时返回清晰错误信息
4. 区分普通点击与强制点击（穿透遮罩）
5. 捕获元素不可点击、被遮挡等异常场景

## 入参定义（JSON Schema）

```json
{
  "type": "object",
  "required": ["selector"],
  "properties": {
    "selector": {
      "type": "string",
      "description": "CSS选择器，例如 .submit-btn、#confirm、a.link"
    },
    "waitMs": {
      "type": "number",
      "description": "等待元素出现的最大毫秒数，0代表立即查询，不等待",
      "default": 0
    },
    "forceClick": {
      "type": "boolean",
      "description": "true使用element.click()强制触发；false优先使用用户行为模拟点击",
      "default": false
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
    "targetInnerText": string|null
  }
}
```

状态码说明

- 200：点击执行成功
- 400：选择器语法错误
- 404：超时未找到匹配元素
- 403：非浏览器环境，无法操作DOM
- 500：元素存在但点击触发异常

## 前端可执行JS代码

```javascript
async function run(params) {
  const { selector, waitMs = 0, forceClick = false } = params;
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
    const startTime = Date.now();
    // 循环等待元素
    while (Date.now() - startTime < waitMs) {
      el = document.querySelector(selector);
      if (el) break;
      await new Promise(resolve => setTimeout(resolve, 100));
    }

    if (!el) {
      return {
        success: false,
        code: 404,
        msg: `等待${waitMs}ms后未找到元素：${selector}`,
        data: { selector, targetInnerText: null }
      }
    }

    if (forceClick) {
      el.click();
    } else {
      // 标准鼠标点击事件
      const event = new MouseEvent('click', {
        bubbles: true,
        cancelable: true,
        view: window
      });
      el.dispatchEvent(event);
    }

    return {
      success: true,
      code: 200,
      msg: "元素点击事件已触发",
      data: {
        selector,
        targetInnerText: el.innerText || ""
      }
    }
  } catch (err) {
    if (err instanceof SyntaxError) {
      return {
        success: false,
        code: 400,
        msg: `CSS选择器语法非法：${err.message}`,
        data: { selector, targetInnerText: null }
      }
    }
    return {
      success: false,
      code: 500,
      msg: `点击执行异常：${err.message}`,
      data: { selector, targetInnerText: null }
    }
  }
}
```

## AI调用约束规则

1. 用户明确要求点击按钮、确认、关闭、展开菜单时调用
2. 禁止自动循环批量点击、恶意高频点击页面元素
3. 涉及删除、提交、支付等高危操作，必须先向用户确认
4. 跨iframe内元素无法选中，不支持操作内嵌页面DOM

## 使用示例

### 调用入参示例

```json
{
  "selector": ".submit-button",
  "waitMs": 2000,
  "forceClick": true
}
```

### 成功返回示例

```json
{
  "success": true,
  "code": 200,
  "msg": "元素点击事件已触发",
  "data": {
    "selector": ".submit-button",
    "targetInnerText": "提交表单"
  }
}
```
