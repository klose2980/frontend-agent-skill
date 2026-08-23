# Skill: storage-local-get 读取LocalStorage数据

## 基础信息

| 字段 | 内容 |
| ---- | ---- |
| skill_id | storage-local-get |
| display_name | 读取浏览器LocalStorage存储数据 |
| version | 1.0.0 |
| author | YourName |
| category | frontend-browser,storage |
| description | 读取当前页面浏览器 localStorage 中指定键名的数据，支持自动解析 JSON 字符串还原对象/数组；仅浏览器可用，无法跨域读取存储内容。 |
| tags | browser,localStorage,本地存储,读取缓存,前端数据 |
| permission | 页面默认存储权限，无需额外授权 |

## 能力说明

1. 支持单个 key 读取与批量 keys 读取两种模式
2. 自动尝试 JSON 反序列化，对象/数组自动还原，解析失败则返回原始字符串
3. 键名不存在时返回 null 并附带明确提示
4. 支持读取当前域名下所有存储键名列表
5. 统一返回键值对结构，便于上层业务处理

## 入参定义（JSON Schema）

```json
{
  "type": "object",
  "properties": {
    "key": {
      "type": "string",
      "description": "单个存储键名，与 keys 参数二选一传入"
    },
    "keys": {
      "type": "array",
      "items": {"type": "string"},
      "description": "批量读取的键名数组，与 key 参数二选一传入"
    },
    "autoParse": {
      "type": "boolean",
      "description": "是否自动解析 JSON 字符串为对象/数组",
      "default": true
    },
    "getAllKeys": {
      "type": "boolean",
      "description": "是否返回当前所有存储键名列表，优先级高于 key/keys",
      "default": false
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
    "values": Record<string, any>,
    "allKeys": string[] | null
  }
}
```

状态码说明

- 200：读取成功（键不存在也返回 200，对应 value 为 null）
- 400：参数错误，未传入 key / keys / getAllKeys 任一参数
- 403：非浏览器环境，无法访问 localStorage

## 前端可执行JS代码

```
async function run(params) {
  const { key, keys, autoParse = true, getAllKeys = false } = params;

  try {
    if (typeof window === 'undefined' || !window.localStorage) {
      return {
        success: false,
        code: 403,
        msg: "当前环境不支持 localStorage，仅浏览器环境可用",
        data: null
      }
    }

    const values = {};
    let allKeys = null;

    if (getAllKeys) {
      allKeys = Object.keys(localStorage);
    }

    // 收集待读取的键名
    let targetKeys = [];
    if (keys && Array.isArray(keys)) {
      targetKeys = keys;
    } else if (key && typeof key === 'string') {
      targetKeys = [key];
    } else if (!getAllKeys) {
      return {
        success: false,
        code: 400,
        msg: "请传入 key、keys 或开启 getAllKeys 参数",
        data: null
      };
    }

    // 读取并解析
    targetKeys.forEach(k => {
      const raw = localStorage.getItem(k);
      if (raw === null) {
        values[k] = null;
        return;
      }
      if (autoParse) {
        try {
          values[k] = JSON.parse(raw);
        } catch {
          values[k] = raw;
        }
      } else {
        values[k] = raw;
      }
    });

    return {
      success: true,
      code: 200,
      msg: "本地存储读取完成",
      data: { values, allKeys }
    }
  } catch (err) {
    return {
      success: false,
      code: 500,
      msg: `读取失败: ${err.message}`,
      data: null
    }
  }
}
```

## AI调用约束规则

1. 用户主动要求查询本地配置、读取缓存数据时调用
2. **禁止主动读取隐私敏感字段**：如用户token、账号密码、身份证信息等
3. 批量读取前先确认范围，禁止无差别遍历导出全部存储数据
4. 读取到敏感信息时不得输出、转发、存储，仅当场按需处理
5. 服务端、Node.js 环境禁止调用该技能

## 使用示例

### 示例1：读取单个键

```
{
  "key": "user_config",
  "autoParse": true
}
```

### 示例2：批量读取多个键

```
{
  "keys": ["theme", "language", "lastVisitTime"],
  "autoParse": true
}
```

### 成功返回示例

```
{
  "success": true,
  "code": 200,
  "msg": "本地存储读取完成",
  "data": {
    "values": {
      "user_config": {
        "theme": "dark",
        "autoRefresh": true
      }
    },
    "allKeys": null
  }
}
```
