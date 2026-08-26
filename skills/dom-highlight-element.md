# Skill: dom-highlight-element 高亮标记页面DOM元素

## 基础信息

| 字段 | 内容 |
| ---- | ---- |
| skill_id | dom-highlight-element |
| display_name | 高亮标记页面DOM元素 |
| version | 1.0.0 |
| author | YourName |
| category | frontend-browser,dom |
| description | 通过CSS选择器定位页面元素，添加高亮边框与背景效果，用于视觉标记目标元素位置；支持自定义颜色、持续时长，到期自动恢复元素原有样式。仅浏览器环境可用。 |
| tags | browser,dom,元素高亮,视觉标记,定位提示 |
| permission | 普通页面DOM访问权限，无需额外浏览器授权 |

## 能力说明

1. 支持标准CSS选择器定位单个或批量元素
2. 支持自定义高亮边框颜色、背景透明度
3. 可设置高亮持续时长，到期自动还原元素原有样式
4. 支持永久高亮模式，需手动调用恢复
5. 自动缓存元素原始样式，恢复无残留，不破坏页面布局
6. 高亮层级可控，不会遮挡页面可交互内容

## 入参定义（JSON Schema）

```json
{
  "type": "object",
  "required": ["selector"],
  "properties": {
    "selector": {
      "type": "string",
      "description": "CSS选择器，例如 .target-item、#result-box"
    },
    "highlightAll": {
      "type": "boolean",
      "description": "true=高亮所有匹配元素；false=仅高亮第一个匹配元素",
      "default": false
    },
    "borderColor": {
      "type": "string",
      "description": "高亮边框颜色，支持十六进制、rgb、颜色名",
      "default": "#ff4d4f"
    },
    "bgColor": {
      "type": "string",
      "description": "高亮背景颜色，建议带透明度",
      "default": "rgba(255, 77, 79, 0.15)"
    },
    "duration": {
      "type": "number",
      "description": "高亮持续时长，单位毫秒；0代表永久高亮，需手动恢复",
      "default": 2000
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
    "highlightCount": number,
    "duration": number
  }
}
```

状态码说明

- 200：高亮效果已添加成功
- 400：选择器语法错误或参数非法
- 404：未找到匹配的DOM元素
- 403：非浏览器环境，无法操作DOM

## 前端可执行JS代码

```javascript
async function run(params) {
  const { selector, highlightAll = false, borderColor = "#ff4d4f", bgColor = "rgba(255, 77, 79, 0.15)", duration = 2000 } = params;

  try {
    if (typeof window === "undefined" || typeof document === "undefined") {
      return {
        success: false,
        code: 403,
        msg: "非浏览器环境，无法操作DOM高亮",
        data: null
      };
    }

    let elements = [];
    if (highlightAll) {
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
        data: { selector, highlightCount: 0, duration }
      };
    }

    // 缓存原始样式并应用高亮
    elements.forEach(el => {
      const originalStyle = {
        border: el.style.border,
        backgroundColor: el.style.backgroundColor,
        boxShadow: el.style.boxShadow
      };
      el._originalHighlightStyle = originalStyle;

      el.style.border = `2px solid ${borderColor}`;
      el.style.backgroundColor = bgColor;
      el.style.boxShadow = `0 0 0 2px ${borderColor}40`;
      el.style.transition = "all 0.2s ease";
    });

    // 定时恢复
    if (duration > 0) {
      setTimeout(() => {
        elements.forEach(el => {
          if (el._originalHighlightStyle) {
            el.style.border = el._originalHighlightStyle.border;
            el.style.backgroundColor = el._originalHighlightStyle.backgroundColor;
            el.style.boxShadow = el._originalHighlightStyle.boxShadow;
            delete el._originalHighlightStyle;
          }
        });
      }, duration);
    }

    return {
      success: true,
      code: 200,
      msg: `已高亮 ${elements.length} 个元素`,
      data: {
        selector,
        highlightCount: elements.length,
        duration
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
      msg: `高亮执行异常：${err.message}`,
      data: null
    };
  }
}
```

## AI调用约束规则

1. 定位到目标元素后，需要向用户可视化展示位置时调用
2. 禁止批量高亮大量元素，避免页面混乱、影响正常浏览
3. 关键操作前可通过高亮确认目标元素，避免误点、误操作
4. 永久高亮模式避免滥用，防止页面样式残留
5. 优先使用柔和配色，避免高饱和颜色长时间停留造成视觉疲劳

## 使用示例

### 调用入参示例

```json
{
  "selector": ".search-result-item",
  "highlightAll": true,
  "borderColor": "#1677ff",
  "bgColor": "rgba(22, 119, 255, 0.12)",
  "duration": 2500
}
```

### 成功返回示例
```json
{
  "success": true,
  "code": 200,
  "msg": "已高亮 5 个元素",
  "data": {
    "selector": ".search-result-item",
    "highlightCount": 5,
    "duration": 2500
  }
}
```
