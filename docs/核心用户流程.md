# 核心用户流程

## 端到端获客流程

从用户输入到获得可行动的线索列表，LeadKits 的核心流程如下：

```mermaid
flowchart TD
    A[用户输入需求] --> B{已有 ICP？}
    B -->|否| C[AI 对话构建 ICP]
    B -->|是| D[上传/导入 ICP]
    C --> E[ICP 确认与属性拆分]
    D --> E

    E --> F[Supervisor Agent 解析任务]
    F --> G[公司搜索]
    F --> H[意向信号检测]

    G --> I[天眼查搜索 API]
    I --> J[天眼查详情 API]
    J --> K[数据融合]
    H --> K

    K --> L[公司 ICP 评分]
    L --> M{达到评分阈值？}
    M -->|否| N[低优先级存储]
    M -->|是| O[Lead 搜索]

    O --> P[天眼查联系方式 API]
    O --> Q[Bing 搜索]
    O --> R[社交平台搜索]
    P & Q & R --> S[Lead 数据融合]

    S --> T[人员 ICP 评分 + 排序]
    T --> U[AI 分析报告生成]
    U --> V[输出线索列表]
    V --> W[CRM 管理 / 导出]
```

## 各阶段详细说明

### 阶段 1：ICP 定义

| 步骤 | 操作 | 系统行为 |
|------|------|----------|
| 1.1 | 用户选择「构建 ICP」或「上传 ICP」 | 进入对应流程 |
| 1.2a | AI 对话模式：用户描述目标客户 | ICP Builder Agent 引导提问，结构化输出 ICP |
| 1.2b | 上传模式：用户上传 ICP 文件 | 系统解析并结构化 |
| 1.3 | 用户确认/调整 ICP | ICP 存储至数据库 |

### 阶段 2：企业搜索

| 步骤 | 操作 | 系统行为 |
|------|------|----------|
| 2.1 | 系统自动从 ICP 拆分搜索属性 | Supervisor Agent 生成搜索策略 |
| 2.2 | 调用天眼查搜索 API | n8n 工作流执行批量搜索 |
| 2.3 | 调用天眼查详情 API | 获取每家公司的详细信息 |
| 2.4 | 意向信号检测 | 分析招聘、融资、技术选型等信号 |
| 2.5 | 数据融合 + ICP 评分 | AI Agent 对每家公司评分 |

### 阶段 3：Lead 发现

| 步骤 | 操作 | 系统行为 |
|------|------|----------|
| 3.1 | 对评分达标的公司搜索 Lead | n8n 触发 Lead 搜索工作流 |
| 3.2 | 天眼查联系方式 API | 获取企业高管信息 |
| 3.3 | Bing 搜索 | 搜索公开联系信息 |
| 3.4 | 社交平台搜索 | 补充社交账号信息 |
| 3.5 | Lead 数据融合 + 评分排序 | AI Agent 评分并按优先级排序 |

### 阶段 4：分析与输出

| 步骤 | 操作 | 系统行为 |
|------|------|----------|
| 4.1 | AI 分析报告生成 | 为每个 Lead 生成分析摘要 |
| 4.2 | 展示线索列表 | 按评分排序，展示公司+Lead+分析 |
| 4.3 | 用户操作 | 加入 CRM / 导出 / 标记状态 |

## 系统交互流程图

```mermaid
sequenceDiagram
    actor U as 用户
    participant F as 前端 (Next.js)
    participant D as Dify (AI Agent)
    participant N as n8n (工作流)
    participant DB as 数据库
    participant API as 外部 API

    U->>F: 输入客户需求
    F->>D: 调用 ICP Builder Agent
    D->>D: 引导对话，构建 ICP
    D->>DB: 存储 ICP
    D->>F: 返回结构化 ICP
    F->>U: 展示 ICP，请求确认

    U->>F: 确认 ICP，开始搜索
    F->>D: 调用 Supervisor Agent
    D->>N: 下发搜索任务
    N->>API: 天眼查搜索 API
    API-->>N: 返回公司列表
    N->>API: 天眼查详情 API
    API-->>N: 返回公司详情
    N->>DB: 存储公司数据

    D->>D: ICP 评分 Agent
    D->>DB: 存储评分结果

    N->>API: Lead 搜索（天眼查+Bing）
    API-->>N: 返回联系人
    N->>DB: 存储 Lead 数据

    D->>D: Lead 评分 + AI 分析
    D->>DB: 存储分析结果

    F->>DB: 查询结果
    DB-->>F: 返回线索列表
    F->>U: 展示评分排序后的线索
```
