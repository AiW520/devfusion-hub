# DataVault 数据分析完整项目指南

> 基于 Python + Pandas + FastAPI + MySQL 的数据分析平台

---

## 📋 项目介绍

### 项目概述
DataVault 是一个数据分析平台，支持数据导入、处理、分析、可视化和报告生成。项目展示了数据分析开发的核心技能，包括数据处理、统计分析、图表生成、API 服务等。

- **数据处理**：Pandas 强大的数据处理能力
- **统计分析**：描述统计、相关性分析、趋势分析
- **可视化**：ECharts 丰富的图表支持
- **面试亮点**：数据分析和工程化能力

### 核心技术亮点
```
后端：Python + FastAPI + Pandas + NumPy
数据库：MySQL + SQLAlchemy
前端：Vue 3 + ECharts
数据处理：Pandas、NumPy、SciPy
统计：statsmodels、scikit-learn（基础）
报告生成：ReportLab、Jinja2
```

---

## 🏗️ 技术架构图

### 系统架构

```
┌─────────────────────────────────────────────────────────────────┐
│                         客户端层                                 │
│  ┌─────────────────────────────────────────────────────────────┐ │
│  │                    Vue 3 Frontend                            │ │
│  │  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐    │ │
│  │  │  数据    │  │  分析    │  │  可视化  │  │  报告    │    │ │
│  │  │  上传    │  │  看板    │  │  图表    │  │  生成    │    │ │
│  │  └──────────┘  └──────────┘  └──────────┘  └──────────┘    │ │
│  └─────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
                           │ REST API
┌──────────────────────────┼────────────────────────────────────┐
│                     FastAPI Backend                             │
│  ┌─────────────────────────────────────────────────────────────┐ │
│  │  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐    │ │
│  │  │  数据    │  │  分析    │  │  统计    │  │  报告    │    │ │
│  │  │  导入    │  │  处理    │  │  计算    │  │  生成    │    │ │
│  │  └──────────┘  └──────────┘  └──────────┘  └──────────┘    │ │
│  └─────────────────────────────────────────────────────────────┘ │
│  ┌─────────────────────────────────────────────────────────────┐ │
│  │              Pandas / NumPy / SciPy                          │ │
│  └─────────────────────────────────────────────────────────────┘ │
└──────────────────────────┼─────────────────────────────────────┘
                           │
          ┌────────────────┴────────────────┐
          │                                 │
┌─────────┴───────┐               ┌─────────┴───────┐
│     MySQL        │               │   文件存储      │
│                  │               │                 │
│  Tables:         │               │  上传文件:       │
│  - datasets     │               │  - CSV/Excel    │
│  - analyses     │               │  - 原始数据      │
│  - reports      │               │  - 缓存文件      │
│  - charts       │               │                 │
└─────────────────┘               └─────────────────┘
```

### 数据处理流程

```
┌─────────────────────────────────────────────────────────────────┐
│                      数据处理流程 (ETL)                          │
│                                                                  │
│  ┌─────────┐    ┌─────────┐    ┌─────────┐    ┌─────────┐     │
│  │ Extract │───>│ Transform│───>│  Load   │───>│  Analyze│     │
│  │ (提取)   │    │  (转换)  │    │  (加载)  │    │  (分析)  │     │
│  └─────────┘    └─────────┘    └─────────┘    └─────────┘     │
│                                                                  │
│  数据源:          清洗处理:         存储:           分析:        │
│  - CSV           - 缺失值处理     - MySQL        - 统计        │
│  - Excel         - 类型转换       - 文件系统      - 趋势        │
│  - Database      - 异常值检测     - 缓存          - 预测        │
│  - API           - 标准化                           - 关联      │
└─────────────────────────────────────────────────────────────────┘
```

---

## 📅 学习计划（7天）

### Day 1：Python 数据处理基础 ⭐

**目标**：掌握 Pandas 和 NumPy 基础

**学习内容**：
- NumPy 数组操作
- Pandas DataFrame
- 数据读写
- 数据清洗基础

**执行命令**：

```bash
# 1. 创建项目
mkdir -p datavault && cd datavault

# 2. 创建虚拟环境
python -m venv venv
source venv/bin/activate  # Windows: venv\Scripts\activate

# 3. 安装依赖
pip install pandas numpy sqlalchemy pymysql fastapi uvicorn
pip install openpyxl xlrd python-multipart
pip install python-jose passlib bcrypt
pip install plotly kaleido  # 图表生成
pip install scipy scikit-learn  # 统计分析

# 4. 创建项目结构
mkdir -p app/{api,core,models,schemas,services,utils}
mkdir -p data/{uploads,cache,exports}
mkdir -p tests
touch app/__init__.py
```

---

### Day 2：数据模型与服务层 ⭐⭐

**目标**：实现数据模型和服务层

**核心代码**：

`datavault/app/core/config.py` - 配置
```python
from pydantic_settings import BaseSettings
from typing import Optional
import os

class Settings(BaseSettings):
    """应用配置"""
    
    APP_NAME: str = "DataVault"
    DEBUG: bool = True
    API_PREFIX: str = "/api/v1"
    
    # 数据库
    DATABASE_URL: str = "mysql+pymysql://root:password@localhost:3306/datavault"
    
    # 文件存储
    UPLOAD_DIR: str = "./data/uploads"
    CACHE_DIR: str = "./data/cache"
    EXPORT_DIR: str = "./data/exports"
    MAX_FILE_SIZE: int = 100 * 1024 * 1024  # 100MB
    
    # JWT
    SECRET_KEY: str = "change-me-in-production"
    ALGORITHM: str = "HS256"
    ACCESS_TOKEN_EXPIRE_MINUTES: int = 30
    
    # 分析配置
    DEFAULT_CHUNK_SIZE: int = 10000
    MAX_ROWS_DISPLAY: int = 1000
    CACHE_TTL: int = 3600
    
    class Config:
        env_file = ".env"
        case_sensitive = True

settings = Settings()

# 确保目录存在
os.makedirs(settings.UPLOAD_DIR, exist_ok=True)
os.makedirs(settings.CACHE_DIR, exist_ok=True)
os.makedirs(settings.EXPORT_DIR, exist_ok=True)
```

`datavault/app/models/dataset.py` - 数据集模型
```python
from sqlalchemy import Column, Integer, String, Text, DateTime, Boolean, BigInteger, JSON
from sqlalchemy.sql import func
from app.db.base import Base

class Dataset(Base):
    """数据集模型"""
    __tablename__ = "datasets"
    
    id = Column(Integer, primary_key=True, index=True)
    name = Column(String(255), nullable=False, index=True)
    description = Column(Text)
    
    # 文件信息
    filename = Column(String(500), nullable=False)
    file_type = Column(String(50), nullable=False)  # csv, excel, json
    file_size = Column(BigInteger)
    file_path = Column(String(1000), nullable=False)
    
    # 数据统计
    row_count = Column(Integer)
    column_count = Column(Integer)
    columns_info = Column(JSON)  # {'col1': {'type': 'int64', 'nulls': 0}, ...}
    
    # 状态
    is_processed = Column(Boolean, default=False)
    is_deleted = Column(Boolean, default=False)
    
    # 元数据
    created_by = Column(Integer, index=True)
    created_at = Column(DateTime(timezone=True), server_default=func.now())
    updated_at = Column(DateTime(timezone=True), onupdate=func.now())
    
    def __repr__(self):
        return f"<Dataset {self.name}>"


class Analysis(Base):
    """分析记录模型"""
    __tablename__ = "analyses"
    
    id = Column(Integer, primary_key=True, index=True)
    name = Column(String(255), nullable=False)
    description = Column(Text)
    
    # 关联数据集
    dataset_id = Column(Integer, index=True)
    
    # 分析配置
    analysis_type = Column(String(100), nullable=False)  # descriptive, correlation, trend, etc.
    config = Column(JSON)  # 分析参数
    
    # 分析结果
    result = Column(JSON)  # 分析结果数据
    summary = Column(Text)  # 摘要
    
    # 状态
    status = Column(String(50), default="pending")  # pending, running, completed, failed
    
    created_by = Column(Integer, index=True)
    created_at = Column(DateTime(timezone=True), server_default=func.now())
    updated_at = Column(DateTime(timezone=True), onupdate=func.now())
    
    def __repr__(self):
        return f"<Analysis {self.name}>"


class Chart(Base):
    """图表配置模型"""
    __tablename__ = "charts"
    
    id = Column(Integer, primary_key=True, index=True)
    name = Column(String(255), nullable=False)
    chart_type = Column(String(50), nullable=False)  # bar, line, pie, scatter, etc.
    
    # 关联
    dataset_id = Column(Integer, index=True)
    analysis_id = Column(Integer, index=True, nullable=True)
    
    # 配置
    config = Column(JSON)  # ECharts 配置
    data = Column(JSON)  # 图表数据
    
    created_by = Column(Integer, index=True)
    created_at = Column(DateTime(timezone=True), server_default=func.now())
    updated_at = Column(DateTime(timezone=True), onupdate=func.now())
    
    def __repr__(self):
        return f"<Chart {self.name}>"
```

`datavault/app/db/base.py` - 数据库基类
```python
from sqlalchemy.ext.asyncio import create_async_engine, AsyncSession, async_sessionmaker
from sqlalchemy.orm import declarative_base
from app.core.config import settings

engine = create_async_engine(
    settings.DATABASE_URL,
    echo=settings.DEBUG,
    pool_pre_ping=True,
    pool_size=10,
    max_overflow=20
)

AsyncSessionLocal = async_sessionmaker(
    engine,
    class_=AsyncSession,
    expire_on_commit=False
)

Base = declarative_base()

async def get_db():
    async with AsyncSessionLocal() as session:
        try:
            yield session
            await session.commit()
        except Exception:
            await session.rollback()
            raise
        finally:
            await session.close()

async def init_db():
    async with engine.begin() as conn:
        await conn.run_sync(Base.metadata.create_all)
```

---

### Day 3：数据处理服务 ⭐⭐⭐

**目标**：实现核心数据处理服务

**核心代码**：

`datavault/app/services/data_processor.py` - 数据处理服务
```python
import pandas as pd
import numpy as np
from typing import List, Dict, Any, Optional, Tuple
from pathlib import Path
import json
import os
from datetime import datetime

class DataProcessor:
    """数据处理服务"""
    
    def __init__(self, file_path: str):
        self.file_path = file_path
        self.df: Optional[pd.DataFrame] = None
        self._load_data()
    
    def _load_data(self):
        """加载数据"""
        file_ext = Path(self.file_path).suffix.lower()
        
        if file_ext == '.csv':
            self.df = pd.read_csv(self.file_path)
        elif file_ext in ['.xlsx', '.xls']:
            self.df = pd.read_excel(self.file_path)
        elif file_ext == '.json':
            self.df = pd.read_json(self.file_path)
        else:
            raise ValueError(f"不支持的文件类型: {file_ext}")
        
        # 自动清理列名
        self.df.columns = self.df.columns.str.strip()
    
    def get_info(self) -> Dict[str, Any]:
        """获取数据集信息"""
        if self.df is None:
            return {}
        
        columns_info = {}
        for col in self.df.columns:
            dtype = str(self.df[col].dtype)
            columns_info[col] = {
                'dtype': dtype,
                'nulls': int(self.df[col].isnull().sum()),
                'unique': int(self.df[col].nunique()),
                'sample': self.df[col].head(3).tolist()
            }
            
            # 数值型额外信息
            if pd.api.types.is_numeric_dtype(self.df[col]):
                columns_info[col].update({
                    'min': float(self.df[col].min()) if not pd.isna(self.df[col].min()) else None,
                    'max': float(self.df[col].max()) if not pd.isna(self.df[col].max()) else None,
                    'mean': float(self.df[col].mean()) if not pd.isna(self.df[col].mean()) else None,
                    'median': float(self.df[col].median()) if not pd.isna(self.df[col].median()) else None,
                })
        
        return {
            'row_count': len(self.df),
            'column_count': len(self.df.columns),
            'columns': list(self.df.columns),
            'columns_info': columns_info,
            'dtypes': {col: str(dtype) for col, dtype in self.df.dtypes.items()}
        }
    
    def get_sample(self, n: int = 100, random: bool = False) -> List[Dict]:
        """获取样本数据"""
        if self.df is None:
            return []
        
        if random:
            sample = self.df.sample(min(n, len(self.df)))
        else:
            sample = self.df.head(n)
        
        return sample.to_dict('records')
    
    def clean_data(self, operations: Dict[str, Any]) -> Dict[str, Any]:
        """数据清洗"""
        if self.df is None:
            return {'success': False, 'error': '数据未加载'}
        
        results = {}
        
        # 处理缺失值
        if 'fillna' in operations:
            fillna_config = operations['fillna']
            for col, value in fillna_config.items():
                if col in self.df.columns:
                    if value == 'mean' and pd.api.types.is_numeric_dtype(self.df[col]):
                        self.df[col].fillna(self.df[col].mean(), inplace=True)
                    elif value == 'median' and pd.api.types.is_numeric_dtype(self.df[col]):
                        self.df[col].fillna(self.df[col].median(), inplace=True)
                    elif value == 'mode':
                        self.df[col].fillna(self.df[col].mode()[0] if len(self.df[col].mode()) > 0 else value, inplace=True)
                    else:
                        self.df[col].fillna(value, inplace=True)
            results['fillna'] = f"已填充 {len(fillna_config)} 列"
        
        # 删除缺失值
        if 'dropna' in operations:
            dropna_config = operations['dropna']
            if dropna_config.get('all', False):
                before = len(self.df)
                self.df.dropna(how='all', inplace=True)
                results['dropna'] = f"删除了 {before - len(self.df)} 行（全为空）"
            elif 'subset' in dropna_config:
                before = len(self.df)
                self.df.dropna(subset=dropna_config['subset'], inplace=True)
                results['dropna'] = f"删除了 {before - len(self.df)} 行"
        
        # 删除重复
        if operations.get('drop_duplicates', False):
            before = len(self.df)
            self.df.drop_duplicates(inplace=True)
            results['drop_duplicates'] = f"删除了 {before - len(self.df)} 行重复数据"
        
        # 类型转换
        if 'astype' in operations:
            for col, dtype in operations['astype'].items():
                if col in self.df.columns:
                    try:
                        self.df[col] = self.df[col].astype(dtype)
                        results[f'astype_{col}'] = f"转换为 {dtype}"
                    except Exception as e:
                        results[f'astype_{col}'] = f"转换失败: {str(e)}"
        
        return results
    
    def filter_data(
        self,
        conditions: List[Dict[str, Any]]
    ) -> 'DataProcessor':
        """数据筛选"""
        if self.df is None:
            return self
        
        mask = pd.Series([True] * len(self.df))
        
        for condition in conditions:
            col = condition['column']
            op = condition['operator']
            value = condition['value']
            
            if col not in self.df.columns:
                continue
            
            if op == '==':
                mask &= (self.df[col] == value)
            elif op == '!=':
                mask &= (self.df[col] != value)
            elif op == '>':
                mask &= (self.df[col] > value)
            elif op == '>=':
                mask &= (self.df[col] >= value)
            elif op == '<':
                mask &= (self.df[col] < value)
            elif op == '<=':
                mask &= (self.df[col] <= value)
            elif op == 'contains':
                mask &= self.df[col].astype(str).str.contains(value, na=False)
            elif op == 'in':
                mask &= self.df[col].isin(value)
        
        self.df = self.df[mask]
        return self
    
    def aggregate_data(
        self,
        group_by: List[str],
        agg_config: Dict[str, List[str]]
    ) -> pd.DataFrame:
        """数据聚合"""
        if self.df is None:
            return pd.DataFrame()
        
        return self.df.groupby(group_by).agg(agg_config)
    
    def pivot_data(
        self,
        index: str,
        columns: str,
        values: str,
        aggfunc: str = 'sum'
    ) -> pd.DataFrame:
        """数据透视"""
        if self.df is None:
            return pd.DataFrame()
        
        return pd.pivot_table(
            self.df,
            index=index,
            columns=columns,
            values=values,
            aggfunc=aggfunc
        )
    
    def save(self, output_path: str):
        """保存数据"""
        if self.df is None:
            raise ValueError("没有可保存的数据")
        
        output_ext = Path(output_path).suffix.lower()
        
        if output_ext == '.csv':
            self.df.to_csv(output_path, index=False)
        elif output_ext in ['.xlsx', '.xls']:
            self.df.to_excel(output_path, index=False)
        elif output_ext == '.json':
            self.df.to_json(output_path, orient='records')
        else:
            raise ValueError(f"不支持的输出格式: {output_ext}")


class AnalysisService:
    """分析服务"""
    
    def __init__(self, df: pd.DataFrame):
        self.df = df
    
    def descriptive_stats(self, columns: Optional[List[str]] = None) -> Dict[str, Any]:
        """描述性统计"""
        if columns:
            data = self.df[columns]
        else:
            data = self.df
        
        # 分离数值和非数值列
        numeric_cols = data.select_dtypes(include=[np.number]).columns
        non_numeric_cols = data.select_dtypes(exclude=[np.number]).columns
        
        results = {
            'numeric': {},
            'categorical': {}
        }
        
        # 数值型统计
        for col in numeric_cols:
            col_data = data[col].dropna()
            results['numeric'][col] = {
                'count': int(len(col_data)),
                'mean': float(col_data.mean()) if len(col_data) > 0 else None,
                'std': float(col_data.std()) if len(col_data) > 0 else None,
                'min': float(col_data.min()) if len(col_data) > 0 else None,
                '25%': float(col_data.quantile(0.25)) if len(col_data) > 0 else None,
                '50%': float(col_data.median()) if len(col_data) > 0 else None,
                '75%': float(col_data.quantile(0.75)) if len(col_data) > 0 else None,
                'max': float(col_data.max()) if len(col_data) > 0 else None,
                'skewness': float(col_data.skew()) if len(col_data) > 0 else None,
                'kurtosis': float(col_data.kurtosis()) if len(col_data) > 0 else None
            }
        
        # 分类型统计
        for col in non_numeric_cols:
            col_data = data[col].dropna()
            value_counts = col_data.value_counts()
            results['categorical'][col] = {
                'count': int(len(col_data)),
                'unique': int(col_data.nunique()),
                'top': str(value_counts.index[0]) if len(value_counts) > 0 else None,
                'freq': int(value_counts.iloc[0]) if len(value_counts) > 0 else None,
                'value_counts': {str(k): int(v) for k, v in value_counts.head(10).items()}
            }
        
        return results
    
    def correlation_analysis(self, columns: Optional[List[str]] = None) -> Dict[str, Any]:
        """相关性分析"""
        if columns:
            data = self.df[columns].select_dtypes(include=[np.number])
        else:
            data = self.df.select_dtypes(include=[np.number])
        
        if data.empty or len(data.columns) < 2:
            return {'error': '需要至少2个数值型列进行相关性分析'}
        
        # 计算相关系数矩阵
        corr_matrix = data.corr()
        
        # 计算 p 值
        from scipy import stats
        
        n = len(data)
        p_values = pd.DataFrame(np.zeros((len(data.columns), len(data.columns))))
        
        for i in range(len(data.columns)):
            for j in range(len(data.columns)):
                if i == j:
                    p_values.iloc[i, j] = 0
                else:
                    _, p = stats.pearsonr(data.iloc[:, i], data.iloc[:, j])
                    p_values.iloc[i, j] = p
        
        return {
            'correlation': corr_matrix.to_dict(),
            'p_values': p_values.to_dict(),
            'method': 'pearson'
        }
    
    def trend_analysis(self, date_col: str, value_col: str) -> Dict[str, Any]:
        """趋势分析"""
        if date_col not in self.df.columns or value_col not in self.df.columns:
            return {'error': f'列 {date_col} 或 {value_col} 不存在'}
        
        df = self.df[[date_col, value_col]].copy()
        df[date_col] = pd.to_datetime(df[date_col], errors='coerce')
        df = df.dropna().sort_values(date_col)
        
        if len(df) < 2:
            return {'error': '数据点不足'}
        
        # 计算移动平均
        df['ma_7'] = df[value_col].rolling(window=7, min_periods=1).mean()
        df['ma_30'] = df[value_col].rolling(window=30, min_periods=1).mean()
        
        # 线性回归
        x = np.arange(len(df))
        y = df[value_col].values
        slope, intercept, r_value, p_value, std_err = stats.linregress(x, y)
        
        return {
            'start_date': str(df[date_col].iloc[0]),
            'end_date': str(df[date_col].iloc[-1]),
            'start_value': float(df[value_col].iloc[0]),
            'end_value': float(df[value_col].iloc[-1]),
            'change': float(df[value_col].iloc[-1] - df[value_col].iloc[0]),
            'change_pct': float((df[value_col].iloc[-1] - df[value_col].iloc[0]) / df[value_col].iloc[0] * 100) if df[value_col].iloc[0] != 0 else 0,
            'trend': {
                'slope': float(slope),
                'intercept': float(intercept),
                'r_squared': float(r_value ** 2),
                'p_value': float(p_value),
                'direction': 'increasing' if slope > 0 else 'decreasing'
            },
            'moving_average': {
                'ma_7': df['ma_7'].tolist(),
                'ma_30': df['ma_30'].tolist()
            },
            'data_points': df[[date_col, value_col]].to_dict('records')
        }
    
    def distribution_analysis(self, column: str, bins: int = 10) -> Dict[str, Any]:
        """分布分析"""
        if column not in self.df.columns:
            return {'error': f'列 {column} 不存在'}
        
        data = self.df[column].dropna()
        
        if not pd.api.types.is_numeric_dtype(data):
            return {'error': f'列 {column} 不是数值型'}
        
        # 直方图数据
        hist, bin_edges = np.histogram(data, bins=bins)
        
        return {
            'histogram': {
                'counts': hist.tolist(),
                'bins': bin_edges.tolist()
            },
            'percentiles': {
                '1%': float(data.quantile(0.01)),
                '5%': float(data.quantile(0.05)),
                '25%': float(data.quantile(0.25)),
                '50%': float(data.quantile(0.50)),
                '75%': float(data.quantile(0.75)),
                '95%': float(data.quantile(0.95)),
                '99%': float(data.quantile(0.99))
            },
            'outliers': {
                'lower_bound': float(data.quantile(0.25) - 1.5 * (data.quantile(0.75) - data.quantile(0.25))),
                'upper_bound': float(data.quantile(0.75) + 1.5 * (data.quantile(0.75) - data.quantile(0.25)))
            },
            'normality': {
                'statistic': float(stats.normaltest(data)[0]),
                'p_value': float(stats.normaltest(data)[1])
            }
        }
```

---

### Day 4：API 服务实现 ⭐⭐

**目标**：实现 FastAPI 后端

**核心代码**：

`datavault/app/api/routes.py` - API 路由
```python
from fastapi import APIRouter, UploadFile, File, HTTPException, BackgroundTasks, Depends
from fastapi.responses import StreamingResponse
from sqlalchemy.ext.asyncio import AsyncSession
from sqlalchemy import select
from typing import List, Optional
import os
import aiofiles
from pathlib import Path
from datetime import datetime

from app.db.base import get_db
from app.models.dataset import Dataset, Analysis, Chart
from app.schemas.dataset import (
    DatasetResponse, DatasetListResponse, DatasetInfoResponse,
    AnalysisRequest, AnalysisResponse,
    ChartRequest, ChartResponse
)
from app.services.data_processor import DataProcessor, AnalysisService

router = APIRouter(prefix="/api/v1")

# ============= 数据集接口 =============

@router.post("/datasets/upload", response_model=DatasetResponse)
async def upload_dataset(
    background_tasks: BackgroundTasks,
    file: UploadFile = File(...),
    name: str = "",
    description: str = "",
    db: AsyncSession = Depends(get_db)
):
    """上传数据集"""
    # 验证文件
    allowed_types = ['.csv', '.xlsx', '.xls', '.json']
    file_ext = Path(file.filename).suffix.lower()
    
    if file_ext not in allowed_types:
        raise HTTPException(
            status_code=400,
            detail=f"不支持的文件类型，仅支持: {', '.join(allowed_types)}"
        )
    
    # 保存文件
    timestamp = datetime.now().strftime("%Y%m%d_%H%M%S")
    filename = f"{timestamp}_{file.filename}"
    file_path = os.path.join("./data/uploads", filename)
    
    async with aiofiles.open(file_path, 'wb') as f:
        content = await file.read()
        await f.write(content)
    
    # 创建数据集记录
    dataset = Dataset(
        name=name or file.filename,
        description=description,
        filename=file.filename,
        file_type=file_ext[1:],
        file_size=len(content),
        file_path=file_path,
        created_by=1  # TODO: 从认证获取
    )
    
    db.add(dataset)
    await db.commit()
    await db.refresh(dataset)
    
    # 后台处理
    def process_dataset():
        try:
            processor = DataProcessor(file_path)
            info = processor.get_info()
            
            # 更新数据集信息
            # 这里需要同步更新数据库
        except Exception as e:
            print(f"处理数据集失败: {e}")
    
    background_tasks.add_task(process_dataset)
    
    return dataset


@router.get("/datasets", response_model=DatasetListResponse)
async def list_datasets(
    skip: int = 0,
    limit: int = 20,
    db: AsyncSession = Depends(get_db)
):
    """列出数据集"""
    # 查询
    result = await db.execute(
        select(Dataset)
        .where(Dataset.is_deleted == False)
        .order_by(Dataset.created_at.desc())
        .offset(skip)
        .limit(limit)
    )
    datasets = result.scalars().all()
    
    # 计数
    from sqlalchemy import func
    count_result = await db.execute(
        select(func.count(Dataset.id))
        .where(Dataset.is_deleted == False)
    )
    total = count_result.scalar()
    
    return DatasetListResponse(
        datasets=datasets,
        total=total,
        skip=skip,
        limit=limit
    )


@router.get("/datasets/{dataset_id}", response_model=DatasetInfoResponse)
async def get_dataset(
    dataset_id: int,
    db: AsyncSession = Depends(get_db)
):
    """获取数据集详情"""
    result = await db.execute(
        select(Dataset).where(Dataset.id == dataset_id)
    )
    dataset = result.scalar_one_or_none()
    
    if not dataset:
        raise HTTPException(status_code=404, detail="数据集不存在")
    
    # 获取数据信息
    processor = DataProcessor(dataset.file_path)
    info = processor.get_info()
    sample = processor.get_sample(100)
    
    return DatasetInfoResponse(
        **dataset.__dict__,
        info=info,
        sample=sample
    )


@router.get("/datasets/{dataset_id}/sample")
async def get_dataset_sample(
    dataset_id: int,
    n: int = 100,
    random: bool = False,
    db: AsyncSession = Depends(get_db)
):
    """获取数据集样本"""
    result = await db.execute(
        select(Dataset).where(Dataset.id == dataset_id)
    )
    dataset = result.scalar_one_or_none()
    
    if not dataset:
        raise HTTPException(status_code=404, detail="数据集不存在")
    
    processor = DataProcessor(dataset.file_path)
    sample = processor.get_sample(n=n, random=random)
    
    return {"data": sample, "count": len(sample)}


@router.post("/datasets/{dataset_id}/clean")
async def clean_dataset(
    dataset_id: int,
    operations: dict,
    db: AsyncSession = Depends(get_db)
):
    """清洗数据集"""
    result = await db.execute(
        select(Dataset).where(Dataset.id == dataset_id)
    )
    dataset = result.scalar_one_or_none()
    
    if not dataset:
        raise HTTPException(status_code=404, detail="数据集不存在")
    
    processor = DataProcessor(dataset.file_path)
    clean_results = processor.clean_data(operations)
    
    return {"success": True, "results": clean_results}


@router.delete("/datasets/{dataset_id}")
async def delete_dataset(
    dataset_id: int,
    db: AsyncSession = Depends(get_db)
):
    """删除数据集"""
    result = await db.execute(
        select(Dataset).where(Dataset.id == dataset_id)
    )
    dataset = result.scalar_one_or_none()
    
    if not dataset:
        raise HTTPException(status_code=404, detail="数据集不存在")
    
    # 软删除
    dataset.is_deleted = True
    await db.commit()
    
    return {"success": True, "message": "数据集已删除"}


# ============= 分析接口 =============

@router.post("/analyses", response_model=AnalysisResponse)
async def create_analysis(
    request: AnalysisRequest,
    db: AsyncSession = Depends(get_db)
):
    """创建分析"""
    # 获取数据集
    result = await db.execute(
        select(Dataset).where(Dataset.id == request.dataset_id)
    )
    dataset = result.scalar_one_or_none()
    
    if not dataset:
        raise HTTPException(status_code=404, detail="数据集不存在")
    
    # 加载数据
    processor = DataProcessor(dataset.file_path)
    analysis_service = AnalysisService(processor.df)
    
    # 执行分析
    if request.analysis_type == "descriptive":
        result_data = analysis_service.descriptive_stats(request.config.get('columns'))
    elif request.analysis_type == "correlation":
        result_data = analysis_service.correlation_analysis(request.config.get('columns'))
    elif request.analysis_type == "trend":
        result_data = analysis_service.trend_analysis(
            request.config['date_column'],
            request.config['value_column']
        )
    elif request.analysis_type == "distribution":
        result_data = analysis_service.distribution_analysis(
            request.config['column'],
            request.config.get('bins', 10)
        )
    else:
        raise HTTPException(status_code=400, detail=f"不支持的分析类型: {request.analysis_type}")
    
    # 保存分析记录
    analysis = Analysis(
        name=request.name,
        description=request.description,
        dataset_id=request.dataset_id,
        analysis_type=request.analysis_type,
        config=request.config,
        result=result_data,
        status="completed"
    )
    
    db.add(analysis)
    await db.commit()
    await db.refresh(analysis)
    
    return analysis


@router.get("/analyses/{analysis_id}")
async def get_analysis(
    analysis_id: int,
    db: AsyncSession = Depends(get_db)
):
    """获取分析结果"""
    result = await db.execute(
        select(Analysis).where(Analysis.id == analysis_id)
    )
    analysis = result.scalar_one_or_none()
    
    if not analysis:
        raise HTTPException(status_code=404, detail="分析不存在")
    
    return analysis


# ============= 图表接口 =============

@router.post("/charts", response_model=ChartResponse)
async def create_chart(
    request: ChartRequest,
    db: AsyncSession = Depends(get_db)
):
    """创建图表"""
    # 获取数据集
    result = await db.execute(
        select(Dataset).where(Dataset.id == request.dataset_id)
    )
    dataset = result.scalar_one_or_none()
    
    if not dataset:
        raise HTTPException(status_code=404, detail="数据集不存在")
    
    # 加载数据
    processor = DataProcessor(dataset.file_path)
    
    # 根据图表类型提取数据
    chart_data = extract_chart_data(processor.df, request.chart_type, request.config)
    
    # 构建 ECharts 配置
    echarts_config = build_echarts_config(request.chart_type, request.config, chart_data)
    
    # 保存图表
    chart = Chart(
        name=request.name,
        chart_type=request.chart_type,
        dataset_id=request.dataset_id,
        config=request.config,
        data=chart_data
    )
    
    db.add(chart)
    await db.commit()
    await db.refresh(chart)
    
    return {
        **chart.__dict__,
        'echarts_config': echarts_config
    }


def extract_chart_data(df, chart_type: str, config: dict) -> dict:
    """提取图表数据"""
    x_col = config.get('x_axis')
    y_cols = config.get('y_axis', [])
    
    if not x_col or not y_cols:
        return {}
    
    data = {
        'categories': df[x_col].tolist(),
        'series': []
    }
    
    for y_col in y_cols:
        if y_col in df.columns:
            data['series'].append({
                'name': y_col,
                'values': df[y_col].tolist()
            })
    
    return data


def build_echarts_config(chart_type: str, config: dict, data: dict) -> dict:
    """构建 ECharts 配置"""
    base_config = {
        "title": {
            "text": config.get('title', ''),
            "left": "center"
        },
        "tooltip": {
            "trigger": "axis" if chart_type != "pie" else "item"
        },
        "legend": {
            "data": [s['name'] for s in data.get('series', [])],
            "top": 30
        },
        "xAxis": {
            "type": "category",
            "data": data.get('categories', []),
            "axisLabel": {
                "rotate": config.get('rotate_labels', 0)
            }
        },
        "yAxis": {
            "type": "value"
        },
        "series": []
    }
    
    # 根据类型配置
    series_config = {
        "bar": {"type": "bar"},
        "line": {"type": "line"},
        "scatter": {"type": "scatter"},
        "pie": {"type": "pie", "radius": "50%"}
    }
    
    for series in data.get('series', []):
        series_item = {
            "name": series['name'],
            "data": series['values'],
            **series_config.get(chart_type, {"type": chart_type})
        }
        base_config["series"].append(series_item)
    
    if chart_type == "pie":
        base_config.pop("xAxis", None)
        base_config.pop("yAxis", None)
        base_config["series"][0]["data"] = [
            {"name": n, "value": v}
            for n, v in zip(data['categories'], data['series'][0]['values'])
        ]
    
    return base_config
```

---

### Day 5：前端可视化 ⭐⭐

**目标**：实现前端数据可视化

**核心代码**：

`frontend/src/components/DataChart.tsx` - 图表组件
```tsx
import React, { useEffect, useRef } from 'react';
import * as echarts from 'echarts';

interface ChartProps {
  option: echarts.EChartsOption;
  width?: string;
  height?: string;
  onClick?: (params: any) => void;
}

export function DataChart({ option, width = '100%', height = '400px', onClick }: ChartProps) {
  const chartRef = useRef<HTMLDivElement>(null);
  const chartInstance = useRef<echarts.ECharts>();

  useEffect(() => {
    if (!chartRef.current) return;

    // 初始化图表
    chartInstance.current = echarts.init(chartRef.current);

    // 设置配置
    chartInstance.current.setOption(option);

    // 绑定点击事件
    if (onClick) {
      chartInstance.current.on('click', onClick);
    }

    // 响应窗口调整
    const handleResize = () => {
      chartInstance.current?.resize();
    };
    window.addEventListener('resize', handleResize);

    return () => {
      window.removeEventListener('resize', handleResize);
      chartInstance.current?.dispose();
    };
  }, [option]);

  return (
    <div
      ref={chartRef}
      style={{ width, height }}
      className="chart-container"
    />
  );
}

// 常用图表类型
export type ChartType = 'bar' | 'line' | 'pie' | 'scatter' | 'heatmap';

// 图表配置生成器
export function generateChartOption(
  type: ChartType,
  data: any,
  title?: string
): echarts.EChartsOption {
  const baseOption: echarts.EChartsOption = {
    title: {
      text: title || '',
      left: 'center',
      textStyle: { fontSize: 16 }
    },
    tooltip: {
      trigger: type === 'pie' ? 'item' : 'axis'
    },
    legend: {
      top: 30,
      data: data.series?.map((s: any) => s.name) || []
    }
  };

  switch (type) {
    case 'bar':
      return {
        ...baseOption,
        xAxis: {
          type: 'category',
          data: data.categories || [],
          axisLabel: { rotate: 30 }
        },
        yAxis: { type: 'value' },
        series: data.series?.map((s: any) => ({
          name: s.name,
          type: 'bar',
          data: s.values
        })) || []
      };

    case 'line':
      return {
        ...baseOption,
        xAxis: {
          type: 'category',
          data: data.categories || []
        },
        yAxis: { type: 'value' },
        series: data.series?.map((s: any) => ({
          name: s.name,
          type: 'line',
          data: s.values,
          smooth: true
        })) || []
      };

    case 'pie':
      return {
        ...baseOption,
        series: [{
          name: data.series?.[0]?.name || '',
          type: 'pie',
          radius: '50%',
          data: data.categories?.map((name: string, i: number) => ({
            name,
            value: data.series?.[0]?.values?.[i] || 0
          })) || []
        }]
      };

    case 'scatter':
      return {
        ...baseOption,
        xAxis: { type: 'value' },
        yAxis: { type: 'value' },
        series: [{
          type: 'scatter',
          symbolSize: 10,
          data: data.series?.[0]?.values?.map((v: number, i: number) => [
            i, v
          ]) || []
        }]
      };

    default:
      return baseOption;
  }
}
```

---

### Day 6 & 7：项目整合与面试准备 ⭐⭐⭐

**Docker 配置**：

`datavault/docker-compose.yml`
```yaml
version: '3.8'

services:
  backend:
    build: ./backend
    ports:
      - "8000:8000"
    environment:
      - DATABASE_URL=mysql+aiomysql://root:password@mysql:3306/datavault
    volumes:
      - ./data:/app/data
    depends_on:
      mysql:
        condition: service_healthy
    networks:
      - datavault-network

  frontend:
    build: ./frontend
    ports:
      - "3000:80"
    depends_on:
      - backend
    networks:
      - datavault-network

  mysql:
    image: mysql:8.0
    ports:
      - "3306:3306"
    environment:
      - MYSQL_ROOT_PASSWORD=password
      - MYSQL_DATABASE=datavault
    volumes:
      - mysql_data:/var/lib/mysql
    healthcheck:
      test: ["CMD", "mysqladmin", "ping", "-h", "localhost"]
      interval: 10s
      timeout: 5s
      retries: 5
    networks:
      - datavault-network

volumes:
  mysql_data:

networks:
  datavault-network:
    driver: bridge
```

---

## 📝 面试要点

### Q1：Pandas 的性能优化方法？

**参考答案**：
```
1. 数据类型优化：
   - 使用 category 类型存储字符串
   - 使用 int32/int64 代替 int64
   - 使用 float32 代替 float64

2. 内存优化：
   - 读取时指定 dtype
   - 使用 chunksize 分块读取
   - 删除不需要的列

3. 操作优化：
   - 使用向量化操作代替循环
   - 使用 inplace=True
   - 使用 eval/express

4. 合并优化：
   - 使用 join 代替 merge（索引对齐）
   - 使用 concat 代替 append
```

### Q2：数据清洗的常见步骤？

**参考答案**：
```
1. 缺失值处理：
   - 删除、填充（均值/中位数/众数）
   - 插值预测

2. 异常值处理：
   - IQR 方法
   - Z-score 方法
   - 领域知识判断

3. 重复数据：
   - 完全重复删除
   - 部分重复合并

4. 格式标准化：
   - 日期格式统一
   - 字符串大小写
   - 数值类型转换

5. 异常数据：
   - 范围检查
   - 枚举值检查
   - 格式正则验证
```

### Q3：如何选择可视化图表？

**参考答案**：
```
1. 比较类：
   - 柱状图：离散比较
   - 折线图：连续趋势
   - 雷达图：多维对比

2. 构成类：
   - 饼图：占比（<5个分类）
   - 堆叠柱状图：部分占整体
   - 瀑布图：变化分解

3. 分布类：
   - 直方图：频率分布
   - 箱线图：统计分布
   - 散点图：双变量分布

4. 关系类：
   - 散点图：相关性
   - 气泡图：三变量关系
   - 热力图：矩阵关系
```

### Q4：统计显著性检验？

**参考答案**：
```
1. t 检验：
   - 单样本 t：样本与已知均值比较
   - 独立样本 t：两组均值比较
   - 配对样本 t：同组前后比较

2. ANOVA：
   - 单因素：多组均值比较
   - 双因素：两因素交互作用

3. 卡方检验：
   - 独立性检验
   - 拟合优度检验

4. 相关性检验：
   - Pearson：线性相关
   - Spearman：等级相关

判断标准：
- p < 0.05：显著
- p < 0.01：非常显著
- p > 0.05：不显著
```

---

## ✅ 项目清单

```
DataVault/
├── backend/
│   ├── app/
│   │   ├── api/
│   │   │   └── routes.py
│   │   ├── core/
│   │   │   └── config.py
│   │   ├── db/
│   │   │   └── base.py
│   │   ├── models/
│   │   │   ├── dataset.py
│   │   │   ├── analysis.py
│   │   │   └── chart.py
│   │   ├── schemas/
│   │   │   └── dataset.py
│   │   └── services/
│   │       ├── data_processor.py
│   │       └── analysis_service.py
│   ├── main.py
│   ├── requirements.txt
│   └── Dockerfile
├── frontend/
│   ├── src/
│   │   ├── components/
│   │   │   ├── DataChart.tsx
│   │   │   └── DataTable.tsx
│   │   ├── pages/
│   │   │   ├── DatasetPage.tsx
│   │   │   ├── AnalysisPage.tsx
│   │   │   └── DashboardPage.tsx
│   │   └── App.tsx
│   ├── package.json
│   └── Dockerfile
├── data/
│   ├── uploads/
│   ├── cache/
│   └── exports/
├── docker-compose.yml
└── README.md
```

---

**文档版本**：v1.0  
**更新日期**：2024
