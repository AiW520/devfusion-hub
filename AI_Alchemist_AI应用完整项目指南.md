# AI Alchemist AI应用完整项目指南

> 基于 Python + LangChain + FastAPI + Ollama 的智能助手应用

---

## 📋 项目介绍

### 项目概述
AI Alchemist 是一个智能助手应用，可以处理文档分析、知识问答、内容生成等任务。项目展示了 LLM 应用开发的核心技能，包括 RAG（检索增强生成）、向量数据库、Prompt 工程等。

- **本地部署**：使用 Ollama 支持本地模型
- **RAG 检索**：基于向量数据库的增强检索
- **多模型支持**：OpenAI/Ollama/本地模型
- **面试亮点**：AI 应用开发核心能力

### 核心技术亮点
```
后端：FastAPI + LangChain + ChromaDB + Ollama
前端：React + TypeScript
向量库：Chroma / FAISS
模型：GPT-4 / Llama2 / 本地模型
特性：RAG、Tool Use、Chain of Thought
```

---

## 🏗️ 技术架构图

### 系统架构

```
┌─────────────────────────────────────────────────────────────────┐
│                         客户端层                                 │
│  ┌─────────────────────────────────────────────────────────────┐ │
│  │                     React Frontend                           │ │
│  │  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐    │ │
│  │  │  聊天    │  │  文档    │  │  知识库  │  │  设置    │    │ │
│  │  │  界面    │  │  上传    │  │  管理    │  │  面板    │    │ │
│  │  └──────────┘  └──────────┘  └──────────┘  └──────────┘    │ │
│  └─────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
                           │ WebSocket + REST
┌──────────────────────────┼────────────────────────────────────┐
│                     FastAPI Backend                             │
│  ┌─────────────────────────────────────────────────────────────┐ │
│  │                     API Layer                                 │ │
│  │  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐    │ │
│  │  │  聊天    │  │  文档    │  │  知识库  │  │  模型    │    │ │
│  │  │  路由    │  │  处理    │  │  检索    │  │  管理    │    │ │
│  │  └──────────┘  └──────────┘  └──────────┘  └──────────┘    │ │
│  └─────────────────────────────────────────────────────────────┘ │
│  ┌─────────────────────────────────────────────────────────────┐ │
│  │                   LangChain Layer                             │ │
│  │  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐    │ │
│  │  │  Chain   │  │  Agent   │  │  Tool    │  │ Prompt  │    │ │
│  │  │  管理    │  │  执行    │  │  调用    │  │  模板    │    │ │
│  │  └──────────┘  └──────────┘  └──────────┘  └──────────┘    │ │
│  └─────────────────────────────────────────────────────────────┘ │
└──────────────────────────┼─────────────────────────────────────┘
                           │
          ┌────────────────┼────────────────┐
          │                │                │
┌─────────┴───────┐ ┌─────┴─────┐ ┌─────────┴───────┐
│   向量存储       │ │   LLM     │ │   文件存储     │
│   ChromaDB      │ │ Ollama    │ │   本地磁盘     │
│                 │ │ /OpenAI   │ │                 │
│  - 文档嵌入     │ │           │ │  - PDF         │
│  - 相似度检索   │ │  - 对话   │ │  - TXT         │
│  - 元数据过滤   │ │  - 推理   │ │  - Markdown    │
└─────────────────┘ └───────────┘ └─────────────────┘
```

### RAG 流程图

```
┌─────────────────────────────────────────────────────────────────┐
│                      文档处理流程 (Indexing)                      │
│                                                                  │
│  ┌─────────┐    ┌─────────┐    ┌─────────┐    ┌─────────┐       │
│  │  文档   │───>│  加载   │───>│  分块   │───>│  嵌入   │       │
│  │  上传   │    │         │    │         │    │         │       │
│  └─────────┘    └─────────┘    └─────────┘    └────┬────┘       │
│                                                    │            │
│                                                    ▼            │
│                                            ┌─────────────┐       │
│                                            │  ChromaDB   │       │
│                                            │  向量存储   │       │
│                                            └─────────────┘       │
└─────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────┐
│                      检索生成流程 (Retrieval)                     │
│                                                                  │
│  ┌─────────┐    ┌─────────┐    ┌─────────┐    ┌─────────┐       │
│  │  用户   │───>│  嵌入   │───>│  相似   │───>│  构造   │       │
│  │  问题   │    │  查询   │    │  度检索 │    │  Prompt │       │
│  └─────────┘    └─────────┘    └─────────┘    └────┬────┘       │
│                                                    │            │
│                                                    ▼            │
│                                            ┌─────────────┐       │
│                                            │    LLM      │       │
│                                            │  生成回答   │       │
│                                            └──────┬──────┘       │
│                                                   │              │
│                                                   ▼              │
│                                            ┌─────────────┐       │
│                                            │   返回给    │       │
│                                            │   用户      │       │
│                                            └─────────────┘       │
└─────────────────────────────────────────────────────────────────┘
```

---

## 📅 学习计划（7天）

### Day 1：Ollama 本地模型配置 ⭐

**目标**：安装和配置 Ollama

**学习内容**：
- Ollama 安装和基础命令
- 模型下载和运行
- API 调用方式
- GPU 配置（可选）

**执行命令**：

```bash
# 1. macOS 安装
brew install ollama

# Linux 安装
curl -fsSL https://ollama.com/install.sh | sh

# Windows 使用 WSL2

# 2. 启动 Ollama 服务
ollama serve

# 3. 拉取模型
ollama pull llama2              # 基础对话
ollama pull llama2:13b           # 更大模型
ollama pull nomic-embed-text     # 嵌入模型
ollama pull mistral              # Mistral 模型

# 4. 测试运行
ollama run llama2 "为什么天空是蓝色的？"

# 5. 查看已安装模型
ollama list

# 6. 模型管理
ollama rm llama2                 # 删除模型
ollama cp llama2 llama2-custom   # 复制模型

# 7. 开放 API（默认端口 11434）
curl http://localhost:11434/api/generate -d '{
  "model": "llama2",
  "prompt": "Hello, world!"
}'
```

---

### Day 2：LangChain 核心概念 ⭐⭐

**目标**：掌握 LangChain 基础

**学习内容**：
- LangChain 组件架构
- LLM 调用
- Prompt 模板
- Chain 组合

**核心代码**：

`ai-alchemist/requirements.txt`
```txt
fastapi==0.109.0
uvicorn[standard]==0.27.0
langchain==0.1.4
langchain-community==0.0.16
langchain-core==0.1.10
langchain-openai==0.0.5
langchain-ollama==0.0.1
chromadb==0.4.22
faiss-cpu==1.7.4
pypdf==4.0.1
python-multipart==0.0.6
pydantic==2.5.3
pydantic-settings==2.1.0
httpx==0.26.0
redis==5.0.1
sse-starlette==1.8.2
python-docx==1.1.0
```

`ai-alchemist/src/config.py` - 配置
```python
from pydantic_settings import BaseSettings
from typing import Optional
import os

class Settings(BaseSettings):
    """应用配置"""
    
    # 应用配置
    APP_NAME: str = "AI Alchemist"
    DEBUG: bool = True
    
    # LLM 配置
    LLM_PROVIDER: str = os.getenv("LLM_PROVIDER", "ollama")  # ollama / openai
    
    # Ollama 配置
    OLLAMA_BASE_URL: str = "http://localhost:11434"
    OLLAMA_MODEL: str = "llama2"
    OLLAMA_EMBED_MODEL: str = "nomic-embed-text"
    
    # OpenAI 配置
    OPENAI_API_KEY: Optional[str] = None
    OPENAI_MODEL: str = "gpt-4"
    
    # 向量数据库
    VECTOR_DB_TYPE: str = "chroma"  # chroma / faiss
    PERSIST_DIRECTORY: str = "./data/chroma_db"
    
    # 文件存储
    UPLOAD_DIRECTORY: str = "./data/uploads"
    MAX_FILE_SIZE: int = 50 * 1024 * 1024  # 50MB
    
    # RAG 配置
    CHUNK_SIZE: int = 1000
    CHUNK_OVERLAP: int = 200
    TOP_K: int = 4
    
    # 认证（可选）
    AUTH_ENABLED: bool = False
    SECRET_KEY: str = "change-me-in-production"
    
    class Config:
        env_file = ".env"

settings = Settings()
```

`ai-alchemist/src/llm/ollama_llm.py` - Ollama LLM
```python
from langchain_ollama import ChatOllama
from langchain_openai import ChatOpenAI
from langchain.schema import HumanMessage, SystemMessage, AIMessage
from typing import Optional, List, Dict, Any
from app.config import settings

class LLMManager:
    """LLM 管理器"""
    
    def __init__(self):
        self.llm = None
        self.embeddings = None
        self._initialize()
    
    def _initialize(self):
        """初始化 LLM"""
        if settings.LLM_PROVIDER == "ollama":
            self._init_ollama()
        elif settings.LLM_PROVIDER == "openai":
            self._init_openai()
        else:
            raise ValueError(f"不支持的 LLM 提供商: {settings.LLM_PROVIDER}")
    
    def _init_ollama(self):
        """初始化 Ollama"""
        from langchain_ollama import ChatOllama, OllamaEmbeddings
        
        self.llm = ChatOllama(
            base_url=settings.OLLAMA_BASE_URL,
            model=settings.OLLAMA_MODEL,
            temperature=0.7,
            streaming=True
        )
        
        self.embeddings = OllamaEmbeddings(
            base_url=settings.OLLAMA_BASE_URL,
            model=settings.OLLAMA_EMBED_MODEL
        )
        
        print(f"Ollama LLM 初始化完成: {settings.OLLAMA_MODEL}")
    
    def _init_openai(self):
        """初始化 OpenAI"""
        from langchain_openai import ChatOpenAI
        
        self.llm = ChatOpenAI(
            api_key=settings.OPENAI_API_KEY,
            model=settings.OPENAI_MODEL,
            temperature=0.7,
            streaming=True
        )
        
        # OpenAI 使用官方嵌入
        from langchain_openai import OpenAIEmbeddings
        self.embeddings = OpenAIEmbeddings()
        
        print(f"OpenAI LLM 初始化完成: {settings.OPENAI_MODEL}")
    
    async def agenerate(
        self,
        messages: List[Dict[str, str]],
        **kwargs
    ) -> str:
        """异步生成回复"""
        # 转换消息格式
        langchain_messages = self._convert_messages(messages)
        
        # 调用 LLM
        response = await self.llm.agenerate([langchain_messages])
        
        return response.generations[0][0].text
    
    def generate(
        self,
        messages: List[Dict[str, str]],
        **kwargs
    ) -> str:
        """同步生成回复"""
        langchain_messages = self._convert_messages(messages)
        response = self.llm.generate([langchain_messages])
        return response.generations[0][0].text
    
    def stream(
        self,
        messages: List[Dict[str, str]],
        **kwargs
    ):
        """流式生成"""
        langchain_messages = self._convert_messages(messages)
        return self.llm.stream(langchain_messages)
    
    def embed_text(self, text: str) -> List[float]:
        """获取文本嵌入"""
        return self.embeddings.embed_query(text)
    
    def embed_documents(self, texts: List[str]) -> List[List[float]]:
        """批量获取嵌入"""
        return self.embeddings.embed_documents(texts)
    
    def _convert_messages(
        self,
        messages: List[Dict[str, str]]
    ) -> List:
        """转换消息格式"""
        langchain_messages = []
        
        for msg in messages:
            role = msg.get("role", "user")
            content = msg.get("content", "")
            
            if role == "system":
                langchain_messages.append(SystemMessage(content=content))
            elif role == "user":
                langchain_messages.append(HumanMessage(content=content))
            elif role == "assistant":
                langchain_messages.append(AIMessage(content=content))
            else:
                langchain_messages.append(HumanMessage(content=content))
        
        return langchain_messages

# 全局实例
llm_manager = LLMManager()
```

`ai-alchemist/src/llm/prompts.py` - Prompt 模板
```python
from langchain.prompts import PromptTemplate
from langchain.chains import LLMChain

# 系统提示词
SYSTEM_PROMPT = """你是一个专业、友好的 AI 助手。

你的能力：
1. 回答各类问题，提供准确、有用的信息
2. 帮助分析文档和提取关键信息
3. 协助写作、编程、数据分析等任务
4. 提供学习指导和建议

行为准则：
- 回答要准确、有据可查
- 不知道的问题要诚实说明
- 保持友好、耐心的态度
- 在引用文档时标注来源

当前上下文：
{context}
"""

# RAG 问答提示词
RAG_QA_PROMPT = PromptTemplate(
    template="""基于以下上下文信息回答用户问题。如果上下文中没有相关信息，请说明无法从提供的内容中找到答案。

上下文：
{context}

用户问题：{question}

请给出详细的回答：""",
    input_variables=["context", "question"]
)

# 文档摘要提示词
SUMMARIZE_PROMPT = PromptTemplate(
    template="""请总结以下文档的主要内容：

文档内容：
{text}

请提供一个简洁的摘要，包含：
1. 文档主题
2. 主要内容
3. 关键要点（3-5个）
4. 文档类型和来源

摘要：""",
    input_variables=["text"]
)

# 代码解释提示词
CODE_EXPLAIN_PROMPT = PromptTemplate(
    template="""请解释以下代码的功能和工作原理：

```{language}
{code}
```

请按以下格式回答：
1. **代码功能**：简要描述代码做什么
2. **工作原理**：解释代码如何实现
3. **关键组件**：解释主要函数/类的用途
4. **可能的改进**：提出优化建议（如果有）""",
    input_variables=["language", "code"]
)

# 对话总结提示词
CONVERSATION_SUMMARY_PROMPT = PromptTemplate(
    template="""请总结以下对话的要点：

对话内容：
{conversation}

摘要要求：
- 提取关键话题
- 总结达成的结论
- 记录未解决的问题""",
    input_variables=["conversation"]
)
```

---

### Day 3：向量数据库与 RAG ⭐⭐⭐

**目标**：实现 RAG 检索系统

**核心代码**：

`ai-alchemist/src/rag/document_loader.py` - 文档加载器
```python
from typing import List, Optional
from langchain_community.document_loaders import (
    PyPDFLoader,
    TextLoader,
    UnstructuredMarkdownLoader,
    Docx2txtLoader,
    UnstructuredFileLoader
)
from langchain.text_splitter import RecursiveCharacterTextSplitter
from langchain.schema import Document
import os
from app.config import settings

class DocumentProcessor:
    """文档处理器"""
    
    def __init__(self):
        self.text_splitter = RecursiveCharacterTextSplitter(
            chunk_size=settings.CHUNK_SIZE,
            chunk_overlap=settings.CHUNK_OVERLAP,
            length_function=len,
            separators=["\n\n", "\n", "。|！|？", " ", ""]
        )
    
    def load_document(
        self,
        file_path: str,
        file_type: Optional[str] = None
    ) -> List[Document]:
        """加载文档"""
        if not os.path.exists(file_path):
            raise FileNotFoundError(f"文件不存在: {file_path}")
        
        # 根据文件类型选择加载器
        file_type = file_type or self._get_file_type(file_path)
        
        loader = self._get_loader(file_path, file_type)
        documents = loader.load()
        
        # 添加元数据
        for doc in documents:
            doc.metadata["source"] = file_path
            doc.metadata["file_type"] = file_type
        
        return documents
    
    def _get_file_type(self, file_path: str) -> str:
        """获取文件类型"""
        ext = os.path.splitext(file_path)[1].lower()
        type_map = {
            ".pdf": "pdf",
            ".txt": "text",
            ".md": "markdown",
            ".docx": "docx",
            ".doc": "doc",
            ".csv": "csv"
        }
        return type_map.get(ext, "unknown")
    
    def _get_loader(self, file_path: str, file_type: str):
        """获取加载器"""
        loaders = {
            "pdf": PyPDFLoader,
            "text": TextLoader,
            "markdown": UnstructuredMarkdownLoader,
            "docx": Docx2txtLoader
        }
        
        loader_class = loaders.get(file_type, UnstructuredFileLoader)
        
        if file_type == "text":
            return loader_class(file_path, encoding="utf-8")
        
        return loader_class(file_path)
    
    def split_documents(
        self,
        documents: List[Document]
    ) -> List[Document]:
        """分割文档"""
        return self.text_splitter.split_documents(documents)
    
    def process_file(
        self,
        file_path: str
    ) -> List[Document]:
        """完整处理流程"""
        # 加载
        documents = self.load_document(file_path)
        
        # 分割
        chunks = self.split_documents(documents)
        
        # 添加块ID
        for i, chunk in enumerate(chunks):
            chunk.metadata["chunk_id"] = i
            chunk.metadata["total_chunks"] = len(chunks)
        
        return chunks
```

`ai-alchemist/src/rag/vector_store.py` - 向量存储
```python
from typing import List, Optional, Dict, Any
from langchain.vectorstores import Chroma, FAISS
from langchain.schema import Document
from app.llm.ollama_llm import llm_manager
from app.config import settings
import os
import shutil

class VectorStore:
    """向量存储管理器"""
    
    def __init__(self):
        self.vector_store = None
        self.embeddings = llm_manager.embeddings
        self._ensure_directory()
    
    def _ensure_directory(self):
        """确保存储目录存在"""
        os.makedirs(settings.PERSIST_DIRECTORY, exist_ok=True)
    
    def create_from_documents(
        self,
        documents: List[Document],
        collection_name: str = "default"
    ):
        """从文档创建向量存储"""
        if settings.VECTOR_DB_TYPE == "chroma":
            self._create_chroma(documents, collection_name)
        elif settings.VECTOR_DB_TYPE == "faiss":
            self._create_faiss(documents, collection_name)
        else:
            raise ValueError(f"不支持的向量数据库: {settings.VECTOR_DB_TYPE}")
    
    def _create_chroma(
        self,
        documents: List[Document],
        collection_name: str
    ):
        """创建 Chroma 向量存储"""
        self.vector_store = Chroma.from_documents(
            documents=documents,
            embedding=self.embeddings,
            persist_directory=os.path.join(
                settings.PERSIST_DIRECTORY,
                collection_name
            ),
            collection_name=collection_name
        )
        self.vector_store.persist()
        print(f"Chroma 向量存储已创建，文档数: {len(documents)}")
    
    def _create_faiss(
        self,
        documents: List[Document],
        collection_name: str
    ):
        """创建 FAISS 向量存储"""
        self.vector_store = FAISS.from_documents(
            documents=documents,
            embedding=self.embeddings
        )
        
        # 保存到磁盘
        save_path = os.path.join(
            settings.PERSIST_DIRECTORY,
            f"faiss_{collection_name}"
        )
        os.makedirs(save_path, exist_ok=True)
        self.vector_store.save_local(save_path)
        
        print(f"FAISS 向量存储已创建，文档数: {len(documents)}")
    
    def load(
        self,
        collection_name: str = "default"
    ) -> bool:
        """加载已存在的向量存储"""
        if settings.VECTOR_DB_TYPE == "chroma":
            return self._load_chroma(collection_name)
        elif settings.VECTOR_DB_TYPE == "faiss":
            return self._load_faiss(collection_name)
        return False
    
    def _load_chroma(self, collection_name: str) -> bool:
        """加载 Chroma"""
        persist_path = os.path.join(
            settings.PERSIST_DIRECTORY,
            collection_name
        )
        
        if not os.path.exists(persist_path):
            return False
        
        try:
            self.vector_store = Chroma(
                persist_directory=persist_path,
                embedding_function=self.embeddings,
                collection_name=collection_name
            )
            print(f"Chroma 向量存储已加载: {collection_name}")
            return True
        except Exception as e:
            print(f"加载 Chroma 失败: {e}")
            return False
    
    def _load_faiss(self, collection_name: str) -> bool:
        """加载 FAISS"""
        load_path = os.path.join(
            settings.PERSIST_DIRECTORY,
            f"faiss_{collection_name}"
        )
        
        if not os.path.exists(load_path):
            return False
        
        try:
            self.vector_store = FAISS.load_local(
                load_path,
                self.embeddings,
                allow_dangerous_deserialization=True
            )
            print(f"FAISS 向量存储已加载: {collection_name}")
            return True
        except Exception as e:
            print(f"加载 FAISS 失败: {e}")
            return False
    
    def similarity_search(
        self,
        query: str,
        k: int = None,
        filter_dict: Dict[str, Any] = None
    ) -> List[Document]:
        """相似度搜索"""
        if not self.vector_store:
            raise ValueError("向量存储未初始化")
        
        return self.vector_store.similarity_search(
            query,
            k=k or settings.TOP_K,
            filter=filter_dict
        )
    
    def similarity_search_with_score(
        self,
        query: str,
        k: int = None
    ) -> List[tuple]:
        """带分数的相似度搜索"""
        if not self.vector_store:
            raise ValueError("向量存储未初始化")
        
        return self.vector_store.similarity_search_with_score(
            query,
            k=k or settings.TOP_K
        )
    
    def add_documents(self, documents: List[Document]):
        """添加文档"""
        if not self.vector_store:
            raise ValueError("向量存储未初始化")
        
        self.vector_store.add_documents(documents)
        
        if settings.VECTOR_DB_TYPE == "chroma":
            self.vector_store.persist()
    
    def delete_by_filter(self, filter_dict: Dict[str, Any]):
        """按过滤器删除"""
        if not self.vector_store:
            raise ValueError("向量存储未初始化")
        
        if settings.VECTOR_DB_TYPE == "chroma":
            self.vector_store._collection.delete(filter=filter_dict)
            self.vector_store.persist()
    
    def get_collection_stats(self) -> Dict[str, Any]:
        """获取集合统计"""
        if not self.vector_store:
            return {"count": 0}
        
        if settings.VECTOR_DB_TYPE == "chroma":
            return {
                "count": self.vector_store._collection.count(),
                "type": "chroma"
            }
        return {"count": 0, "type": "unknown"}

# 全局实例
vector_store = VectorStore()
```

`ai-alchemist/src/rag/rag_chain.py` - RAG Chain
```python
from typing import List, Dict, Optional
from langchain.chains import LLMChain
from langchain.prompts import PromptTemplate
from langchain.schema import Document
from app.llm.ollama_llm import llm_manager
from app.llm.prompts import RAG_QA_PROMPT
from app.rag.vector_store import vector_store
from app.config import settings

class RAGChain:
    """RAG Chain"""
    
    def __init__(self):
        self.llm = llm_manager.llm
        self.vector_store = vector_store
    
    async def query(
        self,
        question: str,
        collection: str = "default",
        use_rag: bool = True,
        **kwargs
    ) -> Dict[str, any]:
        """查询"""
        # 如果使用 RAG，进行检索
        context = ""
        sources = []
        
        if use_rag:
            # 加载向量存储
            if not self.vector_store.load(collection):
                # 如果不存在，创建空存储
                pass
            
            # 检索相关文档
            docs = self.vector_store.similarity_search(
                question,
                k=settings.TOP_K
            )
            
            # 构建上下文
            if docs:
                context = self._build_context(docs)
                sources = self._extract_sources(docs)
        
        # 构建消息
        messages = [
            {"role": "system", "content": f"你是一个有帮助的AI助手。\n\n{context}"},
            {"role": "user", "content": question}
        ]
        
        # 调用 LLM
        answer = await self.llm_manager.agenerate(messages)
        
        return {
            "answer": answer,
            "sources": sources,
            "context_used": bool(context),
            "question": question
        }
    
    def _build_context(self, docs: List[Document]) -> str:
        """构建上下文"""
        context_parts = []
        
        for i, doc in enumerate(docs, 1):
            source = doc.metadata.get("source", "Unknown")
            page = doc.metadata.get("page", "")
            
            context_parts.append(
                f"[文档 {i}] (来源: {source}" + 
                (f", 页码: {page}" if page else "") + 
                f")\n{doc.page_content}"
            )
        
        return "\n\n".join(context_parts)
    
    def _extract_sources(self, docs: List[Document]) -> List[Dict]:
        """提取来源"""
        sources = []
        seen = set()
        
        for doc in docs:
            source = doc.metadata.get("source", "Unknown")
            if source not in seen:
                seen.add(source)
                sources.append({
                    "source": source,
                    "page": doc.metadata.get("page"),
                    "chunk_id": doc.metadata.get("chunk_id")
                })
        
        return sources
    
    def query_stream(
        self,
        question: str,
        collection: str = "default"
    ):
        """流式查询"""
        # 检索
        if self.vector_store.load(collection):
            docs = self.vector_store.similarity_search(question)
            context = self._build_context(docs)
        else:
            context = ""
        
        # 构建消息
        messages = [
            {"role": "system", "content": f"你是一个有帮助的AI助手。\n\n{context}"},
            {"role": "user", "content": question}
        ]
        
        # 流式生成
        return self.llm_manager.stream(messages)
```

---

### Day 4：FastAPI 服务端 ⭐⭐

**目标**：实现 FastAPI 后端

**核心代码**：

`ai-alchemist/src/api/routes.py` - API 路由
```python
from fastapi import APIRouter, UploadFile, File, HTTPException, BackgroundTasks
from fastapi.responses import StreamingResponse
from pydantic import BaseModel
from typing import List, Optional
import os
import json
from app.config import settings
from app.llm.ollama_llm import llm_manager
from app.rag.rag_chain import rag_chain
from app.rag.document_loader import DocumentProcessor
from app.rag.vector_store import vector_store

router = APIRouter(prefix="/api/v1")

# ============= 请求/响应模型 =============

class ChatMessage(BaseModel):
    role: str
    content: str

class ChatRequest(BaseModel):
    messages: List[ChatMessage]
    use_rag: bool = True
    collection: str = "default"
    stream: bool = True

class ChatResponse(BaseModel):
    answer: str
    sources: List[dict] = []
    context_used: bool = False

class DocumentUploadResponse(BaseModel):
    filename: str
    chunks: int
    collection: str

class KnowledgeBaseQuery(BaseModel):
    query: str
    collection: str = "default"
    top_k: int = 4

# ============= 聊天接口 =============

@router.post("/chat", response_model=ChatResponse)
async def chat(request: ChatRequest):
    """聊天接口"""
    # 构建问题（取最后一条用户消息）
    question = ""
    messages = []
    
    for msg in request.messages:
        if msg.role == "user":
            question = msg.content
        messages.append({"role": msg.role, "content": msg.content})
    
    if not question:
        raise HTTPException(status_code=400, detail="未找到用户消息")
    
    # 调用 RAG
    result = await rag_chain.query(
        question=question,
        collection=request.collection,
        use_rag=request.use_rag
    )
    
    return ChatResponse(**result)

@router.post("/chat/stream")
async def chat_stream(request: ChatRequest):
    """流式聊天接口"""
    question = ""
    messages = []
    
    for msg in request.messages:
        if msg.role == "user":
            question = msg.content
        messages.append({"role": msg.role, "content": msg.content})
    
    if not question:
        raise HTTPException(status_code=400, detail="未找到用户消息")
    
    # 检索上下文
    context = ""
    if request.use_rag:
        if vector_store.load(request.collection):
            docs = vector_store.similarity_search(question)
            if docs:
                context = "\n\n".join([f"[{i+1}] {doc.page_content}" for i, doc in enumerate(docs)])
    
    # 构造系统消息
    system_prompt = f"""你是一个专业、友好的 AI 助手。

参考资料：
{context}

请根据以上信息回答用户问题。如果参考资料中没有相关信息，请基于你的知识回答。"""
    
    # 更新消息列表
    final_messages = [{"role": "system", "content": system_prompt}] + messages
    
    # 流式生成
    async def generate():
        try:
            stream = llm_manager.stream(final_messages)
            
            for chunk in stream:
                if chunk.content:
                    yield f"data: {json.dumps({'content': chunk.content}, ensure_ascii=False)}\n\n"
            
            yield f"data: {json.dumps({'done': True}, ensure_ascii=False)}\n\n"
        except Exception as e:
            yield f"data: {json.dumps({'error': str(e)}, ensure_ascii=False)}\n\n"
    
    return StreamingResponse(
        generate(),
        media_type="text/event-stream"
    )

# ============= 文档处理接口 =============

@router.post("/documents/upload", response_model=DocumentUploadResponse)
async def upload_document(
    background_tasks: BackgroundTasks,
    file: UploadFile = File(...),
    collection: str = "default"
):
    """上传并处理文档"""
    # 检查文件大小
    if file.size and file.size > settings.MAX_FILE_SIZE:
        raise HTTPException(
            status_code=400,
            detail=f"文件过大，最大 {settings.MAX_FILE_SIZE // (1024*1024)}MB"
        )
    
    # 保存文件
    os.makedirs(settings.UPLOAD_DIRECTORY, exist_ok=True)
    file_path = os.path.join(settings.UPLOAD_DIRECTORY, file.filename)
    
    try:
        contents = await file.read()
        with open(file_path, "wb") as f:
            f.write(contents)
    except Exception as e:
        raise HTTPException(status_code=500, detail=f"保存文件失败: {str(e)}")
    
    # 后台处理文档
    def process_document():
        try:
            processor = DocumentProcessor()
            chunks = processor.process_file(file_path)
            
            vector_store.create_from_documents(chunks, collection)
            
            print(f"文档处理完成: {file.filename}, {len(chunks)} chunks")
        except Exception as e:
            print(f"文档处理失败: {e}")
    
    background_tasks.add_task(process_document)
    
    return DocumentUploadResponse(
        filename=file.filename,
        chunks=0,  # 后台处理后更新
        collection=collection
    )

@router.get("/documents")
async def list_documents(collection: str = "default"):
    """列出已处理的文档"""
    if not vector_store.load(collection):
        return {"documents": [], "total": 0}
    
    stats = vector_store.get_collection_stats()
    
    return {
        "collection": collection,
        "total_chunks": stats.get("count", 0),
        "documents": []  # 简化实现
    }

@router.delete("/documents/{filename}")
async def delete_document(filename: str, collection: str = "default"):
    """删除文档"""
    if not vector_store.load(collection):
        raise HTTPException(status_code=404, detail="知识库不存在")
    
    try:
        vector_store.delete_by_filter({"source": {"$contains": filename}})
        return {"success": True, "message": f"已删除文档: {filename}"}
    except Exception as e:
        raise HTTPException(status_code=500, detail=str(e))

# ============= 知识库查询 =============

@router.post("/knowledge/query")
async def query_knowledge(request: KnowledgeBaseQuery):
    """知识库查询"""
    if not vector_store.load(request.collection):
        raise HTTPException(status_code=404, detail="知识库不存在")
    
    docs = vector_store.similarity_search(
        request.query,
        k=request.top_k
    )
    
    return {
        "query": request.query,
        "results": [
            {
                "content": doc.page_content,
                "source": doc.metadata.get("source"),
                "score": 1.0  # 简化
            }
            for doc in docs
        ]
    }

# ============= 模型状态 =============

@router.get("/models/status")
async def model_status():
    """获取模型状态"""
    return {
        "provider": settings.LLM_PROVIDER,
        "model": settings.OLLAMA_MODEL if settings.LLM_PROVIDER == "ollama" else settings.OPENAI_MODEL,
        "embed_model": settings.OLLAMA_EMBED_MODEL,
        "status": "running"
    }
```

`ai-alchemist/src/main.py` - 应用入口
```python
from fastapi import FastAPI
from fastapi.middleware.cors import CORSMiddleware
from fastapi.staticfiles import StaticFiles
from app.config import settings
from app.api.routes import router
import os

# 创建应用
app = FastAPI(
    title=settings.APP_NAME,
    description="AI Alchemist - 智能助手应用",
    version="1.0.0"
)

# CORS 配置
app.add_middleware(
    CORSMiddleware,
    allow_origins=["*"],  # 生产环境应限制
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"]
)

# 注册路由
app.include_router(router)

# 健康检查
@app.get("/health")
async def health():
    return {"status": "ok", "app": settings.APP_NAME}

# 启动时初始化
@app.on_event("startup")
async def startup():
    print(f"{settings.APP_NAME} 启动中...")
    
    # 确保目录存在
    os.makedirs(settings.PERSIST_DIRECTORY, exist_ok=True)
    os.makedirs(settings.UPLOAD_DIRECTORY, exist_ok=True)
    
    print("初始化完成")

# 运行
if __name__ == "__main__":
    import uvicorn
    uvicorn.run(
        "app.main:app",
        host="0.0.0.0",
        port=8000,
        reload=settings.DEBUG
    )
```

---

### Day 5：前端开发 ⭐⭐

**目标**：实现 React 前端

**核心代码**：

`frontend/src/App.tsx` - 主应用
```tsx
import React, { useState, useCallback, useRef, useEffect } from 'react';
import {
  Box,
  VStack,
  HStack,
  Input,
  Button,
  Text,
  Container,
  useColorModeValue,
  IconButton,
  Flex,
  Spinner,
  Divider,
  Badge
} from '@chakra-ui/react';
import { SSE } from 'sse';

interface Message {
  id: string;
  role: 'user' | 'assistant';
  content: string;
  sources?: any[];
}

function App() {
  const [messages, setMessages] = useState<Message[]>([]);
  const [input, setInput] = useState('');
  const [loading, setLoading] = useState(false);
  const [useRag, setUseRag] = useState(true);
  const messagesEndRef = useRef<HTMLDivElement>(null);
  
  const bg = useColorModeValue('gray.50', 'gray.900');
  const cardBg = useColorModeValue('white', 'gray.800');
  
  // 滚动到底部
  const scrollToBottom = () => {
    messagesEndRef.current?.scrollIntoView({ behavior: 'smooth' });
  };
  
  useEffect(() => {
    scrollToBottom();
  }, [messages]);
  
  // 发送消息
  const handleSend = useCallback(async () => {
    if (!input.trim() || loading) return;
    
    const userMessage: Message = {
      id: Date.now().toString(),
      role: 'user',
      content: input.trim()
    };
    
    setMessages(prev => [...prev, userMessage]);
    setInput('');
    setLoading(true);
    
    try {
      // 添加空消息占位
      const assistantMessageId = (Date.now() + 1).toString();
      setMessages(prev => [...prev, {
        id: assistantMessageId,
        role: 'assistant',
        content: ''
      }]);
      
      // SSE 流式请求
      const eventSource = new SSE('http://localhost:8000/api/v1/chat/stream', {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json'
        },
        payload: JSON.stringify({
          messages: [...messages.map(m => ({ role: m.role, content: m.content })), userMessage],
          use_rag: useRag,
          stream: true
        })
      });
      
      eventSource.addEventListener('message', (e) => {
        if (!e.data) return;
        
        try {
          const data = JSON.parse(e.data);
          
          if (data.error) {
            console.error('Error:', data.error);
            return;
          }
          
          if (data.done) {
            eventSource.close();
            setLoading(false);
            return;
          }
          
          if (data.content) {
            setMessages(prev => prev.map(msg => 
              msg.id === assistantMessageId 
                ? { ...msg, content: msg.content + data.content }
                : msg
            ));
          }
        } catch (err) {
          console.error('Parse error:', err);
        }
      });
      
      eventSource.addEventListener('error', (e) => {
        console.error('SSE Error:', e);
        setLoading(false);
        eventSource.close();
      });
      
      eventSource.send();
      
    } catch (error) {
      console.error('Send error:', error);
      setLoading(false);
    }
  }, [input, loading, messages, useRag]);
  
  // 键盘事件
  const handleKeyDown = (e: React.KeyboardEvent) => {
    if (e.key === 'Enter' && !e.shiftKey) {
      e.preventDefault();
      handleSend();
    }
  };
  
  return (
    <Box minH="100vh" bg={bg}>
      {/* 头部 */}
      <Box bg={cardBg} borderBottomWidth={1} py={4} px={6}>
        <Container maxW="container.xl">
          <Flex justify="space-between" align="center">
            <VStack align="start" spacing={0}>
              <Text fontSize="2xl" fontWeight="bold">
                🔮 AI Alchemist
              </Text>
              <Text fontSize="sm" color="gray.500">
                智能助手 · 基于 RAG 检索增强生成
              </Text>
            </VStack>
            
            <HStack spacing={4}>
              <Badge colorScheme={useRag ? 'blue' : 'gray'} px={3} py={1}>
                RAG {useRag ? '开启' : '关闭'}
              </Badge>
              <Button
                size="sm"
                variant={useRag ? 'solid' : 'outline'}
                colorScheme="blue"
                onClick={() => setUseRag(!useRag)}
              >
                {useRag ? '关闭知识库' : '开启知识库'}
              </Button>
            </HStack>
          </Flex>
        </Container>
      </Box>
      
      {/* 消息区域 */}
      <Container maxW="container.lg" py={6}>
        <VStack spacing={4} align="stretch">
          {messages.length === 0 && (
            <Box textAlign="center" py={20}>
              <Text fontSize="4xl" mb={4}>🤖</Text>
              <Text fontSize="lg" color="gray.500">
                开始对话吧！上传文档后可基于知识库回答
              </Text>
            </Box>
          )}
          
          {messages.map((msg) => (
            <Box
              key={msg.id}
              bg={msg.role === 'user' ? 'blue.500' : cardBg}
              color={msg.role === 'user' ? 'white' : 'gray.800'}
              p={4}
              borderRadius="lg"
              alignSelf={msg.role === 'user' ? 'flex-end' : 'flex-start'}
              maxW="80%"
            >
              <Text whiteSpace="pre-wrap">{msg.content || '思考中...'}</Text>
              
              {msg.role === 'assistant' && msg.content === '' && loading && (
                <Spinner size="sm" />
              )}
              
              {msg.sources && msg.sources.length > 0 && (
                <Divider my={2} />
              )}
            </Box>
          ))}
          
          <div ref={messagesEndRef} />
        </VStack>
      </Container>
      
      {/* 输入区域 */}
      <Box
        position="fixed"
        bottom={0}
        left={0}
        right={0}
        bg={cardBg}
        borderTopWidth={1}
        py={4}
      >
        <Container maxW="container.lg">
          <HStack spacing={4}>
            <Input
              value={input}
              onChange={(e) => setInput(e.target.value)}
              onKeyDown={handleKeyDown}
              placeholder="输入你的问题... (Enter 发送, Shift+Enter 换行)"
              size="lg"
              bg={useColorModeValue('white', 'gray.700')}
              disabled={loading}
            />
            <Button
              colorScheme="blue"
              size="lg"
              onClick={handleSend}
              isLoading={loading}
              px={8}
            >
              发送
            </Button>
          </HStack>
          
          <Text fontSize="xs" color="gray.500" mt={2} textAlign="center">
            基于 LangChain + Ollama 构建 · 支持本地模型
          </Text>
        </Container>
      </Box>
    </Box>
  );
}

export default App;
```

---

### Day 6 & 7：项目整合与面试准备 ⭐⭐⭐

**目标**：完成项目部署和面试准备

**Docker 部署**：

`docker-compose.yml`
```yaml
version: '3.8'

services:
  backend:
    build: ./backend
    ports:
      - "8000:8000"
    environment:
      - LLM_PROVIDER=ollama
      - OLLAMA_BASE_URL=http://host.docker.internal:11434
    volumes:
      - ./data:/app/data
    networks:
      - ai-network

  frontend:
    build: ./frontend
    ports:
      - "3000:80"
    depends_on:
      - backend
    networks:
      - ai-network

networks:
  ai-network:
    driver: bridge
```

---

## 📝 面试要点

### Q1：什么是 RAG？为什么需要 RAG？

**参考答案**：
```
RAG = Retrieval-Augmented Generation（检索增强生成）

为什么需要：
1. 知识时效性：LLM 知识有截止日期
2. 幻觉问题：LLM 可能产生错误信息
3. 领域知识：通用模型缺乏专业领域知识
4. 可追溯性：基于检索的回答可标注来源
5. 成本控制：比 Fine-tuning 更经济

工作流程：
用户问题 → 向量化 → 相似度检索 → 构造 Prompt → LLM 生成 → 返回答案
```

### Q2：向量数据库与传统数据库的区别？

**参考答案**：
```
1. 数据类型：向量库存 Embedding，传统库存结构化数据
2. 查询方式：向量库用相似度检索，传统库用精确匹配
3. 适用场景：
   - 向量库：语义搜索、推荐系统、图像检索
   - 传统库：事务处理、精确查询
4. 性能指标：向量库关注 ANN 准确率和速度
```

### Q3：LangChain 的核心组件？

**参考答案**：
```
1. Model I/O：LLM 调用、Prompt 模板
2. Retrieval：文档加载、分割、嵌入、向量存储
3. Chains：链接多个组件处理复杂任务
4. Agents：让 LLM 决定使用什么工具
5. Memory：对话历史存储
6. Callbacks：监控和日志
```

### Q4：如何提升 RAG 效果？

**参考答案**：
```
1. 文档处理：
   - 合理的分块大小
   - 保留上下文信息
   - 元数据提取

2. 检索优化：
   - 混合检索（向量+关键词）
   - 重排序（Reranker）
   - 查询扩展/改写

3. 生成优化：
   - 好的 Prompt 设计
   - Few-shot 示例
   - 引用标注
```

---

## ✅ 项目清单

```
AI-Alchemist/
├── backend/
│   ├── src/
│   │   ├── config.py
│   │   ├── main.py
│   │   ├── api/
│   │   │   └── routes.py
│   │   ├── llm/
│   │   │   ├── ollama_llm.py
│   │   │   └── prompts.py
│   │   └── rag/
│   │       ├── document_loader.py
│   │       ├── vector_store.py
│   │       └── rag_chain.py
│   ├── requirements.txt
│   └── Dockerfile
├── frontend/
│   ├── src/
│   │   ├── App.tsx
│   │   └── main.tsx
│   ├── package.json
│   └── Dockerfile
├── data/
│   ├── uploads/
│   └── chroma_db/
├── docker-compose.yml
└── README.md
```

---

**文档版本**：v1.0  
**更新日期**：2024
