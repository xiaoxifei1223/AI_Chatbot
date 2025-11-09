# Phase 1 后端设计文档
# Phase 1 Backend Design Document

---

## 技术栈 | Tech Stack

### 核心框架 | Core Framework

**中文：**
- **Web框架**：FastAPI (Python 3.9+)
- **异步支持**：asyncio + uvicorn
- **数据库**：SQLite (Phase 1) → PostgreSQL (Phase 2+)
- **ORM**：SQLAlchemy 2.0
- **数据验证**：Pydantic
- **LLM集成**：OpenAI Python SDK / LangChain
- **日志**：structlog
- **配置管理**：python-dotenv

**English:**
- **Web Framework**: FastAPI (Python 3.9+)
- **Async Support**: asyncio + uvicorn
- **Database**: SQLite (Phase 1) → PostgreSQL (Phase 2+)
- **ORM**: SQLAlchemy 2.0
- **Data Validation**: Pydantic
- **LLM Integration**: OpenAI Python SDK / LangChain
- **Logging**: structlog
- **Config Management**: python-dotenv

**选型理由 | Rationale:**

| 技术 | 理由 |
|------|------|
| **FastAPI** | • 高性能异步框架<br>• 自动生成API文档<br>• 原生支持Pydantic数据验证<br>• 类型提示友好 |
| **SQLite** | • Phase 1无需复杂部署<br>• 零配置，单文件数据库<br>• 便于开发和测试<br>• 后续易迁移到PostgreSQL |
| **SQLAlchemy** | • 成熟的Python ORM<br>• 支持异步操作<br>• 数据库无关，易迁移 |
| **LangChain** | • 简化LLM应用开发<br>• 内置Prompt模板管理<br>• 支持多种LLM提供商 |

---

## 项目结构 | Project Structure

```
backend/
├── app/
│   ├── __init__.py
│   ├── main.py                      # FastAPI应用入口
│   ├── config.py                    # 配置管理
│   │
│   ├── api/                         # API路由
│   │   ├── __init__.py
│   │   ├── v1/
│   │   │   ├── __init__.py
│   │   │   ├── chat.py             # 聊天接口
│   │   │   ├── sre_check.py        # SRE检查接口
│   │   │   └── stats.py            # 统计接口
│   │   └── deps.py                 # 依赖注入
│   │
│   ├── models/                      # 数据库模型
│   │   ├── __init__.py
│   │   ├── visitor.py              # 访客模型
│   │   ├── chat_log.py             # 聊天记录模型
│   │   └── sre_check_log.py        # SRE检查记录
│   │
│   ├── schemas/                     # Pydantic模型
│   │   ├── __init__.py
│   │   ├── chat.py                 # 聊天相关Schema
│   │   ├── sre.py                  # SRE检查Schema
│   │   └── stats.py                # 统计Schema
│   │
│   ├── services/                    # 业务逻辑层
│   │   ├── __init__.py
│   │   ├── llm_service.py          # LLM服务
│   │   ├── sre_service.py          # SRE检查服务
│   │   └── stats_service.py        # 统计服务
│   │
│   ├── core/                        # 核心功能
│   │   ├── __init__.py
│   │   ├── database.py             # 数据库连接
│   │   ├── security.py             # 安全相关
│   │   └── logging.py              # 日志配置
│   │
│   ├── prompts/                     # Prompt模板
│   │   ├── __init__.py
│   │   ├── sre_checklist.py        # SRE检查Prompt
│   │   ├── code_review.py          # 代码审查Prompt
│   │   └── general_chat.py         # 通用对话Prompt
│   │
│   └── utils/                       # 工具函数
│       ├── __init__.py
│       ├── code_parser.py          # 代码解析
│       └── response_formatter.py   # 响应格式化
│
├── tests/                           # 测试
│   ├── __init__.py
│   ├── test_api/
│   └── test_services/
│
├── alembic/                         # 数据库迁移
│   └── versions/
│
├── requirements.txt                 # 依赖列表
├── .env.example                     # 环境变量示例
├── alembic.ini                      # Alembic配置
└── README.md                        # 后端文档
```

---

## 数据库设计 | Database Design

### ER图 | ER Diagram

```
┌─────────────────────┐
│     visitors        │
├─────────────────────┤
│ id (PK)             │
│ visitor_id (UNIQUE) │
│ first_visit_at      │
│ last_visit_at       │
│ visit_count         │
│ user_agent          │
│ created_at          │
└─────────────────────┘
         │
         │ 1:N
         ▼
┌─────────────────────┐
│    visit_logs       │
├─────────────────────┤
│ id (PK)             │
│ visitor_id (FK)     │
│ visited_at          │
│ ip_address          │
│ user_agent          │
│ referrer            │
│ screen_resolution   │
└─────────────────────┘

┌─────────────────────┐
│    chat_logs        │
├─────────────────────┤
│ id (PK)             │
│ session_id          │
│ visitor_id          │
│ role                │ (user/assistant)
│ content             │
│ sre_analysis_id     │ (nullable)
│ created_at          │
└─────────────────────┘
         │
         │ 1:1
         ▼
┌─────────────────────┐
│  sre_check_results  │
├─────────────────────┤
│ id (PK)             │
│ chat_log_id (FK)    │
│ code_snippet        │
│ language            │
│ check_items (JSON)  │
│ suggestions (JSON)  │
│ overall_score       │
│ created_at          │
└─────────────────────┘
```

---

### 表结构定义 | Table Definitions

#### 1. visitors (访客表)

**中文说明：** 记录唯一访客信息，用于去重统计

**English:** Record unique visitor information for deduplication

```sql
CREATE TABLE visitors (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    visitor_id VARCHAR(100) UNIQUE NOT NULL,     -- 前端生成的访客ID
    first_visit_at TIMESTAMP NOT NULL,            -- 首次访问时间
    last_visit_at TIMESTAMP NOT NULL,             -- 最后访问时间
    visit_count INTEGER DEFAULT 1,                -- 访问次数
    user_agent TEXT,                              -- 浏览器信息
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_visitor_id ON visitors(visitor_id);
CREATE INDEX idx_last_visit ON visitors(last_visit_at);
```

**SQLAlchemy模型 | SQLAlchemy Model:**
```python
# app/models/visitor.py
from sqlalchemy import Column, Integer, String, DateTime, Text
from sqlalchemy.sql import func
from app.core.database import Base

class Visitor(Base):
    __tablename__ = "visitors"
    
    id = Column(Integer, primary_key=True, index=True)
    visitor_id = Column(String(100), unique=True, nullable=False, index=True)
    first_visit_at = Column(DateTime, nullable=False, server_default=func.now())
    last_visit_at = Column(DateTime, nullable=False, server_default=func.now())
    visit_count = Column(Integer, default=1)
    user_agent = Column(Text)
    created_at = Column(DateTime, server_default=func.now())
```

---

#### 2. visit_logs (访问日志表)

**中文说明：** 记录每次访问的详细信息

**English:** Record detailed information for each visit

```sql
CREATE TABLE visit_logs (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    visitor_id VARCHAR(100) NOT NULL,             -- 关联访客ID
    visited_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    ip_address VARCHAR(45),                       -- 支持IPv6
    user_agent TEXT,
    referrer TEXT,
    screen_resolution VARCHAR(20),
    
    FOREIGN KEY (visitor_id) REFERENCES visitors(visitor_id)
);

CREATE INDEX idx_visit_logs_visitor ON visit_logs(visitor_id);
CREATE INDEX idx_visit_logs_time ON visit_logs(visited_at);
```

**SQLAlchemy模型 | SQLAlchemy Model:**
```python
# app/models/visitor.py (续)
class VisitLog(Base):
    __tablename__ = "visit_logs"
    
    id = Column(Integer, primary_key=True, index=True)
    visitor_id = Column(String(100), ForeignKey('visitors.visitor_id'), nullable=False)
    visited_at = Column(DateTime, server_default=func.now())
    ip_address = Column(String(45))
    user_agent = Column(Text)
    referrer = Column(Text)
    screen_resolution = Column(String(20))
```

---

#### 3. chat_logs (聊天记录表)

**中文说明：** 记录所有聊天消息

**English:** Record all chat messages

```sql
CREATE TABLE chat_logs (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    session_id VARCHAR(100) NOT NULL,             -- 会话ID
    visitor_id VARCHAR(100),                      -- 访客ID（可选）
    role VARCHAR(20) NOT NULL,                    -- 'user' 或 'assistant'
    content TEXT NOT NULL,                        -- 消息内容
    sre_analysis_id INTEGER,                      -- SRE分析结果ID（可选）
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    
    FOREIGN KEY (sre_analysis_id) REFERENCES sre_check_results(id)
);

CREATE INDEX idx_chat_session ON chat_logs(session_id);
CREATE INDEX idx_chat_visitor ON chat_logs(visitor_id);
CREATE INDEX idx_chat_time ON chat_logs(created_at);
```

**SQLAlchemy模型 | SQLAlchemy Model:**
```python
# app/models/chat_log.py
from sqlalchemy import Column, Integer, String, Text, DateTime, ForeignKey
from sqlalchemy.sql import func
from app.core.database import Base

class ChatLog(Base):
    __tablename__ = "chat_logs"
    
    id = Column(Integer, primary_key=True, index=True)
    session_id = Column(String(100), nullable=False, index=True)
    visitor_id = Column(String(100), index=True)
    role = Column(String(20), nullable=False)  # 'user' or 'assistant'
    content = Column(Text, nullable=False)
    sre_analysis_id = Column(Integer, ForeignKey('sre_check_results.id'))
    created_at = Column(DateTime, server_default=func.now())
```

---

#### 4. sre_check_results (SRE检查结果表)

**中文说明：** 存储代码SRE检查的详细结果

**English:** Store detailed SRE check results for code

```sql
CREATE TABLE sre_check_results (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    chat_log_id INTEGER,                          -- 关联的聊天记录
    code_snippet TEXT NOT NULL,                   -- 被检查的代码
    language VARCHAR(50),                         -- 编程语言
    check_items TEXT,                             -- JSON格式检查项
    suggestions TEXT,                             -- JSON格式建议
    overall_score INTEGER,                        -- 总体评分 0-100
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    
    FOREIGN KEY (chat_log_id) REFERENCES chat_logs(id)
);

CREATE INDEX idx_sre_check_time ON sre_check_results(created_at);
```

**SQLAlchemy模型 | SQLAlchemy Model:**
```python
# app/models/sre_check_log.py
from sqlalchemy import Column, Integer, String, Text, DateTime, ForeignKey
from sqlalchemy.sql import func
from app.core.database import Base

class SRECheckResult(Base):
    __tablename__ = "sre_check_results"
    
    id = Column(Integer, primary_key=True, index=True)
    chat_log_id = Column(Integer, ForeignKey('chat_logs.id'))
    code_snippet = Column(Text, nullable=False)
    language = Column(String(50))
    check_items = Column(Text)  # JSON string
    suggestions = Column(Text)  # JSON string
    overall_score = Column(Integer)
    created_at = Column(DateTime, server_default=func.now())
```

---

## API接口设计 | API Design

### 1. 聊天接口 | Chat API

#### POST /api/v1/chat

**中文功能：** 发送消息，获取AI回复

**English:** Send message and get AI response

**请求体 | Request Body:**
```json
{
  "message": "如何设计高可用系统？",
  "session_id": "session_abc123",
  "visitor_id": "visitor_1699999999_abc",
  "context": [
    {
      "role": "user",
      "content": "之前的问题"
    },
    {
      "role": "assistant",
      "content": "之前的回答"
    }
  ]
}
```

**响应体 | Response Body:**
```json
{
  "success": true,
  "data": {
    "message": "设计高可用系统需要考虑以下几个关键方面...",
    "session_id": "session_abc123",
    "sre_analysis": null,
    "timestamp": 1699999999999
  }
}
```

**实现示例 | Implementation Example:**
```python
# app/api/v1/chat.py
from fastapi import APIRouter, Depends
from sqlalchemy.ext.asyncio import AsyncSession
from app.schemas.chat import ChatRequest, ChatResponse
from app.services.llm_service import LLMService
from app.api.deps import get_db

router = APIRouter()

@router.post("/chat", response_model=ChatResponse)
async def chat(
    request: ChatRequest,
    db: AsyncSession = Depends(get_db)
):
    """
    聊天接口
    """
    llm_service = LLMService(db)
    
    # 保存用户消息
    await llm_service.save_message(
        session_id=request.session_id,
        visitor_id=request.visitor_id,
        role="user",
        content=request.message
    )
    
    # 调用LLM获取回复
    response = await llm_service.get_response(
        message=request.message,
        context=request.context
    )
    
    # 保存AI回复
    await llm_service.save_message(
        session_id=request.session_id,
        visitor_id=request.visitor_id,
        role="assistant",
        content=response
    )
    
    return ChatResponse(
        success=True,
        data={
            "message": response,
            "session_id": request.session_id,
            "timestamp": int(time.time() * 1000)
        }
    )
```

---

### 2. SRE代码检查接口 | SRE Code Check API

#### POST /api/v1/sre/check

**中文功能：** 检查代码的SRE合规性

**English:** Check code SRE compliance

**请求体 | Request Body:**
```json
{
  "code": "def get_users():\n    return db.query('SELECT * FROM users')",
  "language": "python",
  "session_id": "session_abc123",
  "visitor_id": "visitor_1699999999_abc"
}
```

**响应体 | Response Body:**
```json
{
  "success": true,
  "data": {
    "message": "代码SRE检查完成",
    "sre_analysis": {
      "check_items": [
        {
          "id": "1",
          "category": "reliability",
          "status": "fail",
          "message": "缺少错误处理机制"
        },
        {
          "id": "2",
          "category": "monitoring",
          "status": "warning",
          "message": "建议添加日志记录"
        },
        {
          "id": "3",
          "category": "performance",
          "status": "warning",
          "message": "SQL查询未使用参数化，存在注入风险"
        }
      ],
      "suggestions": [
        {
          "id": "1",
          "text": "添加try-except错误处理",
          "priority": "high"
        },
        {
          "id": "2",
          "text": "使用ORM或参数化查询防止SQL注入",
          "priority": "high"
        },
        {
          "id": "3",
          "text": "添加数据库连接超时设置",
          "priority": "medium"
        }
      ],
      "overall_score": 45
    }
  }
}
```

**实现示例 | Implementation Example:**
```python
# app/api/v1/sre_check.py
from fastapi import APIRouter, Depends
from sqlalchemy.ext.asyncio import AsyncSession
from app.schemas.sre import SRECheckRequest, SRECheckResponse
from app.services.sre_service import SREService
from app.api.deps import get_db

router = APIRouter()

@router.post("/sre/check", response_model=SRECheckResponse)
async def check_code(
    request: SRECheckRequest,
    db: AsyncSession = Depends(get_db)
):
    """
    SRE代码检查接口
    """
    sre_service = SREService(db)
    
    # 执行SRE检查
    analysis = await sre_service.check_code(
        code=request.code,
        language=request.language
    )
    
    # 保存检查结果
    await sre_service.save_check_result(
        session_id=request.session_id,
        visitor_id=request.visitor_id,
        code=request.code,
        language=request.language,
        analysis=analysis
    )
    
    return SRECheckResponse(
        success=True,
        data={
            "message": "代码SRE检查完成",
            "sre_analysis": analysis
        }
    )
```

---

### 3. 访客统计接口 | Visitor Statistics API

#### POST /api/v1/stats/visit

**中文功能：** 记录访问并返回总访问量

**English:** Record visit and return total visits

**请求体 | Request Body:**
```json
{
  "visitor_id": "visitor_1699999999_abc",
  "timestamp": 1699999999999,
  "user_agent": "Mozilla/5.0...",
  "screen_resolution": "1920x1080",
  "referrer": "https://google.com"
}
```

**响应体 | Response Body:**
```json
{
  "success": true,
  "data": {
    "total_visits": 1234,
    "visitor_id": "visitor_1699999999_abc"
  }
}
```

#### GET /api/v1/stats/visitors

**中文功能：** 获取访客统计信息

**English:** Get visitor statistics

**响应体 | Response Body:**
```json
{
  "success": true,
  "data": {
    "total_visits": 1234,
    "unique_visitors": 567,
    "today_visits": 89
  }
}
```

**实现示例 | Implementation Example:**
```python
# app/api/v1/stats.py
from fastapi import APIRouter, Depends, Request
from sqlalchemy.ext.asyncio import AsyncSession
from app.schemas.stats import VisitRequest, VisitResponse, StatsResponse
from app.services.stats_service import StatsService
from app.api.deps import get_db

router = APIRouter()

@router.post("/stats/visit", response_model=VisitResponse)
async def record_visit(
    visit_data: VisitRequest,
    request: Request,
    db: AsyncSession = Depends(get_db)
):
    """
    记录访问
    """
    stats_service = StatsService(db)
    
    # 获取IP地址
    ip_address = request.client.host
    
    # 记录访问
    total_visits = await stats_service.record_visit(
        visitor_id=visit_data.visitor_id,
        ip_address=ip_address,
        user_agent=visit_data.user_agent,
        referrer=visit_data.referrer,
        screen_resolution=visit_data.screen_resolution
    )
    
    return VisitResponse(
        success=True,
        data={
            "total_visits": total_visits,
            "visitor_id": visit_data.visitor_id
        }
    )

@router.get("/stats/visitors", response_model=StatsResponse)
async def get_stats(db: AsyncSession = Depends(get_db)):
    """
    获取访客统计
    """
    stats_service = StatsService(db)
    stats = await stats_service.get_statistics()
    
    return StatsResponse(
        success=True,
        data=stats
    )
```

---

## SRE知识库设计 | SRE Knowledge Base Design

### Prompt模板结构 | Prompt Template Structure

**中文说明：** 使用LangChain管理Prompt模板，便于维护和更新

**English:** Use LangChain to manage prompt templates for easy maintenance

#### 1. SRE检查清单Prompt | SRE Checklist Prompt

```python
# app/prompts/sre_checklist.py
from langchain.prompts import PromptTemplate

SRE_CHECKLIST_PROMPT = PromptTemplate(
    input_variables=["code", "language"],
    template="""
你是一位资深的SRE（Site Reliability Engineering）专家。
请对以下{language}代码进行全面的SRE原则检查。

代码:
```{language}
{code}
```

请从以下维度进行分析，并以JSON格式返回结果：

1. **可用性 (Availability)**
   - 是否有错误处理机制？
   - 是否有超时设置？
   - 是否有降级策略？
   - 是否有健康检查？

2. **可靠性 (Reliability)**
   - 是否有重试机制？
   - 是否有幂等性保证？
   - 是否处理了边界情况？
   - 数据一致性如何保证？

3. **监控与可观测性 (Monitoring & Observability)**
   - 是否有日志记录？
   - 日志级别是否合理？
   - 是否有指标采集点？
   - 是否有链路追踪？

4. **性能 (Performance)**
   - 是否存在性能瓶颈？
   - 资源使用是否合理？
   - 是否有缓存策略？
   - 数据库查询是否优化？

5. **安全性 (Security)**
   - 是否有输入验证？
   - 是否防范了常见漏洞（SQL注入、XSS等）？
   - 敏感信息是否加密？
   - 是否有访问控制？

返回格式示例：
{{
  "check_items": [
    {{
      "id": "1",
      "category": "availability",
      "status": "fail",
      "message": "缺少错误处理机制"
    }}
  ],
  "suggestions": [
    {{
      "id": "1",
      "text": "添加try-except错误处理",
      "priority": "high"
    }}
  ],
  "overall_score": 45
}}
"""
)
```

#### 2. 代码审查Prompt | Code Review Prompt

```python
# app/prompts/code_review.py
from langchain.prompts import PromptTemplate

CODE_REVIEW_PROMPT = PromptTemplate(
    input_variables=["code", "language", "focus_area"],
    template="""
作为SRE专家，请对以下代码进行审查，重点关注：{focus_area}

代码语言：{language}

代码：
```{language}
{code}
```

请提供：
1. 发现的问题（按严重程度排序）
2. 改进建议（包含代码示例）
3. SRE最佳实践建议
"""
)
```

#### 3. 通用SRE咨询Prompt | General SRE Consultation Prompt

```python
# app/prompts/general_chat.py
from langchain.prompts import PromptTemplate

GENERAL_SRE_PROMPT = PromptTemplate(
    input_variables=["question", "context"],
    template="""
你是一位经验丰富的SRE工程师，专注于帮助开发团队构建高可用、高可靠的系统。

{context}

用户问题：{question}

请提供专业、实用的建议，包括：
1. 核心概念解释
2. 最佳实践
3. 实际案例或代码示例
4. 常见陷阱和注意事项

回答应该简洁明了，重点突出，便于工程师理解和实施。
"""
)
```

---

### SRE知识库内容 | SRE Knowledge Base Content

#### 核心检查维度 | Core Check Dimensions

**中文：**
```python
# app/prompts/sre_checklist.py (续)

SRE_CHECK_CATEGORIES = {
    "availability": {
        "name": "可用性",
        "checks": [
            "错误处理机制",
            "超时设置",
            "降级策略",
            "健康检查",
            "熔断器模式"
        ]
    },
    "reliability": {
        "name": "可靠性",
        "checks": [
            "重试机制",
            "幂等性保证",
            "边界情况处理",
            "数据一致性",
            "事务管理"
        ]
    },
    "monitoring": {
        "name": "监控与可观测性",
        "checks": [
            "日志记录",
            "日志级别",
            "指标采集",
            "链路追踪",
            "告警配置"
        ]
    },
    "performance": {
        "name": "性能",
        "checks": [
            "性能瓶颈",
            "资源使用",
            "缓存策略",
            "数据库优化",
            "并发控制"
        ]
    },
    "security": {
        "name": "安全性",
        "checks": [
            "输入验证",
            "SQL注入防护",
            "XSS防护",
            "敏感信息加密",
            "访问控制"
        ]
    }
}
```

---

## 服务层实现 | Service Layer Implementation

### LLM服务 | LLM Service

```python
# app/services/llm_service.py
from typing import List, Dict, Optional
from openai import AsyncOpenAI
from sqlalchemy.ext.asyncio import AsyncSession
from app.core.config import settings
from app.models.chat_log import ChatLog

class LLMService:
    def __init__(self, db: AsyncSession):
        self.db = db
        self.client = AsyncOpenAI(api_key=settings.OPENAI_API_KEY)
    
    async def get_response(
        self,
        message: str,
        context: Optional[List[Dict]] = None
    ) -> str:
        """
        获取LLM回复
        """
        # 构建消息列表
        messages = [
            {
                "role": "system",
                "content": "你是一位资深的SRE专家，帮助工程师遵循SRE最佳实践。"
            }
        ]
        
        # 添加上下文
        if context:
            messages.extend(context)
        
        # 添加当前消息
        messages.append({"role": "user", "content": message})
        
        # 调用OpenAI API
        response = await self.client.chat.completions.create(
            model=settings.LLM_MODEL,  # "gpt-3.5-turbo" or "gpt-4"
            messages=messages,
            temperature=0.7,
            max_tokens=2000
        )
        
        return response.choices[0].message.content
    
    async def save_message(
        self,
        session_id: str,
        visitor_id: str,
        role: str,
        content: str
    ) -> ChatLog:
        """
        保存聊天消息
        """
        chat_log = ChatLog(
            session_id=session_id,
            visitor_id=visitor_id,
            role=role,
            content=content
        )
        self.db.add(chat_log)
        await self.db.commit()
        await self.db.refresh(chat_log)
        return chat_log
```

---

### SRE检查服务 | SRE Check Service

```python
# app/services/sre_service.py
import json
from typing import Dict
from sqlalchemy.ext.asyncio import AsyncSession
from langchain.chains import LLMChain
from langchain.chat_models import ChatOpenAI
from app.prompts.sre_checklist import SRE_CHECKLIST_PROMPT
from app.models.sre_check_log import SRECheckResult
from app.core.config import settings

class SREService:
    def __init__(self, db: AsyncSession):
        self.db = db
        self.llm = ChatOpenAI(
            api_key=settings.OPENAI_API_KEY,
            model=settings.LLM_MODEL,
            temperature=0.3  # 较低温度确保一致性
        )
        self.chain = LLMChain(llm=self.llm, prompt=SRE_CHECKLIST_PROMPT)
    
    async def check_code(self, code: str, language: str) -> Dict:
        """
        执行SRE代码检查
        """
        # 调用LLM进行检查
        result = await self.chain.arun(code=code, language=language)
        
        # 解析JSON结果
        try:
            analysis = json.loads(result)
        except json.JSONDecodeError:
            # 如果解析失败，使用默认结构
            analysis = {
                "check_items": [],
                "suggestions": [],
                "overall_score": 0
            }
        
        return analysis
    
    async def save_check_result(
        self,
        session_id: str,
        visitor_id: str,
        code: str,
        language: str,
        analysis: Dict
    ) -> SRECheckResult:
        """
        保存检查结果
        """
        result = SRECheckResult(
            code_snippet=code,
            language=language,
            check_items=json.dumps(analysis.get("check_items", [])),
            suggestions=json.dumps(analysis.get("suggestions", [])),
            overall_score=analysis.get("overall_score", 0)
        )
        self.db.add(result)
        await self.db.commit()
        await self.db.refresh(result)
        return result
```

---

### 统计服务 | Statistics Service

```python
# app/services/stats_service.py
from datetime import datetime, timedelta
from sqlalchemy import select, func
from sqlalchemy.ext.asyncio import AsyncSession
from app.models.visitor import Visitor, VisitLog

class StatsService:
    def __init__(self, db: AsyncSession):
        self.db = db
    
    async def record_visit(
        self,
        visitor_id: str,
        ip_address: str,
        user_agent: str,
        referrer: str = None,
        screen_resolution: str = None
    ) -> int:
        """
        记录访问并返回总访问量
        """
        # 查找或创建访客记录
        stmt = select(Visitor).where(Visitor.visitor_id == visitor_id)
        result = await self.db.execute(stmt)
        visitor = result.scalar_one_or_none()
        
        if visitor:
            # 更新现有访客
            visitor.last_visit_at = datetime.utcnow()
            visitor.visit_count += 1
        else:
            # 创建新访客
            visitor = Visitor(
                visitor_id=visitor_id,
                first_visit_at=datetime.utcnow(),
                last_visit_at=datetime.utcnow(),
                visit_count=1,
                user_agent=user_agent
            )
            self.db.add(visitor)
        
        # 记录访问日志
        visit_log = VisitLog(
            visitor_id=visitor_id,
            ip_address=ip_address,
            user_agent=user_agent,
            referrer=referrer,
            screen_resolution=screen_resolution
        )
        self.db.add(visit_log)
        
        await self.db.commit()
        
        # 获取总访问量
        total_stmt = select(func.sum(Visitor.visit_count))
        total_result = await self.db.execute(total_stmt)
        total_visits = total_result.scalar() or 0
        
        return total_visits
    
    async def get_statistics(self) -> Dict:
        """
        获取统计信息
        """
        # 总访问量
        total_stmt = select(func.sum(Visitor.visit_count))
        total_result = await self.db.execute(total_stmt)
        total_visits = total_result.scalar() or 0
        
        # 独立访客数
        unique_stmt = select(func.count(Visitor.id))
        unique_result = await self.db.execute(unique_stmt)
        unique_visitors = unique_result.scalar() or 0
        
        # 今日访问量
        today = datetime.utcnow().date()
        today_stmt = select(func.count(VisitLog.id)).where(
            func.date(VisitLog.visited_at) == today
        )
        today_result = await self.db.execute(today_stmt)
        today_visits = today_result.scalar() or 0
        
        return {
            "total_visits": total_visits,
            "unique_visitors": unique_visitors,
            "today_visits": today_visits
        }
```

---

## 配置管理 | Configuration Management

### 环境变量 | Environment Variables

```bash
# .env.example

# 应用配置
APP_NAME=SRE AI Chatbot
APP_VERSION=1.0.0
DEBUG=False
HOST=0.0.0.0
PORT=8000

# 数据库配置
DATABASE_URL=sqlite+aiosqlite:///./sre_chatbot.db
# PostgreSQL示例（Phase 2+）
# DATABASE_URL=postgresql+asyncpg://user:password@localhost/sre_chatbot

# OpenAI配置
OPENAI_API_KEY=sk-your-api-key-here
LLM_MODEL=gpt-3.5-turbo
# 或使用 gpt-4 for better quality

# CORS配置
ALLOWED_ORIGINS=http://localhost:3000,http://localhost:5173

# 日志配置
LOG_LEVEL=INFO
LOG_FILE=logs/app.log

# 安全配置
SECRET_KEY=your-secret-key-change-in-production
```

### 配置类 | Configuration Class

```python
# app/config.py
from pydantic_settings import BaseSettings
from typing import List

class Settings(BaseSettings):
    # 应用配置
    APP_NAME: str = "SRE AI Chatbot"
    APP_VERSION: str = "1.0.0"
    DEBUG: bool = False
    HOST: str = "0.0.0.0"
    PORT: int = 8000
    
    # 数据库配置
    DATABASE_URL: str = "sqlite+aiosqlite:///./sre_chatbot.db"
    
    # OpenAI配置
    OPENAI_API_KEY: str
    LLM_MODEL: str = "gpt-3.5-turbo"
    
    # CORS配置
    ALLOWED_ORIGINS: List[str] = ["http://localhost:3000"]
    
    # 日志配置
    LOG_LEVEL: str = "INFO"
    LOG_FILE: str = "logs/app.log"
    
    # 安全配置
    SECRET_KEY: str
    
    class Config:
        env_file = ".env"

settings = Settings()
```

---

## 下一步开发任务 | Next Development Tasks

**中文：**
1. 初始化FastAPI项目结构
2. 安装必要依赖（requirements.txt）
3. 配置数据库连接和ORM
4. 创建数据库表（Alembic迁移）
5. 实现访客统计API
6. 集成OpenAI API
7. 实现聊天接口
8. 实现SRE检查接口
9. 编写Prompt模板
10. 添加日志和错误处理
11. 编写单元测试
12. API文档完善（自动生成Swagger）

**English:**
1. Initialize FastAPI project structure
2. Install necessary dependencies (requirements.txt)
3. Configure database connection and ORM
4. Create database tables (Alembic migration)
5. Implement visitor statistics API
6. Integrate OpenAI API
7. Implement chat interface
8. Implement SRE check interface
9. Write prompt templates
10. Add logging and error handling
11. Write unit tests
12. Complete API documentation (auto-generate Swagger)

---

## 依赖清单 | Dependencies List

```txt
# requirements.txt

# Web框架
fastapi==0.104.1
uvicorn[standard]==0.24.0
pydantic==2.5.0
pydantic-settings==2.1.0

# 数据库
sqlalchemy==2.0.23
aiosqlite==0.19.0
alembic==1.12.1
# asyncpg==0.29.0  # PostgreSQL（Phase 2+）

# LLM集成
openai==1.3.5
langchain==0.0.340
tiktoken==0.5.1

# 工具库
python-dotenv==1.0.0
structlog==23.2.0
python-multipart==0.0.6

# 测试
pytest==7.4.3
pytest-asyncio==0.21.1
httpx==0.25.1

# 代码质量
black==23.11.0
flake8==6.1.0
mypy==1.7.0
```

---

## 性能与安全考虑 | Performance & Security Considerations

### 性能优化 | Performance Optimization

**中文：**
1. **数据库连接池**：使用SQLAlchemy连接池管理
2. **异步处理**：所有IO操作使用async/await
3. **缓存策略**：Phase 2引入Redis缓存
4. **LLM调用优化**：
   - 设置合理的max_tokens限制
   - 使用流式响应（Phase 2）
   - 考虑本地缓存常见问题答案

### 安全措施 | Security Measures

**中文：**
1. **API密钥保护**：使用环境变量，不提交到代码库
2. **输入验证**：使用Pydantic严格验证所有输入
3. **SQL注入防护**：使用ORM参数化查询
4. **CORS配置**：严格限制允许的来源
5. **速率限制**：Phase 2添加API速率限制
6. **日志脱敏**：不记录敏感信息到日志

**English:**
1. **API Key Protection**: Use env vars, never commit to repo
2. **Input Validation**: Strict validation with Pydantic
3. **SQL Injection Prevention**: Use ORM parameterized queries
4. **CORS Configuration**: Strictly limit allowed origins
5. **Rate Limiting**: Add API rate limits in Phase 2
6. **Log Sanitization**: Never log sensitive information
