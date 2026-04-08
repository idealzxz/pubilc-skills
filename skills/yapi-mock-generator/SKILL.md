---
name: yapi-mock-generator
description: 从 YApi (docs.dui88.com) 接口文档自动生成 mock 数据和 API 定义。当用户提供 YApi 链接、要求生成 mock、创建 mock 文件、更新 mock 接口、或提到 YApi 接口文档时使用。
---

# YApi Mock 数据生成器

从 YApi 接口文档平台读取接口定义，自动生成 mock 数据到 `mock/` 目录，并同步更新 `src/api/index.js` 中的 API 定义。

## 触发场景

| 用户指令 | 动作 |
|---------|------|
| 提供 YApi URL + token | 首次生成：拉取接口 → 生成 mock 文件 → 更新 API 定义 |
| "更新 mock 接口" | 重新拉取：用已有 token 重新请求 YApi → 覆盖 mock 文件 → 同步 API 定义 |

## 输入参数

用户需提供：
- **YApi URL**：分类页或接口页地址
- **Token**：YApi 项目 token，用于 API 认证

首次提供后应记住 token，后续"更新 mock"时复用。

## 工作流程

### Step 1: 解析 YApi URL

从 URL 中提取 `host`、`projectId` 和 `catId` / `apiId`：

| URL 类型 | 格式 |
|---------|------|
| 分类页 | `{host}/project/{projectId}/interface/api/cat_{catId}` |
| 接口页 | `{host}/project/{projectId}/interface/api/{apiId}` |

### Step 2: 通过 Shell curl 获取接口数据

**必须使用 Shell 工具执行 curl**（WebFetch 无法访问内网），所有请求需带 token 参数。

获取分类下接口列表：
```bash
curl -s --connect-timeout 15 "{host}/api/interface/list_cat?catid={catId}&limit=100&page=1&token={token}"
```

获取单个接口详情（对列表中每个 `_id` 执行）：
```bash
curl -s --connect-timeout 15 "{host}/api/interface/get?id={apiId}&token={token}"
```

可并行请求多个接口详情以提高效率。

### Step 3: 解析接口定义

从接口详情中提取：

| 字段 | 说明 |
|------|------|
| `method` | 请求方法（GET/POST） |
| `path` | 接口路径（如 `/main/index.do`） |
| `title` | 接口名称（用作注释） |
| `desc` | 接口描述 |
| `res_body` | 响应体 JSON Schema（字符串，需 `JSON.parse`） |

### Step 4: 生成 mock 数据

根据 `res_body` 的 JSON Schema 生成 mock 数据：

| Schema 类型 | 生成规则 |
|------------|---------|
| `string` | 参考字段名和 `description` 生成合理文本 |
| `number` / `integer` | 参考字段名和 `description` 生成合理数字 |
| `boolean` | 参考 `description` 语义，默认 `true` |
| `array` | 生成 1-3 个元素的数组 |
| `object` | 递归生成各属性 |
| 有 `enum` | 取第一个枚举值 |
| 有 `default` | 使用默认值 |

> 如果 `res_body` 本身已包含 `success`、`data` 等外层字段，直接按 schema 生成完整响应，不要重复嵌套。

### Step 5: 写入 mock 文件

#### 文件位置与命名

在 `mock/project/` 目录下创建或更新文件：
- 文件名基于接口路径前缀或分类名，如 `/main/xxx.do` → `home.js`
- **更新场景**：直接覆盖对应 mock 文件的内容（保留文件结构，更新数据）

#### 文件格式

```javascript
module.exports = {
    'GET /main/index.do': {
        "success": true,
        "code": null,
        "message": null,
        "data": {
            // 根据 schema 生成的字段
        }
    },

    'POST /main/join.do': {
        "success": true,
        "code": null,
        "message": null,
        "data": {
            // 根据 schema 生成的字段
        }
    },
}
```

#### 自动注册

`mock/project/index.js` 会自动扫描并加载目录下所有 JS 文件，新建文件无需手动注册。

#### 加密接口

如果接口需要 AES 加密，在 `mock/index.js` 中手动处理：

```javascript
const { AESEncrypt } = require("./Crypto");

"GET /spring/start.do": {
    "success": true,
    "message": "message",
    "timeStamp": Date.now(),
    "data": AESEncrypt(JSON.stringify({ /* data */ }), "密钥Key", "密钥IV"),
},
```

### Step 6: 同步更新 API 定义

**此步骤为必须执行**，每次生成或更新 mock 时都要同步检查 `src/api/index.js`。

#### 命名规则

根据接口的 `method` 和 `path` 生成 API 名称：

| 接口 | API 名称 | 说明 |
|------|---------|------|
| `GET /main/index.do` | `getHomeInfo` | get + 语义名 |
| `POST /main/join.do` | `doJoin` | do + 动作名 |
| `POST /main/draw.do` | `doDraw` | do + 动作名 |

#### 格式

```javascript
const API = generateAPI({
    // ...已有接口...

    /** 接口标题（来自 YApi title 字段） */
    getXxx: 'path/to.do',

    /** 接口标题 */
    doXxx: {
        uri: 'path/to.do',
        method: 'post'
    },
})
```

#### 规则

- GET 请求：直接用字符串 `'path/to.do'`
- POST 请求：用对象 `{ uri: 'path/to.do', method: 'post' }`
- 路径**不带**前导 `/`（如 `main/index.do` 而非 `/main/index.do`）
- 注释使用 YApi 上的 `title` 字段
- **已存在的 API 不要重复添加**，但如果 path 变了需要更新
- 如果接口需要携带 token，添加 `withToken: true`

### Step 7: 验证

使用 Shell 工具执行验证：

```bash
node -e "const m = require('./mock/project'); console.log(Object.keys(m));"
```

确保所有路由都已正确注册。

## 更新 mock 流程

当用户说"更新 mock 接口"或类似指令时：

1. 使用已保存的 YApi URL 和 token 重新执行 Step 2 拉取最新数据
2. 对比接口列表，处理新增/删除/变更的接口
3. 重新生成 mock 数据（覆盖已有文件中对应路由的数据）
4. 同步更新 `src/api/index.js`（新增缺少的、移除已删除的）
5. 执行 Step 7 验证

## 注意事项

- mock 路由 key 的路径带前导 `/`（如 `'GET /main/index.do'`）
- `src/api/index.js` 中的 uri 路径**不带**前导 `/`（如 `'main/index.do'`）
- mock 文件使用 CommonJS（`module.exports`），API 文件使用 ES Module（`import/export`）
- 并行请求多个接口详情以提高效率
