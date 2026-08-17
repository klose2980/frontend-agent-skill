# Skill: download-file 浏览器端生成并下载文件

## 基础信息

| 字段 | 内容 |
| ---- | ---- |
| skill_id | download-file |
| display_name | 前端生成文件并触发下载 |
| version | 1.0.0 |
| author | YourName |
| category | frontend-browser,file |
| description | 在浏览器环境中纯前端生成文件并触发原生下载，支持文本、JSON、CSV、Markdown 等纯文本格式，可自定义文件名与后缀；无需后端接口，仅浏览器环境可用。 |
| tags | browser,download,文件导出,blob,前端下载 |
| permission | 无需额外浏览器权限，触发原生下载行为 |

## 能力说明

1. 支持文本、JSON、CSV、Markdown 等字符串内容生成文件
2. 可自定义完整文件名与后缀（如 `.json` `.txt` `.csv` `.md`）
3. 基于 Blob + ObjectURL 实现，兼容主流现代浏览器
4. 自动释放内存，避免页面内存泄漏
5. 下载失败返回清晰错误原因与参数提示

## 入参定义（JSON Schema）

```json
{
  "type": "object",
  "required": ["filename", "content"],
  "properties": {
    "filename": {
      "type": "string",
      "description": "下载保存的完整文件名，需包含后缀，例如 user-config.json"
    },
    "content": {
      "type": "string",
      "description": "文件的正文内容，纯文本格式，JSON/CSV 需提前序列化为字符串"
    },
    "fileType": {
      "type": "string",
      "description": "文件 MIME 类型，默认 text/plain；常用：application/json、text/csv、text/markdown",
      "default": "text/plain"
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
    "filename": string,
    "fileSize": number,
    "fileType": string
  }
}
```

状态码说明
- 200：已成功触发浏览器下载
- 400：参数不合法（文件名为空、内容为空）
- 403：非浏览器环境，无法执行下载
- 500：下载执行异常

## 前端可执行JS代码

```javascript
async function run(params) {
  const { filename, content, fileType = "text/plain" } = params;

  try {
    if (typeof window === "undefined" || typeof document === "undefined") {
      return {
        success: false,
        code: 403,
        msg: "非浏览器环境，无法触发文件下载",
        data: null
      };
    }

    if (!filename || !filename.trim()) {
      return {
        success: false,
        code: 400,
        msg: "文件名不能为空，且需包含后缀名",
        data: null
      };
    }

    const blob = new Blob([content], { type: fileType });
    const url = URL.createObjectURL(blob);

    const link = document.createElement("a");
    link.href = url;
    link.download = filename.trim();
    link.style.display = "none";
    document.body.appendChild(link);
    link.click();
    document.body.removeChild(link);

    // 释放内存
    setTimeout(() => URL.revokeObjectURL(url), 1000);

    return {
      success: true,
      code: 200,
      msg: "文件下载已触发，请查看浏览器下载栏",
      data: {
        filename: filename.trim(),
        fileSize: blob.size,
        fileType
      }
    };
  } catch (err) {
    return {
      success: false,
      code: 500,
      msg: `文件下载异常：${err.message}`,
      data: null
    };
  }
}
```

## AI调用约束规则

1. 用户明确要求导出、下载、保存为文件时调用
2. **禁止生成并下载违规、侵权、隐私敏感内容**
3. 内容体积较大时，先告知用户文件大小并确认后再执行下载
4. JSON/CSV 格式需确保内容已正确序列化，避免下载文件格式损坏
5. 禁止高频循环调用批量生成大量文件

## 使用示例

### 调用入参示例

```json
{
  "filename": "agent-config.json",
  "content": "{\"theme\":\"dark\",\"autoSave\":true,\"skills\":[\"dom-click\",\"toast\"]}",
  "fileType": "application/json"
}
```

### 成功返回示例

```json
{
  "success": true,
  "code": 200,
  "msg": "文件下载已触发，请查看浏览器下载栏",
  "data": {
    "filename": "agent-config.json",
    "fileSize": 72,
    "fileType": "application/json"
  }
}
```
