# Tokio 异步服务方案：方案1 - 高性能 API 网关

## 项目介绍

### 项目是什么
这是一个基于 Rust + Tokio 构建的高性能 API 网关，支持请求路由、负载均衡、熔断器、限流、认证和日志追踪等功能。

### 解决什么问题
- 统一入口：所有前端请求通过网关访问后端服务
- 负载均衡：将请求分发到多个后端实例
- 熔断保护：防止级联故障
- 请求限流：保护后端服务
- 统一认证：JWT 令牌验证
- 日志追踪：分布式请求追踪

### 核心技术亮点
| 特性 | 技术实现 |
|------|----------|
| 异步运行时 | Tokio + async/await |
| HTTP 框架 | Axum 0.7 |
| 连接池 | SQLx |
| 缓存层 | Redis |
| 中间件 | Tower |
| 熔断器 | 自定义实现 |
| 限流器 | Token Bucket |
| 日志 | tracing + opentelemetry |
| 配置 | config-rs |

### 适合什么场景
- 微服务架构的 API 入口
- 需要高并发处理的场景
- 需要统一认证和限流的 API 服务
- 需要请求追踪和监控的后端系统

---

## 完整可运行代码

### 目录结构
```
api-gateway/
├── src/
│   ├── main.rs              # 程序入口
│   ├── lib.rs               # 库入口
│   ├── config.rs            # 配置管理
│   ├── routes/
│   │   ├── mod.rs           # 路由模块
│   │   ├── proxy.rs         # 代理路由
│   │   ├── auth.rs          # 认证路由
│   │   └── health.rs        # 健康检查
│   ├── middleware/
│   │   ├── mod.rs           # 中间件模块
│   │   ├── rate_limiter.rs  # 限流器
│   │   ├── circuit_breaker.rs  # 熔断器
│   │   ├── auth.rs          # 认证中间件
│   │   └── logging.rs       # 日志中间件
│   ├── services/
│   │   ├── mod.rs           # 服务模块
│   │   ├── upstream.rs      # 上游服务调用
│   │   ├── cache.rs         # 缓存服务
│   │   └── metrics.rs       # 指标收集
│   ├── models/
│   │   ├── mod.rs           # 模型模块
│   │   ├── request.rs       # 请求模型
│   │   └── response.rs      # 响应模型
│   └── utils/
│       ├── mod.rs           # 工具模块
│       └── jwt.rs           # JWT 工具
├── tests/
│   └── integration_tests.rs # 集成测试
├── Cargo.toml
└── config.yaml
```

### 1. Cargo.toml
```toml
[package]
name = "api-gateway"
version = "1.0.0"
edition = "2021"
authors = ["尘先生"]
description = "High-performance API Gateway built with Rust and Tokio"

[dependencies]
# ============ 异步运行时 ============
tokio = { version = "1.35", features = ["full"] }
tokio-rustls = "0.24"      # TLS 支持

# ============ Web 框架 ============
axum = { version = "0.7", features = ["macros"] }
tower = "0.4"              # 中间件库
tower-http = { version = "0.5", features = ["cors", "trace"] }

# ============ HTTP 客户端 ============
reqwest = { version = "0.11", features = ["json", "rustls-tls"], default-features = false }

# ============ 数据库 ============
sqlx = { version = "0.7", features = ["runtime-tokio", "postgres", "migrate", "chrono"] }

# ============ 缓存 ============
redis = { version = "0.24", features = ["tokio-comp", "connection-manager"] }

# ============ 序列化 ============
serde = { version = "1.0", features = ["derive"] }
serde_json = "1.0"

# ============ 配置 ============
config = "0.14"

# ============ JWT ============
jsonwebtoken = "9.2"

# ============ 日志和追踪 ============
tracing = "0.1"
tracing-subscriber = { version = "0.3", features = ["env-filter", "json"] }
tracing-opentelemetry = "0.22"
opentelemetry = { version = "0.21", features = ["trace"] }

# ============ 限流 ============
governor = "0.6"

# ============ 错误处理 ============
thiserror = "1.0"
anyhow = "1.0"

# ============ 时间 ============
chrono = { version = "0.4", features = ["serde"] }

# ============ UUID ============
uuid = { version = "1.6", features = ["v4", "serde"] }

# ============ 异步工具 ============
futures = "0.3"
async-trait = "0.1"

[dev-dependencies]
tower = { version = "0.4", features = ["util"] }
hyper = "1.1"
tokio-test = "0.4"

[profile.release]
opt-level = 3
lto = true
codegen-units = 1
```

### 2. 配置文件：config.yaml
```yaml
# API Gateway 配置文件
app:
  name: "api-gateway"
  host: "0.0.0.0"
  port: 8080
  workers: 4  # Tokio worker 数量

# 上游服务配置
upstream:
  # 用户服务
  user_service:
    url: "http://localhost:8081"
    timeout: 5s
    weight: 100
    
  # 订单服务
  order_service:
    url: "http://localhost:8082"
    timeout: 10s
    weight: 50
    
  # 商品服务
  product_service:
    url: "http://localhost:8083"
    timeout: 5s
    weight: 80

# 限流配置
rate_limit:
  enabled: true
  default_rps: 100  # 每秒请求数
  burst_size: 200    # 突发大小
  
  # 按 IP 限流
  by_ip:
    rps: 10
    burst: 20

# 熔断器配置
circuit_breaker:
  enabled: true
  failure_threshold: 5   # 失败次数阈值
  success_threshold: 2   # 成功次数阈值
  timeout: 60s          # 熔断超时

# JWT 配置
jwt:
  secret: "your-secret-key-change-in-production"
  expiry: 24h
  issuer: "api-gateway"

# Redis 配置
redis:
  host: "localhost"
  port: 6379
  password: ""
  db: 0
  pool_size: 10

# PostgreSQL 配置
database:
  url: "postgres://user:pass@localhost/gateway"
  max_connections: 10

# 日志配置
logging:
  level: "info"
  format: "json"
  tracing:
    enabled: true
    service_name: "api-gateway"
```

### 3. 配置管理：src/config.rs
```rust
// config.rs - 配置管理模块
// 作用：加载和解析 YAML 配置文件

use serde::Deserialize;
use std::path::Path;
use thiserror::Error;

// ==================== 配置错误类型 ====================

#[derive(Debug, Error)]
pub enum ConfigError {
    #[error("配置加载失败: {0}")]
    Load(#[from] config::ConfigError),
    
    #[error("配置解析失败: {0}")]
    Parse(String),
    
    #[error("配置文件不存在: {0}")]
    NotFound(String),
}

// ==================== 配置结构体 ====================

/// 应用配置
#[derive(Debug, Clone, Deserialize)]
pub struct AppConfig {
    pub name: String,
    pub host: String,
    pub port: u16,
    pub workers: usize,
}

/// 上游服务配置
#[derive(Debug, Clone, Deserialize)]
pub struct UpstreamConfig {
    pub url: String,
    pub timeout: Duration,
    pub weight: u32,
}

/// 上游服务集合
#[derive(Debug, Clone, Deserialize)]
pub struct UpstreamServices {
    #[serde(rename = "user_service")]
    pub user: Option<UpstreamConfig>,
    #[serde(rename = "order_service")]
    pub order: Option<UpstreamConfig>,
    #[serde(rename = "product_service")]
    pub product: Option<UpstreamConfig>,
}

/// 限流配置
#[derive(Debug, Clone, Deserialize)]
pub struct RateLimitConfig {
    pub enabled: bool,
    pub default_rps: u32,
    pub burst_size: u32,
}

/// IP 限流配置
#[derive(Debug, Clone, Deserialize)]
pub struct IpRateLimitConfig {
    pub rps: u32,
    pub burst: u32,
}

/// 限流完整配置
#[derive(Debug, Clone, Deserialize)]
pub struct RateLimitSettings {
    pub enabled: bool,
    pub default_rps: u32,
    pub burst_size: u32,
    #[serde(rename = "by_ip")]
    pub by_ip: Option<IpRateLimitConfig>,
}

/// 熔断器配置
#[derive(Debug, Clone, Deserialize)]
pub struct CircuitBreakerConfig {
    pub enabled: bool,
    pub failure_threshold: u32,
    pub success_threshold: u32,
    pub timeout: Duration,
}

/// JWT 配置
#[derive(Debug, Clone, Deserialize)]
pub struct JwtConfig {
    pub secret: String,
    pub expiry: Duration,
    pub issuer: String,
}

/// Redis 配置
#[derive(Debug, Clone, Deserialize)]
pub struct RedisConfig {
    pub host: String,
    pub port: u16,
    pub password: Option<String>,
    pub db: u8,
    pub pool_size: u32,
}

/// 数据库配置
#[derive(Debug, Clone, Deserialize)]
pub struct DatabaseConfig {
    pub url: String,
    pub max_connections: u32,
}

/// 日志配置
#[derive(Debug, Clone, Deserialize)]
pub struct LoggingConfig {
    pub level: String,
    pub format: String,
    pub tracing: TracingConfig,
}

/// 追踪配置
#[derive(Debug, Clone, Deserialize)]
pub struct TracingConfig {
    pub enabled: bool,
    pub service_name: String,
}

/// 完整配置
#[derive(Debug, Clone, Deserialize)]
pub struct Config {
    pub app: AppConfig,
    pub upstream: Option<UpstreamServices>,
    #[serde(rename = "rate_limit")]
    pub rate_limit: Option<RateLimitSettings>,
    #[serde(rename = "circuit_breaker")]
    pub circuit_breaker: Option<CircuitBreakerConfig>,
    pub jwt: JwtConfig,
    pub redis: Option<RedisConfig>,
    pub database: Option<DatabaseConfig>,
    pub logging: Option<LoggingConfig>,
}

/// Duration 辅助类型
/// 将字符串如 "5s", "1m", "24h" 解析为 Duration
#[derive(Debug, Clone)]
pub struct Duration(pub std::time::Duration);

impl Duration {
    /// 从字符串解析 Duration
    /// 格式: 数字 + 单位(s/m/h/d)
    pub fn from_str(s: &str) -> Result<Self, ConfigError> {
        let s = s.trim();
        
        let (num_str, unit) = s
            .trim_end_matches(|c: char| c.is_ascii_digit())
            .split_at(s.len().saturating_sub(1));
        
        let num: u64 = num_str
            .parse()
            .map_err(|_| ConfigError::Parse(format!("无效的数字: {}", num_str)))?;
        
        let duration = match unit {
            "s" => std::time::Duration::from_secs(num),
            "m" => std::time::Duration::from_secs(num * 60),
            "h" => std::time::Duration::from_secs(num * 3600),
            "d" => std::time::Duration::from_secs(num * 86400),
            _ => return Err(ConfigError::Parse(format!("无效的时间单位: {}", unit))),
        };
        
        Ok(Duration(duration))
    }
}

impl<'de> Deserialize<'de> for Duration {
    fn deserialize<D>(deserializer: D) -> Result<Self, D::Error>
    where
        D: serde::Deserializer<'de>,
    {
        let s = String::deserialize(deserializer)?;
        Duration::from_str(&s).map_err(serde::de::Error::custom)
    }
}

// ==================== 配置加载 ====================

impl Config {
    /// 从文件加载配置
    pub fn from_file<P: AsRef<Path>>(path: P) -> Result<Self, ConfigError> {
        let path = path.as_ref();
        
        if !path.exists() {
            return Err(ConfigError::NotFound(path.display().to_string()));
        }
        
        // 使用 config 库加载配置
        let settings = config::Config::builder()
            // 从文件加载配置
            .add_source(config::File::from(path))
            // 允许环境变量覆盖
            .add_source(config::Environment::with_prefix("GATEWAY"))
            .build()?;
        
        // 反序列化为 Config 结构
        let config: Config = settings.try_deserialize()?;
        
        Ok(config)
    }
    
    /// 加载默认配置
    pub fn load() -> Result<Self, ConfigError> {
        Self::from_file("config.yaml")
    }
    
    /// 获取上游服务 URL
    pub fn get_upstream_url(&self, name: &str) -> Option<String> {
        match name {
            "user" | "user_service" => self.upstream.as_ref()?.user.clone().map(|c| c.url),
            "order" | "order_service" => self.upstream.as_ref()?.order.clone().map(|c| c.url),
            "product" | "product_service" => self.upstream.as_ref()?.product.clone().map(|c| c.url),
            _ => None,
        }
    }
}

// ==================== 默认配置 ====================

impl Default for Config {
    fn default() -> Self {
        Self {
            app: AppConfig {
                name: "api-gateway".to_string(),
                host: "0.0.0.0".to_string(),
                port: 8080,
                workers: 4,
            },
            upstream: None,
            rate_limit: None,
            circuit_breaker: None,
            jwt: JwtConfig {
                secret: "change-me-in-production".to_string(),
                expiry: Duration(std::time::Duration::from_secs(86400)),
                issuer: "api-gateway".to_string(),
            },
            redis: None,
            database: None,
            logging: None,
        }
    }
}
```

### 4. 模型定义：src/models/mod.rs
```rust
// models/mod.rs - 数据模型定义
// 作用：定义请求、响应和内部使用的数据结构

pub mod request;
pub mod response;

pub use request::*;
pub use response::*;

// ==================== 通用类型 ====================

/// 请求 ID 类型
pub type RequestId = uuid::Uuid;

/// 路由路径
pub type RoutePath = String;

/// 状态码
pub type StatusCode = u16;

// ==================== 常用类型别名 ====================

/// 异步结果类型
pub type Result<T> = std::result::Result<T, GatewayError>;

/// HTTP 请求
pub type HttpRequest = axum::http::Request<axum::body::Body>;

/// HTTP 响应
pub type HttpResponse = axum::http::Response<axum::body::Body>;
```

### 5. 请求模型：src/models/request.rs
```rust
// models/request.rs - 请求相关模型

use axum::{extract::State, http::Request, body::Body};
use serde::{Deserialize, Serialize};
use std::collections::HashMap;

// ==================== 代理请求 ====================

/// 代理请求上下文
/// 包含原始请求的所有信息
#[derive(Debug, Clone)]
pub struct ProxyRequest {
    /// 请求 ID
    pub request_id: uuid::Uuid,
    /// 目标服务名称
    pub service: String,
    /// 目标路径
    pub path: String,
    /// HTTP 方法
    pub method: String,
    /// 请求头
    pub headers: HashMap<String, String>,
    /// 查询参数
    pub query: HashMap<String, String>,
    /// 请求体（如果是 POST/PUT）
    pub body: Option<Vec<u8>>,
    /// 客户端 IP
    pub client_ip: Option<String>,
    /// 认证用户 ID
    pub user_id: Option<String>,
    /// 开始时间
    pub start_time: std::time::Instant,
}

impl ProxyRequest {
    /// 创建新的代理请求
    pub fn new(
        service: String,
        path: String,
        method: String,
        headers: HashMap<String, String>,
    ) -> Self {
        Self {
            request_id: uuid::Uuid::new_v4(),
            service,
            path,
            method,
            headers,
            query: HashMap::new(),
            body: None,
            client_ip: None,
            user_id: None,
            start_time: std::time::Instant::now(),
        }
    }
}

// ==================== 认证请求 ====================

/// 登录请求
#[derive(Debug, Deserialize)]
pub struct LoginRequest {
    pub username: String,
    pub password: String,
}

/// 注册请求
#[derive(Debug, Deserialize)]
pub struct RegisterRequest {
    pub username: String,
    pub email: String,
    pub password: String,
}

/// 刷新 Token 请求
#[derive(Debug, Deserialize)]
pub struct RefreshTokenRequest {
    pub refresh_token: String,
}

// ==================== 通用请求 ====================

/// 通用分页请求
#[derive(Debug, Deserialize)]
pub struct PaginatedRequest {
    #[serde(default = "default_page")]
    pub page: u32,
    #[serde(default = "default_page_size")]
    pub page_size: u32,
}

fn default_page() -> u32 {
    1
}

fn default_page_size() -> u32 {
    20
}

/// 搜索请求
#[derive(Debug, Deserialize)]
pub struct SearchRequest {
    pub query: String,
    #[serde(default)]
    pub filters: HashMap<String, String>,
    #[serde(flatten)]
    pub pagination: PaginatedRequest,
}
```

### 6. 响应模型：src/models/response.rs
```rust
// models/response.rs - 响应相关模型

use axum::{http::StatusCode, response::{IntoResponse, Response}};
use serde::{Deserialize, Serialize};
use chrono::{DateTime, Utc};

// ==================== API 响应结构 ====================

/// 统一 API 响应格式
/// 所有 API 响应都使用这个格式
#[derive(Debug, Serialize)]
pub struct ApiResponse<T> {
    /// 响应码
    pub code: u16,
    /// 消息
    pub message: String,
    /// 数据
    #[serde(skip_serializing_if = "Option::is_none")]
    pub data: Option<T>,
    /// 请求 ID
    pub request_id: Option<String>,
    /// 时间戳
    pub timestamp: DateTime<Utc>,
}

impl<T> ApiResponse<T> {
    /// 创建成功响应
    pub fn success(data: T) -> Self {
        Self {
            code: 200,
            message: "成功".to_string(),
            data: Some(data),
            request_id: None,
            timestamp: Utc::now(),
        }
    }
    
    /// 创建成功响应（无数据）
    pub fn success_no_data() -> ApiResponse<()> {
        ApiResponse {
            code: 200,
            message: "成功".to_string(),
            data: None,
            request_id: None,
            timestamp: Utc::now(),
        }
    }
    
    /// 创建错误响应
    pub fn error(code: u16, message: impl Into<String>) -> ApiResponse<()> {
        ApiResponse {
            code,
            message: message.into(),
            data: None,
            request_id: None,
            timestamp: Utc::now(),
        }
    }
    
    /// 创建未授权响应
    pub fn unauthorized(message: impl Into<String>) -> ApiResponse<()> {
        Self::error(401, message)
    }
    
    /// 创建禁止响应
    pub fn forbidden(message: impl Into<String>) -> ApiResponse<()> {
        Self::error(403, message)
    }
    
    /// 创建未找到响应
    pub fn not_found(message: impl Into<String>) -> ApiResponse<()> {
        Self::error(404, message)
    }
    
    /// 创建内部错误响应
    pub fn internal_error(message: impl Into<String>) -> ApiResponse<()> {
        Self::error(500, message)
    }
    
    /// 添加请求 ID
    pub fn with_request_id(mut self, request_id: &str) -> Self {
        self.request_id = Some(request_id.to_string());
        self
    }
}

// ==================== 分页响应 ====================

/// 分页响应
#[derive(Debug, Serialize)]
pub struct PaginatedResponse<T> {
    /// 数据列表
    pub items: Vec<T>,
    /// 总数
    pub total: u64,
    /// 当前页
    pub page: u32,
    /// 每页数量
    pub page_size: u32,
    /// 总页数
    pub total_pages: u64,
}

impl<T> PaginatedResponse<T> {
    pub fn new(items: Vec<T>, total: u64, page: u32, page_size: u32) -> Self {
        let total_pages = if page_size > 0 {
            (total + page_size as u64 - 1) / page_size as u64
        } else {
            0
        };
        
        Self {
            items,
            total,
            page,
            page_size,
            total_pages,
        }
    }
}

// ==================== 错误类型 ====================

/// 网关错误类型
#[derive(Debug, thiserror::Error)]
pub enum GatewayError {
    #[error("认证失败: {0}")]
    Unauthorized(String),
    
    #[error("禁止访问: {0}")]
    Forbidden(String),
    
    #[error("资源未找到: {0}")]
    NotFound(String),
    
    #[error("请求验证失败: {0}")]
    ValidationError(String),
    
    #[error("限流: {0}")]
    RateLimited(String),
    
    #[error("熔断器打开: {0}")]
    CircuitOpen(String),
    
    #[error("上游服务错误: {0}")]
    UpstreamError(String),
    
    #[error("内部错误: {0}")]
    InternalError(String),
    
    #[error("超时: {0}")]
    Timeout(String),
    
    #[error("无效请求: {0}")]
    BadRequest(String),
}

impl IntoResponse for GatewayError {
    fn into_response(self) -> Response {
        let (status, message) = match &self {
            GatewayError::Unauthorized(msg) => (StatusCode::UNAUTHORIZED, msg.clone()),
            GatewayError::Forbidden(msg) => (StatusCode::FORBIDDEN, msg.clone()),
            GatewayError::NotFound(msg) => (StatusCode::NOT_FOUND, msg.clone()),
            GatewayError::ValidationError(msg) => (StatusCode::BAD_REQUEST, msg.clone()),
            GatewayError::RateLimited(msg) => (StatusCode::TOO_MANY_REQUESTS, msg.clone()),
            GatewayError::CircuitOpen(msg) => (StatusCode::SERVICE_UNAVAILABLE, msg.clone()),
            GatewayError::UpstreamError(msg) => (StatusCode::BAD_GATEWAY, msg.clone()),
            GatewayError::InternalError(msg) => (StatusCode::INTERNAL_SERVER_ERROR, msg.clone()),
            GatewayError::Timeout(msg) => (StatusCode::GATEWAY_TIMEOUT, msg.clone()),
            GatewayError::BadRequest(msg) => (StatusCode::BAD_REQUEST, msg.clone()),
        };
        
        let body = ApiResponse::<()>::error(status.as_u16(), message);
        
        (status, axum::Json(body)).into_response()
    }
}

// ==================== 认证响应 ====================

/// 登录响应
#[derive(Debug, Serialize)]
pub struct LoginResponse {
    pub access_token: String,
    pub refresh_token: String,
    pub expires_in: u64,
    pub token_type: String,
}

/// Token 信息
#[derive(Debug, Serialize)]
pub struct TokenInfo {
    pub user_id: String,
    pub username: String,
    pub roles: Vec<String>,
    pub exp: u64,
}
```

### 7. 限流器：src/middleware/rate_limiter.rs
```rust
// middleware/rate_limiter.rs - 限流器中间件
// 作用：实现基于 Token Bucket 算法的请求限流

use axum::{
    body::Body,
    extract::Request,
    middleware::Next,
    response::Response,
};
use governor::{
    clock::DefaultClock,
    middleware::NoOpMiddleware,
    state::{InMemoryState, NotKeyed},
    Quota, RateLimiter as GovRateLimiter,
};
use std::sync::Arc;
use std::time::Duration;
use tower::{Layer, Service};
use std::future::Future;
use std::pin::Pin;

// ==================== 限流器类型 ====================

/// 基于 Governor 的限流器
pub type RateLimiter = GovRateLimiter<NotKeyed, InMemoryState, DefaultClock, NoOpMiddleware>;

/// 创建限流器
/// rps: 每秒请求数
/// burst: 突发容量
pub fn create_rate_limiter(rps: u32, burst: u32) -> Arc<RateLimiter> {
    let quota = Quota::per_second(std::num::NonZeroU32::new(rps).unwrap())
        .allow_burst(std::num::NonZeroU32::new(burst).unwrap());
    
    Arc::new(GovRateLimiter::direct(quota))
}

// ==================== IP 限流器 ====================

/// IP 限流器存储
pub type IpRateLimiterStore = std::collections::HashMap<String, Arc<RateLimiter>>;

/// IP 限流器配置
#[derive(Debug, Clone)]
pub struct IpRateLimitConfig {
    pub rps: u32,
    pub burst: u32,
}

/// 创建 IP 限流器存储
pub fn create_ip_rate_limiter_store(config: &IpRateLimitConfig) -> Arc<parking_lot::RwLock<IpRateLimiterStore>> {
    Arc::new(parking_lot::RwLock::new(std::collections::HashMap::new()))
}

/// 获取或创建 IP 限流器
pub fn get_or_create_ip_limiter(
    store: &Arc<parking_lot::RwLock<IpRateLimiterStore>>,
    ip: &str,
    config: &IpRateLimitConfig,
) -> Arc<RateLimiter> {
    let mut guard = store.write();
    
    if let Some(limiter) = guard.get(ip) {
        return limiter.clone();
    }
    
    let limiter = create_rate_limiter(config.rps, config.burst);
    guard.insert(ip.to_string(), limiter.clone());
    limiter
}

// ==================== 限流中间件 ====================

/// 限流状态
#[derive(Clone)]
pub struct RateLimitState {
    pub limiter: Arc<RateLimiter>,
    pub enabled: bool,
}

impl RateLimitState {
    pub fn new(rps: u32, burst: u32, enabled: bool) -> Self {
        Self {
            limiter: create_rate_limiter(rps, burst),
            enabled,
        }
    }
}

/// 限流中间件处理函数
pub async fn rate_limit_middleware(
    State(state): State<RateLimitState>,
    request: Request,
    next: Next,
) -> Result<Response, std::convert::Infallible> {
    // 如果限流未启用，直接放行
    if !state.enabled {
        return Ok(next.run(request).await);
    }
    
    // 检查是否超过限制
    match state.limiter.check() {
        Ok(_) => {
            // 未超过限制，继续处理请求
            Ok(next.run(request).await)
        }
        Err(not_until) => {
            // 超过限制，返回 429
            let retry_after = not_until.wait_time_from(DefaultClock::default());
            let retry_secs = retry_after.as_secs();
            
            let mut response = Response::new(Body::from(
                serde_json::json!({
                    "code": 429,
                    "message": format!("请求过于频繁，请在 {} 秒后重试", retry_secs),
                })
                .to_string(),
            ));
            
            *response.status_mut() = axum::http::StatusCode::TOO_MANY_REQUESTS;
            response.headers_mut().insert(
                axum::http::header::RETRY_AFTER,
                axum::http::HeaderValue::from_secs(retry_secs),
            );
            
            Ok(response)
        }
    }
}

// ==================== Tower Layer 实现 ====================

/// 限流 Layer
#[derive(Clone)]
pub struct RateLimitLayer {
    state: RateLimitState,
}

impl RateLimitLayer {
    pub fn new(rps: u32, burst: u32, enabled: bool) -> Self {
        Self {
            state: RateLimitState::new(rps, burst, enabled),
        }
    }
}

impl<S> Layer<S> for RateLimitLayer {
    type Service = RateLimitService<S>;
    
    fn layer(&self, inner: S) -> Self::Service {
        RateLimitService {
            inner,
            state: self.state.clone(),
        }
    }
}

/// 限流服务包装器
#[derive(Clone)]
pub struct RateLimitService<S> {
    inner: S,
    state: RateLimitState,
}

impl<S, B> Service<Request<B>> for RateLimitService<S>
where
    S: Service<Request<B>, Response = Response> + Clone + Send + 'static,
    S::Future: Send + 'static,
    B: Send + 'static,
{
    type Response = Response;
    type Error = std::convert::Infallible;
    type Future = Pin<Box<dyn Future<Output = Result<Self::Response, Self::Error>> + Send>>;
    
    fn poll_ready(
        &mut self,
        cx: &mut std::task::Context<'_>,
    ) -> std::task::Poll<Result<(), Self::Error>> {
        self.inner.poll_ready(cx)
    }
    
    fn call(&mut self, request: Request<B>) -> Self::Future {
        // 如果限流未启用，直接转发
        if !self.state.enabled {
            let mut inner = self.inner.clone();
            let future = inner.call(request);
            return Box::pin(async move {
                Ok(future.await.unwrap_or_else(|_| {
                    Response::builder()
                        .status(500)
                        .body(Body::empty())
                        .unwrap()
                }))
            });
        }
        
        // 检查限流
        match self.state.limiter.check() {
            Ok(_) => {
                // 未超过限制
                let mut inner = self.inner.clone();
                let future = inner.call(request);
                Box::pin(async move {
                    Ok(future.await.unwrap_or_else(|_| {
                        Response::builder()
                            .status(500)
                            .body(Body::empty())
                            .unwrap()
                    }))
                })
            }
            Err(not_until) => {
                // 超过限制
                let retry_after = not_until.wait_time_from(DefaultClock::default());
                let retry_secs = retry_after.as_secs();
                
                let response = Response::builder()
                    .status(axum::http::StatusCode::TOO_MANY_REQUESTS)
                    .header(axum::http::header::RETRY_AFTER, retry_secs.to_string())
                    .body(Body::from(
                        serde_json::json!({
                            "code": 429,
                            "message": format!("Rate limit exceeded. Retry after {} seconds", retry_secs),
                        })
                        .to_string(),
                    ))
                    .unwrap();
                
                Box::pin(async move { Ok(response) })
            }
        }
    }
}
```

### 8. 熔断器：src/middleware/circuit_breaker.rs
```rust
// middleware/circuit_breaker.rs - 熔断器实现
// 作用：实现熔断器模式，防止级联故障

use std::sync::atomic::{AtomicU64, Ordering};
use std::sync::Arc;
use std::time::{Duration, Instant};
use tokio::sync::RwLock;
use parking_lot::Mutex;

// ==================== 熔断器状态 ====================

/// 熔断器状态
#[derive(Debug, Clone, Copy, PartialEq, Eq)]
pub enum CircuitState {
    /// 关闭状态：正常请求
    Closed,
    /// 打开状态：拒绝请求
    Open,
    /// 半开状态：尝试放行一个请求测试
    HalfOpen,
}

impl std::fmt::Display for CircuitState {
    fn fmt(&self, f: &mut std::fmt::Formatter<'_>) -> std::fmt::Result {
        match self {
            CircuitState::Closed => write!(f, "Closed"),
            CircuitState::Open => write!(f, "Open"),
            CircuitState::HalfOpen => write!(f, "HalfOpen"),
        }
    }
}

// ==================== 熔断器配置 ====================

/// 熔断器配置
#[derive(Debug, Clone)]
pub struct CircuitBreakerConfig {
    /// 失败次数阈值
    pub failure_threshold: u32,
    /// 成功次数阈值（半开状态下）
    pub success_threshold: u32,
    /// 熔断超时时间
    pub timeout: Duration,
}

impl CircuitBreakerConfig {
    pub fn new(failure_threshold: u32, success_threshold: u32, timeout_secs: u64) -> Self {
        Self {
            failure_threshold,
            success_threshold,
            timeout: Duration::from_secs(timeout_secs),
        }
    }
}

impl Default for CircuitBreakerConfig {
    fn default() -> Self {
        Self {
            failure_threshold: 5,
            success_threshold: 2,
            timeout: Duration::from_secs(60),
        }
    }
}

// ==================== 熔断器 ====================

/// 熔断器内部状态
struct CircuitBreakerInner {
    /// 当前状态
    state: CircuitState,
    /// 失败计数
    failure_count: u32,
    /// 成功计数（半开状态）
    success_count: u32,
    /// 最后失败时间
    last_failure_time: Option<Instant>,
    /// 最后状态变更时间
    last_state_change: Instant,
}

impl CircuitBreakerInner {
    fn new() -> Self {
        Self {
            state: CircuitState::Closed,
            failure_count: 0,
            success_count: 0,
            last_failure_time: None,
            last_state_change: Instant::now(),
        }
    }
}

/// 熔断器
pub struct CircuitBreaker {
    config: CircuitBreakerConfig,
    inner: Arc<Mutex<CircuitBreakerInner>>,
}

impl CircuitBreaker {
    /// 创建新的熔断器
    pub fn new(config: CircuitBreakerConfig) -> Self {
        Self {
            config,
            inner: Arc::new(Mutex::new(CircuitBreakerInner::new())),
        }
    }
    
    /// 获取当前状态
    pub fn state(&self) -> CircuitState {
        let inner = self.inner.lock();
        inner.state
    }
    
    /// 检查是否允许请求
    pub fn is_allowed(&self) -> bool {
        let mut inner = self.inner.lock();
        
        match inner.state {
            CircuitState::Closed => true,
            CircuitState::Open => {
                // 检查超时
                let elapsed = inner.last_state_change.elapsed();
                if elapsed >= self.config.timeout {
                    // 超时，切换到半开状态
                    inner.state = CircuitState::HalfOpen;
                    inner.success_count = 0;
                    inner.last_state_change = Instant::now();
                    true
                } else {
                    false
                }
            }
            CircuitState::HalfOpen => true,
        }
    }
    
    /// 记录成功
    pub fn record_success(&self) {
        let mut inner = self.inner.lock();
        
        match inner.state {
            CircuitState::Closed => {
                // 成功后重置失败计数
                inner.failure_count = 0;
            }
            CircuitState::HalfOpen => {
                inner.success_count += 1;
                if inner.success_count >= self.config.success_threshold {
                    // 连续成功，关闭熔断器
                    inner.state = CircuitState::Closed;
                    inner.failure_count = 0;
                    inner.success_count = 0;
                    inner.last_state_change = Instant::now();
                }
            }
            CircuitState::Open => {
                // 不应该发生
            }
        }
    }
    
    /// 记录失败
    pub fn record_failure(&self) {
        let mut inner = self.inner.lock();
        
        match inner.state {
            CircuitState::Closed => {
                inner.failure_count += 1;
                inner.last_failure_time = Some(Instant::now());
                
                if inner.failure_count >= self.config.failure_threshold {
                    // 失败过多，打开熔断器
                    inner.state = CircuitState::Open;
                    inner.last_state_change = Instant::now();
                }
            }
            CircuitState::HalfOpen => {
                // 半开状态下失败，立即打开
                inner.state = CircuitState::Open;
                inner.last_state_change = Instant::now();
                inner.success_count = 0;
            }
            CircuitState::Open => {
                // 不应该发生
            }
        }
    }
    
    /// 获取指标
    pub fn metrics(&self) -> CircuitBreakerMetrics {
        let inner = self.inner.lock();
        CircuitBreakerMetrics {
            state: inner.state,
            failure_count: inner.failure_count,
            success_count: inner.success_count,
            last_failure_time: inner.last_failure_time,
        }
    }
}

/// 熔断器指标
#[derive(Debug, Clone)]
pub struct CircuitBreakerMetrics {
    pub state: CircuitState,
    pub failure_count: u32,
    pub success_count: u32,
    pub last_failure_time: Option<Instant>,
}

// ==================== 熔断器存储 ====================

/// 熔断器存储（按服务名）
pub type CircuitBreakerStore = Arc<RwLock<std::collections::HashMap<String, CircuitBreaker>>>;

/// 创建熔断器存储
pub fn create_breaker_store() -> CircuitBreakerStore {
    Arc::new(RwLock::new(std::collections::HashMap::new()))
}

/// 获取或创建熔断器
pub fn get_or_create_breaker(
    store: &CircuitBreakerStore,
    service: &str,
    config: &CircuitBreakerConfig,
) -> CircuitBreaker {
    // 尝试读取
    if let Some(breaker) = store.read().get(service).cloned() {
        return breaker;
    }
    
    // 创建新的
    let breaker = CircuitBreaker::new(config.clone());
    store.write().insert(service.to_string(), breaker.clone());
    breaker
}

// ==================== 熔断器包装器（用于异步操作） ====================

/// 熔断器异步包装器
pub async fn with_circuit_breaker<F, T, E>(
    breaker: &CircuitBreaker,
    operation: F,
) -> Result<T, CircuitBreakerError<E>>
where
    F: std::future::Future<Output = Result<T, E>>,
{
    // 检查是否允许请求
    if !breaker.is_allowed() {
        return Err(CircuitBreakerError::CircuitOpen);
    }
    
    // 执行操作
    match operation.await {
        Ok(result) => {
            breaker.record_success();
            Ok(result)
        }
        Err(e) => {
            breaker.record_failure();
            Err(CircuitBreakerError::UpstreamError(e))
        }
    }
}

/// 熔断器错误类型
#[derive(Debug, thiserror::Error)]
pub enum CircuitBreakerError<E> {
    #[error("熔断器打开")]
    CircuitOpen,
    
    #[error("上游服务错误: {0}")]
    UpstreamError(#[from] E),
}
```

### 9. 认证中间件：src/middleware/auth.rs
```rust
// middleware/auth.rs - 认证中间件
// 作用：JWT 令牌验证和用户认证

use axum::{
    body::Body,
    extract::Request,
    http::{StatusCode, HeaderMap},
    middleware::Next,
    response::{IntoResponse, Response},
    Extension,
};
use jsonwebtoken::{decode, encode, DecodingKey, EncodingKey, Header, Validation};
use serde::{Deserialize, Serialize};
use std::sync::Arc;

// ==================== JWT 声明 ====================

/// JWT 声明
#[derive(Debug, Serialize, Deserialize, Clone)]
pub struct Claims {
    /// 用户 ID
    pub sub: String,
    /// 用户名
    pub username: String,
    /// 角色
    pub roles: Vec<String>,
    /// 签发者
    pub iss: String,
    /// 过期时间
    pub exp: usize,
    /// 签发时间
    pub iat: usize,
}

impl Claims {
    /// 创建新声明
    pub fn new(user_id: &str, username: &str, roles: Vec<String>, expiry_hours: u64) -> Self {
        let now = chrono::Utc::now();
        let exp = now.timestamp() as usize + (expiry_hours * 3600) as usize;
        
        Self {
            sub: user_id.to_string(),
            username: username.to_string(),
            roles,
            iss: "api-gateway".to_string(),
            exp,
            iat: now.timestamp() as usize,
        }
    }
}

// ==================== JWT 服务 ====================

/// JWT 服务
#[derive(Clone)]
pub struct JwtService {
    encoding_key: EncodingKey,
    decoding_key: DecodingKey,
    validation: Validation,
}

impl JwtService {
    /// 创建新的 JWT 服务
    pub fn new(secret: &str, issuer: &str) -> Self {
        let encoding_key = EncodingKey::from_secret(secret.as_bytes());
        let decoding_key = DecodingKey::from_secret(secret.as_bytes());
        
        let mut validation = Validation::default();
        validation.set_issuer(&[issuer]);
        validation.validate_exp = true;
        
        Self {
            encoding_key,
            decoding_key,
            validation,
        }
    }
    
    /// 生成 Token
    pub fn generate_token(&self, claims: &Claims) -> Result<String, jsonwebtoken::errors::Error> {
        encode(&Header::default(), claims, &self.encoding_key)
    }
    
    /// 验证 Token
    pub fn verify_token(&self, token: &str) -> Result<Claims, jsonwebtoken::errors::Error> {
        let token_data = decode::<Claims>(token, &self.decoding_key, &self.validation)?;
        Ok(token_data.claims)
    }
}

// ==================== 用户上下文 ====================

/// 已认证用户信息
#[derive(Debug, Clone)]
pub struct AuthenticatedUser {
    pub user_id: String,
    pub username: String,
    pub roles: Vec<String>,
}

impl From<Claims> for AuthenticatedUser {
    fn from(claims: Claims) -> Self {
        Self {
            user_id: claims.sub,
            username: claims.username,
            roles: claims.roles,
        }
    }
}

// ==================== 认证中间件 ====================

/// 认证状态
#[derive(Clone)]
pub struct AuthState {
    jwt_service: JwtService,
    enabled: bool,
}

impl AuthState {
    pub fn new(secret: &str, issuer: &str, enabled: bool) -> Self {
        Self {
            jwt_service: JwtService::new(secret, issuer),
            enabled,
        }
    }
}

/// 从请求中提取 Token
fn extract_token(headers: &HeaderMap) -> Option<String> {
    // 尝试从 Authorization header 提取
    if let Some(auth_header) = headers.get("authorization") {
        if let Ok(auth_str) = auth_header.to_str() {
            if auth_str.starts_with("Bearer ") {
                return Some(auth_str[7..].to_string());
            }
        }
    }
    
    // 尝试从 query parameter 提取
    None
}

/// 认证中间件处理函数
pub async fn auth_middleware(
    State(state): State<AuthState>,
    mut request: Request,
    next: Next,
) -> Response {
    // 如果认证未启用，直接放行
    if !state.enabled {
        return next.run(request).await;
    }
    
    // 提取 Token
    let token = match extract_token(request.headers()) {
        Some(t) => t,
        None => {
            return auth_error("Missing authorization token", StatusCode::UNAUTHORIZED);
        }
    };
    
    // 验证 Token
    match state.jwt_service.verify_token(&token) {
        Ok(claims) => {
            // 认证成功，将用户信息添加到请求扩展
            let user: AuthenticatedUser = claims.into();
            request.extensions_mut().insert(user);
            next.run(request).await
        }
        Err(e) => {
            tracing::warn!("Token verification failed: {:?}", e);
            auth_error("Invalid token", StatusCode::UNAUTHORIZED)
        }
    }
}

/// 返回认证错误
fn auth_error(message: &str, status: StatusCode) -> Response {
    let body = serde_json::json!({
        "code": status.as_u16(),
        "message": message,
    });
    
    (status, axum::Json(body)).into_response()
}

// ==================== 可选认证中间件 ====================

/// 可选认证中间件（Token 存在时验证，不存在也放行）
pub async fn optional_auth_middleware(
    State(state): State<AuthState>,
    mut request: Request,
    next: Next,
) -> Response {
    // 如果认证未启用，直接放行
    if !state.enabled {
        return next.run(request).await;
    }
    
    // 提取 Token
    if let Some(token) = extract_token(request.headers()) {
        if let Ok(claims) = state.jwt_service.verify_token(&token) {
            let user: AuthenticatedUser = claims.into();
            request.extensions_mut().insert(user);
        }
    }
    
    next.run(request).await
}

// ==================== 角色检查中间件 ====================

/// 要求特定角色的中间件
pub fn require_role(required_role: &str) -> impl Fn(Request, Next) -> Response + Clone {
    move |request: Request, next: Next| {
        let user = request.extensions().get::<AuthenticatedUser>();
        
        if let Some(user) = user {
            if user.roles.contains(&required_role.to_string()) {
                return next.run(request).await;
            }
            return auth_error("Insufficient permissions", StatusCode::FORBIDDEN);
        }
        
        auth_error("Authentication required", StatusCode::UNAUTHORIZED)
    }
}
```

### 10. 日志中间件：src/middleware/logging.rs
```rust
// middleware/logging.rs - 日志和追踪中间件
// 作用：请求日志、追踪和指标收集

use axum::{
    body::Body,
    extract::Request,
    http::{HeaderName, HeaderValue},
    middleware::Next,
    response::Response,
};
use std::time::Instant;
use tracing::Span;
use uuid::Uuid;

// ==================== 请求 ID 生成 ====================

/// 请求 ID header 名称
pub const REQUEST_ID_HEADER: &str = "x-request-id";

/// 生成请求 ID
pub fn generate_request_id() -> String {
    Uuid::new_v4().to_string()
}

/// 从请求中获取或生成请求 ID
pub fn get_or_create_request_id(request: &Request) -> String {
    // 尝试从 header 获取
    if let Some(id) = request.headers().get(REQUEST_ID_HEADER) {
        if let Ok(id_str) = id.to_str() {
            return id_str.to_string();
        }
    }
    
    // 生成新的
    generate_request_id()
}

// ==================== 日志中间件 ====================

/// 请求日志信息
#[derive(Debug, Clone)]
pub struct RequestLog {
    pub request_id: String,
    pub method: String,
    pub path: String,
    pub query: Option<String>,
    pub client_ip: Option<String>,
    pub user_agent: Option<String>,
    pub user_id: Option<String>,
    pub start_time: Instant,
    pub duration_ms: u64,
    pub status_code: u16,
    pub response_size: Option<u64>,
}

impl RequestLog {
    pub fn from_request(request: &Request, request_id: String) -> Self {
        let uri = request.uri();
        let headers = request.headers();
        
        Self {
            request_id,
            method: request.method().to_string(),
            path: uri.path().to_string(),
            query: uri.query().map(|s| s.to_string()),
            client_ip: headers
                .get("x-forwarded-for")
                .and_then(|v| v.to_str().ok())
                .map(|s| s.to_string())
                .or_else(|| {
                    headers
                        .get("x-real-ip")
                        .and_then(|v| v.to_str().ok())
                        .map(|s| s.to_string())
                }),
            user_agent: headers
                .get("user-agent")
                .and_then(|v| v.to_str().ok())
                .map(|s| s.to_string()),
            user_id: None, // 从认证中间件获取
            start_time: Instant::now(),
            duration_ms: 0,
            status_code: 0,
            response_size: None,
        }
    }
    
    /// 完成日志记录
    pub fn finish(&mut self, status_code: u16, response_size: Option<u64>) {
        self.duration_ms = self.start_time.elapsed().as_millis() as u64;
        self.status_code = status_code;
        self.response_size = response_size;
    }
}

/// 日志中间件处理函数
pub async fn logging_middleware(
    request: Request,
    next: Next,
) -> Response {
    let request_id = get_or_create_request_id(&request);
    let mut log = RequestLog::from_request(&request, request_id.clone());
    
    // 创建追踪 Span
    let span = tracing::info_span!(
        "http_request",
        request_id = %request_id,
        method = %request.method(),
        path = %request.uri().path(),
    );
    
    let _guard = span.enter();
    
    // 执行请求
    let response = next.run(request).await;
    
    // 获取响应信息
    let status_code = response.status().as_u16();
    let response_size = response.body().and_then(|body| {
        // 注意：这只是估算
        Some(body.size_hint().upper().unwrap_or(0))
    });
    
    // 完成日志
    log.finish(status_code, response_size);
    
    // 记录日志
    tracing::info!(
        request_id = %log.request_id,
        method = %log.method,
        path = %log.path,
        status = %log.status_code,
        duration_ms = %log.duration_ms,
        client_ip = ?log.client_ip,
        "HTTP request completed"
    );
    
    // 添加 request ID 到响应 header
    let mut response = response;
    response.headers_mut().insert(
        HeaderName::from_static(REQUEST_ID_HEADER),
        HeaderValue::from_str(&request_id).unwrap_or_else(|_| HeaderValue::from_static("")),
    );
    
    response
}

// ==================== 追踪中间件 ====================

/// OpenTelemetry 追踪中间件
pub async fn tracing_middleware(
    request: Request,
    next: Next,
) -> Response {
    let request_id = get_or_create_request_id(&request);
    
    // 创建 OpenTelemetry Span
    let span = tracing::info_span!(
        "HTTP Request",
        "otel.name" = format!("{} {}", request.method(), request.uri()),
        "http.method" = %request.method(),
        "http.url" = %request.uri(),
        "http.request_id" = %request_id,
    );
    
    let _guard = span.enter();
    
    let response = next.run(request).await;
    
    // 添加追踪信息到响应
    span.record("http.status_code", response.status().as_u16());
    
    response
}
```

### 11. 上游服务调用：src/services/upstream.rs
```rust
// services/upstream.rs - 上游服务调用
// 作用：实现到后端服务的代理调用

use crate::config::UpstreamConfig;
use crate::middleware::circuit_breaker::{CircuitBreaker, CircuitBreakerConfig, with_circuit_breaker};
use crate::models::{GatewayError, ProxyRequest, Result};
use axum::{
    body::Body,
    http::{Request, Method, HeaderMap, Uri},
    Response,
};
use reqwest::Client;
use std::sync::Arc;
use std::time::Duration;
use tokio::sync::RwLock;

// ==================== 上游服务客户端 ====================

/// 上游服务客户端
pub struct UpstreamClient {
    client: Client,
    base_url: String,
    timeout: Duration,
    circuit_breaker: Option<CircuitBreaker>,
}

impl UpstreamClient {
    /// 创建新的上游服务客户端
    pub fn new(config: &UpstreamConfig) -> Self {
        let client = Client::builder()
            .timeout(config.timeout)
            .pool_max_idle_per_host(10)
            .build()
            .expect("Failed to create HTTP client");
        
        Self {
            client,
            base_url: config.url.clone(),
            timeout: config.timeout,
            circuit_breaker: Some(CircuitBreaker::new(CircuitBreakerConfig::default())),
        }
    }
    
    /// 创建无熔断器的客户端
    pub fn new_without_breaker(config: &UpstreamConfig) -> Self {
        let client = Client::builder()
            .timeout(config.timeout)
            .build()
            .expect("Failed to create HTTP client");
        
        Self {
            client,
            base_url: config.url.clone(),
            timeout: config.timeout,
            circuit_breaker: None,
        }
    }
    
    /// 代理请求到上游服务
    pub async fn proxy(&self, request: ProxyRequest) -> Result<Response<Body>> {
        let url = format!("{}{}", self.base_url, request.path);
        
        // 构建新的请求
        let mut req_builder = self.client.request(
            Method::from_bytes(request.method.as_bytes()).unwrap(),
            &url,
        );
        
        // 添加请求头（过滤 hop-by-hop headers）
        for (key, value) in &request.headers {
            let key_lower = key.to_lowercase();
            if !is_hop_by_hop_header(&key_lower) {
                req_builder = req_builder.header(key.as_str(), value.as_str());
            }
        }
        
        // 添加认证信息
        if let Some(user_id) = &request.user_id {
            req_builder = req_builder.header("x-user-id", user_id);
        }
        
        // 添加请求 ID
        req_builder = req_builder.header("x-request-id", request.request_id.to_string());
        
        // 添加查询参数
        if !request.query.is_empty() {
            let query_str: String = request.query
                .iter()
                .map(|(k, v)| format!("{}={}", urlencoding::encode(k), urlencoding::encode(v)))
                .collect::<Vec<_>>()
                .join("&");
            req_builder = req_builder.query(&request.query);
        }
        
        // 添加请求体
        if let Some(body) = request.body {
            req_builder = req_builder.body(body);
        }
        
        // 执行请求（使用熔断器）
        let response = if let Some(breaker) = &self.circuit_breaker {
            with_circuit_breaker(breaker, async {
                req_builder.send().await.map_err(|e| {
                    if e.is_timeout() {
                        GatewayError::Timeout(e.to_string())
                    } else {
                        GatewayError::UpstreamError(e.to_string())
                    }
                })
            })
            .await
            .map_err(|e| match e {
                crate::middleware::circuit_breaker::CircuitBreakerError::CircuitOpen => 
                    GatewayError::CircuitOpen(self.base_url.clone()),
                crate::middleware::circuit_breaker::CircuitBreakerError::UpstreamError(e) => e,
            })?
        } else {
            req_builder.send().await.map_err(|e| {
                if e.is_timeout() {
                    GatewayError::Timeout(e.to_string())
                } else {
                    GatewayError::UpstreamError(e.to_string())
                }
            })?
        };
        
        // 转换响应
        let status = response.status();
        let headers = response.headers().clone();
        let body = response.bytes().await.map_err(|e| 
            GatewayError::UpstreamError(e.to_string())
        )?;
        
        // 构建响应
        let mut resp_builder = Response::builder().status(status);
        
        for (key, value) in headers.iter() {
            let key_lower = key.as_str().to_lowercase();
            if !is_hop_by_hop_header(&key_lower) {
                resp_builder = resp_builder.header(key.as_str(), value.as_str());
            }
        }
        
        Ok(resp_builder.body(Body::from(body.to_vec())).unwrap())
    }
}

/// 检查是否是 hop-by-hop header
fn is_hop_by_hop_header(header: &str) -> bool {
    matches!(
        header,
        "connection"
            | "keep-alive"
            | "proxy-authenticate"
            | "proxy-authorization"
            | "te"
            | "trailers"
            | "transfer-encoding"
            | "upgrade"
            | "host"
    )
}

// ==================== 上游服务存储 ====================

/// 上游服务存储
pub type UpstreamStore = Arc<RwLock<std::collections::HashMap<String, UpstreamClient>>>;

/// 创建上游服务存储
pub fn create_upstream_store() -> UpstreamStore {
    Arc::new(RwLock::new(std::collections::HashMap::new()))
}

/// 获取或创建上游服务客户端
pub async fn get_or_create_upstream(
    store: &UpstreamStore,
    name: &str,
    config: Option<&UpstreamConfig>,
) -> Option<UpstreamClient> {
    // 尝试读取
    if let Some(client) = store.read().get(name).cloned() {
        return Some(client);
    }
    
    // 如果没有配置，无法创建
    let config = config?;
    
    // 创建新的
    let client = UpstreamClient::new(config);
    store.write().await.insert(name.to_string(), client.clone());
    
    Some(client)
}
```

### 12. 路由定义：src/routes/proxy.rs
```rust
// routes/proxy.rs - 代理路由
// 作用：处理到后端服务的代理请求

use crate::models::{ProxyRequest, Result};
use crate::services::upstream::{UpstreamStore, get_or_create_upstream};
use crate::config::Config;
use axum::{
    extract::{Path, State, Extension, Query},
    http::{HeaderMap, Method, Request},
    body::Body,
    response::Response,
    Router,
};
use std::collections::HashMap;
use std::sync::Arc;
use tower::ServiceBuilder;
use std::convert::Infallible;

// ==================== 代理状态 ====================

#[derive(Clone)]
pub struct ProxyState {
    pub upstream_store: UpstreamStore,
    pub config: Arc<Config>,
}

// ==================== 路由工厂 ====================

/// 创建代理路由
pub fn create_proxy_routes(state: ProxyState) -> Router {
    Router::new()
        // 用户服务代理
        .route("/api/user/*path", axum::routing::any(proxy_handler))
        .route("/api/user", axum::routing::any(proxy_handler))
        // 订单服务代理
        .route("/api/order/*path", axum::routing::any(proxy_handler))
        .route("/api/order", axum::routing::any(proxy_handler))
        // 商品服务代理
        .route("/api/product/*path", axum::routing::any(proxy_handler))
        .route("/api/product", axum::routing::any(proxy_handler))
        .with_state(state)
}

/// 代理请求处理函数
async fn proxy_handler(
    // 路径参数
    Path((service, path)): Path<(String, String)>,
    // 查询参数
    Query(query): Query<HashMap<String, String>>,
    // 请求方法
    method: Method,
    // 请求头
    headers: HeaderMap,
    // 请求体
    request: Request<Body>,
    // 状态
    State(state): State<ProxyState>,
    // 用户信息（如果已认证）
    Extension(user): Extension<Option<crate::middleware::auth::AuthenticatedUser>>,
) -> Result<Response> {
    // 确定目标服务
    let service_name = match service.as_str() {
        "user" => "user",
        "order" => "order",
        "product" => "product",
        _ => return Err(crate::models::GatewayError::NotFound(format!("Unknown service: {}", service))),
    };
    
    // 获取客户端
    let config = state.config.get_upstream_url(service_name);
    let client = get_or_create_upstream(&state.upstream_store, service_name, config.as_deref()).await;
    let client = client.ok_or_else(|| 
        crate::models::GatewayError::UpstreamError(format!("Service {} not configured", service_name))
    )?;
    
    // 构建目标路径
    let target_path = if path.is_empty() {
        "/".to_string()
    } else {
        format!("/{}", path)
    };
    
    // 转换请求头
    let mut req_headers = HashMap::new();
    for (key, value) in headers.iter() {
        if let (Ok(k), Ok(v)) = (key.to_str(), value.to_str()) {
            req_headers.insert(k.to_string(), v.to_string());
        }
    }
    
    // 创建代理请求
    let mut proxy_request = ProxyRequest::new(
        service_name.to_string(),
        target_path,
        method.to_string(),
        req_headers,
    );
    
    // 设置查询参数
    proxy_request.query = query;
    
    // 设置用户信息
    if let Some(user) = user {
        proxy_request.user_id = Some(user.user_id);
    }
    
    // 代理请求
    client.proxy(proxy_request).await
}

// ==================== 动态代理路由 ====================

/// 动态代理路由（根据路径中的服务名）
pub async fn dynamic_proxy(
    Path(path_segments): Path<Vec<String>>,
    method: Method,
    headers: HeaderMap,
    request: Request<Body>,
    State(state): State<ProxyState>,
) -> Result<Response> {
    // 路径格式: /{service}/{path...}
    if path_segments.is_empty() {
        return Err(crate::models::GatewayError::BadRequest("Missing service name".to_string()));
    }
    
    let service = &path_segments[0];
    let path = if path_segments.len() > 1 {
        path_segments[1..].join("/")
    } else {
        String::new()
    };
    
    // 创建代理请求
    let mut req_headers = HashMap::new();
    for (key, value) in headers.iter() {
        if let (Ok(k), Ok(v)) = (key.to_str(), value.to_str()) {
            req_headers.insert(k.to_string(), v.to_string());
        }
    }
    
    let proxy_request = ProxyRequest::new(
        service.clone(),
        format!("/{}", path),
        method.to_string(),
        req_headers,
    );
    
    // 获取客户端
    let config = state.config.get_upstream_url(service);
    let client = get_or_create_upstream(&state.upstream_store, service, config.as_deref()).await;
    let client = client.ok_or_else(|| 
        crate::models::GatewayError::UpstreamError(format!("Service {} not configured", service))
    )?;
    
    client.proxy(proxy_request).await
}
```

### 13. 认证路由：src/routes/auth.rs
```rust
// routes/auth.rs - 认证路由
// 作用：处理登录、注册、Token 刷新等认证相关请求

use crate::middleware::auth::{AuthState, Claims, JwtService, AuthenticatedUser};
use crate::models::{ApiResponse, GatewayError, LoginRequest, LoginResponse, RegisterRequest, RefreshTokenRequest};
use axum::{
    extract::{Extension, State},
    http::StatusCode,
    response::IntoResponse,
    routing::{post, get},
    Router,
};
use serde::Serialize;

// ==================== 认证状态 ====================

#[derive(Clone)]
pub struct AuthRouteState {
    pub jwt_service: JwtService,
    pub state: AuthState,
}

// ==================== 路由工厂 ====================

/// 创建认证路由
pub fn create_auth_routes(state: AuthRouteState) -> Router {
    Router::new()
        .route("/login", post(login_handler))
        .route("/register", post(register_handler))
        .route("/refresh", post(refresh_handler))
        .route("/logout", post(logout_handler))
        .route("/me", get(me_handler))
        .with_state(state)
}

// ==================== 处理器 ====================

/// 登录处理
async fn login_handler(
    State(state): State<AuthRouteState>,
    Json(request): Json<LoginRequest>,
) -> Result<impl IntoResponse, GatewayError> {
    // 验证用户（实际应用中应该查询数据库）
    // 这里简化处理，使用硬编码验证
    if request.username == "admin" && request.password == "admin123" {
        // 生成 Token
        let claims = Claims::new(
            "1",
            &request.username,
            vec!["admin".to_string(), "user".to_string()],
            24, // 24 小时
        );
        
        let access_token = state.jwt_service.generate_token(&claims)
            .map_err(|e| GatewayError::InternalError(e.to_string()))?;
        
        // 生成刷新 Token（简化处理）
        let refresh_claims = Claims::new(
            "1",
            &request.username,
            vec![],
            7 * 24, // 7 天
        );
        
        let refresh_token = state.jwt_service.generate_token(&refresh_claims)
            .map_err(|e| GatewayError::InternalError(e.to_string()))?;
        
        let response = LoginResponse {
            access_token,
            refresh_token,
            expires_in: 24 * 3600,
            token_type: "Bearer".to_string(),
        };
        
        Ok((StatusCode::OK, ApiResponse::success(response)))
    } else {
        Err(GatewayError::Unauthorized("Invalid username or password".to_string()))
    }
}

/// 注册处理
async fn register_handler(
    State(_state): State<AuthRouteState>,
    Json(request): Json<RegisterRequest>,
) -> Result<impl IntoResponse, GatewayError> {
    // 实际应用中应该：
    // 1. 验证输入
    // 2. 检查用户名是否已存在
    // 3. 哈希密码
    // 4. 保存到数据库
    // 5. 生成用户 ID
    // 6. 返回 Token
    
    // 简化处理
    tracing::info!("User registration: {}", request.username);
    
    Ok((
        StatusCode::CREATED,
        ApiResponse::<()>::success_no_data().with_request_id("mock-request-id"),
    ))
}

/// Token 刷新处理
async fn refresh_handler(
    State(state): State<AuthRouteState>,
    Json(request): Json<RefreshTokenRequest>,
) -> Result<impl IntoResponse, GatewayError> {
    // 验证刷新 Token
    let claims = state.jwt_service.verify_token(&request.refresh_token)
        .map_err(|_| GatewayError::Unauthorized("Invalid refresh token".to_string()))?;
    
    // 生成新的 Access Token
    let new_claims = Claims::new(
        &claims.sub,
        &claims.username,
        claims.roles,
        24,
    );
    
    let access_token = state.jwt_service.generate_token(&new_claims)
        .map_err(|e| GatewayError::InternalError(e.to_string()))?;
    
    let response = LoginResponse {
        access_token,
        refresh_token: request.refresh_token, // 返回原刷新 Token
        expires_in: 24 * 3600,
        token_type: "Bearer".to_string(),
    };
    
    Ok((StatusCode::OK, ApiResponse::success(response)))
}

/// 登出处理
async fn logout_handler(
    Extension(_user): Extension<AuthenticatedUser>,
) -> Result<impl IntoResponse, GatewayError> {
    // 实际应用中应该：
    // 1. 将 Token 加入黑名单
    // 2. 或者从 Redis 中删除
    
    Ok((StatusCode::OK, ApiResponse::<()>::success_no_data()))
}

/// 获取当前用户信息
async fn me_handler(
    Extension(user): Extension<AuthenticatedUser>,
) -> impl IntoResponse {
    ApiResponse::success(user)
}
```

### 14. 健康检查路由：src/routes/health.rs
```rust
// routes/health.rs - 健康检查路由
// 作用：提供健康检查和就绪检查端点

use crate::models::ApiResponse;
use axum::{
    extract::State,
    response::IntoResponse,
    routing::{get, MethodRouter},
    Router,
};
use serde::Serialize;
use std::sync::Arc;

/// 健康检查状态
#[derive(Clone)]
pub struct HealthState {
    pub version: String,
    pub start_time: std::time::Instant,
}

/// 健康信息
#[derive(Serialize)]
pub struct HealthInfo {
    pub status: String,
    pub version: String,
    pub uptime_seconds: u64,
    pub memory_usage_mb: Option<f64>,
}

/// 就绪信息
#[derive(Serialize)]
pub struct ReadinessInfo {
    pub ready: bool,
    pub upstream_services: Vec<ServiceStatus>,
}

/// 服务状态
#[derive(Serialize)]
pub struct ServiceStatus {
    pub name: String,
    pub available: bool,
}

/// 创建健康检查路由
pub fn create_health_routes(state: HealthState) -> Router {
    Router::new()
        .route("/health", get(health_handler))
        .route("/ready", get(readiness_handler))
        .with_state(state)
}

/// 健康检查处理器
async fn health_handler(
    State(state): State<HealthState>,
) -> impl IntoResponse {
    let uptime = state.start_time.elapsed().as_secs();
    
    let info = HealthInfo {
        status: "healthy".to_string(),
        version: state.version.clone(),
        uptime_seconds: uptime,
        memory_usage_mb: Some(get_memory_usage()),
    };
    
    ApiResponse::success(info)
}

/// 就绪检查处理器
async fn readiness_handler(
    State(_state): State<HealthState>,
) -> impl IntoResponse {
    // 实际应用中应该检查：
    // 1. 数据库连接
    // 2. Redis 连接
    // 3. 上游服务健康状态
    
    let info = ReadinessInfo {
        ready: true,
        upstream_services: vec![
            ServiceStatus {
                name: "user".to_string(),
                available: true,
            },
            ServiceStatus {
                name: "order".to_string(),
                available: true,
            },
            ServiceStatus {
                name: "product".to_string(),
                available: true,
            },
        ],
    };
    
    ApiResponse::success(info)
}

/// 获取内存使用（MB）
fn get_memory_usage() -> f64 {
    // 简化实现
    // 实际应该使用 process::memory_usage() 或类似方法
    0.0
}
```

### 15. 主程序：src/main.rs
```rust
// main.rs - API Gateway 主程序
// 作用：程序入口，组装所有组件

mod config;
mod models;
mod middleware;
mod routes;
mod services;

use crate::config::Config;
use crate::middleware::auth::AuthState;
use crate::middleware::rate_limiter::RateLimitLayer;
use crate::middleware::logging::logging_middleware;
use crate::routes::auth::{create_auth_routes, AuthRouteState};
use crate::routes::health::{create_health_routes, HealthState};
use crate::routes::proxy::{create_proxy_routes, ProxyState};
use crate::services::upstream::create_upstream_store;

use axum::{
    middleware as axum_middleware,
    routing::get,
    Router,
};
use std::sync::Arc;
use tower_http::cors::{Any, CorsLayer};
use tower_http::trace::TraceLayer;
use tracing_subscriber::{layer::SubscriberExt, util::SubscriberInitExt};

#[tokio::main]
async fn main() -> anyhow::Result<()> {
    // ==================== 初始化日志 ====================
    tracing_subscriber::registry()
        .with(
            tracing_subscriber::EnvFilter::try_from_default_env()
                .unwrap_or_else(|_| "api_gateway=debug,tower_http=debug".into()),
        )
        .with(tracing_subscriber::fmt::layer())
        .init();
    
    tracing::info!("🚀 启动 API Gateway...");
    
    // ==================== 加载配置 ====================
    let config = Config::load().unwrap_or_else(|e| {
        tracing::warn!("配置加载失败，使用默认配置: {}", e);
        Config::default()
    });
    
    let config = Arc::new(config);
    
    tracing::info!(
        "📋 配置加载完成: {} ({}:{})",
        config.app.name,
        config.app.host,
        config.app.port
    );
    
    // ==================== 创建状态 ====================
    
    // 上游服务存储
    let upstream_store = create_upstream_store();
    
    // 认证状态
    let auth_state = AuthState::new(
        &config.jwt.secret,
        &config.jwt.issuer,
        true,
    );
    
    // 限流配置
    let rate_limit_config = config.rate_limit.as_ref();
    let (default_rps, burst, enabled) = rate_limit_config
        .map(|c| (c.default_rps, c.burst_size, c.enabled))
        .unwrap_or((100, 200, false));
    
    // 健康检查状态
    let health_state = HealthState {
        version: env!("CARGO_PKG_VERSION").to_string(),
        start_time: std::time::Instant::now(),
    };
    
    // ==================== 创建路由 ====================
    
    let app = Router::new()
        // 根路径
        .route("/", get(|| async { "API Gateway is running!" }))
        // 健康检查
        .nest("/api", create_health_routes(health_state.clone()))
        // 认证路由
        .nest("/auth", create_auth_routes(AuthRouteState {
            jwt_service: crate::middleware::auth::JwtService::new(
                &config.jwt.secret,
                &config.jwt.issuer,
            ),
            state: auth_state.clone(),
        }))
        // 代理路由（需要认证）
        .nest("/proxy", create_proxy_routes(ProxyState {
            upstream_store: upstream_store.clone(),
            config: config.clone(),
        }))
        // 全局中间件
        .layer(axum_middleware::from_fn(logging_middleware))
        .layer(TraceLayer::new_for_http())
        .layer(
            CorsLayer::new()
                .allow_origin(Any)
                .allow_methods(Any)
                .allow_headers(Any),
        )
        // 限流
        .layer(RateLimitLayer::new(default_rps, burst, enabled));
    
    // ==================== 启动服务器 ====================
    
    let addr = format!("{}:{}", config.app.host, config.app.port);
    
    tracing::info!("🌐 服务器启动在 http://{}", addr);
    tracing::info!("📖 健康检查: http://{}/api/health", addr);
    tracing::info!("🔐 认证接口: http://{}/auth/login", addr);
    
    let listener = tokio::net::TcpListener::bind(&addr).await?;
    
    axum::serve(listener, app)
        .with_graceful_shutdown(shutdown_signal())
        .await?;
    
    tracing::info!("👋 API Gateway 已关闭");
    Ok(())
}

/// 优雅关闭信号处理
async fn shutdown_signal() {
    use tokio::signal;
    
    let ctrl_c = async {
        signal::ctrl_c()
            .await
            .expect("failed to install Ctrl+C handler");
    };
    
    #[cfg(unix)]
    let terminate = async {
        signal::unix::signal(signal::unix::SignalKind::terminate())
            .expect("failed to install SIGTERM handler")
            .recv()
            .await;
    };
    
    #[cfg(not(unix))]
    let terminate = std::future::pending::<()>();
    
    tokio::select! {
        _ = ctrl_c => {},
        _ = terminate => {},
    }
    
    tracing::info!("📤 收到关闭信号，开始优雅关闭...");
}
```

---

## 代码关键点说明

### 1. Tokio 异步运行时
```rust
#[tokio::main]  // 异步 main 函数
async fn main() {
    // Tokio 提供多线程运行时
    // 默认使用多线程 runtime，可以处理阻塞 I/O
    
    // spawn 创建新任务
    let handle = tokio::spawn(async {
        // 异步操作
        tokio::time::sleep(Duration::from_secs(1)).await;
        println!("任务完成");
    });
    
    // await 等待任务完成
    handle.await.unwrap();
}
```

### 2. 异步中间件模式
```rust
// 使用 Tower 的 Service 模式
// 中间件是包装另一个 Service 的 Service

pub async fn middleware(
    request: Request,
    next: Next,
) -> Response {
    // 前置处理
    let start = Instant::now();
    
    // 调用下一个 Service
    let response = next.run(request).await;
    
    // 后置处理
    let duration = start.elapsed();
    tracing::info!("请求耗时: {:?}", duration);
    
    response
}
```

### 3. 熔断器状态机
```
         ┌──────────────────────────────────┐
         │                                  │
         │                                  ▼
    ┌────┴────┐     失败>=阈值      ┌──────┴──────┐
    │ Closed  │ ─────────────────▶ │   Open     │
    │ (正常)  │                    │ (熔断)      │
    └────┬────┘                    └──────┬──────┘
         │                                 │
         │ 成功                           │ 超时
         │                                 ▼
         │                           ┌───────────┐
         │◀─────────────────────────│ HalfOpen  │
         │  成功>=阈值               │ (半开)    │
         └───────────────────────────┴───────────┘
```

### 4. JWT 认证流程
```
请求 → 提取 Token → 验证签名 → 验证过期 → 提取 Claims → 注入用户信息 → 放行
```

---

## 学习计划（5天）

### 第1天：Tokio 异步基础与项目搭建
**目标**：理解 Tokio 运行时，搭建项目结构

| 时间 | 任务 | 产出 |
|------|------|------|
| 上午 | Tokio 概念学习 | async/await 理解 |
| 上午 | 创建项目 | Cargo.toml |
| 下午 | Axum 框架 | Hello World |
| 下午 | 配置管理 | config.yaml |

### 第2天：中间件系统
**目标**：实现中间件系统

| 时间 | 任务 | 产出 |
|------|------|------|
| 上午 | Tower 中间件 | Service 模式 |
| 上午 | 限流器实现 | Token Bucket |
| 下午 | 熔断器实现 | 状态机 |
| 下午 | 日志中间件 | tracing |

### 第3天：认证与路由
**目标**：实现 JWT 认证和路由

| 时间 | 任务 | 产出 |
|------|------|------|
| 上午 | JWT 基础 | Claims 验证 |
| 上午 | 认证中间件 | Token 提取 |
| 下午 | 路由系统 | 嵌套路由 |
| 下午 | 代理路由 | Upstream 调用 |

### 第4天：高级特性
**目标**：实现高级特性

| 时间 | 任务 | 产出 |
|------|------|------|
| 上午 | Redis 缓存 | 连接池 |
| 上午 | 数据库 | SQLx |
| 下午 | 健康检查 | /health |
| 下午 | OpenTelemetry | 分布式追踪 |

### 第5天：测试与部署
**目标**：测试和部署

| 时间 | 任务 | 产出 |
|------|------|------|
| 上午 | 单元测试 | #[test] |
| 上午 | 集成测试 | #[tokio::test] |
| 下午 | Docker 部署 | Dockerfile |
| 下午 | 面试准备 | 题目答案 |

---

## 面试要点

### 面试题 1：Tokio 异步运行时是如何工作的？

**标准答案**：
1. **多线程运行时**：Tokio 使用工作线程池执行任务
2. **任务调度**：任务被分配到线程上执行
3. **I/O 驱动**：使用 epoll/kqueue/IOCP 处理异步 I/O
4. **协作式调度**：`await` 点让出执行权

```rust
#[tokio::main]
async fn main() {
    // tokio::spawn 创建并发任务
    let handle = tokio::spawn(async {
        tokio::time::sleep(Duration::from_secs(1)).await;
        "done"
    });
    
    // 任务在后台运行
    let result = handle.await.unwrap();
}
```

### 面试题 2：async/await 的原理是什么？

**标准答案**：
1. **状态机转换**：async 函数被编译为状态机
2. **Future**：表示尚未完成的异步操作
3. **Poll**：轮询 Future 直到完成
4. **Waker**：I/O 完成时通知调度器

```rust
// async 函数编译为 Future
// await 点是状态转换

async fn example() {
    let data = fetch_data().await;  // 状态 1
    process(data).await;            // 状态 2
}

// 等价于
fn example() -> impl Future<Output = ()> {
    async {
        let data = fetch_data().await;
        process(data).await;
    }
}
```

### 面试题 3：Rust 的 Trait 对象和泛型有什么区别？

**标准答案**：
| 对比项 | 泛型 | Trait 对象 |
|--------|------|------------|
| 分发时机 | 编译时 | 运行时 |
| 性能 | 零成本 | 有虚表调度开销 |
| 类型 | 具体类型 | 动态类型 |
| 适用场景 | 静态类型 | 需要运行时多态 |

```rust
// 泛型（静态分发）
fn process<T: Service>(service: T) {
    // 编译时确定类型
}

// Trait 对象（动态分发）
fn process(service: &dyn Service) {
    // 运行时确定类型
}
```

### 面试题 4：熔断器模式的原理是什么？

**标准答案**：
1. **Closed**：正常状态，统计失败次数
2. **Open**：失败超阈值，拒绝所有请求
3. **HalfOpen**：超时后放行一个请求测试

```rust
// 熔断器检查
if !breaker.is_allowed() {
    return Err(CircuitOpen);
}

// 执行操作
match operation.await {
    Ok(_) => breaker.record_success(),
    Err(_) => breaker.record_failure(),
}
```

### 面试题 5：JWT 的工作原理？

**标准答案**：
1. **Header**：包含算法信息
2. **Payload**：包含声明（Claims）
3. **Signature**：使用密钥签名

```rust
// 生成 Token
let claims = Claims::new(user_id, username, roles, expiry_hours);
let token = encode(&Header::default(), &claims, &encoding_key)?;

// 验证 Token
let token_data = decode::<Claims>(token, &decoding_key, &validation)?;
let user = token_data.claims;
```
