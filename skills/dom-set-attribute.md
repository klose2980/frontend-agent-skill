# Skill: dom-set-attribute 设置DOM元素属性
## 基础信息
| 字段 | 内容 |
| ---- | ---- |
| skill_id | dom-set-attribute |
| display_name | 设置或移除DOM元素HTML属性 |
| version | 1.0.0 |
| author | YourName |
| category | frontend-browser,dom |
| description | 通过CSS选择器定位页面元素，设置、修改或移除元素的HTML属性，支持标准属性、自定义data-*属性、布尔状态属性；仅浏览器环境可用。 |
| tags | browser,dom,setAttribute,元素属性,状态修改 |
| permission | 普通页面DOM访问权限，无需额外浏览器授权 |

## 能力说明
1. 支持标准CSS选择器定位单个或批量匹配元素
2. 支持设置任意合法HTML属性，包含自定义 data-* 系列属性
3. 支持移除指定属性（属性值传 null 即执行移除）
4. 自动适配布尔属性：disabled、checked、readonly、hidden 等
5. 返回操作前后的属性值，便于校验执行结果
6. 元素不存在时返回清晰错误，不抛出阻塞性异常

## 入参定义（JSON Schema）
```json
{
  "type": "object",
  "required": ["selector", "attrName"],
  "properties": {
    "selector": {
      "type": "string",
      "description": "CSS选择器，例如 #submit-btn、.input-field、img.avatar"
    },
    "attrName": {
      "type": "string",
      "description": "目标属性名称，例如 src、href、disabled、placeholder、data-id"
    },
    "attrValue": {
      "type": ["string", "boolean", "number", "null"],
      "description": "属性值；传 null 则代表移除该属性"
    },
    "operateAll": {
      "type": "boolean",
      "description": "true=操作所有匹配元素；false=仅操作第一个匹配元素",
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
    "operateCount": number,
    "attrName": string,
    "oldValue": string | null,
    "newValue": string | null
  }
}
```
状态码说明
- 200：属性设置/移除成功
- 400：参数错误或属性名非法
- 404：未找到匹配的DOM元素
- 403：非浏览器环境，无法操作DOM

## 前端可执行JS代码
```javascript
async function run(params) {
  const { selector, attrName, attrValue, operateAll = false } = params;

  try {
    if (typeof window === "undefined" || typeof document === "undefined") {
      return {
        success: false,
        code: 403,
        msg: "非浏览器环境，无法操作DOM属性",
        data: null
      };
    }

    if (!attrName || !attrName.trim()) {
      return {
        success: false,
        code: 400,
        msg: "属性名 attrName 不能为空",
        data: null
      };
    }

    let elements = [];
    if (operateAll) {
      elements = Array.from(document.querySelectorAll(selector));
    } else {
      const el = document.querySelector(selector);
      if (el) elements = [el];
    }

    if (elements.length === 0) {
      return {
        success: false,
        code: 404,
        msg: `未找到匹配元素：${selector}`,
        data: { selector, operateCount: 0, attrName, oldValue: null, newValue: null }
      };
    }

    const oldValue = elements[0].getAttribute(attrName);
    let newValue = null;

    // 布尔属性特殊处理
    const boolAttrs = ["disabled", "checked", "readonly", "hidden", "required", "open"];
    const isBoolAttr = boolAttrs.includes(attrName.toLowerCase());

    elements.forEach(el => {
      if (attrValue === null) {
        el.removeAttribute(attrName);
        newValue = null;
      } else if (isBoolAttr) {
        if (attrValue) {
          el.setAttribute(attrName, attrName);
          el[attrName] = true;
          newValue = attrName;
        } else {
          el.removeAttribute(attrName);
          el[attrName] = false;
          newValue = null;
        }
      } else {
        const val = String(attrValue);
        el.setAttribute(attrName, val);
        newValue = val;
      }
    });

    return {
      success: true,
      code: 200,
      msg: `已对 ${elements.length} 个元素执行属性操作`,
      data: {
        selector,
        operateCount: elements.length,
        attrName,
        oldValue,
        newValue
      }
    };
  } catch (err) {
    if (err instanceof SyntaxError) {
      return {
        success: false,
        code: 400,
        msg: `CSS选择器语法非法：${err.message}`,
        data: null
      };
    }
    return {
      success: false,
      code: 500,
      msg: `属性操作异常：${err.message}`,
      data: null
    };
  }
}
```

## AI调用约束规则
1. 用户要求修改元素状态、调整属性值、切换禁用状态时调用
2. 禁止修改表单提交地址、跳转链接、安全校验等关键属性诱导用户
3. 禁止移除 integrity、crossorigin、nonce 等安全相关属性
4. 涉及修改页面核心功能的属性前，需确认用户意图再执行
5. 批量修改属性前先确认范围，避免大面积影响页面正常功能
6. 禁止通过修改属性窃取数据、篡改页面展示内容误导用户

## 使用示例
### 示例1：禁用提交按钮
```json
{
  "selector": "#submit-btn",
  "attrName": "disabled",
  "attrValue": true,
  "operateAll": false
}
```

### 示例2：移除元素的自定义属性
```json
{
  "selector": ".card-item",
  "attrName": "data-temp-mark",
  "attrValue": null,
  "operateAll": true
}
```

### 成功返回示例
```json
{
  "success": true,
  "code": 200,
  "msg": "已对 1 个元素执行属性操作",
  "data": {
    "selector": "#submit-btn",
    "operateCount": 1,
    "attrName": "disabled",
    "oldValue": null,
    "newValue": "disabled"
  }
}
```
