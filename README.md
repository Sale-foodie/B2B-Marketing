# LeadKits — B2B 智能营销获客平台

> ICP 驱动的一站式 B2B 获客解决方案，从目标客户定义到精准触达

![Status](https://img.shields.io/badge/Status-MVP%20开发中-orange)
![License](https://img.shields.io/badge/License-MIT-blue)

---

## 产品简介

LeadKits 是一款面向国内中小 B2B 团队的智能获客平台。以 ICP（理想客户画像）为核心驱动，通过 AI Agent 自动构建客户画像、精准搜索目标企业与决策人、智能评分排序，帮助用户用最低的成本完成从线索发现到商机转化的全流程闭环。

## 核心功能（MVP）

| 模块 | 说明 | 文档 |
|------|------|------|
| ICP 构建 | AI 对话式构建理想客户画像，或直接上传已有 ICP | [详情](docs/features/icp-builder.md) |
| 公司搜索 | 基于 ICP 属性，通过天眼查 API 多维度精准筛选目标企业 | [详情](docs/features/company-search.md) |
| Lead 发现 | 定位关键决策人，通过天眼查/网络搜索/社交平台获取联系方式 | [详情](docs/features/lead-discovery.md) |
| ICP 评分 | 基于 ICP 匹配度对公司和联系人智能评分，自动分级 | [详情](docs/features/icp-scoring.md) |
| AI 分析报告 | AI 生成公司分析摘要、Lead 洞察和触达建议 | [详情](docs/features/ai-analysis.md) |
| CRM 管理 | 站内线索管理、搜索数据管理、状态追踪 | [详情](docs/features/crm-management.md) |

## 技术栈

| 层级 | 技术 | 用途 |
|------|------|------|
| 前端 | Next.js / React / TailwindCSS | 产品 Web 界面 |
| LLM 编排 | Dify | ICP 构建、评分、分析等 AI Agent |
| 工作流引擎 | n8n | 公司搜索、Lead 搜索、数据存储等自动化流程 |
| 数据库 | PostgreSQL | 公司、Lead、ICP、会话等数据存储 |
| 外部 API | 天眼查 / Bing 搜索 | 企业数据与联系人数据来源 |

## 文档导航

### 产品设计

| 文档 | 说明 |
|------|------|
| [产品概述](docs/overview.md) | 愿景、核心问题与解决方案 |
| [市场分析](docs/market-analysis.md) | 市场背景、竞品对比与差异化定位 |
| [用户画像](docs/user-personas.md) | 目标用户角色与使用场景 |
| [用户流程](docs/user-flow.md) | 端到端核心流程与交互设计 |

### 功能模块

| 文档 | 说明 |
|------|------|
| [ICP 构建](docs/features/icp-builder.md) | AI 对话构建 / 上传 ICP，属性维度定义 |
| [公司搜索](docs/features/company-search.md) | 天眼查 API 驱动的多维企业搜索 |
| [Lead 发现](docs/features/lead-discovery.md) | 多渠道联系人发现与信息获取 |
| [ICP 评分](docs/features/icp-scoring.md) | AI 驱动的公司与人员匹配评分 |
| [AI 分析报告](docs/features/ai-analysis.md) | 智能分析摘要与触达建议生成 |
| [CRM 管理](docs/features/crm-management.md) | 站内线索与数据管理 |

### 技术设计

| 文档 | 说明 |
|------|------|
| [系统架构](docs/technical/architecture.md) | 三层架构总览与组件职责 |
| [数据库设计](docs/technical/database.md) | 完整 Schema 与 ER 关系 |
| [工作流引擎](docs/technical/workflow-engine.md) | Dify + n8n 协作设计 |
| [API 集成](docs/technical/api-integrations.md) | 天眼查、Bing 等第三方 API |
| [知识库设计](docs/technical/knowledge-base.md) | 7 大知识库内容与用途 |

### 运营与规划

| 文档 | 说明 |
|------|------|
| [指标体系](docs/metrics.md) | 各模块核心 KPI 定义 |
| [非功能需求](docs/non-functional.md) | 性能、安全与合规要求 |
| [产品路线图](docs/roadmap.md) | MVP → V1 → V2 里程碑规划 |

## 快速开始

```bash
# 克隆项目
git clone <repo-url>
cd B2B获客

# 安装依赖
npm install

# 配置环境变量
cp .env.example .env.local

# 启动开发服务器
npm run dev
```

## License

MIT
