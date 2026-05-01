# LiteLynx 轻快全栈完整项目指南

> 基于 Python + FastAPI + React + PostgreSQL + Redis 的高性能任务管理系统

---

## 📋 项目介绍

### 项目概述
LiteLynx 是一个现代化的任务管理 API 系统，专为学习和实战而设计。它具有以下特点：

- **快速开发**：使用 FastAPI 的自动文档生成，效率提升 300%
- **性能优异**：Redis 缓存加速查询响应，支持高并发
- **架构清晰**：前后端分离，便于理解和扩展
- **面试友好**：涵盖全栈开发核心知识点

### 解决的问题
- 个人任务管理：待办事项、番茄钟、任务分类
- 团队协作基础：任务分配、状态流转
- 学习成果展示：完整的前后端交互项目

### 核心技术亮点
```
后端：FastAPI + SQLAlchemy + Redis + PostgreSQL
前端：React 18 + TypeScript + React Query + TailwindCSS
亮点：异步编程、缓存策略、RESTful API 设计
```

---

## 🏗️ 技术架构图

### 系统架构

```
┌─────────────────────────────────────────────────────────────────┐
│                         客户端层 (Client Layer)                  │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐             │
│  │   React     │  │   移动端    │  │    浏览器   │             │
│  │   Web App   │  │   (PWA)     │  │   Postman   │             │
│  └──────┬──────┘  └──────┬──────┘  └──────┬──────┘             │
└─────────┼────────────────┼────────────────┼─────────────────────┘
          │                │                │
          └────────────────┼────────────────┘
                           │ HTTP/REST
┌──────────────────────────┼────────────────────────────────────┐
│                     API 网关层                                   │
│  ┌─────────────────────────────────────────────────────────────┐│
│  │                      FastAPI Server                          ││
│  │  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐    ││
│  │  │  路由    │  │  中间件  │  │  认证    │  │  验证    │    ││
│  │  │ Router   │  │ Middleware│ │  JWT     │  │ Pydantic │    ││
│  │  └──────────┘  └──────────┘  └──────────┘  └──────────┘    ││
│  └─────────────────────────────────────────────────────────────┘│
└──────────────────────────┼─────────────────────────────────────┘
                           │
          ┌────────────────┼────────────────┐
          │                │                │
┌─────────┴───────┐ ┌─────┴─────┐ ┌─────────┴───────┐
│   缓存层         │ │   数据层  │ │   业务逻辑层    │
│   Redis         │ │ PostgreSQL│ │   Services     │
│   Cluster       │ │           │ │                 │
│   (热点数据)     │ │ (持久存储) │ │  TaskService    │
│                 │ │           │ │  UserService    │
│  - 会话管理     │ │  - 用户表  │ │  AuthService    │
│  - 热点缓存     │ │  - 任务表  │ │                 │
│  - 限流计数     │ │  - 分类表  │ │                 │
└─────────────────┘ └───────────┘ └─────────────────┘

```

### 技术栈层级

```
┌────────────────────────────────────────────────┐
│                 表示层 (Presentation)           │
│         React + TypeScript + TailwindCSS        │
├────────────────────────────────────────────────┤
│                 API 层 (API Layer)              │
│         FastAPI + Pydantic + OpenAPI           │
├────────────────────────────────────────────────┤
│               业务逻辑层 (Business)              │
│    Services + Cache + Auth + Validation         │
├────────────────────────────────────────────────┤
│                 数据访问层 (DAO)                │
│        SQLAlchemy ORM + Redis Client           │
├────────────────────────────────────────────────┤
│                 数据存储层 (Data)               │
│         PostgreSQL + Redis + Migrations        │
└────────────────────────────────────────────────┘
```

---

## 📅 学习计划（7天）

### Day 1：项目初始化与环境搭建 ⭐

**目标**：搭建完整的后端项目结构

**学习内容**：
- Python 虚拟环境配置
- FastAPI 项目结构设计
- 依赖包管理（requirements.txt / pyproject.toml）
- 数据库连接配置

**产出**：可运行的后端项目骨架

**执行命令**：

```bash
# 1. 创建项目目录
mkdir -p litelynx-backend
cd litelynx-backend

# 2. 创建虚拟环境
python -m venv venv

# 3. 激活虚拟环境（Windows）
# venv\Scripts\activate

# 激活虚拟环境（Linux/Mac）
source venv/bin/activate

# 4. 创建 requirements.txt
cat > requirements.txt << 'EOF'
fastapi==0.109.0
uvicorn[standard]==0.27.0
sqlalchemy==2.0.25
asyncpg==0.29.0
psycopg2-binary==2.9.9
redis==5.0.1
pydantic==2.5.3
pydantic-settings==2.1.0
python-jose[cryptography]==3.3.0
passlib[bcrypt]==1.7.4
python-multipart==0.0.6
alembic==1.13.1
httpx==0.26.0
pytest==7.4.4
pytest-asyncio==0.23.3
EOF

# 5. 安装依赖
pip install -r requirements.txt

# 6. 创建项目结构
mkdir -p app/{api,core,db,models,schemas,services,utils}
touch app/__init__.py app/api/__init__.py app/core/__init__.py
touch app/db/__init__.py app/models/__init__.py app/schemas/__init__.py
touch app/services/__init__.py app/utils/__init__.py
mkdir -p tests alembic/versions
```

---

### Day 2：数据库设计与模型定义 ⭐⭐

**目标**：完成数据库表设计和 SQLAlchemy 模型

**学习内容**：
- PostgreSQL 数据库设计
- SQLAlchemy ORM 模型创建
- 数据库关系（一对多、多对多）
- Alembic 数据库迁移

**产出**：完整的数据库模型和迁移脚本

**核心代码**：

`app/core/config.py` - 配置文件
```python
from pydantic_settings import BaseSettings
from typing import Optional

class Settings(BaseSettings):
    """应用配置类"""
    
    # 应用基础配置
    APP_NAME: str = "LiteLynx API"
    APP_VERSION: str = "1.0.0"
    DEBUG: bool = True
    
    # 数据库配置
    DATABASE_URL: str = "postgresql+asyncpg://postgres:postgres@localhost:5432/litelynx"
    DATABASE_URL_SYNC: str = "postgresql://postgres:postgres@localhost:5432/litelynx"
    
    # Redis 配置
    REDIS_HOST: str = "localhost"
    REDIS_PORT: int = 6379
    REDIS_DB: int = 0
    REDIS_PASSWORD: Optional[str] = None
    
    # JWT 配置
    SECRET_KEY: str = "your-super-secret-key-change-in-production"
    ALGORITHM: str = "HS256"
    ACCESS_TOKEN_EXPIRE_MINUTES: int = 30
    REFRESH_TOKEN_EXPIRE_DAYS: int = 7
    
    # CORS 配置
    BACKEND_CORS_ORIGINS: list[str] = ["http://localhost:3000"]
    
    class Config:
        env_file = ".env"
        case_sensitive = True

settings = Settings()
```

`app/db/base.py` - 数据库基础
```python
from sqlalchemy.ext.asyncio import create_async_engine, AsyncSession, async_sessionmaker
from sqlalchemy.orm import declarative_base
from app.core.config import settings

# 异步引擎
engine = create_async_engine(
    settings.DATABASE_URL,
    echo=settings.DEBUG,
    pool_pre_ping=True,
    pool_size=10,
    max_overflow=20
)

# 异步会话工厂
AsyncSessionLocal = async_sessionmaker(
    engine,
    class_=AsyncSession,
    expire_on_commit=False,
    autocommit=False,
    autoflush=False
)

# 声明基类
Base = declarative_base()

async def get_db():
    """获取数据库会话的依赖"""
    async with AsyncSessionLocal() as session:
        try:
            yield session
            await session.commit()
        except Exception:
            await session.rollback()
            raise
        finally:
            await session.close()
```

`app/models/user.py` - 用户模型
```python
from sqlalchemy import Column, Integer, String, Boolean, DateTime, ForeignKey, Text
from sqlalchemy.orm import relationship
from sqlalchemy.sql import func
from app.db.base import Base

class User(Base):
    """用户模型"""
    __tablename__ = "users"
    
    id = Column(Integer, primary_key=True, index=True)
    email = Column(String(255), unique=True, index=True, nullable=False)
    username = Column(String(100), unique=True, index=True, nullable=False)
    hashed_password = Column(String(255), nullable=False)
    full_name = Column(String(200))
    is_active = Column(Boolean, default=True)
    is_superuser = Column(Boolean, default=False)
    avatar_url = Column(String(500))
    bio = Column(Text)
    created_at = Column(DateTime(timezone=True), server_default=func.now())
    updated_at = Column(DateTime(timezone=True), onupdate=func.now())
    
    # 关系
    tasks = relationship("Task", back_populates="owner", cascade="all, delete-orphan")
    categories = relationship("Category", back_populates="owner", cascade="all, delete-orphan")
    
    def __repr__(self):
        return f"<User {self.username}>"
```

`app/models/task.py` - 任务模型
```python
from sqlalchemy import Column, Integer, String, Boolean, DateTime, ForeignKey, Text, Enum as SQLEnum
from sqlalchemy.orm import relationship
from sqlalchemy.sql import func
import enum
from app.db.base import Base

class TaskPriority(str, enum.Enum):
    """任务优先级枚举"""
    LOW = "low"
    MEDIUM = "medium"
    HIGH = "high"
    URGENT = "urgent"

class TaskStatus(str, enum.Enum):
    """任务状态枚举"""
    TODO = "todo"
    IN_PROGRESS = "in_progress"
    DONE = "done"
    CANCELLED = "cancelled"

class Task(Base):
    """任务模型"""
    __tablename__ = "tasks"
    
    id = Column(Integer, primary_key=True, index=True)
    title = Column(String(300), nullable=False, index=True)
    description = Column(Text)
    status = Column(SQLEnum(TaskStatus), default=TaskStatus.TODO, index=True)
    priority = Column(SQLEnum(TaskPriority), default=TaskPriority.MEDIUM)
    due_date = Column(DateTime(timezone=True), nullable=True)
    completed_at = Column(DateTime(timezone=True), nullable=True)
    estimated_minutes = Column(Integer, nullable=True)
    actual_minutes = Column(Integer, nullable=True)
    
    # 外键
    owner_id = Column(Integer, ForeignKey("users.id"), nullable=False, index=True)
    category_id = Column(Integer, ForeignKey("categories.id"), nullable=True, index=True)
    
    # 时间戳
    created_at = Column(DateTime(timezone=True), server_default=func.now())
    updated_at = Column(DateTime(timezone=True), onupdate=func.now())
    
    # 关系
    owner = relationship("User", back_populates="tasks")
    category = relationship("Category", back_populates="tasks")
    tags = relationship("Tag", secondary="task_tags", back_populates="tasks")
    
    def __repr__(self):
        return f"<Task {self.title}>"
```

`app/models/category.py` - 分类模型
```python
from sqlalchemy import Column, Integer, String, ForeignKey, DateTime, UniqueConstraint
from sqlalchemy.orm import relationship
from sqlalchemy.sql import func
from app.db.base import Base

class Category(Base):
    """任务分类模型"""
    __tablename__ = "categories"
    
    id = Column(Integer, primary_key=True, index=True)
    name = Column(String(100), nullable=False, index=True)
    color = Column(String(7), default="#3B82F6")  # HEX 颜色
    icon = Column(String(50), default="folder")
    owner_id = Column(Integer, ForeignKey("users.id"), nullable=False, index=True)
    created_at = Column(DateTime(timezone=True), server_default=func.now())
    
    # 关系
    owner = relationship("User", back_populates="categories")
    tasks = relationship("Task", back_populates="category")
    
    # 唯一约束：同一用户不能有同名分类
    __table_args__ = (
        UniqueConstraint('owner_id', 'name', name='unique_category_name_per_user'),
    )
    
    def __repr__(self):
        return f"<Category {self.name}>"
```

`app/models/tag.py` - 标签模型（多对多）
```python
from sqlalchemy import Column, Integer, String, ForeignKey, DateTime, Table
from sqlalchemy.orm import relationship
from sqlalchemy.sql import func
from app.db.base import Base

# 多对多关联表
task_tags = Table(
    'task_tags',
    Base.metadata,
    Column('task_id', Integer, ForeignKey('tasks.id', ondelete='CASCADE'), primary_key=True),
    Column('tag_id', Integer, ForeignKey('tags.id', ondelete='CASCADE'), primary_key=True),
    Column('created_at', DateTime(timezone=True), server_default=func.now())
)

class Tag(Base):
    """任务标签模型"""
    __tablename__ = "tags"
    
    id = Column(Integer, primary_key=True, index=True)
    name = Column(String(50), nullable=False, unique=True, index=True)
    color = Column(String(7), default="#10B981")
    created_at = Column(DateTime(timezone=True), server_default=func.now())
    
    # 关系
    tasks = relationship("Task", secondary=task_tags, back_populates="tags")
    
    def __repr__(self):
        return f"<Tag {self.name}>"
```

**数据库迁移命令**：

```bash
# 初始化 Alembic
alembic init alembic

# 配置 alembic.ini（修改 sqlalchemy.url）
# sqlalchemy.url = postgresql://postgres:postgres@localhost:5432/litelynx

# 配置 env.py 添加模型导入
cat > alembic/env.py << 'EOF'
from logging.config import fileConfig
from sqlalchemy import engine_from_config, pool
from alembic import context
import asyncio
from sqlalchemy.ext.asyncio import AsyncEngine

# 导入模型
from app.db.base import Base
from app.models.user import User
from app.models.task import Task
from app.models.category import Category
from app.models.tag import Tag
from app.core.config import settings

config = context.config
config.set_main_option("sqlalchemy.url", settings.DATABASE_URL_SYNC)

if config.config_file_name is not None:
    fileConfig(config.config_file_name)

target_metadata = Base.metadata

def run_migrations_offline() -> None:
    url = config.get_main_option("sqlalchemy.url")
    context.configure(
        url=url,
        target_metadata=target_metadata,
        literal_binds=True,
        dialect_opts={"paramstyle": "named"},
    )
    with context.begin_transaction():
        context.run_migrations()

async def run_migrations_online() -> None:
    connectable = AsyncEngine(engine)
    async with connectable.connect() as connection:
        await connection.run_sync(do_run_migrations)
        
def do_run_migrations(connection):
    context.configure(connection=connection, target_metadata=target_metadata)
    with context.begin_transaction():
        context.run_migrations()

if context.is_offline_mode():
    run_migrations_offline()
else:
    asyncio.run(run_migrations_online())
EOF

# 生成迁移文件
alembic revision --autogenerate -m "initial migration"

# 执行迁移
alembic upgrade head
```

---

### Day 3：认证系统与 API 路由 ⭐⭐

**目标**：实现完整的用户认证系统和 API 接口

**学习内容**：
- JWT 认证原理
- 密码加密与验证
- Pydantic 数据验证
- FastAPI 路由设计

**产出**：完整的认证 API 和任务 CRUD 接口

**核心代码**：

`app/schemas/user.py` - 用户 Schema
```python
from pydantic import BaseModel, EmailStr, Field, ConfigDict
from typing import Optional
from datetime import datetime

class UserBase(BaseModel):
    email: EmailStr
    username: str = Field(..., min_length=3, max_length=100)
    full_name: Optional[str] = None

class UserCreate(UserBase):
    password: str = Field(..., min_length=6, max_length=100)

class UserUpdate(BaseModel):
    email: Optional[EmailStr] = None
    username: Optional[str] = Field(None, min_length=3, max_length=100)
    full_name: Optional[str] = None
    password: Optional[str] = Field(None, min_length=6)
    avatar_url: Optional[str] = None
    bio: Optional[str] = None

class UserResponse(UserBase):
    id: int
    is_active: bool
    is_superuser: bool
    avatar_url: Optional[str] = None
    bio: Optional[str] = None
    created_at: datetime
    
    model_config = ConfigDict(from_attributes=True)

class UserInDB(UserResponse):
    hashed_password: str
```

`app/schemas/task.py` - 任务 Schema
```python
from pydantic import BaseModel, Field, ConfigDict
from typing import Optional, List
from datetime import datetime
from app.models.task import TaskPriority, TaskStatus

class TaskBase(BaseModel):
    title: str = Field(..., min_length=1, max_length=300)
    description: Optional[str] = None
    priority: TaskPriority = TaskPriority.MEDIUM
    due_date: Optional[datetime] = None
    estimated_minutes: Optional[int] = Field(None, ge=1, le=1440)
    category_id: Optional[int] = None

class TaskCreate(TaskBase):
    tag_ids: Optional[List[int]] = []

class TaskUpdate(BaseModel):
    title: Optional[str] = Field(None, min_length=1, max_length=300)
    description: Optional[str] = None
    status: Optional[TaskStatus] = None
    priority: Optional[TaskPriority] = None
    due_date: Optional[datetime] = None
    completed_at: Optional[datetime] = None
    estimated_minutes: Optional[int] = Field(None, ge=1, le=1440)
    actual_minutes: Optional[int] = Field(None, ge=1, le=1440)
    category_id: Optional[int] = None
    tag_ids: Optional[List[int]] = None

class TagResponse(BaseModel):
    id: int
    name: str
    color: str
    
    model_config = ConfigDict(from_attributes=True)

class CategoryResponse(BaseModel):
    id: int
    name: str
    color: str
    icon: str
    
    model_config = ConfigDict(from_attributes=True)

class TaskResponse(TaskBase):
    id: int
    status: TaskStatus
    due_date: Optional[datetime] = None
    completed_at: Optional[datetime] = None
    estimated_minutes: Optional[int] = None
    actual_minutes: Optional[int] = None
    owner_id: int
    category_id: Optional[int] = None
    category: Optional[CategoryResponse] = None
    tags: List[TagResponse] = []
    created_at: datetime
    updated_at: Optional[datetime] = None
    
    model_config = ConfigDict(from_attributes=True)

class TaskListResponse(BaseModel):
    tasks: List[TaskResponse]
    total: int
    page: int
    page_size: int
```

`app/schemas/auth.py` - 认证 Schema
```python
from pydantic import BaseModel, EmailStr
from typing import Optional

class Token(BaseModel):
    access_token: str
    refresh_token: str
    token_type: str = "bearer"

class TokenPayload(BaseModel):
    sub: Optional[str] = None
    exp: Optional[int] = None
    type: Optional[str] = None

class LoginRequest(BaseModel):
    username: str
    password: str

class RefreshTokenRequest(BaseModel):
    refresh_token: str
```

`app/utils/security.py` - 安全工具
```python
from datetime import datetime, timedelta
from typing import Optional, Any
from jose import jwt, JWTError
from passlib.context import CryptContext
from app.core.config import settings

# 密码加密上下文
pwd_context = CryptContext(schemes=["bcrypt"], deprecated="auto")

def verify_password(plain_password: str, hashed_password: str) -> bool:
    """验证密码"""
    return pwd_context.verify(plain_password, hashed_password)

def get_password_hash(password: str) -> str:
    """获取密码哈希"""
    return pwd_context.hash(password)

def create_access_token(subject: str, expires_delta: Optional[timedelta] = None) -> str:
    """创建访问令牌"""
    if expires_delta:
        expire = datetime.utcnow() + expires_delta
    else:
        expire = datetime.utcnow() + timedelta(minutes=settings.ACCESS_TOKEN_EXPIRE_MINUTES)
    
    to_encode = {
        "exp": expire,
        "sub": str(subject),
        "type": "access"
    }
    return jwt.encode(to_encode, settings.SECRET_KEY, algorithm=settings.ALGORITHM)

def create_refresh_token(subject: str) -> str:
    """创建刷新令牌"""
    expire = datetime.utcnow() + timedelta(days=settings.REFRESH_TOKEN_EXPIRE_DAYS)
    to_encode = {
        "exp": expire,
        "sub": str(subject),
        "type": "refresh"
    }
    return jwt.encode(to_encode, settings.SECRET_KEY, algorithm=settings.ALGORITHM)

def decode_token(token: str) -> Optional[dict]:
    """解码令牌"""
    try:
        payload = jwt.decode(token, settings.SECRET_KEY, algorithms=[settings.ALGORITHM])
        return payload
    except JWTError:
        return None
```

`app/api/deps.py` - 依赖注入
```python
from typing import Annotated
from fastapi import Depends, HTTPException, status
from fastapi.security import OAuth2PasswordBearer
from sqlalchemy.ext.asyncio import AsyncSession
from sqlalchemy import select
from app.db.base import get_db
from app.models.user import User
from app.utils.security import decode_token

oauth2_scheme = OAuth2PasswordBearer(tokenUrl="/api/v1/auth/login")

async def get_current_user(
    token: Annotated[str, Depends(oauth2_scheme)],
    db: AsyncSession = Depends(get_db)
) -> User:
    """获取当前登录用户"""
    credentials_exception = HTTPException(
        status_code=status.HTTP_401_UNAUTHORIZED,
        detail="无法验证凭据",
        headers={"WWW-Authenticate": "Bearer"},
    )
    
    payload = decode_token(token)
    if payload is None:
        raise credentials_exception
    
    user_id: str = payload.get("sub")
    token_type: str = payload.get("type")
    
    if user_id is None or token_type != "access":
        raise credentials_exception
    
    # 从数据库获取用户
    result = await db.execute(select(User).where(User.id == int(user_id)))
    user = result.scalar_one_or_none()
    
    if user is None:
        raise credentials_exception
    
    if not user.is_active:
        raise HTTPException(
            status_code=status.HTTP_403_FORBIDDEN,
            detail="用户已被禁用"
        )
    
    return user

def get_current_active_user(
    current_user: Annotated[User, Depends(get_current_user)]
) -> User:
    """获取当前活跃用户"""
    if not current_user.is_active:
        raise HTTPException(
            status_code=status.HTTP_403_FORBIDDEN,
            detail="用户已被禁用"
        )
    return current_user

async def get_current_superuser(
    current_user: Annotated[User, Depends(get_current_user)]
) -> User:
    """获取当前超级用户"""
    if not current_user.is_superuser:
        raise HTTPException(
            status_code=status.HTTP_403_FORBIDDEN,
            detail="权限不足"
        )
    return current_user
```

`app/api/v1/auth.py` - 认证路由
```python
from fastapi import APIRouter, Depends, HTTPException, status
from fastapi.security import OAuth2PasswordRequestForm
from sqlalchemy.ext.asyncio import AsyncSession
from sqlalchemy import select
from datetime import timedelta
from app.db.base import get_db
from app.models.user import User
from app.schemas.auth import Token, LoginRequest, RefreshTokenRequest
from app.schemas.user import UserCreate, UserResponse
from app.utils.security import (
    verify_password, get_password_hash, 
    create_access_token, create_refresh_token, decode_token
)
from app.core.config import settings

router = APIRouter(prefix="/auth", tags=["认证"])

@router.post("/register", response_model=UserResponse, status_code=status.HTTP_201_CREATED)
async def register(
    user_in: UserCreate,
    db: AsyncSession = Depends(get_db)
):
    """用户注册"""
    # 检查邮箱是否已存在
    result = await db.execute(select(User).where(User.email == user_in.email))
    if result.scalar_one_or_none():
        raise HTTPException(
            status_code=status.HTTP_400_BAD_REQUEST,
            detail="该邮箱已被注册"
        )
    
    # 检查用户名是否已存在
    result = await db.execute(select(User).where(User.username == user_in.username))
    if result.scalar_one_or_none():
        raise HTTPException(
            status_code=status.HTTP_400_BAD_REQUEST,
            detail="该用户名已被使用"
        )
    
    # 创建用户
    user = User(
        email=user_in.email,
        username=user_in.username,
        full_name=user_in.full_name,
        hashed_password=get_password_hash(user_in.password)
    )
    db.add(user)
    await db.commit()
    await db.refresh(user)
    
    return user

@router.post("/login", response_model=Token)
async def login(
    form_data: OAuth2PasswordRequestForm = Depends(),
    db: AsyncSession = Depends(get_db)
):
    """用户登录（OAuth2兼容）"""
    # 查找用户
    result = await db.execute(select(User).where(User.username == form_data.username))
    user = result.scalar_one_or_none()
    
    if not user or not verify_password(form_data.password, user.hashed_password):
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="用户名或密码错误",
            headers={"WWW-Authenticate": "Bearer"},
        )
    
    if not user.is_active:
        raise HTTPException(
            status_code=status.HTTP_403_FORBIDDEN,
            detail="用户已被禁用"
        )
    
    # 创建令牌
    access_token = create_access_token(user.id)
    refresh_token = create_refresh_token(user.id)
    
    return Token(
        access_token=access_token,
        refresh_token=refresh_token
    )

@router.post("/refresh", response_model=Token)
async def refresh_token(
    request: RefreshTokenRequest,
    db: AsyncSession = Depends(get_db)
):
    """刷新令牌"""
    payload = decode_token(request.refresh_token)
    
    if payload is None or payload.get("type") != "refresh":
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="无效的刷新令牌"
        )
    
    user_id = payload.get("sub")
    result = await db.execute(select(User).where(User.id == int(user_id)))
    user = result.scalar_one_or_none()
    
    if not user or not user.is_active:
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="用户不存在或已被禁用"
        )
    
    access_token = create_access_token(user.id)
    new_refresh_token = create_refresh_token(user.id)
    
    return Token(
        access_token=access_token,
        refresh_token=new_refresh_token
    )

@router.get("/me", response_model=UserResponse)
async def get_current_user_info(
    db: AsyncSession = Depends(get_db),
    current_user: User = Depends(get_db)
):
    """获取当前用户信息"""
    return current_user
```

`app/api/v1/tasks.py` - 任务路由
```python
from fastapi import APIRouter, Depends, HTTPException, status, Query
from sqlalchemy.ext.asyncio import AsyncSession
from sqlalchemy import select, func
from sqlalchemy.orm import selectinload
from typing import List, Optional
from datetime import datetime
from app.db.base import get_db
from app.models.user import User
from app.models.task import Task, TaskStatus, TaskPriority
from app.models.category import Category
from app.models.tag import Tag, task_tags
from app.schemas.task import (
    TaskCreate, TaskUpdate, TaskResponse, TaskListResponse
)
from app.schemas.user import UserResponse
from app.api.deps import get_current_user
from app.utils.redis import redis_client

router = APIRouter(prefix="/tasks", tags=["任务管理"])

@router.get("/", response_model=TaskListResponse)
async def list_tasks(
    page: int = Query(1, ge=1),
    page_size: int = Query(20, ge=1, le=100),
    status: Optional[TaskStatus] = None,
    priority: Optional[TaskPriority] = None,
    category_id: Optional[int] = None,
    search: Optional[str] = None,
    db: AsyncSession = Depends(get_db),
    current_user: User = Depends(get_current_user)
):
    """获取任务列表"""
    # 构建查询
    query = select(Task).where(Task.owner_id == current_user.id)
    count_query = select(func.count(Task.id)).where(Task.owner_id == current_user.id)
    
    # 筛选条件
    if status:
        query = query.where(Task.status == status)
        count_query = count_query.where(Task.status == status)
    if priority:
        query = query.where(Task.priority == priority)
        count_query = count_query.where(Task.priority == priority)
    if category_id:
        query = query.where(Task.category_id == category_id)
        count_query = count_query.where(Task.category_id == category_id)
    if search:
        query = query.where(Task.title.ilike(f"%{search}%"))
        count_query = count_query.where(Task.title.ilike(f"%{search}%"))
    
    # 总数
    total_result = await db.execute(count_query)
    total = total_result.scalar()
    
    # 分页
    offset = (page - 1) * page_size
    query = query.options(
        selectinload(Task.category),
        selectinload(Task.tags)
    ).order_by(Task.created_at.desc()).offset(offset).limit(page_size)
    
    result = await db.execute(query)
    tasks = result.scalars().all()
    
    return TaskListResponse(
        tasks=[TaskResponse.model_validate(task) for task in tasks],
        total=total,
        page=page,
        page_size=page_size
    )

@router.post("/", response_model=TaskResponse, status_code=status.HTTP_201_CREATED)
async def create_task(
    task_in: TaskCreate,
    db: AsyncSession = Depends(get_db),
    current_user: User = Depends(get_current_user)
):
    """创建任务"""
    # 验证分类
    if task_in.category_id:
        result = await db.execute(
            select(Category).where(
                Category.id == task_in.category_id,
                Category.owner_id == current_user.id
            )
        )
        if not result.scalar_one_or_none():
            raise HTTPException(
                status_code=status.HTTP_404_NOT_FOUND,
                detail="分类不存在"
            )
    
    # 创建任务
    task = Task(
        title=task_in.title,
        description=task_in.description,
        priority=task_in.priority,
        due_date=task_in.due_date,
        estimated_minutes=task_in.estimated_minutes,
        owner_id=current_user.id,
        category_id=task_in.category_id
    )
    db.add(task)
    await db.flush()
    
    # 添加标签
    if task_in.tag_ids:
        result = await db.execute(select(Tag).where(Tag.id.in_(task_in.tag_ids)))
        tags = result.scalars().all()
        task.tags = list(tags)
    
    await db.commit()
    await db.refresh(task)
    
    # 清除缓存
    await redis_client.delete_pattern(f"tasks:{current_user.id}:*")
    
    return task

@router.get("/{task_id}", response_model=TaskResponse)
async def get_task(
    task_id: int,
    db: AsyncSession = Depends(get_db),
    current_user: User = Depends(get_current_user)
):
    """获取单个任务"""
    result = await db.execute(
        select(Task)
        .options(selectinload(Task.category), selectinload(Task.tags))
        .where(Task.id == task_id, Task.owner_id == current_user.id)
    )
    task = result.scalar_one_or_none()
    
    if not task:
        raise HTTPException(
            status_code=status.HTTP_404_NOT_FOUND,
            detail="任务不存在"
        )
    
    return task

@router.put("/{task_id}", response_model=TaskResponse)
async def update_task(
    task_id: int,
    task_in: TaskUpdate,
    db: AsyncSession = Depends(get_db),
    current_user: User = Depends(get_current_user)
):
    """更新任务"""
    result = await db.execute(
        select(Task)
        .options(selectinload(Task.tags))
        .where(Task.id == task_id, Task.owner_id == current_user.id)
    )
    task = result.scalar_one_or_none()
    
    if not task:
        raise HTTPException(
            status_code=status.HTTP_404_NOT_FOUND,
            detail="任务不存在"
        )
    
    # 更新字段
    update_data = task_in.model_dump(exclude_unset=True)
    
    # 处理标签更新
    if "tag_ids" in update_data:
        tag_ids = update_data.pop("tag_ids")
        if tag_ids is not None:
            result = await db.execute(select(Tag).where(Tag.id.in_(tag_ids)))
            tags = result.scalars().all()
            task.tags = list(tags)
    
    # 处理完成状态
    if task_in.status == TaskStatus.DONE and task.status != TaskStatus.DONE:
        update_data["completed_at"] = datetime.utcnow()
    elif task_in.status and task_in.status != TaskStatus.DONE:
        update_data["completed_at"] = None
    
    # 更新其他字段
    for field, value in update_data.items():
        setattr(task, field, value)
    
    await db.commit()
    await db.refresh(task)
    
    # 清除缓存
    await redis_client.delete_pattern(f"tasks:{current_user.id}:*")
    
    return task

@router.delete("/{task_id}", status_code=status.HTTP_204_NO_CONTENT)
async def delete_task(
    task_id: int,
    db: AsyncSession = Depends(get_db),
    current_user: User = Depends(get_current_user)
):
    """删除任务"""
    result = await db.execute(
        select(Task).where(Task.id == task_id, Task.owner_id == current_user.id)
    )
    task = result.scalar_one_or_none()
    
    if not task:
        raise HTTPException(
            status_code=status.HTTP_404_NOT_FOUND,
            detail="任务不存在"
        )
    
    await db.delete(task)
    await db.commit()
    
    # 清除缓存
    await redis_client.delete_pattern(f"tasks:{current_user.id}:*")

@router.post("/{task_id}/complete", response_model=TaskResponse)
async def complete_task(
    task_id: int,
    db: AsyncSession = Depends(get_db),
    current_user: User = Depends(get_current_user)
):
    """完成任务"""
    result = await db.execute(
        select(Task).where(Task.id == task_id, Task.owner_id == current_user.id)
    )
    task = result.scalar_one_or_none()
    
    if not task:
        raise HTTPException(
            status_code=status.HTTP_404_NOT_FOUND,
            detail="任务不存在"
        )
    
    task.status = TaskStatus.DONE
    task.completed_at = datetime.utcnow()
    await db.commit()
    await db.refresh(task)
    
    return task
```

---

### Day 4：Redis 缓存与性能优化 ⭐⭐⭐

**目标**：实现 Redis 缓存和性能优化

**学习内容**：
- Redis 数据结构和命令
- 缓存策略设计
- 缓存穿透、缓存雪崩解决方案
- 异步任务队列

**产出**：完整的缓存层实现

**核心代码**：

`app/utils/redis.py` - Redis 工具
```python
import json
import redis.asyncio as redis
from typing import Optional, Any, List
from app.core.config import settings

class RedisClient:
    """Redis 客户端封装"""
    
    def __init__(self):
        self._client: Optional[redis.Redis] = None
    
    async def connect(self):
        """连接 Redis"""
        self._client = redis.Redis(
            host=settings.REDIS_HOST,
            port=settings.REDIS_PORT,
            db=settings.REDIS_DB,
            password=settings.REDIS_PASSWORD,
            decode_responses=True
        )
        # 测试连接
        await self._client.ping()
        print(f"Redis 连接成功: {settings.REDIS_HOST}:{settings.REDIS_PORT}")
    
    async def close(self):
        """关闭连接"""
        if self._client:
            await self._client.close()
    
    @property
    def client(self) -> redis.Redis:
        if self._client is None:
            raise RuntimeError("Redis 未连接")
        return self._client
    
    # 基础操作
    async def get(self, key: str) -> Optional[str]:
        """获取值"""
        return await self.client.get(key)
    
    async def set(
        self, 
        key: str, 
        value: Any, 
        expire: Optional[int] = None
    ) -> bool:
        """设置值"""
        if isinstance(value, (dict, list)):
            value = json.dumps(value, default=str)
        if expire:
            return await self.client.setex(key, expire, value)
        return await self.client.set(key, value)
    
    async def delete(self, *keys: str) -> int:
        """删除键"""
        return await self.client.delete(*keys)
    
    async def delete_pattern(self, pattern: str) -> int:
        """删除匹配的所有键"""
        count = 0
        async for key in self.client.scan_iter(match=pattern):
            await self.client.delete(key)
            count += 1
        return count
    
    async def exists(self, key: str) -> bool:
        """检查键是否存在"""
        return await self.client.exists(key) > 0
    
    async def expire(self, key: str, seconds: int) -> bool:
        """设置过期时间"""
        return await self.client.expire(key, seconds)
    
    # Hash 操作
    async def hset(self, name: str, key: str, value: Any) -> int:
        """设置 Hash 字段"""
        if isinstance(value, (dict, list)):
            value = json.dumps(value, default=str)
        return await self.client.hset(name, key, value)
    
    async def hget(self, name: str, key: str) -> Optional[str]:
        """获取 Hash 字段"""
        return await self.client.hget(name, key)
    
    async def hgetall(self, name: str) -> dict:
        """获取所有 Hash 字段"""
        return await self.client.hgetall(name)
    
    async def hdel(self, name: str, *keys: str) -> int:
        """删除 Hash 字段"""
        return await self.client.hdel(name, *keys)
    
    # List 操作
    async def lpush(self, name: str, *values: Any) -> int:
        """左侧推入"""
        values = [json.dumps(v, default=str) if isinstance(v, (dict, list)) else v for v in values]
        return await self.client.lpush(name, *values)
    
    async def rpop(self, name: str) -> Optional[str]:
        """右侧弹出"""
        return await self.client.rpop(name)
    
    async def lrange(self, name: str, start: int = 0, end: int = -1) -> List[str]:
        """获取列表范围"""
        return await self.client.lrange(name, start, end)
    
    # 计数器
    async def incr(self, key: str, amount: int = 1) -> int:
        """递增"""
        return await self.client.incrby(key, amount)
    
    async def decr(self, key: str, amount: int = 1) -> int:
        """递减"""
        return await self.client.decrby(key, amount)
    
    # 缓存装饰器
    def cached(self, expire: int = 300, key_prefix: str = ""):
        """缓存装饰器"""
        def decorator(func):
            async def wrapper(*args, **kwargs):
                # 生成缓存键
                cache_key = f"{key_prefix}:{func.__name__}:{str(args)}:{str(kwargs)}"
                
                # 尝试获取缓存
                cached_value = await self.get(cache_key)
                if cached_value:
                    return json.loads(cached_value)
                
                # 执行函数
                result = await func(*args, **kwargs)
                
                # 存入缓存
                await self.set(cache_key, result, expire)
                
                return result
            return wrapper
        return decorator

# 全局实例
redis_client = RedisClient()

# 缓存键前缀常量
class CacheKeys:
    USER = "user"
    TASK = "task"
    TASK_LIST = "tasks"
    CATEGORY = "category"
    STATS = "stats"
    
    @staticmethod
    def user(user_id: int) -> str:
        return f"user:{user_id}"
    
    @staticmethod
    def task(task_id: int) -> str:
        return f"task:{task_id}"
    
    @staticmethod
    def user_tasks(user_id: int) -> str:
        return f"tasks:{user_id}"
    
    @staticmethod
    def user_stats(user_id: int) -> str:
        return f"stats:{user_id}"
```

`app/services/task_service.py` - 任务服务（带缓存）
```python
from typing import Optional, List
from sqlalchemy.ext.asyncio import AsyncSession
from sqlalchemy import select, func
from sqlalchemy.orm import selectinload
from app.models.task import Task, TaskStatus, TaskPriority
from app.models.user import User
from app.utils.redis import redis_client, CacheKeys

class TaskService:
    """任务服务（带缓存）"""
    
    # 缓存时间（秒）
    CACHE_TTL = 300  # 5分钟
    STATS_TTL = 60   # 1分钟
    
    @staticmethod
    async def get_task_with_cache(
        db: AsyncSession,
        task_id: int,
        user_id: int
    ) -> Optional[Task]:
        """获取单个任务（带缓存）"""
        cache_key = CacheKeys.task(task_id)
        
        # 尝试从缓存获取
        cached = await redis_client.get(cache_key)
        if cached:
            import json
            return json.loads(cached)
        
        # 从数据库获取
        result = await db.execute(
            select(Task)
            .options(selectinload(Task.category), selectinload(Task.tags))
            .where(Task.id == task_id, Task.owner_id == user_id)
        )
        task = result.scalar_one_or_none()
        
        # 存入缓存
        if task:
            import json
            await redis_client.set(
                cache_key, 
                json.dumps({
                    "id": task.id,
                    "title": task.title,
                    "status": task.status.value,
                    # ... 其他字段
                }, default=str),
                TaskService.CACHE_TTL
            )
        
        return task
    
    @staticmethod
    async def get_user_stats(
        db: AsyncSession,
        user_id: int
    ) -> dict:
        """获取用户任务统计（带缓存）"""
        cache_key = CacheKeys.user_stats(user_id)
        
        # 尝试从缓存获取
        cached = await redis_client.get(cache_key)
        if cached:
            import json
            return json.loads(cached)
        
        # 从数据库统计
        total_result = await db.execute(
            select(func.count(Task.id)).where(Task.owner_id == user_id)
        )
        total = total_result.scalar()
        
        # 统计各状态数量
        status_stats = {}
        for status in TaskStatus:
            result = await db.execute(
                select(func.count(Task.id)).where(
                    Task.owner_id == user_id,
                    Task.status == status
                )
            )
            status_stats[status.value] = result.scalar()
        
        stats = {
            "total": total,
            "todo": status_stats.get("todo", 0),
            "in_progress": status_stats.get("in_progress", 0),
            "done": status_stats.get("done", 0),
            "cancelled": status_stats.get("cancelled", 0)
        }
        
        # 存入缓存
        await redis_client.set(cache_key, stats, TaskService.STATS_TTL)
        
        return stats
    
    @staticmethod
    async def invalidate_user_cache(user_id: int):
        """清除用户相关缓存"""
        await redis_client.delete_pattern(f"tasks:{user_id}:*")
        await redis_client.delete(CacheKeys.user_stats(user_id))
```

---

### Day 5：前端 React 项目搭建 ⭐⭐

**目标**：搭建 React + TypeScript 前端项目

**学习内容**：
- React 18 新特性
- TypeScript 类型系统
- React Query 数据请求
- TailwindCSS 样式

**产出**：完整可运行的前端项目

**执行命令**：

```bash
# 创建前端项目
cd ..
npx create-vite@latest litelynx-frontend --template react-ts
cd litelynx-frontend

# 安装依赖
npm install
npm install react-router-dom@6 @tanstack/react-query axios
npm install -D tailwindcss postcss autoprefixer
npx tailwindcss init -p

# 安装 UI 组件库
npm install @headlessui/react @heroicons/react date-fns clsx
```

**核心代码**：

`src/types/index.ts` - 类型定义
```typescript
// 用户相关类型
export interface User {
  id: number;
  email: string;
  username: string;
  full_name?: string;
  is_active: boolean;
  is_superuser: boolean;
  avatar_url?: string;
  bio?: string;
  created_at: string;
}

export interface LoginRequest {
  username: string;
  password: string;
}

export interface RegisterRequest {
  email: string;
  username: string;
  password: string;
  full_name?: string;
}

// 任务相关类型
export enum TaskPriority {
  LOW = 'low',
  MEDIUM = 'medium',
  HIGH = 'high',
  URGENT = 'urgent',
}

export enum TaskStatus {
  TODO = 'todo',
  IN_PROGRESS = 'in_progress',
  DONE = 'done',
  CANCELLED = 'cancelled',
}

export interface Category {
  id: number;
  name: string;
  color: string;
  icon: string;
}

export interface Tag {
  id: number;
  name: string;
  color: string;
}

export interface Task {
  id: number;
  title: string;
  description?: string;
  status: TaskStatus;
  priority: TaskPriority;
  due_date?: string;
  completed_at?: string;
  estimated_minutes?: number;
  actual_minutes?: number;
  owner_id: number;
  category_id?: number;
  category?: Category;
  tags: Tag[];
  created_at: string;
  updated_at?: string;
}

export interface TaskCreate {
  title: string;
  description?: string;
  priority?: TaskPriority;
  due_date?: string;
  estimated_minutes?: number;
  category_id?: number;
  tag_ids?: number[];
}

export interface TaskUpdate {
  title?: string;
  description?: string;
  status?: TaskStatus;
  priority?: TaskPriority;
  due_date?: string;
  completed_at?: string;
  estimated_minutes?: number;
  actual_minutes?: number;
  category_id?: number;
  tag_ids?: number[];
}

export interface TaskListResponse {
  tasks: Task[];
  total: number;
  page: number;
  page_size: number;
}

export interface UserStats {
  total: number;
  todo: number;
  in_progress: number;
  done: number;
  cancelled: number;
}

// API 响应类型
export interface ApiResponse<T> {
  data: T;
  message?: string;
}

export interface AuthResponse {
  access_token: string;
  refresh_token: string;
  token_type: string;
}
```

`src/api/client.ts` - API 客户端
```typescript
import axios from 'axios';

const API_BASE_URL = import.meta.env.VITE_API_URL || 'http://localhost:8000/api/v1';

export const apiClient = axios.create({
  baseURL: API_BASE_URL,
  timeout: 10000,
  headers: {
    'Content-Type': 'application/json',
  },
});

// 请求拦截器：添加 Token
apiClient.interceptors.request.use(
  (config) => {
    const token = localStorage.getItem('access_token');
    if (token) {
      config.headers.Authorization = `Bearer ${token}`;
    }
    return config;
  },
  (error) => Promise.reject(error)
);

// 响应拦截器：处理 Token 过期
apiClient.interceptors.response.use(
  (response) => response,
  async (error) => {
    const originalRequest = error.config;
    
    if (error.response?.status === 401 && !originalRequest._retry) {
      originalRequest._retry = true;
      
      const refreshToken = localStorage.getItem('refresh_token');
      if (refreshToken) {
        try {
          const response = await axios.post(`${API_BASE_URL}/auth/refresh`, {
            refresh_token: refreshToken,
          });
          
          const { access_token, refresh_token } = response.data;
          localStorage.setItem('access_token', access_token);
          localStorage.setItem('refresh_token', refresh_token);
          
          originalRequest.headers.Authorization = `Bearer ${access_token}`;
          return apiClient(originalRequest);
        } catch (refreshError) {
          // 刷新失败，清除 Token
          localStorage.removeItem('access_token');
          localStorage.removeItem('refresh_token');
          window.location.href = '/login';
        }
      }
    }
    
    return Promise.reject(error);
  }
);

// Auth API
export const authApi = {
  login: (data: { username: string; password: string }) =>
    apiClient.post('/auth/login', data),
  
  register: (data: { email: string; username: string; password: string }) =>
    apiClient.post('/auth/register', data),
  
  refresh: (refreshToken: string) =>
    apiClient.post('/auth/refresh', { refresh_token: refreshToken }),
  
  getMe: () => apiClient.get('/auth/me'),
};

// Task API
export const taskApi = {
  list: (params?: {
    page?: number;
    page_size?: number;
    status?: string;
    priority?: string;
    category_id?: number;
    search?: string;
  }) => apiClient.get('/tasks/', { params }),
  
  get: (id: number) => apiClient.get(`/tasks/${id}`),
  
  create: (data: any) => apiClient.post('/tasks/', data),
  
  update: (id: number, data: any) => apiClient.put(`/tasks/${id}`, data),
  
  delete: (id: number) => apiClient.delete(`/tasks/${id}`),
  
  complete: (id: number) => apiClient.post(`/tasks/${id}/complete`),
  
  stats: () => apiClient.get('/tasks/stats'),
};

// Category API
export const categoryApi = {
  list: () => apiClient.get('/categories/'),
  
  create: (data: { name: string; color?: string; icon?: string }) =>
    apiClient.post('/categories/', data),
  
  update: (id: number, data: any) => apiClient.put(`/categories/${id}`, data),
  
  delete: (id: number) => apiClient.delete(`/categories/${id}`),
};
```

`src/hooks/useAuth.ts` - 认证 Hook
```typescript
import { useMutation, useQuery, useQueryClient } from '@tanstack/react-query';
import { useNavigate } from 'react-router-dom';
import { authApi, LoginRequest, RegisterRequest } from '../api/client';
import { User } from '../types';

export function useLogin() {
  const navigate = useNavigate();
  const queryClient = useQueryClient();
  
  return useMutation({
    mutationFn: (data: LoginRequest) => authApi.login(data),
    onSuccess: (response) => {
      const { access_token, refresh_token } = response.data;
      localStorage.setItem('access_token', access_token);
      localStorage.setItem('refresh_token', refresh_token);
      queryClient.invalidateQueries({ queryKey: ['currentUser'] });
      navigate('/');
    },
  });
}

export function useRegister() {
  const navigate = useNavigate();
  
  return useMutation({
    mutationFn: (data: RegisterRequest) => authApi.register(data),
    onSuccess: () => {
      navigate('/login');
    },
  });
}

export function useLogout() {
  const navigate = useNavigate();
  const queryClient = useQueryClient();
  
  return () => {
    localStorage.removeItem('access_token');
    localStorage.removeItem('refresh_token');
    queryClient.clear();
    navigate('/login');
  };
}

export function useCurrentUser() {
  return useQuery({
    queryKey: ['currentUser'],
    queryFn: async (): Promise<User | null> => {
      const token = localStorage.getItem('access_token');
      if (!token) return null;
      
      try {
        const response = await authApi.getMe();
        return response.data;
      } catch {
        localStorage.removeItem('access_token');
        localStorage.removeItem('refresh_token');
        return null;
      }
    },
    retry: false,
    staleTime: 5 * 60 * 1000, // 5 分钟
  });
}
```

`src/pages/LoginPage.tsx` - 登录页面
```typescript
import { useState } from 'react';
import { Link, useNavigate } from 'react-router-dom';
import { useLogin } from '../hooks/useAuth';

export default function LoginPage() {
  const [username, setUsername] = useState('');
  const [password, setPassword] = useState('');
  const [error, setError] = useState('');
  
  const navigate = useNavigate();
  const loginMutation = useLogin();
  
  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();
    setError('');
    
    try {
      await loginMutation.mutateAsync({ username, password });
    } catch (err: any) {
      setError(err.response?.data?.detail || '登录失败');
    }
  };
  
  return (
    <div className="min-h-screen flex items-center justify-center bg-gray-100">
      <div className="bg-white p-8 rounded-lg shadow-md w-full max-w-md">
        <h1 className="text-2xl font-bold text-center mb-6">LiteLynx 登录</h1>
        
        {error && (
          <div className="bg-red-100 border border-red-400 text-red-700 px-4 py-3 rounded mb-4">
            {error}
          </div>
        )}
        
        <form onSubmit={handleSubmit} className="space-y-4">
          <div>
            <label className="block text-sm font-medium text-gray-700">用户名</label>
            <input
              type="text"
              value={username}
              onChange={(e) => setUsername(e.target.value)}
              className="mt-1 block w-full px-3 py-2 border border-gray-300 rounded-md shadow-sm focus:outline-none focus:ring-blue-500 focus:border-blue-500"
              required
            />
          </div>
          
          <div>
            <label className="block text-sm font-medium text-gray-700">密码</label>
            <input
              type="password"
              value={password}
              onChange={(e) => setPassword(e.target.value)}
              className="mt-1 block w-full px-3 py-2 border border-gray-300 rounded-md shadow-sm focus:outline-none focus:ring-blue-500 focus:border-blue-500"
              required
            />
          </div>
          
          <button
            type="submit"
            disabled={loginMutation.isPending}
            className="w-full bg-blue-600 text-white py-2 px-4 rounded-md hover:bg-blue-700 disabled:opacity-50 disabled:cursor-not-allowed transition-colors"
          >
            {loginMutation.isPending ? '登录中...' : '登录'}
          </button>
        </form>
        
        <p className="mt-4 text-center text-sm text-gray-600">
          还没有账号？{' '}
          <Link to="/register" className="text-blue-600 hover:underline">
            注册
          </Link>
        </p>
      </div>
    </div>
  );
}
```

`src/pages/DashboardPage.tsx` - 仪表盘页面
```typescript
import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query';
import { taskApi, categoryApi } from '../api/client';
import { Task, TaskStatus, TaskPriority } from '../types';
import { format } from 'date-fns';

export default function DashboardPage() {
  const queryClient = useQueryClient();
  
  // 获取任务列表
  const { data: taskData, isLoading } = useQuery({
    queryKey: ['tasks'],
    queryFn: async () => {
      const response = await taskApi.list({ page_size: 50 });
      return response.data;
    },
  });
  
  // 获取分类
  const { data: categories } = useQuery({
    queryKey: ['categories'],
    queryFn: async () => {
      const response = await categoryApi.list();
      return response.data;
    },
  });
  
  // 完成任务
  const completeMutation = useMutation({
    mutationFn: (id: number) => taskApi.complete(id),
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['tasks'] });
    },
  });
  
  // 删除任务
  const deleteMutation = useMutation({
    mutationFn: (id: number) => taskApi.delete(id),
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['tasks'] });
    },
  });
  
  const getPriorityColor = (priority: TaskPriority) => {
    const colors = {
      [TaskPriority.LOW]: 'bg-gray-100 text-gray-600',
      [TaskPriority.MEDIUM]: 'bg-blue-100 text-blue-600',
      [TaskPriority.HIGH]: 'bg-orange-100 text-orange-600',
      [TaskPriority.URGENT]: 'bg-red-100 text-red-600',
    };
    return colors[priority];
  };
  
  const getStatusColor = (status: TaskStatus) => {
    const colors = {
      [TaskStatus.TODO]: 'bg-gray-100 text-gray-600',
      [TaskStatus.IN_PROGRESS]: 'bg-blue-100 text-blue-600',
      [TaskStatus.DONE]: 'bg-green-100 text-green-600',
      [TaskStatus.CANCELLED]: 'bg-red-100 text-red-600',
    };
    return colors[status];
  };
  
  if (isLoading) {
    return (
      <div className="flex items-center justify-center min-h-screen">
        <div className="text-lg">加载中...</div>
      </div>
    );
  }
  
  return (
    <div className="min-h-screen bg-gray-100">
      {/* 头部 */}
      <header className="bg-white shadow">
        <div className="max-w-7xl mx-auto py-6 px-4 sm:px-6 lg:px-8">
          <div className="flex justify-between items-center">
            <h1 className="text-3xl font-bold text-gray-900">LiteLynx 任务管理</h1>
            <button
              onClick={() => {/* 退出登录 */}}
              className="text-gray-600 hover:text-gray-900"
            >
              退出登录
            </button>
          </div>
        </div>
      </header>
      
      {/* 统计卡片 */}
      <main className="max-w-7xl mx-auto py-6 sm:px-6 lg:px-8">
        <div className="grid grid-cols-1 md:grid-cols-4 gap-4 mb-8">
          <StatCard title="总任务" value={taskData?.total || 0} color="blue" />
          <StatCard 
            title="待办" 
            value={taskData?.tasks.filter(t => t.status === TaskStatus.TODO).length || 0} 
            color="gray" 
          />
          <StatCard 
            title="进行中" 
            value={taskData?.tasks.filter(t => t.status === TaskStatus.IN_PROGRESS).length || 0} 
            color="yellow" 
          />
          <StatCard 
            title="已完成" 
            value={taskData?.tasks.filter(t => t.status === TaskStatus.DONE).length || 0} 
            color="green" 
          />
        </div>
        
        {/* 任务列表 */}
        <div className="bg-white shadow rounded-lg overflow-hidden">
          <div className="px-4 py-5 sm:px-6 flex justify-between items-center">
            <h2 className="text-lg font-medium text-gray-900">任务列表</h2>
            <button className="bg-blue-600 text-white px-4 py-2 rounded-md hover:bg-blue-700">
              新建任务
            </button>
          </div>
          
          <div className="divide-y divide-gray-200">
            {taskData?.tasks.map((task: Task) => (
              <div key={task.id} className="p-4 hover:bg-gray-50 transition-colors">
                <div className="flex items-center justify-between">
                  <div className="flex-1">
                    <div className="flex items-center gap-2">
                      <h3 className="text-sm font-medium text-gray-900">{task.title}</h3>
                      <span className={`px-2 py-0.5 text-xs rounded-full ${getPriorityColor(task.priority)}`}>
                        {task.priority}
                      </span>
                      <span className={`px-2 py-0.5 text-xs rounded-full ${getStatusColor(task.status)}`}>
                        {task.status}
                      </span>
                    </div>
                    {task.description && (
                      <p className="text-sm text-gray-500 mt-1">{task.description}</p>
                    )}
                    <div className="flex items-center gap-4 mt-2 text-xs text-gray-400">
                      {task.due_date && (
                        <span>截止: {format(new Date(task.due_date), 'yyyy-MM-dd')}</span>
                      )}
                      {task.category && (
                        <span style={{ color: task.category.color }}>{task.category.name}</span>
                      )}
                    </div>
                  </div>
                  <div className="flex items-center gap-2">
                    {task.status !== TaskStatus.DONE && (
                      <button
                        onClick={() => completeMutation.mutate(task.id)}
                        className="text-green-600 hover:text-green-800 text-sm"
                      >
                        完成
                      </button>
                    )}
                    <button
                      onClick={() => deleteMutation.mutate(task.id)}
                      className="text-red-600 hover:text-red-800 text-sm"
                    >
                      删除
                    </button>
                  </div>
                </div>
              </div>
            ))}
            
            {taskData?.tasks.length === 0 && (
              <div className="p-8 text-center text-gray-500">
                暂无任务，点击「新建任务」开始
              </div>
            )}
          </div>
        </div>
      </main>
    </div>
  );
}

function StatCard({ title, value, color }: { title: string; value: number; color: string }) {
  const colorMap: Record<string, string> = {
    blue: 'bg-blue-500',
    gray: 'bg-gray-500',
    yellow: 'bg-yellow-500',
    green: 'bg-green-500',
  };
  
  return (
    <div className="bg-white rounded-lg shadow p-4">
      <div className="flex items-center">
        <div className={`w-2 h-2 rounded-full ${colorMap[color]} mr-2`} />
        <span className="text-sm text-gray-500">{title}</span>
      </div>
      <p className="text-2xl font-bold mt-2">{value}</p>
    </div>
  );
}
```

---

### Day 6：Docker 部署与项目整合 ⭐⭐⭐

**目标**：完成 Docker 部署配置

**学习内容**：
- Docker 容器化
- Docker Compose 多服务编排
- 环境变量管理
- 生产环境配置

**产出**：完整的 Docker 部署方案

**Docker 配置文件**：

`docker-compose.yml` - Docker Compose 配置
```yaml
version: '3.8'

services:
  # PostgreSQL 数据库
  postgres:
    image: postgres:15-alpine
    container_name: litelynx-postgres
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: postgres123
      POSTGRES_DB: litelynx
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 10s
      timeout: 5s
      retries: 5
    networks:
      - litelynx-network

  # Redis 缓存
  redis:
    image: redis:7-alpine
    container_name: litelynx-redis
    ports:
      - "6379:6379"
    volumes:
      - redis_data:/data
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 10s
      timeout: 5s
      retries: 5
    networks:
      - litelynx-network

  # 后端 API
  backend:
    build:
      context: ./backend
      dockerfile: Dockerfile
    container_name: litelynx-backend
    environment:
      - DATABASE_URL=postgresql+asyncpg://postgres:postgres123@postgres:5432/litelynx
      - REDIS_HOST=redis
      - REDIS_PORT=6379
      - SECRET_KEY=your-super-secret-key-production
    ports:
      - "8000:8000"
    depends_on:
      postgres:
        condition: service_healthy
      redis:
        condition: service_healthy
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8000/health"]
      interval: 30s
      timeout: 10s
      retries: 3
    networks:
      - litelynx-network

  # 前端
  frontend:
    build:
      context: ./frontend
      dockerfile: Dockerfile
    container_name: litelynx-frontend
    ports:
      - "3000:80"
    depends_on:
      - backend
    networks:
      - litelynx-network

volumes:
  postgres_data:
  redis_data:

networks:
  litelynx-network:
    driver: bridge
```

`backend/Dockerfile` - 后端 Dockerfile
```dockerfile
# Python 运行时
FROM python:3.11-slim

# 设置工作目录
WORKDIR /app

# 安装系统依赖
RUN apt-get update && apt-get install -y \
    gcc \
    postgresql-client \
    curl \
    && rm -rf /var/lib/apt/lists/*

# 复制依赖文件
COPY requirements.txt .

# 安装 Python 依赖
RUN pip install --no-cache-dir -r requirements.txt

# 复制应用代码
COPY . .

# 创建非 root 用户
RUN adduser --disabled-password --gecos '' appuser && \
    chown -R appuser:appuser /app
USER appuser

# 暴露端口
EXPOSE 8000

# 健康检查
HEALTHCHECK --interval=30s --timeout=10s --start-period=5s --retries=3 \
    CMD curl -f http://localhost:8000/health || exit 1

# 启动命令
CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8000"]
```

`backend/.dockerignore`
```
__pycache__
*.pyc
*.pyo
*.pyd
.Python
env/
venv/
.venv/
*.egg-info/
dist/
build/
.git/
.gitignore
.coverage
htmlcov/
.pytest_cache/
alembic/versions/*.py
!alembic/versions/*.py.example
.env
.env.local
docker-compose.yml
Dockerfile
.dockerignore
README.md
```

---

### Day 7：项目总结与面试准备 ⭐⭐⭐

**目标**：总结项目亮点，准备面试

**学习内容**：
- 项目亮点总结
- 面试问题准备
- 技术原理讲解

---

## 📝 面试要点

### Q1：FastAPI 相比 Flask 有哪些优势？

**参考答案**：
```
1. 性能：FastAPI 基于 Starlette，异步支持更好，性能比 Flask 高 2-3 倍
2. 类型安全：基于 Pydantic 的自动数据验证，减少运行时错误
3. 自动文档：自动生成 OpenAPI/Swagger 文档
4. 现代化：支持类型提示、依赖注入、异步编程
5. 生态：与 SQLAlchemy、Pydantic 等现代库集成更好
```

### Q2：为什么要使用 Redis 缓存？

**参考答案**：
```
1. 性能提升：内存访问速度比数据库快 10-100 倍
2. 减轻数据库压力：热点数据直接从缓存返回
3. 支持高并发：单节点 Redis 可支持 10万+ QPS
4. 丰富的数据结构：String、Hash、List、Set 等满足不同场景

缓存策略：
- Cache-Aside：读时先查缓存，未命中查数据库并写入缓存
- Write-Through：写时同时更新缓存和数据库
- TTL 过期：设置合理的过期时间，避免数据不一致
```

### Q3：SQLAlchemy 异步和同步的区别？

**参考答案**：
```
1. 同步模式：每个请求占用一个连接，连接耗尽会阻塞
2. 异步模式：使用 async/await，不阻塞等待 IO，提高并发

代码区别：
# 同步
session.query(User).all()

# 异步
await session.execute(select(User))
```

### Q4：JWT 认证的原理？

**参考答案**：
```
JWT 由三部分组成：Header.Payload.Signature

1. 用户登录后，服务端生成 Token 返回
2. 客户端请求时在 Header 携带 Token
3. 服务端验证 Token 签名和过期时间
4. 验证通过后获取用户信息

安全问题：
- Access Token：短期有效（15-30分钟）
- Refresh Token：长期有效（7天），用于续期
- Token 黑名单：支持手动失效
```

### Q5：如何保证接口安全性？

**参考答案**：
```
1. 认证授权：JWT Token + 权限验证
2. 参数校验：Pydantic 自动验证输入
3. SQL 注入防护：ORM 参数化查询
4. XSS 防护：输入转义 + CSP
5. CORS 配置：限制跨域请求
6. 限流：Redis 计数器防止滥用
7. 日志审计：记录敏感操作
```

### Q6：Docker 在项目中的作用？

**参考答案**：
```
1. 环境一致性：开发、测试、生产环境完全一致
2. 快速部署：一条命令启动所有服务
3. 资源隔离：服务之间相互隔离
4. 弹性伸缩：支持水平扩展
5. CI/CD 集成：便于自动化部署

Docker Compose 优势：
- 一键启动多容器应用
- 服务依赖自动等待
- 统一网络配置
- 便捷的日志查看
```

---

## ✅ 项目清单

### 目录结构
```
LiteLynx/
├── backend/                    # 后端项目
│   ├── app/
│   │   ├── api/
│   │   │   └── v1/
│   │   │       ├── auth.py     # 认证接口
│   │   │       ├── tasks.py    # 任务接口
│   │   │       └── deps.py     # 依赖注入
│   │   ├── core/
│   │   │   └── config.py       # 配置文件
│   │   ├── db/
│   │   │   └── base.py         # 数据库连接
│   │   ├── models/
│   │   │   ├── user.py         # 用户模型
│   │   │   ├── task.py         # 任务模型
│   │   │   ├── category.py     # 分类模型
│   │   │   └── tag.py          # 标签模型
│   │   ├── schemas/
│   │   │   ├── user.py         # 用户 Schema
│   │   │   ├── task.py         # 任务 Schema
│   │   │   └── auth.py         # 认证 Schema
│   │   ├── services/
│   │   │   └── task_service.py # 业务逻辑
│   │   └── utils/
│   │       ├── security.py     # 安全工具
│   │       └── redis.py        # Redis 工具
│   ├── alembic/                # 数据库迁移
│   ├── tests/                  # 测试
│   ├── requirements.txt
│   ├── Dockerfile
│   └── main.py                 # 应用入口
│
├── frontend/                   # 前端项目
│   ├── src/
│   │   ├── api/
│   │   │   └── client.ts       # API 客户端
│   │   ├── hooks/
│   │   │   └── useAuth.ts      # 认证 Hook
│   │   ├── pages/
│   │   │   ├── LoginPage.tsx    # 登录页
│   │   │   └── DashboardPage.tsx # 仪表盘
│   │   ├── types/
│   │   │   └── index.ts        # 类型定义
│   │   ├── App.tsx
│   │   └── main.tsx
│   ├── package.json
│   └── Dockerfile
│
└── docker-compose.yml          # Docker 编排
```

### 快速启动命令
```bash
# 1. 启动所有服务
docker-compose up -d

# 2. 初始化数据库
docker-compose exec backend alembic upgrade head

# 3. 访问应用
# 前端：http://localhost:3000
# 后端 API：http://localhost:8000
# API 文档：http://localhost:8000/docs
```

---

**文档版本**：v1.0  
**作者**：LiteLynx Team  
**更新日期**：2024
