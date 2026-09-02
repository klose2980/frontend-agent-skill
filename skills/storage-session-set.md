# Skill: storage-session-set 写入SessionStorage

## 基础信息

| 字段 | 内容 |
| ---- | ---- |
| skill_id | storage-session-set |
| display_name | 浏览器SessionStorage写入数据 |
| version | 1.0.0 |
| author | YourName |
| category | frontend-browser,storage |
| description | 在当前页面浏览器环境写入 sessionStorage 会话级数据，支持字符串、对象自动序列化；数据仅当前会话有效，关闭标签页后自动清除，无法跨域读写。仅浏览器可用。 |
| tags | browser,sessionStorage,会话存储,临时缓存,前端存储 |
| permission | 页面默认存储权限，无需额外授权 |

## 能力说明

1. 接收 key 和 value，自动对对象/数组进行 JSON 序列化
2. 支持写入文本、数字、对象、数组等常规数据类型
3. 同名key直接覆盖原有会话存储内容
4. 捕获存储容量溢出异常并返回错误信息
5. 会话级生命周期，页面关闭自动清理，适合存临时状态
6. 不支持存储 File、Blob、函数等特殊类型

## 入参定义（JSON Schema）

```json
{
  "type": "object",
  "required": ["key", "value"],
  "properties": {
    "key": {
      "type": "string",
      "description": "sessionStorage存储键名"
    },
    "value": {
      "type": ["string", "number", "boolean", "object", "array"],
      "description": "需要存储的数据，对象和数组会自动转为JSON字符串保存"
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
    "key": string,
    "storedValue": string
  }
}
```

状态码说明

- 200：写入成功
- 400：数据无法序列化（存在循环引用等）
- 403：非浏览器环境，sessionStorage不存在
- 507：存储空间已满，无法写入

## 前端可执行JS代码

```javascript
async function run(params) {
  const { key, value } = params;
  try {
    if (typeof window === 'undefined' || !window.sessionStorage) {
      return {
        success: false,
        code: 403,
        msg: "当前环境不支持 sessionStorage，仅浏览器环境可用",
        data: null
      }
    }
    let storeStr;
    if (typeof value === 'object' && value !== null) {
      storeStr = JSON.stringify(value);
    } else {
      storeStr = String(value);
    }
    sessionStorage.setItem(key, storeStr);
    return {
      success: true,
      code: 200,
      msg: "会话存储写入成功",
      data: {
        key,
        storedValue: storeStr
      }
    }
  } catch (err) {
    if (err.name === 'QuotaExceededError') {
      return {
        success: false,
        code: 507,
        msg: "SessionStorage存储空间已满，写入失败",
        data: null
      }
    }
    return {
      success: false,
      code: 400,
      msg: `写入失败: ${err.message}`,
      data: null
    }
  }
}
```

## AI调用约束规则

1. 用户主动要求保存临时状态、会话级配置、单次访问记录时调用
2. **禁止存储敏感隐私数据**：身份证、手机号、账号密码、授权Token
3. 不要一次性写入超大体积数据，避免占满会话存储
4. 持久化数据优先使用 localStorage，临时状态使用 sessionStorage
5. 服务端、Node.js环境禁止调用该技能

## 使用示例

### 调用入参示例

```json
{
  "key": "temp_form_data",
  "value": {
    "step": 2,
    "draftContent": "临时编辑内容",
    "expireFlag": true
  }
}
```

### 成功返回示例

```json
{
  "success": true,
  "code": 200,
  "msg": "会话存储写入成功",
  "data": {
    "key": "temp_form_data",
    "storedValue": "{\"step\":2,\"draftContent\":\"临时编辑内容\",\"expireFlag\":true}"
  }
}
```
