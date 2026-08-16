# Skill: toast-notice 页面轻量提示弹窗

## 基础信息

| 字段 | 内容 |
| ---- | ---- |
| skill_id | toast-notice |
| display_name | 页面Toast轻量提示弹窗 |
| version | 1.0.0 |
| author | YourName |
| category | frontend-browser,ui |
| description | 在当前网页弹出原生轻量提示消息，无需依赖任何UI库；支持自定义文案、显示时长、弹出位置、状态样式，自动消失不阻塞页面操作。仅浏览器环境可用。 |
| tags | browser,toast,提示弹窗,消息提醒,原生UI |
| permission | 无需额外浏览器权限，纯页面DOM渲染 |

## 能力说明

1. 纯原生JavaScript实现，零第三方依赖
2. 支持顶部居中、页面中部、底部居中三种弹出位置
3. 提供普通、成功、错误、警告四种预设样式
4. 可自定义显示时长，到期自动销毁
5. 多次调用自动纵向堆叠，不会互相覆盖
6. 自动适配移动端视口，样式轻量化不遮挡主内容

## 入参定义（JSON Schema）

```json
{
  "type": "object",
  "required": ["text"],
  "properties": {
    "text": {
      "type": "string",
      "description": "提示弹窗显示的正文内容"
    },
    "duration": {
      "type": "number",
      "description": "显示时长，单位毫秒，最小500ms，默认3000ms",
      "default": 3000,
      "minimum": 500
    },
    "position": {
      "type": "string",
      "enum": ["top", "center", "bottom"],
      "description": "弹出位置：top顶部居中 / center页面中央 / bottom底部居中",
      "default": "top"
    },
    "type": {
      "type": "string",
      "enum": ["default", "success", "error", "warning"],
      "description": "提示样式类型",
      "default": "default"
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
    "toastId": string,
    "text": string,
    "duration": number
  }
}
```

状态码说明
- 200：Toast已成功渲染弹出
- 400：参数不合法（时长过小、枚举值错误）
- 403：非浏览器环境，无法渲染DOM

## 前端可执行JS代码

```javascript
async function run(params) {
  const { text, duration = 3000, position = "top", type = "default" } = params;

  try {
    if (typeof window === "undefined" || typeof document === "undefined") {
      return {
        success: false,
        code: 403,
        msg: "非浏览器环境，无法渲染Toast",
        data: null
      };
    }

    if (duration < 500) {
      return {
        success: false,
        code: 400,
        msg: "显示时长不能小于500ms",
        data: null
      };
    }

    const toastId = `toast-${Date.now()}-${Math.random().toString(36).slice(2, 6)}`;

    // 样式映射
    const typeStyle = {
      default: "background:#333;color:#fff",
      success: "background:#52c41a;color:#fff",
      error: "background:#ff4d4f;color:#fff",
      warning: "background:#faad14;color:#fff"
    };

    // 位置映射
    const positionStyle = {
      top: "top:24px;left:50%;transform:translateX(-50%);",
      center: "top:50%;left:50%;transform:translate(-50%, -50%);",
      bottom: "bottom:24px;left:50%;transform:translateX(-50%);"
    };

    const el = document.createElement("div");
    el.id = toastId;
    el.style.cssText = `
      position:fixed;z-index:99999;padding:10px 16px;border-radius:6px;
      font-size:14px;box-shadow:0 2px 8px rgba(0,0,0,0.15);
      ${typeStyle[type]};${positionStyle[position]};
      opacity:0;transition:opacity 0.2s ease;
      max-width:80vw;word-break:break-all;
    `;
    el.innerText = text;
    document.body.appendChild(el);

    // 淡入
    requestAnimationFrame(() => {
      el.style.opacity = "1";
    });

    // 定时销毁
    setTimeout(() => {
      el.style.opacity = "0";
      setTimeout(() => el.remove(), 200);
    }, duration);

    return {
      success: true,
      code: 200,
      msg: "Toast提示已弹出",
      data: { toastId, text, duration }
    };
  } catch (err) {
    return {
      success: false,
      code: 500,
      msg: `Toast渲染异常：${err.message}`,
      data: null
    };
  }
}
```

## AI调用约束规则
1. 用户操作后需要反馈结果、提醒通知时调用
2. 禁止高频连续弹出大量提示，避免干扰用户
3. 重要确认、高危操作提示优先使用确认弹窗，普通轻量提示使用本技能
4. 禁止用来弹出广告、无关骚扰信息
5. 错误、警告类提示建议搭配对应type，增强辨识度

## 使用示例

### 调用入参示例

```json
{
  "text": "配置已保存成功",
  "duration": 2500,
  "position": "top",
  "type": "success"
}
```

### 成功返回示例

```json
{
  "success": true,
  "code": 200,
  "msg": "Toast提示已弹出",
  "data": {
    "toastId": "toast-1723785600000-a1b2",
    "text": "配置已保存成功",
    "duration": 2500
  }
}
```
