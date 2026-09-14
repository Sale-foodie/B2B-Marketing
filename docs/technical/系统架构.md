# 系统架构

## 架构总览

LeadKits 采用三层架构：前端展示层、工作流引擎层、数据层，通过 Dify（AI 编排）和 n8n（数据工作流）实现业务逻辑。

```mermaid
graph TD
    subgraph 前端层["前端展示层 (Next.js)"]
        UI[Web 界面]
        Pages[页面路由]
        Components[组件库]
    end

    subgraph 工作流层["工作流引擎层"]
        subgraph Dify["Dify (AI Agent 编排)"]
            ICP_Agent[ICP Builder Agent]
            Supervisor[Supervisor Agent]
            Scoring_Agent[ICP Scoring Agent]
            Analysis_Agent[AI Analysis Agent]
        end

        subgraph N8N["n8n (数据工作流)"]
            Company_WF[公司搜索工作流]
            Lead_WF[Lead 搜索工作流]
            Storage_WF[数据存储工作流]
            Export_WF[数据导出工作流]
        end
    end

    subgraph 数据层["数据层"]
        DB[(PostgreSQL)]
        KB[知识库]
    end

    subgraph 外部服务["外部 API"]
        TYC[天眼查 API]
        Bing[Bing 搜索 API]
        LLM[LLM API]
    end

    UI --> Dify
    UI --> N8N
    UI --> DB
    Dify <--> N8N
    Dify --> LLM
    Dify --> KB
    N8N --> TYC
    N8N --> Bing
    N8N --> DB
```

## 组件职责

### 前端层 (Next.js)

| 组件 | 职责 |
|------|------|
| **Web 界面** | 用户交互界面，响应式设计 |
| **App Router** | 页面路由管理 |
| **API Routes** | 轻量 BFF 层，转发请求到 Dify/n8n |
| **状态管理** | 前端数据状态（线索列表、筛选条件等） |

### 工作流引擎层

#### Dify — AI Agent 编排

负责所有需要 LLM 参与的智能处理：

| Agent | 职责 | 调用方式 |
|-------|------|----------|
| **ICP Builder** | 对话式 ICP 构建 | 前端直接调用 Dify API |
| **Supervisor** | 解析 ICP，拆分搜索任务 | 前端调用，输出任务给 n8n |
| **ICP Scoring** | 公司和 Lead 的 ICP 匹配评分 | n8n 回调触发 |
| **AI Analysis** | 生成分析报告和触达建议 | 评分完成后触发 |

#### n8n — 数据工作流

负责所有数据获取、处理、存储的自动化流程：

| 工作流 | 职责 | 触发方式 |
|--------|------|----------|
| **公司搜索** | 调用天眼查 API 搜索+获取详情 | Supervisor 下发任务 |
| **Lead 搜索** | 多渠道搜索联系人 | 公司评分完成后触发 |
| **数据存储** | 清洗、去重、写入数据库 | 搜索结果返回后 |
| **数据导出** | 生成 CSV/Excel 导出文件 | 用户手动触发 |

### 数据层

| 组件 | 用途 |
|------|------|
| **PostgreSQL** | 核心业务数据存储（公司、Lead、ICP、会话） |
| **知识库（Dify）** | Dify 内置向量知识库，支撑 Agent 的行业知识 |

## 数据流向

```mermaid
sequenceDiagram
    participant U as 用户
    participant FE as Next.js
    participant D as Dify
    participant N as n8n
    participant API as 外部 API
    participant DB as PostgreSQL

    U->>FE: 1. 输入需求
    FE->>D: 2. 调用 ICP Builder
    D-->>FE: 3. 返回 ICP
    FE->>DB: 4. 存储 ICP

    U->>FE: 5. 确认，开始搜索
    FE->>D: 6. 调用 Supervisor
    D->>N: 7. 下发搜索任务
    N->>API: 8. 天眼查搜索
    API-->>N: 9. 返回数据
    N->>DB: 10. 存储公司数据
    N->>D: 11. 回调触发评分

    D->>DB: 12. 读取公司数据
    D->>DB: 13. 存储评分结果
    D->>N: 14. 触发 Lead 搜索

    N->>API: 15. Lead 搜索
    API-->>N: 16. 返回 Lead
    N->>DB: 17. 存储 Lead 数据
    N->>D: 18. 回调触发分析

    D->>DB: 19. 存储分析报告
    FE->>DB: 20. 查询展示结果
    DB-->>FE: 21. 返回线索列表
    FE-->>U: 22. 展示结果
```

## 部署架构

```
┌──────────────────────────────────────────┐
│              部署环境                      │
├──────────┬──────────┬──────────┬─────────┤
│ Next.js  │   Dify   │   n8n   │ Postgres │
│ (Vercel/ │ (Docker) │ (Docker)│ (Docker/ │
│  Docker) │          │         │  Cloud)  │
└──────────┴──────────┴──────────┴─────────┘
```

**MVP 部署方案**：
- Next.js：Vercel 或 Docker
- Dify：Docker 自部署
- n8n：Docker 自部署
- PostgreSQL：Docker 或云数据库（阿里云 RDS）
