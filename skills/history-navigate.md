# Skill: history-navigate 浏览器历史导航与页面刷新

## 基础信息

| 字段 | 内容 |
| ---- | ---- |
| skill_id | history-navigate |
| display_name | 浏览器历史导航与页面刷新 |
| version | 1.0.0 |
| author | YourName |
| category | frontend-browser,navigation |
| description | 控制浏览器历史记录执行后退、前进操作，或刷新当前页面；支持强制刷新绕过本地缓存。仅浏览器环境可用，后退/前进操作会触发页面跳转。 |
| tags | browser,history,后退,前进,刷新,页面导航 |
| permission | 无需额外浏览器权限，原生历史记录API |

## 能力说明

1. 支持三种导航模式：历史后退、历史前进、刷新当前页面
2. 刷新模式支持普通刷新与强制刷新（绕过缓存，等价于 Ctrl+F5）
3. 纯原生浏览器 API 实现，无任何第三方依赖
4. 自动校验运行环境，非浏览器场景直接返回错误
5. 由于浏览器安全限制，无法读取历史栈深度，仅保证执行对应 API 调用

## 入参定义（JSON Schema）

```json
{
  "type": "object",
  "required": ["mode"],
  "properties": {
    "mode": {
      "type": "string",
      "enum": ["back", "forward", "refresh"],
      "description": "back：后退一页；forward：前进一页；refresh：刷新当前页面"
    },
    "force": {
      "type": "boolean",
      "description": "仅refresh模式生效，true=强制刷新绕过缓存，false=普通刷新",
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
    "mode": string,
    "force": boolean
  }
}
```

状态码说明

- 200：导航指令已执行（后退/前进/刷新指令已发出）
- 400：mode 参数非法
- 403：非浏览器环境，无法执行导航操作

> 备注：后退、前进、刷新操作会导致页面重载或跳转，当前页面脚本可能被卸载，存在返回结果无法被上层接收的可能。

## 前端可执行JS代码

```javascript
async function run(params) {
  const { mode, force = false } = params;

  try {
    if (typeof window === "undefined" || typeof history === "undefined") {
      return {
        success: false,
        code: 403,
        msg: "非浏览器环境，无法执行历史导航操作",
        data: null
      };
    }

    if (!["back", "forward", "refresh"].includes(mode)) {
      return {
        success: false,
        code: 400,
        msg: "mode 仅支持 back / forward / refresh",
        data: null
      };
    }

    switch (mode) {
      case "back":
        history.back();
        break;
      case "forward":
        history.forward();
        break;
      case "refresh":
        location.reload(force);
        break;
    }

    return {
      success: true,
      code: 200,
      msg: `已执行${mode}操作，页面即将跳转/刷新`,
      data: { mode, force }
    };
  } catch (err) {
    return {
      success: false,
      code: 500,
      msg: `导航执行异常：${err.message}`,
      data: null
    };
  }
}
```

## AI调用约束规则

1. 用户明确要求返回上一页、前进一页、刷新页面时调用
2. 表单填写、内容编辑类页面，执行刷新/后退前必须向用户确认，避免未保存内容丢失
3. 禁止高频循环调用刷新操作，避免页面反复重载影响使用
4. 后退/前进/刷新均会触发页面生命周期重置，执行前需告知用户页面会跳转
5. 禁止用于绕过页面离开确认弹窗的强制跳转

## 使用示例

### 示例1：后退一页

```json
{
  "mode": "back"
}
```

### 示例2：强制刷新页面

```json
{
  "mode": "refresh",
  "force": true
}
```

### 成功返回示例

```json
{
  "success": true,
  "code": 200,
  "msg": "已执行refresh操作，页面即将跳转/刷新",
  "data": {
    "mode": "refresh",
    "force": true
  }
}
```
