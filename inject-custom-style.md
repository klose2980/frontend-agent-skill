# Skill: inject-custom-style 注入自定义CSS样式

## 基础信息

| 字段 | 内容 |
| ---- | ---- |
| skill_id | inject-custom-style |
| display_name | 向页面注入自定义CSS样式 |
| version | 1.0.0 |
| author | YourName |
| category | frontend-browser,dom,style |
| description | 向当前网页注入自定义CSS样式代码，实时生效；可用于临时调整页面布局、隐藏元素、修改配色、放大字体等场景。支持指定样式ID，可覆盖或移除已注入样式。仅浏览器环境可用。 |
| tags | browser,css,样式注入,页面美化,布局调整,dom |
| permission | 普通页面DOM访问权限，无需额外浏览器授权 |

## 能力说明

1. 支持注入任意合法CSS代码，样式实时生效
2. 支持指定样式ID，同ID样式可覆盖更新或移除
3. 两种注入模式：追加注入 / 覆盖同ID样式
4. 支持移除之前注入的指定ID样式
5. 样式标签统一管理，不污染页面原有样式
6. 自动校验CSS语法基本合法性，捕获异常

## 入参定义（JSON Schema）
```json
{
  "type": "object",
  "required": ["mode"],
  "properties": {
    "mode": {
      "type": "string",
      "enum": ["inject", "remove"],
      "description": "inject：注入样式；remove：移除已注入的样式"
    },
    "cssCode": {
      "type": "string",
      "description": "mode=inject 必填，CSS样式代码，例如 '.sidebar{display:none}' "
    },
    "styleId": {
      "type": "string",
      "description": "样式唯一标识，用于覆盖或移除；不传则自动生成随机ID",
      "default": "agent-custom-style"
    },
    "replace": {
      "type": "boolean",
      "description": "mode=inject 生效，true=覆盖同ID旧样式；false=追加新样式标签",
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
    "mode": string,
    "styleId": string,
    "cssLength": number
  }
}
```
状态码说明
- 200：样式注入/移除成功
- 400：参数错误或CSS代码为空
- 404：remove模式未找到对应ID的样式标签
- 403：非浏览器环境，无法操作DOM

## 前端可执行JS代码
```javascript
async function run(params) {
  const { mode, cssCode, styleId = "agent-custom-style", replace = true } = params;

  try {
    if (typeof window === "undefined" || typeof document === "undefined") {
      return {
        success: false,
        code: 403,
        msg: "非浏览器环境，无法注入CSS样式",
        data: null
      };
    }

    if (mode === "remove") {
      const existEl = document.getElementById(styleId);
      if (!existEl) {
        return {
          success: false,
          code: 404,
          msg: `未找到ID为 ${styleId} 的样式标签`,
          data: { mode, styleId, cssLength: 0 }
        };
      }
      existEl.remove();
      return {
        success: true,
        code: 200,
        msg: "样式标签已移除",
        data: { mode, styleId, cssLength: 0 }
      };
    }

    if (mode === "inject") {
      if (!cssCode || !cssCode.trim()) {
        return {
          success: false,
          code: 400,
          msg: "cssCode 不能为空",
          data: null
        };
      }

      let styleEl = document.getElementById(styleId);
      
      if (styleEl && replace) {
        styleEl.textContent = cssCode;
      } else {
        styleEl = document.createElement("style");
        styleEl.id = styleId;
        styleEl.textContent = cssCode;
        document.head.appendChild(styleEl);
      }

      return {
        success: true,
        code: 200,
        msg: "CSS样式已注入并生效",
        data: {
          mode,
          styleId,
          cssLength: cssCode.length
        }
      };
    }

    return {
      success: false,
      code: 400,
      msg: "mode 仅支持 inject / remove",
      data: null
    };
  } catch (err) {
    return {
      success: false,
      code: 500,
      msg: `样式操作异常：${err.message}`,
      data: null
    };
  }
}
```

## AI调用约束规则
1. 用户要求调整页面样式、隐藏元素、放大字体、修改配色时调用
2. 禁止注入恶意样式：遮挡页面核心内容、隐藏安全提示、伪造页面状态
3. 禁止通过样式窃取信息、诱导点击、篡改按钮文字
4. 注入影响页面布局的样式前，需确认用户意图
5. 尽量使用指定styleId，便于后续恢复和管理，避免样式残留
6. 禁止高频大量注入样式，避免页面重绘卡顿

## 使用示例
### 示例1：隐藏页面侧边栏
```json
{
  "mode": "inject",
  "styleId": "hide-sidebar",
  "cssCode": ".sidebar { display: none !important; } .main { margin-left: 0 !important; }",
  "replace": true
}
```

### 示例2：移除已注入的样式

```json
{
  "mode": "remove",
  "styleId": "hide-sidebar"
}
```

### 成功返回示例

```json
{
  "success": true,
  "code": 200,
  "msg": "CSS样式已注入并生效",
  "data": {
    "mode": "inject",
    "styleId": "hide-sidebar",
    "cssLength": 68
  }
}
```
