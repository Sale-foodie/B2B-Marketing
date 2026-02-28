# API 与第三方集成

## 概述

LeadKits MVP 阶段集成以下外部 API：

| API | 用途 | 调用方 |
|-----|------|--------|
| 天眼查搜索 API | 按条件搜索企业 | n8n |
| 天眼查详情 API | 获取企业详细信息 | n8n |
| 天眼查联系方式 API | 获取企业高管联系方式 | n8n |
| Bing 搜索 API | 搜索公开的联系人信息 | n8n |
| LLM API | AI Agent 的大模型调用 | Dify |

## 天眼查 API

### 1. 搜索 API

**用途**：根据关键词和筛选条件搜索企业列表

**调用场景**：公司搜索工作流的第一步

```
GET /services/open/search/2.0
```

**请求参数**：

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| word | string | 是 | 搜索关键词 |
| pageNum | int | 否 | 页码，默认 1 |
| pageSize | int | 否 | 每页条数，默认 20 |

**返回字段**：
- 公司名称
- 公司 ID
- 法定代表人
- 注册资本
- 成立日期
- 经营状态
- 匹配度分数

**调用频率限制**：需根据天眼查 API 套餐确定

### 2. 详情 API

**用途**：获取单个企业的完整工商信息

**调用场景**：搜索到公司后，逐一获取详情

```
GET /services/open/ic/baseinfo/normal
```

**请求参数**：

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| keyword | string | 是 | 公司名称或公司 ID |

**返回字段**：
- 完整工商注册信息
- 股东信息
- 主要人员
- 经营范围
- 地址信息
- 行业分类

### 3. 联系方式 API

**用途**：获取企业高管的联系方式

**调用场景**：Lead 搜索工作流

```
GET /services/open/ic/contact
```

**请求参数**：

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| keyword | string | 是 | 公司名称或公司 ID |

**返回字段**：
- 联系人姓名
- 职位
- 手机号码
- 邮箱

**合规注意**：
- 仅返回公开可获取的联系信息
- 需遵守天眼查数据使用协议

## Bing 搜索 API

### Web Search API

**用途**：搜索互联网上公开的联系人信息

**调用场景**：Lead 搜索工作流，补充天眼查未覆盖的信息

```
GET https://api.bing.microsoft.com/v7.0/search
```

**请求参数**：

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| q | string | 是 | 搜索查询 |
| count | int | 否 | 返回结果数 |
| mkt | string | 否 | 市场，如 zh-CN |

**搜索策略**：
```
"[公司名] [职位] 联系方式"
"[公司名] [人名] 邮箱"
"[公司名] CTO site:linkedin.com"
```

**API Key 管理**：
- 通过 Azure Cognitive Services 申请
- 免费层：每月 1000 次调用
- 付费层：按需扩展

## LLM API

### Dify 内置调用

Dify 平台内部调用 LLM，支持以下模型：

| 模型 | 用途 | 特点 |
|------|------|------|
| GPT-4o | ICP 构建、AI 分析 | 推理能力强，适合复杂分析 |
| GPT-4o-mini | ICP 评分 | 性价比高，适合批量评分 |
| Claude 3.5 Sonnet | 备选模型 | 长文本理解好 |

**配置方式**：在 Dify 平台设置中配置 API Key

## API 错误处理

### 重试策略

```mermaid
flowchart TD
    A[API 调用] --> B{成功？}
    B -->|是| C[处理返回数据]
    B -->|否| D{错误类型}
    D -->|限流 429| E[等待后重试]
    D -->|服务端错误 5xx| F[间隔 5s 重试]
    D -->|客户端错误 4xx| G[记录错误，跳过]
    D -->|网络超时| F
    E --> H{重试次数 < 3?}
    F --> H
    H -->|是| A
    H -->|否| I[标记失败，继续下一条]
```

### 限流管理

| API | 限流策略 |
|-----|----------|
| 天眼查 | 按套餐 QPS 限制，n8n 中设置请求间隔 |
| Bing | 免费层 3 QPS，n8n 中控制并发 |
| LLM | Dify 平台自带限流管理 |

## API Key 管理

- 所有 API Key 通过环境变量管理，不硬编码
- n8n 使用 Credential 管理器存储 API Key
- Dify 通过平台设置管理 LLM API Key

```env
# .env.example
TIANYANCHA_API_TOKEN=your_token_here
BING_SEARCH_API_KEY=your_key_here
DATABASE_URL=postgresql://user:pass@localhost:5432/leadkits
```
