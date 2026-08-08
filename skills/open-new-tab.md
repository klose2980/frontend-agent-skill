# Skill: open-new-tab 打开新标签页

## 基础信息

| 字段 | 内容 |
| ---- | ---- |
| skill_id | open-new-tab |
| display_name | 浏览器打开新标签页 |
| version | 1.0.0 |
| author | YourName |
| category | frontend-browser,navigation |
| description | 在当前浏览器打开新标签页访问指定链接；支持控制是否前台激活标签。受浏览器弹窗策略限制，必须由用户主动行为触发才能正常打开。 |
| tags | browser,新标签页,跳转,window.open,页面导航 |
| permission | 受浏览器弹窗拦截策略限制，AI无法静默自动打开，需要绑定用户交互触发 |

## 能力说明

1. 支持打开 http/https 网页链接
2. 可配置新标签是否立即激活（聚焦）
3. 自动校验链接合法性，非法URL直接返回错误
4. 捕获浏览器弹窗拦截异常并提示用户
5. 不支持本地文件协议、跨域受限特殊协议

## 入参定义（JSON Schema）

```json
{
  "type": "object",
  "required": ["url"],
  "properties": {
    "url": {
      "type": "string",
      "description": "目标网页地址，完整http/https链接"
    },
    "active": {
      "type": "boolean",
      "description": "新标签是否自动激活并前置显示，false则后台打开",
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
    "url": string
  }
}
```

状态码说明
- 200：成功打开新标签
- 400：URL格式不合法
- 403：浏览器拦截弹窗，无法打开新标签
- 405：非浏览器环境，无法执行

## 前端可执行JS代码

```javascript
async function run(params) {
  const { url, active = true } = params;
  try {
    if (typeof window === 'undefined') {
      return {
        success: false,
        code: 405,
        msg: "当前环境不是浏览器，无法打开标签页",
        data: null
      }
    }
    // 校验URL
    const targetUrl = new URL(url);
    if (!["http:", "https:"].includes(targetUrl.protocol)) {
      return {
        success: false,
        code: 400,
        msg: "仅支持 http / https 协议链接",
        data: null
      }
    }
    const newWindow = window.open(url, "_blank");
    if (!newWindow || newWindow.closed) {
      return {
        success: false,
        code: 403,
        msg: "浏览器弹窗拦截，无法打开新标签，请手动允许弹窗权限",
        data: { url }
      }
    }
    if (active) {
      newWindow.focus();
    }
    return {
      success: true,
      code: 200,
      msg: "新标签页已打开",
      data: { url }
    }
  } catch (err) {
    if (err instanceof TypeError) {
      return {
        success: false,
        code: 400,
        msg: "URL格式错误，请传入合法链接",
        data: null
      }
    }
    return {
      success: false,
      code: 500,
      msg: `打开标签失败：${err.message}`,
      data: null
    }
  }
}
```

## AI调用约束规则

1. 用户明确要求打开链接、访问网页时调用
2. **禁止主动静默打开未知外链**，需要确认用户意愿
3. 提醒：浏览器弹窗拦截会导致技能失效，尽量引导用户点击触发
4. 禁止生成、打开钓鱼、违规网站链接

## 使用示例

### 调用入参示例

```json
{
  "url": "https://github.com",
  "active": true
}
```

### 成功返回示例

```json
{
  "success": true,
  "code": 200,
  "msg": "新标签页已打开",
  "data": {
    "url": "https://github.com"
  }
}
```
