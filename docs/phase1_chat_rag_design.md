# Phase 1 聊天与RAG混合系统设计
# Phase 1 Chat & RAG Hybrid System Design

---

## 设计概述 | Design Overview

**中文：**
Phase 1采用**LLM + RAG混合架构**，通过意图分类器自动路由不同类型的问题：
- **通用SRE咨询** → 直接使用LLM（快速响应）
- **代码审查请求** → Prompt工程 + SRE检查清单
- **深度专业问题** → RAG检索 + LLM增强（高准确性）

**English:**
Phase 1 adopts **LLM + RAG hybrid architecture**, automatically routing different question types via intent classifier:
- **General SRE consultation** → Direct LLM (fast response)
- **Code review requests** → Prompt engineering + SRE checklist
- **Deep professional questions** → RAG retrieval + LLM enhancement (high accuracy)

---

## 系统架构 | System Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                     前端用户输入                              │
│                  Frontend User Input                         │
└────────────────────────┬────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────────┐
│                  POST /api/v1/chat                           │
│                   (统一聊天接口)                              │
└────────────────────────┬────────────────────────────────────┘
                         │
                         ▼
            ┌────────────────────────┐
            │   IntentClassifier     │
            │    (意图分类器)         │
            └────────────┬───────────┘
                         │
        ┌────────────────┼────────────────┐
        │                │                │
        ▼                ▼                ▼
┌──────────────┐  ┌──────────────┐  ┌──────────────┐
│ GENERAL_CHAT │  │ CODE_REVIEW  │  │ SPECIFIC_TECH│
│  通用对话     │  │  代码检查     │  │  深度问题     │
└──────┬───────┘  └──────┬───────┘  └──────┬───────┘
       │                 │                 │
       ▼                 ▼                 ▼
┌──────────────┐  ┌──────────────┐  ┌──────────────┐
│ LLMService   │  │ SREService   │  │ RAGService   │
│ (纯LLM)      │  │(Prompt工程)  │  │(检索增强)    │
└──────┬───────┘  └──────┬───────┘  └──────┬───────┘
       │                 │                 │
       └─────────────────┼─────────────────┘
                         │
                         ▼
                 ┌───────────────┐
                 │  保存聊天记录  │
                 │  返回响应      │
                 └───────────────┘
```

---

## 意图分类器设计 | Intent Classifier Design

### 问题类型定义 | Question Type Definition

```python
# app/services/intent_classifier.py
from typing import Literal
from enum import Enum

class QuestionType(str, Enum):
    """问题类型枚举"""
    GENERAL_CHAT = "general_chat"           # 通用对话
    CODE_REVIEW = "code_review"             # 代码检查
    SRE_CONCEPT = "sre_concept"             # SRE概念问题
    SPECIFIC_TECH = "specific_tech"         # 具体技术问题（需要RAG）
```

---

### 分类器实现 | Classifier Implementation

**文件路径 | File Path**: `app/services/intent_classifier.py`

```python
from typing import Literal
from enum import Enum

class QuestionType(str, Enum):
    GENERAL_CHAT = "general_chat"
    CODE_REVIEW = "code_review"
    SRE_CONCEPT = "sre_concept"
    SPECIFIC_TECH = "specific_tech"

class IntentClassifier:
    """
    问题意图分类器
    Phase 1: 基于规则的轻量级实现
    Phase 2: 可升级为小型BERT分类模型
    """
    
    def __init__(self):
        # 代码块检测模式
        self.code_patterns = [
            "```",              # Markdown代码块
            "def ",             # Python函数
            "function ",        # JavaScript函数
            "class ",           # 类定义
            "import ",          # 导入语句
            "检查代码",
            "review code",
            "看看这段",
            "帮我审查"
        ]
        
        # 需要RAG的深度问题关键词
        self.rag_keywords = [
            "最佳实践",
            "best practice",
            "具体案例",
            "详细步骤",
            "完整方案",
            "官方文档",
            "权威",
            "标准",
            "规范",
            "如何实现",
            "生产环境",
            "真实案例"
        ]
        
        # SRE核心概念
        self.sre_concepts = [
            "SLO", "SLI", "SLA",
            "error budget", "错误预算",
            "可用性", "availability",
            "可靠性", "reliability",
            "监控", "monitoring",
            "告警", "alerting",
            "四大黄金指标",
            "golden signals",
            "熔断", "circuit breaker",
            "降级", "degradation",
            "限流", "rate limiting"
        ]
    
    async def classify(self, message: str) -> QuestionType:
        """
        分类问题类型
        
        Args:
            message: 用户输入的消息
            
        Returns:
            QuestionType: 问题类型
        """
        message_lower = message.lower()
        
        # 优先级1: 检测代码块（最明确）
        if self._contains_code(message):
            return QuestionType.CODE_REVIEW
        
        # 优先级2: 检测需要RAG的深度问题
        if self._needs_rag(message_lower):
            return QuestionType.SPECIFIC_TECH
        
        # 优先级3: 检测SRE概念问题（可以用纯LLM）
        if self._is_sre_concept(message_lower):
            return QuestionType.SRE_CONCEPT
        
        # 优先级4: 默认通用对话
        return QuestionType.GENERAL_CHAT
    
    def _contains_code(self, message: str) -> bool:
        """检测是否包含代码"""
        return any(pattern in message for pattern in self.code_patterns)
    
    def _needs_rag(self, message_lower: str) -> bool:
        """检测是否需要RAG增强"""
        return any(keyword in message_lower for keyword in self.rag_keywords)
    
    def _is_sre_concept(self, message_lower: str) -> bool:
        """检测是否为SRE概念问题"""
        return any(concept.lower() in message_lower for concept in self.sre_concepts)
    
    async def should_use_rag(self, question_type: QuestionType) -> bool:
        """
        判断是否需要使用RAG
        
        Args:
            question_type: 问题类型
            
        Returns:
            bool: 是否使用RAG
        """
        return question_type in [
            QuestionType.SPECIFIC_TECH,
            # QuestionType.CODE_REVIEW  # Phase 1代码检查暂时用Prompt工程
        ]
```

---

## 统一聊天接口 | Unified Chat API

**文件路径 | File Path**: `app/api/v1/chat.py`

```python
from fastapi import APIRouter, Depends
from sqlalchemy.ext.asyncio import AsyncSession
import time

from app.schemas.chat import ChatRequest, ChatResponse
from app.services.llm_service import LLMService
from app.services.rag_service import RAGService
from app.services.sre_service import SREService
from app.services.intent_classifier import IntentClassifier, QuestionType
from app.api.deps import get_db

router = APIRouter()

@router.post("/chat", response_model=ChatResponse)
async def chat(
    request: ChatRequest,
    db: AsyncSession = Depends(get_db)
):
    """
    智能聊天接口 - 自动路由到合适的处理方式
    
    工作流程：
    1. 意图分类
    2. 根据类型选择处理服务（LLM/RAG/SRE）
    3. 生成回答
    4. 保存聊天记录
    5. 返回响应
    """
    
    # 步骤1: 分类问题类型
    classifier = IntentClassifier()
    question_type = await classifier.classify(request.message)
    
    # 步骤2: 根据类型路由到对应服务
    response_text = ""
    sre_analysis = None
    
    if question_type == QuestionType.CODE_REVIEW:
        # 代码检查 - 使用SREService
        sre_service = SREService(db)
        
        # 提取代码片段（简化处理）
        code = request.message
        language = _detect_language(code)
        
        # 执行SRE检查
        sre_analysis = await sre_service.check_code(
            code=code,
            language=language
        )
        
        # 格式化检查结果为文本
        response_text = _format_sre_result(sre_analysis)
        
    elif await classifier.should_use_rag(question_type):
        # 深度专业问题 - 使用RAG增强
        rag_service = RAGService(db)
        response_text = await rag_service.get_rag_enhanced_response(
            message=request.message,
            context=request.context
        )
        
    else:
        # 通用问题或SRE概念 - 直接使用LLM
        llm_service = LLMService(db)
        response_text = await llm_service.get_response(
            message=request.message,
            context=request.context
        )
    
    # 步骤3: 保存聊天记录
    llm_service = LLMService(db)
    
    # 保存用户消息
    await llm_service.save_message(
        session_id=request.session_id,
        visitor_id=request.visitor_id,
        role="user",
        content=request.message
    )
    
    # 保存AI回复
    await llm_service.save_message(
        session_id=request.session_id,
        visitor_id=request.visitor_id,
        role="assistant",
        content=response_text
    )
    
    # 步骤4: 返回响应
    return ChatResponse(
        success=True,
        data={
            "message": response_text,
            "session_id": request.session_id,
            "question_type": question_type,  # 返回问题类型供前端参考
            "sre_analysis": sre_analysis,    # 如果是代码检查，返回结构化结果
            "timestamp": int(time.time() * 1000)
        }
    )


def _detect_language(code: str) -> str:
    """简单的编程语言检测"""
    if "def " in code or "import " in code:
        return "python"
    elif "function " in code or "const " in code:
        return "javascript"
    elif "public class" in code or "private " in code:
        return "java"
    else:
        return "unknown"


def _format_sre_result(analysis: dict) -> str:
    """格式化SRE检查结果为友好的文本"""
    result = "## 🔍 SRE代码检查结果\n\n"
    
    # 总体评分
    score = analysis.get("overall_score", 0)
    result += f"**总体评分**: {score}/100\n\n"
    
    # 检查项
    check_items = analysis.get("check_items", [])
    if check_items:
        result += "### 检查项详情\n\n"
        for item in check_items:
            status_icon = "✅" if item["status"] == "pass" else "⚠️" if item["status"] == "warning" else "❌"
            result += f"{status_icon} **{item['category']}**: {item['message']}\n"
        result += "\n"
    
    # 改进建议
    suggestions = analysis.get("suggestions", [])
    if suggestions:
        result += "### 💡 改进建议\n\n"
        for i, sug in enumerate(suggestions, 1):
            priority_icon = "🔴" if sug["priority"] == "high" else "🟡" if sug["priority"] == "medium" else "🟢"
            result += f"{i}. {priority_icon} {sug['text']}\n"
    
    return result
```

---

## RAG服务实现 | RAG Service Implementation

**文件路径 | File Path**: `app/services/rag_service.py`

```python
from typing import List, Dict, Optional
from langchain.vectorstores import Chroma
from langchain.embeddings import OpenAIEmbeddings
from langchain.text_splitter import RecursiveCharacterTextSplitter
from langchain.chains import RetrievalQA
from langchain.chat_models import ChatOpenAI
from sqlalchemy.ext.asyncio import AsyncSession

from app.core.config import settings

class RAGService:
    """
    RAG服务 - 检索增强生成
    Phase 1: 使用本地Chroma向量数据库
    """
    
    def __init__(self, db: AsyncSession):
        self.db = db
        
        # 初始化Embedding模型
        self.embeddings = OpenAIEmbeddings(
            api_key=settings.OPENAI_API_KEY
        )
        
        # 初始化本地向量数据库
        self.vectorstore = Chroma(
            persist_directory="./data/chroma_db",
            embedding_function=self.embeddings,
            collection_name="sre_knowledge"
        )
        
        # 初始化LLM
        self.llm = ChatOpenAI(
            api_key=settings.OPENAI_API_KEY,
            model=settings.LLM_MODEL,
            temperature=0.3  # 较低温度保证准确性
        )
    
    async def get_rag_enhanced_response(
        self,
        message: str,
        context: Optional[List[Dict]] = None
    ) -> str:
        """
        使用RAG增强的回答
        
        Args:
            message: 用户问题
            context: 对话上下文（暂时不用于RAG）
            
        Returns:
            str: 增强后的回答
        """
        # 1. 配置检索器
        retriever = self.vectorstore.as_retriever(
            search_type="similarity",
            search_kwargs={
                "k": 3  # 返回最相关的3个文档片段
            }
        )
        
        # 2. 构建RetrievalQA链
        qa_chain = RetrievalQA.from_chain_type(
            llm=self.llm,
            chain_type="stuff",  # Phase 1使用简单的stuff方式
            retriever=retriever,
            return_source_documents=True,  # 返回来源文档
            chain_type_kwargs={
                "prompt": self._get_qa_prompt()
            }
        )
        
        # 3. 执行查询
        result = await qa_chain.ainvoke({"query": message})
        
        # 4. 格式化回答（包含来源）
        answer = result["result"]
        sources = result.get("source_documents", [])
        
        # 5. 添加来源引用
        if sources:
            answer += "\n\n---\n📚 **参考来源**：\n"
            for i, doc in enumerate(sources[:2], 1):  # 只显示前2个来源
                source = doc.metadata.get("source", "SRE内部知识库")
                snippet = doc.page_content[:100] + "..."
                answer += f"\n{i}. {source}\n   > {snippet}\n"
        
        return answer
    
    def _get_qa_prompt(self):
        """获取QA Prompt模板"""
        from langchain.prompts import PromptTemplate
        
        template = """
你是一位资深的SRE专家。请基于以下参考文档回答用户的问题。

参考文档：
{context}

用户问题：
{question}

回答要求：
1. 基于参考文档提供准确、专业的建议
2. 如果参考文档不足以回答问题，请明确说明
3. 提供具体的实施步骤或代码示例
4. 突出SRE最佳实践

回答：
"""
        return PromptTemplate(
            template=template,
            input_variables=["context", "question"]
        )
    
    async def initialize_knowledge_base(self, documents: List[str]):
        """
        初始化知识库（一次性操作）
        
        Args:
            documents: 文档列表
        """
        # 1. 文档分块
        text_splitter = RecursiveCharacterTextSplitter(
            chunk_size=1000,
            chunk_overlap=200,
            separators=["\n\n", "\n", " ", ""]
        )
        
        splits = text_splitter.create_documents(documents)
        
        # 2. 添加元数据
        for i, split in enumerate(splits):
            split.metadata = {
                "source": f"SRE最佳实践文档 {i+1}",
                "doc_id": i
            }
        
        # 3. 存入向量数据库
        self.vectorstore = Chroma.from_documents(
            documents=splits,
            embedding=self.embeddings,
            persist_directory="./data/chroma_db",
            collection_name="sre_knowledge"
        )
        
        # 4. 持久化
        self.vectorstore.persist()
        
        print(f"✅ 已加载 {len(splits)} 个文档片段到知识库")
    
    async def add_document(self, content: str, metadata: dict = None):
        """
        动态添加单个文档到知识库
        
        Args:
            content: 文档内容
            metadata: 文档元数据
        """
        text_splitter = RecursiveCharacterTextSplitter(
            chunk_size=1000,
            chunk_overlap=200
        )
        
        splits = text_splitter.create_documents([content])
        
        # 添加元数据
        for split in splits:
            split.metadata = metadata or {"source": "动态添加"}
        
        # 添加到向量库
        self.vectorstore.add_documents(splits)
        self.vectorstore.persist()
```

---

## SRE知识库初始化 | SRE Knowledge Base Initialization

**文件路径 | File Path**: `scripts/init_knowledge_base.py`

```python
"""
初始化SRE知识库脚本
Phase 1: 手动维护的核心SRE最佳实践文档
"""

SRE_KNOWLEDGE_DOCS = [
    # 文档1: SRE四大黄金指标
    """
# SRE四大黄金指标 (The Four Golden Signals)

SRE的监控核心围绕四个关键指标展开：

## 1. 延迟 (Latency)
**定义**: 服务处理请求所需的时间

**监控建议**:
- 使用分位数统计：P50、P95、P99
- 区分成功请求和失败请求的延迟
- 建议告警阈值：P95 > 200ms，P99 > 500ms

**代码示例** (Python):
```python
import time
from prometheus_client import Histogram

request_latency = Histogram(
    'http_request_duration_seconds',
    'HTTP request latency',
    ['method', 'endpoint']
)

@request_latency.time()
def handle_request():
    # 处理请求
    pass
```

## 2. 流量 (Traffic)
**定义**: 系统每秒处理的请求数量

**监控指标**:
- QPS (Queries Per Second)
- RPS (Requests Per Second)
- 吞吐量 (Throughput)

**容量规划**:
- 建立基线：正常流量范围
- 峰值流量识别
- 设置扩容阈值

## 3. 错误 (Errors)
**定义**: 请求失败的比率

**监控维度**:
- HTTP 4xx: 客户端错误
- HTTP 5xx: 服务端错误
- 业务错误率

**SLO建议**:
- 目标：错误率 < 0.1% (即99.9%成功率)
- 关键接口：错误率 < 0.01% (99.99%成功率)

## 4. 饱和度 (Saturation)
**定义**: 系统资源的利用率

**监控资源**:
- CPU使用率
- 内存使用率
- 磁盘IO
- 网络带宽

**告警阈值**:
- 通用服务：80%告警，90%紧急
- 数据库：70%告警，85%紧急
""",

    # 文档2: 错误预算
    """
# 错误预算 (Error Budget)

## 概念
错误预算是SRE中平衡可靠性与快速迭代的核心工具。

**定义**: 在满足SLO的前提下，允许的最大错误量。

## 计算方法

### 基于可用性的错误预算
假设SLO = 99.9% (三个九)

**月度错误预算**:
- 一个月 = 30天 = 43200分钟
- 错误预算 = 43200 × (1 - 0.999) = 43.2分钟
- 即：每月可以停机43.2分钟

**季度错误预算**:
- 一个季度 = 90天
- 错误预算 = 129.6分钟 ≈ 2.16小时

### 基于请求的错误预算
假设月度100万次请求，SLO = 99.9%

**错误预算**:
- 允许失败请求 = 1,000,000 × 0.001 = 1,000次
- 每天预算 ≈ 33次失败请求

## 使用场景

### 1. 发布决策
```
if 错误预算充足 (>50%):
    ✅ 可以加快功能发布
    ✅ 可以进行实验性变更
else if 错误预算紧张 (20%-50%):
    ⚠️ 谨慎发布，增加测试
    ⚠️ 避免风险变更
else:  # 错误预算耗尽 (<20%)
    ❌ 冻结功能发布
    ❌ 专注稳定性改进
    ❌ 修复已知问题
```

### 2. 团队激励
- 错误预算是客观指标，避免主观判断
- SRE团队和开发团队共同对错误预算负责
- 预算充足时鼓励创新，预算紧张时专注质量

## 监控建议
```python
# 实时追踪错误预算消耗
def calculate_error_budget_burn_rate():
    current_error_rate = get_current_error_rate()
    slo_target = 0.999
    
    burn_rate = current_error_rate / (1 - slo_target)
    
    if burn_rate > 10:
        alert("错误预算消耗过快！")
    elif burn_rate > 5:
        warning("错误预算消耗偏高")
    
    return burn_rate
```

## 最佳实践
1. **可视化**: 实时Dashboard显示剩余预算
2. **自动化**: 预算告警自动触发
3. **透明化**: 全团队可见错误预算状态
4. **定期Review**: 每周/月回顾预算使用情况
""",

    # 文档3: 代码审查SRE检查清单
    """
# 代码审查SRE检查清单
# SRE Code Review Checklist

在进行代码审查时，从SRE角度需要关注以下维度：

## ✅ 1. 可靠性 (Reliability)

### 异常处理
- [ ] **是否有异常处理机制？**
  ```python
  # ❌ 错误示例
  def get_user(user_id):
      user = db.query(f"SELECT * FROM users WHERE id={user_id}")
      return user
  
  # ✅ 正确示例
  def get_user(user_id):
      try:
          user = db.query("SELECT * FROM users WHERE id=?", (user_id,))
          return user
      except DatabaseError as e:
          logger.error(f"Failed to get user {user_id}: {e}")
          raise ServiceUnavailableError("Database temporarily unavailable")
  ```

### 超时设置
- [ ] **数据库操作是否有超时？**
  ```python
  # ✅ 设置超时
  db.execute(query, timeout=5)  # 5秒超时
  ```

### 重试机制
- [ ] **是否有重试机制（幂等操作）？**
  ```python
  from tenacity import retry, stop_after_attempt, wait_exponential
  
  @retry(
      stop=stop_after_attempt(3),
      wait=wait_exponential(multiplier=1, min=1, max=10)
  )
  def call_external_api():
      response = requests.get(api_url, timeout=5)
      response.raise_for_status()
      return response.json()
  ```

### 熔断器
- [ ] **关键服务是否有熔断保护？**
  ```python
  from pybreaker import CircuitBreaker
  
  breaker = CircuitBreaker(fail_max=5, timeout_duration=60)
  
  @breaker
  def call_payment_service():
      return payment_api.charge(...)
  ```

## ✅ 2. 可观测性 (Observability)

### 日志记录
- [ ] **关键路径是否有日志？**
- [ ] **日志级别是否合理？**
  ```python
  # ✅ 合理的日志
  logger.info(f"User {user_id} login successful")  # 正常流程
  logger.warning(f"Retry attempt {retry_count}")   # 异常但可恢复
  logger.error(f"Payment failed: {error}")         # 错误需要关注
  ```

- [ ] **避免过多DEBUG日志影响性能**

### 指标埋点
- [ ] **是否有性能指标？**
  ```python
  from prometheus_client import Counter, Histogram
  
  request_count = Counter('api_requests_total', 'Total API requests')
  request_duration = Histogram('api_request_duration_seconds', 'API request duration')
  
  @request_duration.time()
  def handle_api():
      request_count.inc()
      # 处理逻辑
  ```

### 链路追踪
- [ ] **分布式调用是否有trace_id？**

## ✅ 3. 性能 (Performance)

### 数据库优化
- [ ] **查询是否有索引？**
- [ ] **是否避免N+1查询？**
  ```python
  # ❌ N+1查询
  users = User.query.all()
  for user in users:
      user.profile  # 每次触发单独查询
  
  # ✅ 使用JOIN或预加载
  users = User.query.options(joinedload(User.profile)).all()
  ```

### 缓存策略
- [ ] **热点数据是否缓存？**
  ```python
  from functools import lru_cache
  
  @lru_cache(maxsize=1000)
  def get_config(key):
      return db.get_config(key)
  ```

### 分页
- [ ] **大数据集是否分页？**

## ✅ 4. 安全性 (Security)

### 输入验证
- [ ] **用户输入是否验证？**
  ```python
  from pydantic import BaseModel, validator
  
  class UserInput(BaseModel):
      email: str
      
      @validator('email')
      def validate_email(cls, v):
          if '@' not in v:
              raise ValueError('Invalid email')
          return v
  ```

### SQL注入防护
- [ ] **SQL是否参数化？**
  ```python
  # ❌ SQL注入风险
  query = f"SELECT * FROM users WHERE name='{user_input}'"
  
  # ✅ 参数化查询
  query = "SELECT * FROM users WHERE name=?"
  db.execute(query, (user_input,))
  ```

### 敏感信息
- [ ] **密码、密钥是否加密存储？**
- [ ] **日志是否包含敏感信息？**

## ✅ 5. 可用性 (Availability)

### 降级策略
- [ ] **关键功能失败时是否有降级方案？**
  ```python
  def get_recommendations(user_id):
      try:
          return recommendation_service.get(user_id)
      except ServiceError:
          # 降级：返回热门推荐
          return get_popular_items()
  ```

### 健康检查
- [ ] **是否提供健康检查端点？**
  ```python
  @app.get("/health")
  async def health_check():
      # 检查数据库连接
      db_ok = await check_database()
      # 检查缓存
      cache_ok = await check_cache()
      
      if db_ok and cache_ok:
          return {"status": "healthy"}
      else:
          return {"status": "unhealthy"}, 503
  ```
""",

    # 文档4: SLO/SLI/SLA定义
    """
# SLO、SLI、SLA详解

## 核心概念

### SLI (Service Level Indicator) - 服务等级指标
**定义**: 衡量服务质量的具体指标

**常见SLI**:
- **可用性**: 成功请求 / 总请求
- **延迟**: P95响应时间 < 200ms
- **吞吐量**: QPS > 1000
- **正确性**: 数据一致性 > 99.99%

**示例**:
```
SLI: 过去30天内，95%的HTTP请求在200ms内返回
计算: P95延迟 = 180ms ✅ (满足)
```

### SLO (Service Level Objective) - 服务等级目标
**定义**: 对SLI设定的目标值

**示例**:
```
SLO: API可用性 ≥ 99.9%
SLO: P95延迟 ≤ 200ms
SLO: 错误率 ≤ 0.1%
```

**设定原则**:
1. 不要设置100%（不现实）
2. 基于用户体验设定（用户能感知的阈值）
3. 留有改进空间

### SLA (Service Level Agreement) - 服务等级协议
**定义**: 与客户的正式承诺，违反有赔偿

**示例**:
```
SLA: 月度可用性 ≥ 99.95%
如果低于99.95%: 退款10%
如果低于99.9%: 退款25%
```

## 关系
```
SLI (测量指标)
  ↓
SLO (内部目标，通常严格于SLA)
  ↓
SLA (对外承诺，法律约束)
```

## 设定示例

### Web API服务
```yaml
SLI:
  - 可用性: 成功响应数 / 总请求数
  - 延迟: P95响应时间
  - 错误率: 5xx错误数 / 总请求数

SLO:
  - 可用性 ≥ 99.9% (月度)
  - P95延迟 ≤ 200ms
  - 错误率 ≤ 0.1%

SLA:
  - 可用性 ≥ 99.5% (低于SLO，留有buffer)
```

### 数据管道
```yaml
SLI:
  - 数据新鲜度: 最新数据时间戳距离现在的时间
  - 数据完整性: 成功处理记录数 / 总记录数

SLO:
  - 数据延迟 ≤ 5分钟
  - 数据完整性 ≥ 99.99%

SLA:
  - 数据延迟 ≤ 15分钟
```

## 监控实现
```python
from prometheus_client import Counter, Histogram, Gauge

# SLI指标
request_total = Counter('http_requests_total', 'Total HTTP requests', ['status'])
request_duration = Histogram('http_request_duration_seconds', 'HTTP request latency')
error_rate = Gauge('http_error_rate', 'Current error rate')

def calculate_sli():
    # 计算可用性SLI
    total_requests = request_total._value.sum()
    success_requests = request_total.labels(status='2xx')._value.sum()
    availability_sli = success_requests / total_requests if total_requests > 0 else 1
    
    # 计算延迟SLI
    p95_latency = request_duration.observe(0.95)
    
    return {
        'availability': availability_sli,
        'latency_p95': p95_latency
    }
```
""",

    # 文档5: 告警设计最佳实践
    """
# 告警设计最佳实践

## 告警原则

### 1. 告警必须可操作
**规则**: 每条告警必须有明确的处理步骤

❌ **错误示例**:
```
告警: CPU使用率超过80%
问题: 然后呢？需要做什么？
```

✅ **正确示例**:
```
告警: API服务CPU使用率超过80%
处理步骤:
1. 检查是否有异常流量
2. 查看慢查询日志
3. 如果持续5分钟，触发自动扩容
4. 如果无法自动恢复，升级到on-call工程师
```

### 2. 避免告警疲劳
**规则**: 减少噪音，只告警真正的问题

**策略**:
- 使用阈值组合：CPU > 80% **且** 持续5分钟
- 设置告警频率限制：同一问题10分钟内只告警一次
- 区分严重级别：P0紧急、P1重要、P2一般

### 3. 基于症状而非原因告警
✅ 告警: "API响应时间P95 > 500ms"（用户能感知）
❌ 告警: "Redis连接数 > 100"（技术指标，用户不一定受影响）

## 告警级别

### P0 - 紧急 (Critical)
**定义**: 影响核心业务，需要立即响应

**触发条件**:
- 核心API完全不可用
- 数据丢失风险
- 安全漏洞

**响应时间**: 5分钟内
**通知方式**: 电话 + 短信 + IM

### P1 - 重要 (High)
**定义**: 部分功能不可用或严重降级

**触发条件**:
- 错误率 > 1%
- P95延迟 > 1秒
- 部分区域服务中断

**响应时间**: 30分钟内
**通知方式**: 短信 + IM

### P2 - 一般 (Medium)
**定义**: 性能下降，但在可接受范围

**触发条件**:
- 错误率 0.5% - 1%
- P95延迟 500ms - 1秒

**响应时间**: 工作时间内处理
**通知方式**: IM

### P3 - 低 (Low)
**定义**: 潜在问题，不影响用户

**触发条件**:
- 磁盘使用率 > 70%（还有缓冲）
- 依赖服务延迟升高

**响应时间**: 一周内处理
**通知方式**: 工单

## 告警规则示例

### 黄金指标告警
```yaml
# Prometheus告警规则
groups:
  - name: golden_signals
    interval: 30s
    rules:
      # 延迟告警
      - alert: HighLatency
        expr: histogram_quantile(0.95, http_request_duration_seconds) > 0.5
        for: 5m
        labels:
          severity: P1
        annotations:
          summary: "API延迟过高"
          description: "{{ $labels.endpoint }} P95延迟 {{ $value }}秒"
      
      # 错误率告警
      - alert: HighErrorRate
        expr: |
          sum(rate(http_requests_total{status=~"5.."}[5m])) 
          / 
          sum(rate(http_requests_total[5m])) > 0.01
        for: 5m
        labels:
          severity: P0
        annotations:
          summary: "错误率超过1%"
          
      # 饱和度告警
      - alert: HighCPU
        expr: cpu_usage > 0.8
        for: 10m
        labels:
          severity: P2
        annotations:
          summary: "CPU使用率持续高于80%"
```

## 告警降噪技巧

### 1. 使用时间窗口
```
# 避免短暂抖动
for: 5m  # 持续5分钟才告警
```

### 2. 聚合告警
```python
# 单个实例故障不告警，多个实例故障才告警
if failed_instances >= total_instances * 0.3:
    trigger_alert()
```

### 3. 依赖告警抑制
```
# 如果数据库已经告警，抑制依赖数据库的服务告警
if database_down:
    suppress(api_service_alerts)
```

### 4. 静默期
```
# 已知维护窗口，静默告警
silence_alerts(
    start='2024-01-01 02:00',
    end='2024-01-01 04:00',
    reason='Database migration'
)
```

## OnCall最佳实践

1. **轮换制度**: 避免同一人长期OnCall
2. **文档先行**: Runbook必须完善
3. **事后回顾**: 每次告警后Review
4. **自动化优先**: 能自动修复的不要人工介入
"""
]


async def main():
    """执行知识库初始化"""
    import asyncio
    from app.services.rag_service import RAGService
    
    print("🚀 开始初始化SRE知识库...")
    
    # 注意：这里传None是因为初始化不需要数据库会话
    rag_service = RAGService(db=None)
    
    # 初始化知识库
    await rag_service.initialize_knowledge_base(SRE_KNOWLEDGE_DOCS)
    
    print("✅ SRE知识库初始化完成！")
    print(f"📚 已加载 {len(SRE_KNOWLEDGE_DOCS)} 个核心SRE文档")
    print("\n文档列表：")
    print("1. SRE四大黄金指标")
    print("2. 错误预算详解")
    print("3. 代码审查SRE检查清单")
    print("4. SLO/SLI/SLA定义")
    print("5. 告警设计最佳实践")


if __name__ == "__main__":
    import asyncio
    asyncio.run(main())
```

---

## Schema定义 | Schema Definitions

**文件路径 | File Path**: `app/schemas/chat.py`

```python
from pydantic import BaseModel, Field
from typing import List, Dict, Optional

class ChatRequest(BaseModel):
    """聊天请求Schema"""
    message: str = Field(..., description="用户消息")
    session_id: str = Field(..., description="会话ID")
    visitor_id: str = Field(..., description="访客ID")
    context: Optional[List[Dict]] = Field(
        default=None,
        description="对话上下文"
    )

class ChatResponse(BaseModel):
    """聊天响应Schema"""
    success: bool
    data: Dict = Field(..., description="响应数据")
    
    class Config:
        schema_extra = {
            "example": {
                "success": True,
                "data": {
                    "message": "这是AI的回答",
                    "session_id": "session_123",
                    "question_type": "general_chat",
                    "timestamp": 1699999999999
                }
            }
        }
```

---

## 依赖更新 | Dependencies Update

在 `requirements.txt` 中添加RAG相关依赖：

```txt
# ... 现有依赖 ...

# RAG相关
chromadb==0.4.18
sentence-transformers==2.2.2  # 如果使用本地Embedding（可选）
```

---

## 使用流程 | Usage Flow

### 1. 初始化知识库（首次部署）

```bash
# 在项目根目录执行
python scripts/init_knowledge_base.py
```

### 2. 启动后端服务

```bash
uvicorn app.main:app --reload
```

### 3. 前端调用示例

```javascript
// 通用问题（自动路由到LLM）
const response1 = await fetch('/api/v1/chat', {
  method: 'POST',
  body: JSON.stringify({
    message: "什么是SLO？",
    session_id: "session_abc",
    visitor_id: "visitor_123"
  })
});

// 深度问题（自动路由到RAG）
const response2 = await fetch('/api/v1/chat', {
  method: 'POST',
  body: JSON.stringify({
    message: "请详细说明SRE四大黄金指标的最佳实践和具体实现方案",
    session_id: "session_abc",
    visitor_id: "visitor_123"
  })
});

// 代码检查（自动路由到SRE检查）
const response3 = await fetch('/api/v1/chat', {
  method: 'POST',
  body: JSON.stringify({
    message: "请检查这段代码：\n```python\ndef get_users():\n    return db.query('SELECT * FROM users')\n```",
    session_id: "session_abc",
    visitor_id: "visitor_123"
  })
});
```

---

## Phase 1与Phase 2+对比 | Phase 1 vs Phase 2+ Comparison

| 特性 | Phase 1 | Phase 2+ |
|------|---------|----------|
| **意图分类** | 基于规则的关键词匹配 | BERT分类模型 |
| **向量数据库** | 本地Chroma | 云端Pinecone/Weaviate |
| **知识库规模** | 20-30个核心文档 | 100+文档 + 动态更新 |
| **检索策略** | 简单相似度检索 | 混合检索（语义+关键词+重排序） |
| **来源展示** | 简单文本引用 | 可点击的文档链接 |
| **知识库更新** | 手动维护 | 用户自定义 + 自动爬取 |
| **多模态** | 纯文本 | 支持图片、PDF、代码仓库 |

---

## 性能优化建议 | Performance Optimization

### 1. Embedding缓存
```python
# 缓存常见问题的Embedding
from functools import lru_cache

@lru_cache(maxsize=1000)
def get_question_embedding(question: str):
    return embeddings.embed_query(question)
```

### 2. 批量检索
```python
# 一次检索多个相关文档，减少调用次数
retriever.get_relevant_documents(query, k=5)
```

### 3. 异步处理
```python
# 使用异步方法避免阻塞
await qa_chain.ainvoke({"query": message})
```

---

## 监控指标 | Monitoring Metrics

```python
# app/services/rag_service.py 中添加监控
from prometheus_client import Counter, Histogram

# RAG调用统计
rag_requests_total = Counter('rag_requests_total', 'Total RAG requests')
rag_latency = Histogram('rag_request_duration_seconds', 'RAG request latency')
rag_relevance_score = Histogram('rag_relevance_score', 'Document relevance score')

@rag_latency.time()
async def get_rag_enhanced_response(self, message: str, context=None):
    rag_requests_total.inc()
    # ... 现有逻辑
    
    # 记录检索相关性
    for doc in sources:
        score = doc.metadata.get('score', 0)
        rag_relevance_score.observe(score)
```

---

## 常见问题 | FAQ

### Q1: 为什么Phase 1不直接使用云端向量数据库？
**A**: 控制成本和复杂度。本地Chroma足够满足MVP需求，且便于开发测试。

### Q2: 如何判断RAG效果好坏？
**A**: 
- 监控检索文档的相关性分数
- 收集用户反馈（有用/无用）
- A/B测试对比纯LLM和RAG的回答质量

### Q3: 知识库如何持续更新？
**A**:
- Phase 1: 手动添加文档（`scripts/add_document.py`）
- Phase 2: 提供管理后台，支持在线上传
- Phase 3: 自动从公司Wiki/Confluence同步

### Q4: 如果检索不到相关文档怎么办？
**A**: 
```python
if not sources or relevance_score < 0.5:
    # 降级到纯LLM回答
    return await llm_service.get_response(message)
```

---

## 下一步开发任务 | Next Steps

**Phase 1:**
1. ✅ 实现IntentClassifier
2. ✅ 实现RAGService
3. ✅ 更新chat.py路由逻辑
4. ✅ 编写知识库初始化脚本
5. ✅ 测试混合路由效果

**Phase 2:**
1. 训练意图分类模型
2. 优化检索算法（Hybrid Search）
3. 添加用户反馈收集
4. 知识库管理后台

**Phase 3:**
1. 迁移到云端向量数据库
2. 多模态RAG支持
3. 自定义知识库
4. RAG可信度评分
