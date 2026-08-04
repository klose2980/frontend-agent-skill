# Skill: dom-query-element 页面DOM元素查询

## 基础信息

| 字段 | 内容 |
| ---- | ---- |
| skill_id | dom-query-element |
| display_name | 查询页面DOM元素信息 |
| version | 1.0.0 |
| author | YourName |
| category | frontend-browser,dom |
| description | 在当前网页中通过选择器查询DOM元素，支持获取元素文本、属性、宽高、位置信息；仅浏览器环境可用。支持 css selector，如 #id、.class、div[name]。 |
| tags | browser,dom,querySelector,页面元素,前端选择器 |
| permission | 页面正常DOM访问权限，无额外浏览器权限要求 |

## 能力说明

1. 支持标准CSS选择器查询单个/多个页面元素
2. 可配置需要获取的信息：元素内文本、属性列表、坐标、尺寸
3. 找不到元素时清晰返回空结果，不会抛出阻塞异常
4. 限制：无法跨iframe查询外部DOM

## 入参定义（JSON Schema）

```json
{
  "type": "object",
  "required": ["selector"],
  "properties": {
    "selector": {
      "type": "string",
      "description": "CSS选择器，例如 .search-btn、#header、div.container"
    },
    "queryAll": {
      "type": "boolean",
      "description": "true = querySelectorAll 查询多个；false = querySelector 查询单个元素",
      "default": false
    },
    "getRect": {
      "type": "boolean",
      "description": "是否获取元素位置与尺寸(left,top,width,height)",
      "default": false
    },
    "getAttrs": {
      "type": "array",
      "items": {"type":"string"},
      "description": "需要读取的属性列表，例如 ['href','src','data-id']，留空则不读取属性"
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
    "list": [
      {
        "innerText": string,
        "innerHTML": string,
        "attrs": Record<string,string>,
        "rect": {
          "left": number,
          "top": number,
          "width": number,
          "height": number
        } | null
      }
    ]
  }
}
```

状态码说明

- 200：查询成功（即使没有匹配元素也返回200，list为空数组）
- 400：选择器语法非法
- 403：非浏览器环境，DOM不可访问

## 前端可执行JS代码

```javascript
async function run(params) {
  const { selector, queryAll = false, getRect = false, getAttrs = [] } = params;
  try {
    let nodeList = [];
    if (queryAll) {
      nodeList = Array.from(document.querySelectorAll(selector));
    } else {
      const el = document.querySelector(selector);
      if (el) nodeList = [el];
    }

    const list = nodeList.map(el => {
      const item = {
        innerText: el.innerText,
        innerHTML: el.innerHTML,
        attrs: {},
        rect: null
      };
      if(getRect){
        const rect = el.getBoundingClientRect();
        item.rect = {
          left: rect.left,
          top: rect.top,
          width: rect.width,
          height: rect.height
        };
      }
      getAttrs.forEach(key=>{
        item.attrs[key] = el.getAttribute(key) ?? "";
      })
      return item;
    })

    return {
      success: true,
      code: 200,
      msg: "DOM查询完成",
      data: { list }
    }
  } catch(err){
    return {
      success: false,
      code: 400,
      msg: `查询失败: ${err.message}`,
      data: { list: [] }
    }
  }
}
```

## AI调用约束规则

1. 仅用户需要获取页面内容、查找页面按钮/文本信息时调用
2. 禁止用来批量爬取大量页面敏感信息
3. 复杂选择器优先简化，避免性能消耗
4. Node.js、服务端环境禁止调用

## 使用示例

### 调用入参示例

```json
{
  "selector": ".title",
  "queryAll": true,
  "getRect": true,
  "getAttrs": ["class"]
}
```

### 成功返回示例

```json
{
  "success": true,
  "code": 200,
  "msg": "DOM查询完成",
  "data": {
    "list": [
      {
        "innerText": "欢迎页面",
        "innerHTML": "<span>欢迎页面</span>",
        "attrs": {"class":"title"},
        "rect": {"left":20,"top":100,"width":200,"height":32}
      }
    ]
  }
}
```
