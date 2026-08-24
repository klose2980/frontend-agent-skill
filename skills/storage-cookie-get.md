# Skill: storage-cookie-get 读取浏览器 Cookie

## 基础信息

| 字段 | 内容 |
| ---- | ---- |
| skill_id | storage-cookie-get |
| display_name | 读取当前页面Cookie数据 |
| version | 1.0.0 |
| author | YourName |
| category | frontend-browser,storage |
| description | 读取当前域名下的浏览器 Cookie 数据，支持单条读取与批量读取，自动解析键值对；仅浏览器环境可用，受同源策略限制无法读取跨域 Cookie。 |
| tags | browser,cookie,本地存储,会话存储,前端数据 |
| permission | 页面默认 Cookie 访问权限，无法读取 HttpOnly 标记的 Cookie |

## 能力说明

1. 支持读取指定名称的单条 Cookie
2. 支持读取当前域名下所有可访问 Cookie
3. 自动解码 URI 编码的 Cookie 值
4. 键不存在时返回 null 并附带明确提示
5. 自动过滤 HttpOnly Cookie（浏览器安全限制，前端不可读）

## 入参定义（JSON Schema）

```json
{
  "type": "object",
  "properties": {
    "name": {
      "type": "string",
      "description": "要读取的 Cookie 名称，不传则返回所有可读取的 Cookie"
    },
    "decode": {
      "type": "boolean",
      "description": "是否自动对 Cookie 值进行 decodeURIComponent 解码",
      "default": true
    }
  }
}
```

## 出参结构

```
{
  "success": boolean,
  "code": number,
  "msg": string,
  "data": {
    "cookies": Record<string, string>,
    "count": number
  }
}
```

状态码说明

- 200：读取成功（Cookie 不存在也返回 200，对应 value 为 null）
- 403：非浏览器环境，无法访问 Cookie

## 前端可执行JS代码

```
async function run(params) {
  const { name, decode = true } = params;

  try {
    if (typeof document === "undefined" || typeof document.cookie !== "string") {
      return {
        success: false,
        code: 403,
        msg: "非浏览器环境，无法读取 Cookie",
        data: null
      };
    }

    const cookieStr = document.cookie;
    const cookies = {};

    if (cookieStr) {
      cookieStr.split(";").forEach(item => {
        const [key, ...valueParts] = item.trim().split("=");
        if (!key) return;
        let value = valueParts.join("=");
        if (decode) {
          try {
            value = decodeURIComponent(value);
          } catch {
            // 解码失败保留原值
          }
        }
        cookies[key.trim()] = value;
      });
    }

    const result = {};
    if (name) {
      result[name] = cookies[name] ?? null;
    } else {
      Object.assign(result, cookies);
    }

    return {
      success: true,
      code: 200,
      msg: "Cookie 读取完成",
      data: {
        cookies: result,
        count: Object.keys(result).filter(k => result[k] !== null).length
      }
    };
  } catch (err) {
    return {
      success: false,
      code: 500,
      msg: `读取 Cookie 异常：${err.message}`,
      data: null
    };
  }
}
```

## AI调用约束规则

1. 用户主动要求查看 Cookie、校验会话状态时调用
2. **禁止主动读取并输出身份凭证、Token、会话 ID 等敏感 Cookie**
3. 禁止批量导出、转发用户 Cookie 数据，避免账号安全风险
4. 仅可读取当前域名下的 Cookie，无法跨域获取
5. HttpOnly 标记的 Cookie 前端不可读取，需向用户说明

## 使用示例

### 示例1：读取指定 Cookie

```
{
  "name": "theme",
  "decode": true
}
```

### 示例2：读取所有可访问 Cookie

```
{
  "decode": true
}
```

### 成功返回示例

```
{
  "success": true,
  "code": 200,
  "msg": "Cookie 读取完成",
  "data": {
    "cookies": {
      "theme": "dark",
      "lang": "zh-CN"
    },
    "count": 2
  }
}
```
