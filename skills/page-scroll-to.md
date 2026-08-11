# Skill: page-scroll-to 页面滚动控制

## 基础信息

| 字段 | 内容 |
| ---- | ---- |
| skill_id | page-scroll-to |
| display_name | 浏览器页面滚动控制 |
| version | 1.0.0 |
| author | YourName |
| category | frontend-browser,dom,scroll |
| description | 控制当前页面滚动，支持按坐标滚动、滚动到指定DOM元素；支持平滑滚动/瞬间跳转。仅浏览器环境可用。 |
| tags | browser,scroll,页面滚动,滚动到元素,平滑滚动 |
| permission | 无需额外权限，常规页面DOM能力 |

## 能力说明

1. 两种模式：按坐标滚动 / 根据CSS选择器滚动至目标元素
2. 支持平滑滚动 auto / smooth
3. 可设置滚动偏移量（避开顶部导航栏遮挡）
4. 元素不存在时返回错误提示
5. 支持纵向、横向滚动

## 入参定义（JSON Schema）

```json
{
  "type": "object",
  "required": ["mode"],
  "properties": {
    "mode": {
      "type": "string",
      "enum": ["coordinate", "element"],
      "description": "coordinate：坐标滚动；element：滚动到元素"
    },
    "selector": {
      "type": "string",
      "description": "mode=element 生效，CSS选择器"
    },
    "x": {
      "type": "number",
      "description": "mode=coordinate 生效，横向滚动像素值",
      "default": 0
    },
    "y": {
      "type": "number",
      "description": "mode=coordinate 生效，纵向滚动像素值",
      "default": 0
    },
    "offsetY": {
      "type": "number",
      "description": "滚动纵向偏移，例如 -80 避开顶部固定导航",
      "default": 0
    },
    "behavior": {
      "type": "string",
      "enum": ["smooth", "auto"],
      "description": "smooth平滑滚动，auto瞬间跳转",
      "default": "smooth"
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
    "scrollX": number,
    "scrollY": number
  }
}
```

状态码说明

- 200：滚动指令执行成功
- 400：参数配置错误（mode与参数不匹配）
- 404：选择器未找到对应DOM元素
- 403：非浏览器环境，无法执行滚动

## 前端可执行JS代码

```js
async function run(params) {
  const { mode, selector, x = 0, y = 0, offsetY = 0, behavior = "smooth" } = params;

  try {
    if (typeof window === "undefined") {
      return {
        success: false,
        code: 403,
        msg: "非浏览器环境，无法执行页面滚动",
        data: null
      };
    }

    if (mode === "element") {
      if (!selector) {
        return {
          success: false,
          code: 400,
          msg: "mode=element 必须传入 selector",
          data: null
        };
      }
      const el = document.querySelector(selector);
      if (!el) {
        return {
          success: false,
          code: 404,
          msg: `未找到元素：${selector}`,
          data: null
        };
      }
      const rect = el.getBoundingClientRect();
      const targetY = window.scrollY + rect.top + offsetY;
      window.scrollTo({
        top: targetY,
        left: window.scrollX,
        behavior
      });
    } else if (mode === "coordinate") {
      window.scrollTo({
        top: y,
        left: x,
        behavior
      });
    } else {
      return {
        success: false,
        code: 400,
        msg: "mode仅支持 coordinate / element",
        data: null
      };
    }

    return {
      success: true,
      code: 200,
      msg: "滚动指令已执行",
      data: {
        scrollX: window.scrollX,
        scrollY: window.scrollY
      }
    };
  } catch (err) {
    return {
      success: false,
      code: 500,
      msg: `滚动异常: ${err.message}`,
      data: null
    };
  }
}
```

## AI调用约束规则

1. 用户要求滑动页面、滚动到某处、查看下方内容时调用
2. 尽量优先使用 element 模式，不要固定写死坐标
3. 长页面查询内容时，可滚动后再调用dom查询技能
4. 禁止高频循环持续滚动，避免页面卡顿

## 使用示例

### 示例1：滚动到元素

```json
{
  "mode": "element",
  "selector": ".article-content",
  "offsetY": -60,
  "behavior": "smooth"
}
```

### 示例2：滚动到指定坐标

```json
{
  "mode": "coordinate",
  "y": 1200,
  "behavior": "smooth"
}
```

### 成功返回示例

```json
{
  "success": true,
  "code": 200,
  "msg": "滚动指令已执行",
  "data": {
    "scrollX": 0,
    "scrollY": 1200
  }
}
```
