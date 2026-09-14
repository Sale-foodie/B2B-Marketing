# 工作流引擎设计

## 概述

LeadKits 的业务逻辑通过 **Dify + n8n** 双引擎协作实现：

- **Dify**：负责所有需要 LLM 参与的智能处理（ICP 构建、评分、分析）
- **n8n**：负责所有数据获取、处理、存储的自动化流程（API 调用、数据清洗、存储）

```mermaid
flowchart LR
    subgraph Dify["Dify (AI 智能层)"]
        A1[ICP Builder Agent]
        A2[Supervisor Agent]
        A3[ICP Scoring Agent]
        A4[AI Analysis Agent]
    end

    subgraph N8N["n8n (数据操作层)"]
        W1[公司搜索工作流]
        W2[Lead 搜索工作流]
        W3[数据存储工作流]
        W4[数据导出工作流]
    end

    A2 -->|下发搜索任务| W1
    W1 -->|搜索完成回调| A3
    A3 -->|评分完成，触发 Lead 搜索| W2
    W2 -->|Lead 数据就绪| A3
    A3 -->|高分线索| A4
    W1 & W2 --> W3
```

## Dify Agent 详细设计

### Agent 1：ICP Builder

| 配置项 | 说明 |
|--------|------|
| **类型** | 对话型 Chatflow |
| **触发方式** | 前端 Dify API 直接调用 |
| **输入** | 用户自然语言描述 |
| **输出** | 结构化 ICP JSON |
| **知识库** | ICP 最佳实践、行业采购结构链、行业分类、行业与产业关联 |
| **LLM** | GPT-4 / Claude |

**Chatflow 设计**：

```mermaid
flowchart TD
    Start[用户输入] --> Classify{输入分类}
    Classify -->|描述需求| Guide[引导提问流程]
    Classify -->|上传文件| Parse[解析文件内容]

    Guide --> Q1[提问：产品/服务是什么？]
    Q1 --> Q2[提问：目标行业和规模？]
    Q2 --> Q3[提问：目标决策人？]
    Q3 --> Q4[提问：购买信号？]
    Q4 --> KB[知识库检索：补充行业信息]
    KB --> Generate[生成结构化 ICP]

    Parse --> Validate[验证字段完整性]
    Validate --> Generate

    Generate --> Confirm[输出 ICP，请用户确认]
    Confirm -->|需要调整| Adjust[调整 ICP]
    Adjust --> Confirm
    Confirm -->|确认| Save[保存 ICP]
```

### Agent 2：Supervisor

| 配置项 | 说明 |
|--------|------|
| **类型** | Workflow（非对话） |
| **触发方式** | 前端 API 调用 |
| **输入** | ICP JSON |
| **输出** | 搜索任务列表（JSON） |
| **工具** | HTTP 请求（触发 n8n Webhook） |
| **LLM 角色** | 分析 ICP，拆分为可执行的搜索策略 |

**主要逻辑**：
1. 接收确认的 ICP
2. LLM 分析 ICP 属性，生成搜索策略（如何组合关键词、如何分批搜索）
3. 通过 HTTP Request 工具调用 n8n Webhook，传递搜索参数
4. 等待 n8n 回调，协调后续流程

### Agent 3：ICP Scoring

| 配置项 | 说明 |
|--------|------|
| **类型** | Workflow（非对话） |
| **触发方式** | n8n 回调触发 |
| **输入** | ICP 基准 + 公司/Lead 数据 |
| **输出** | 各维度得分 + 总分 + 等级 + 理由 |
| **LLM 角色** | 评估意向信号、模糊匹配判断 |
| **规则引擎** | 硬指标（行业/地区/规模）用 Code 节点计算 |

**评分流程**：

```mermaid
flowchart TD
    Input[接收公司/Lead 数据] --> Rule[规则评分：行业/地区/规模/融资]
    Rule --> LLM[LLM 评估：意向信号分析]
    LLM --> Calc[加权计算总分]
    Calc --> Grade{分级}
    Grade -->|85-100| A[A 级]
    Grade -->|70-84| B[B 级]
    Grade -->|50-69| C[C 级]
    Grade -->|0-49| D[D 级]
    A & B & C & D --> Output[输出评分结果]
```

### Agent 4：AI Analysis

| 配置项 | 说明 |
|--------|------|
| **类型** | Workflow（非对话） |
| **触发方式** | 评分完成后触发（仅 A/B 级） |
| **输入** | 公司详情 + Lead 信息 + 评分结果 |
| **输出** | 结构化分析报告 JSON |
| **知识库** | B2B 销售方法论、话术模版 |
| **LLM** | GPT-4 / Claude（需要强推理） |

## n8n 工作流详细设计

### 工作流 1：公司搜索

```mermaid
flowchart TD
    A[Webhook 接收搜索任务] --> B[解析搜索参数]
    B --> C[调用天眼查搜索 API]
    C --> D[解析搜索结果列表]
    D --> E{遍历每家公司}
    E --> F[调用天眼查详情 API]
    F --> G[数据清洗与标准化]
    G --> H[检测意向信号]
    H --> I[写入 PostgreSQL]
    I --> J{还有下一家？}
    J -->|是| E
    J -->|否| K[回调 Dify：搜索完成]
```

**关键配置**：
- Webhook 节点：接收 Dify Supervisor 的搜索请求
- HTTP Request 节点：调用天眼查 API
- Code 节点：数据清洗、信号检测
- Postgres 节点：数据写入
- 错误处理：API 限流重试、异常数据跳过

### 工作流 2：Lead 搜索

```mermaid
flowchart TD
    A[Webhook 接收 Lead 搜索任务] --> B[获取目标公司列表]
    B --> C{遍历每家公司}
    C --> D[天眼查联系方式 API]
    C --> E[Bing 搜索联系人]
    D & E --> F[多源数据融合]
    F --> G[去重处理]
    G --> H[可信度评估]
    H --> I[写入 PostgreSQL]
    I --> J{还有下一家？}
    J -->|是| C
    J -->|否| K[回调 Dify：Lead 搜索完成]
```

### 工作流 3：数据存储

通用的数据写入工作流，被其他工作流调用：
- 数据校验
- 去重检查
- 批量插入/更新
- 写入日志

### 工作流 4：数据导出

- 接收导出请求（筛选条件 + 导出字段）
- 从 PostgreSQL 查询数据
- 生成 CSV/Excel 文件
- 返回下载链接

## Dify ↔ n8n 通信方式

| 方向 | 方式 | 说明 |
|------|------|------|
| Dify → n8n | HTTP Webhook | Dify 的 HTTP Request 工具调用 n8n Webhook URL |
| n8n → Dify | HTTP API | n8n 的 HTTP Request 节点调用 Dify API 触发工作流 |
| 共享数据 | PostgreSQL | 两者共用同一数据库，通过数据库交换复杂数据 |

## 环境配置

```yaml
# docker-compose.yml 示例
services:
  dify:
    image: langgenius/dify
    ports:
      - "3000:3000"
    environment:
      - DATABASE_URL=postgresql://...

  n8n:
    image: n8nio/n8n
    ports:
      - "5678:5678"
    environment:
      - DB_TYPE=postgresdb
      - DB_POSTGRESDB_HOST=...

  postgres:
    image: postgres:16
    ports:
      - "5432:5432"
```
