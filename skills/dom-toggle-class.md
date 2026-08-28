# Skill: dom-toggle-class 切换 DOM 元素 CSS 类名

## 基础信息

| 字段 | 内容 |
| ---- | ---- |
| skill_id | dom-toggle-class |
| display_name | 切换页面 DOM 元素 CSS 类名 |
| version | 1.0.0 |
| author | YourName |
| category | frontend-browser, dom |
| description | 通过 CSS 选择器定位页面元素，对指定元素执行添加、移除、切换CSS类名操作；可用于控制显示隐藏、主题切换、状态切换、展开收起等场景。仅浏览器环境可用。 |
| tags | browser,dom,classList,样式切换,显示隐藏,主题切换 |
| permission | 普通页面DOM访问权限，无需额外浏览器授权 |

## 能力说明

1. 支持三种操作模式：添加类、移除类、切换类（存在则移除，不存在则添加）
2. 支持单个元素或批量匹配元素操作
3. 支持同时操作多个类名，空格分隔一次性传入
4. 返回操作后元素的最终类名列表，便于校验结果
5. 自动兼容 classList API，操作无残留，不影响原有其他样式
6. 元素不存在时返回清晰错误，不抛出异常阻塞流程

## 入参定义（JSON Schema）

```json
{
  "type": "object",
  "required": ["selector", "mode", "className"],
  "properties": {
    "selector": {
      "type": "string",
      "description": "CSS选择器，例如 .panel、#menu、.theme-dark"
    },
    "mode": {
      "type": "string",
      "enum": ["add", "remove", "toggle"],
      "description": "操作模式：add添加类 / remove移除类 / toggle切换类"
    },
    "className": {
      "type": "string",
      "description": "目标CSS类名，多个类名用空格分隔，例如 'active show'"
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
    "finalClassList": string[]
  }
}
```

状态码说明

- 200：类名操作执行成功
- 400：参数错误或选择器语法非法
- 404：未找到匹配的 DOM 元素
- 403：非浏览器环境，无法操作 DOM

## 前端可执行 JS 代码

```javascript
async function run(params) {
  const { selector, mode, className, operateAll = false } = params;

  try {
    if (typeof window === "undefined" || typeof document === "undefined") {
      return {
        success: false,
        code: 403,
        msg: "非浏览器环境，无法操作DOM类名",
        data: null
      };
    }

    if (!["add", "remove", "toggle"].includes(mode)) {
      return {
        success: false,
        code: 400,
        msg: "mode 仅支持 add / remove / toggle",
        data: null
      };
    }

    if (!className || !className.trim()) {
      return {
        success: false,
        code: 400,
        msg: "className 不能为空",
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
        data: { selector, operateCount: 0, finalClassList: [] }
      };
    }

    const classList = className.trim().split(/\s+/).filter(Boolean);

    elements.forEach(el => {
      classList.forEach(cls => {
        switch (mode) {
          case "add":
            el.classList.add(cls);
            break;
          case "remove":
            el.classList.remove(cls);
            break;
          case "toggle":
            el.classList.toggle(cls);
            break;
        }
      });
    });

    const finalClassList = Array.from(elements[0].classList);

    return {
      success: true,
      code: 200,
      msg: `已对 ${elements.length} 个元素执行${mode}类名操作`,
      data: {
        selector,
        operateCount: elements.length,
        finalClassList
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
      msg: `类名操作异常：${err.message}`,
      data: null
    };
  }
}
```

## AI 调用约束规则

1. 用户要求切换显示隐藏、展开收起、切换主题、切换状态时调用
2. 禁止批量移除页面核心布局类名，避免页面结构崩溃
3. 涉及页面关键区域样式变更前，需确认用户意图再执行
4. 禁止高频循环切换类名，避免页面反复重绘造成卡顿
5. 操作后可结合DOM查询技能校验最终样式状态

## 使用示例

### 调用入参示例

```json
{
  "selector": ".sidebar",
  "mode": "toggle",
  "className": "collapsed",
  "operateAll": false
}
```

### 成功返回示例

```json
{
  "success": true,
  "code": 200,
  "msg": "已对 1 个元素执行 toggle 类名操作",
  "data": {
    "selector": ".sidebar",
    "operateCount": 1,
    "finalClassList": ["sidebar", "width-200", "collapsed"]
  }
}
```
