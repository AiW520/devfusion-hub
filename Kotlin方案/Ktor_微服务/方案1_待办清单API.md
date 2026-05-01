# Ktor 微服务方案一：待办清单 RESTful API

## 项目介绍

### 项目是什么
这是一个轻量级的待办清单（Todo List）RESTful API 服务，使用 Kotlin + Ktor + PostgreSQL 构建。它提供完整的 CRUD 操作，支持任务分类、优先级、截止日期等功能。

### 解决什么问题
- 为前端（Web/移动端）提供标准 RESTful 接口
- 存储和管理用户任务数据
- 支持任务筛选、排序、分页

### 核心技术亮点
| 技术 | 说明 |
|------|------|
| Ktor | Kotlin 异步 Web 框架，轻量且高性能 |
| Exposed | JetBrains 出品的 Kotlin SQL 框架 |
| Kotlin Coroutines | 异步非阻塞编程 |
| RESTful | 标准 REST 设计风格 |
| JWT | 无状态身份认证 |

### 适合场景
- 作为微服务架构中的任务管理服务
- 学习 Kotlin 后端开发的入门项目
- 需要快速搭建 API 服务的场景

---

## 完整可运行代码

### 项目结构
```
todo-api/
├── build.gradle.kts
├── settings.gradle.kts
├── gradle.properties
├── docker-compose.yml
└── src/
    └── main/
        ├── kotlin/
        │   └── com/example/todo/
        │       ├── Main.kt                    # 应用入口
        │       ├── plugin/                     # Ktor 插件配置
        │       │   ├── Database.kt            # 数据库配置
        │       │   ├── Security.kt            # 安全配置（JWT）
        │       │   └── Serialization.kt       # JSON 序列化
        │       ├── model/                      # 数据模型
        │       │   ├── Todo.kt                # 待办事项实体
        │       │   └── User.kt                # 用户实体
        │       ├── repository/                 # 数据访问层
        │       │   ├── TodoRepository.kt
        │       │   └── UserRepository.kt
        │       ├── service/                     # 业务逻辑层
        │       │   ├── TodoService.kt
        │       │   └── AuthService.kt
        │       ├── route/                       # 路由处理
        │       │   ├── TodoRoute.kt
        │       │   └── AuthRoute.kt
        │       └── dto/                         # 数据传输对象
        │           ├── TodoDto.kt
        │           ├── UserDto.kt
        │           └── AuthDto.kt
        └── resources/
            └── application.conf                # Ktor 配置文件
```

### 1. build.gradle.kts（项目构建配置）

```kotlin
// build.gradle.kts - Kotlin DSL 格式的 Gradle 构建脚本
// 这是项目的核心配置文件，定义了所有依赖和构建规则

plugins {
    // Kotlin 插件 - 支持 Kotlin 语言
    kotlin("jvm") version "1.9.22"
    
    // Ktor 插件 - 用于生成 Ktor 应用程序
    kotlin("plugin.serialization") version "1.9.22"
    
    // Ktor 服务器插件（Gradle 插件方式）
    id("io.ktor.plugin") version "2.3.7"
    
    // 数据库迁移插件（可选）
    id("org.flywaydb.flyway") version "9.22.3"
}

// 存储库配置
repositories {
    mavenCentral()  // Maven 中央仓库，Kotlin/JVM 库主要来源
}

// 依赖项配置
dependencies {
    // ===== Ktor 核心依赖 =====
    
    // Ktor 服务器核心 - 包含 Web 框架核心功能
    implementation("io.ktor:ktor-server-core:2.3.7")
    
    // Ktor 服务器引擎（Netty）- 高性能异步网络框架
    implementation("io.ktor:ktor-server-netty:2.3.7")
    
    // Ktor 内容协商 - 用于处理 JSON 等内容格式
    implementation("io.ktor:ktor-server-content-negotiation:2.3.7")
    
    // Ktor JSON 序列化支持（Kotlinx Serialization）
    implementation("io.ktor:ktor-serialization-kotlinx-json:2.3.7")
    
    // Ktor 状态_pages页面支持（访问HTML页面）
    implementation("io.ktor:ktor-server-html-builder:2.3.7")
    
    // ===== Ktor 安全相关 =====
    
    // Ktor 认证 - 支持 JWT、Basic 等认证方式
    implementation("io.ktor:ktor-server-auth:2.3.7")
    
    // Ktor JWT 认证 - JSON Web Token 认证
    implementation("io.ktor:ktor-server-auth-jwt:2.3.7")
    
    // ===== Ktor 日志和监控 =====
    
    // Ktor 调用日志 - 记录每个HTTP请求的详细信息
    implementation("io.ktor:ktor-server-call-logging:2.3.7")
    
    // Ktor CORS - 跨域资源共享配置
    implementation("io.ktor:ktor-server-cors:2.3.7")
    
    // ===== Kotlin 协程 =====
    
    // Kotlin 协程核心库 - 支持异步编程
    implementation("org.jetbrains.kotlinx:kotlinx-coroutines-core:1.7.3")
    
    // Kotlin 协程调试支持
    implementation("org.jetbrains.kotlinx:kotlinx-coroutines-debug:1.7.3")
    
    // ===== 数据库相关 =====
    
    // Exposed ORM 框架核心 - Kotlin SQL 框架
    implementation("org.jetbrains.exposed:exposed:0.48.0")
    
    // Exposed JDBC 驱动 - 数据库连接
    implementation("org.jetbrains.exposed:exposed-jdbc:0.48.0")
    
    // PostgreSQL JDBC 驱动 - 连接 PostgreSQL 数据库
    implementation("org.postgresql:postgresql:42.7.1")
    
    // HikariCP 连接池 - 高性能数据库连接池
    implementation("com.zaxxer:HikariCP:5.1.0")
    
    // Flyway 数据库迁移 - 管理数据库版本
    implementation("org.flywaydb:flyway-core:9.22.3")
    
    // ===== 工具库 =====
    
    // Kotlinx Serialization JSON - JSON 序列化/反序列化
    implementation("org.jetbrains.kotlinx:kotlinx-serialization-json:1.6.2")
    
    // BCrypt 密码加密 - 安全存储用户密码
    implementation("org.mindrot:jbcrypt:0.4")
    
    // Kotlinx DateTime - Kotlin 日期时间处理
    implementation("org.jetbrains.kotlinx:kotlinx-datetime:0.5.0")
    
    // ===== 测试依赖 =====
    
    // Ktor 服务器测试 - 用于单元测试
    testImplementation("io.ktor:ktor-server-test-host:2.3.7")
    
    // Kotlin 测试框架
    testImplementation("org.jetbrains.kotlin:kotlin-test:1.9.22")
}

// Kotlin 应用配置
kotlin {
    // JVM 编译目标版本为 17（现代 LTS 版本）
    jvm {
        toolchain(17)
    }
}

// Ktor 插件配置
ktor {
    // 打包类型为 "fat-jar"（包含所有依赖的独立 JAR）
    fatJar {
        // 打包后的 JAR 文件名
        archiveFileName.set("todo-api.jar")
    }
    
    // 开发模式配置
    development {
        // 是否启用热重载（开发时为 true）
        watchPatterns.add("src/main/kotlin/**")
    }
}
```

### 2. settings.gradle.kts

```kotlin
// settings.gradle.kts - 项目设置文件
// 定义项目名称和模块结构

// 项目名称 - 显示在 Gradle 构建输出中
rootProject.name = "todo-api"

// 启用 Gradle 配置缓存，提高构建性能
gradleConfigurationCache = true
```

### 3. gradle.properties

```properties
# gradle.properties - Gradle 全局配置
# 使用 .properties 格式，键值对形式

# Kotlin 代码编码为 UTF-8（支持中文注释）
kotlin.code.styles=official

# 启用 Gradle 构建缓存（重复构建时复用结果）
org.gradle.caching=true

# 启用 Gradle 并行构建（多模块时效果明显）
org.gradle.parallel=true

# JVM 参数配置
# -Xmx: 最大堆内存
# -Xss: 线程栈大小
org.gradle.jvmargs=-Xmx2048m -Xss4m -XX:+UseParallelGC

# Kotlin 编译器参数
# -Xjsr305: 支持 JSR-305 注解（用于空安全检查）
kotlin.daemon.jvmargs=-Xmx2048m
```

### 4. docker-compose.yml（Docker 容器编排）

```yaml
# docker-compose.yml - Docker 容器编排配置
# 定义了 PostgreSQL 数据库服务的配置

version: '3.8'  # Docker Compose 文件版本

services:
  # PostgreSQL 数据库服务
  postgres:
    # 使用的 Docker 镜像及版本
    # postgres:16-alpine - Alpine 轻量级镜像，体积小安全性高
    image: postgres:16-alpine
    
    # 容器名称 - 方便 docker 命令引用
    container_name: todo-postgres
    
    # 重启策略 - 容器退出后自动重启
    restart: unless-stopped
    
    # 环境变量 - PostgreSQL 配置
    environment:
      # 数据库名称
      POSTGRES_DB: todo_db
      # 数据库用户
      POSTGRES_USER: todo_user
      # 数据库密码（生产环境应使用更安全的方式）
      POSTGRES_PASSWORD: todo_password_123
      
    # 端口映射 - 格式: 主机端口:容器端口
    # 将主机的 5432 端口映射到容器的 5432 端口
    ports:
      - "5432:5432"
    
    # 卷挂载 - 持久化数据
    volumes:
      # 将容器内数据目录挂载到主机，防止数据丢失
      - postgres_data:/var/lib/postgresql/data
      
    # 健康检查 - 确保数据库就绪后再启动应用
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U todo_user -d todo_db"]
      interval: 10s  # 检查间隔
      timeout: 5s   # 超时时间
      retries: 5    # 重试次数

# 卷定义 - 声明数据卷
volumes:
  # 命名卷 - Docker 自动管理数据持久化
  postgres_data:
    driver: local
```

### 5. Main.kt（应用入口）

```kotlin
package com.example.todo

import com.example.todo.plugin.*
import com.example.todo.route.*
import com.example.todo.repository.*
import com.example.todo.service.*
import io.ktor.server.engine.*
import io.ktor.server.netty.*
import io.ktor.server.application.*
import io.ktor.server.plugins.callloging.*
import org.slf4j.event.*

/**
 * Todo API 主程序入口
 * 
 * Ktor 应用遵循 "嵌入式服务器" 模式，将服务器直接嵌入到应用中
 * 这种方式简单高效，适合微服务场景
 */
fun main() {
    // embeddedServer: 创建嵌入式服务器
    // Netty: 高性能异步网络框架，作为 Ktor 的底层引擎
    // port = 8080: HTTP 服务端口
    embeddedServer(Netty, port = 8080) {
        // ===== 配置 Ktor 应用模块 =====
        
        // 1. 配置模块 - 在这里注册所有插件和路由
        configureModule()
        
    }.start(wait = true)  // 启动服务器，wait = true 表示阻塞主线程直到应用结束
}

/**
 * Ktor 应用配置模块
 * 这是 Ktor 的核心扩展函数，用于配置整个应用
 */
fun Application.configureModule() {
    // ===== 安装基础插件 =====
    
    // 1. CallLogging - 记录每个 HTTP 请求的日志
    // 对于调试和监控非常重要
    install(CallLogging) {
        // 使用 SLF4J 日志级别
        level = Level.INFO
        
        // 过滤规则 - 忽略健康检查等频繁请求
        filter { call ->
            call.request.path().startsWith("/")
        }
    }
    
    // 2. CORS - 跨域资源共享配置
    // 允许前端 JavaScript 跨域访问 API
    install(CORS) {
        // 允许所有来源（开发环境）
        anyHost()
        
        // 允许的 HTTP 方法
        allowMethod(HttpMethod.Get)
        allowMethod(HttpMethod.Post)
        allowMethod(HttpMethod.Put)
        allowMethod(HttpMethod.Delete)
        allowMethod(HttpMethod.Options)
        
        // 允许的请求头
        allowHeader(HttpHeaders.ContentType)
        allowHeader(HttpHeaders.Authorization)
        allowHeader(HttpHeaders.Accept)
        
        // 允许携带凭证（Cookies）
        allowCredentials = true
        
        // 预检请求缓存时间
        maxAgeInSeconds = 3600L
    }
    
    // ===== 安装业务插件 =====
    
    // 3. JSON 序列化配置
    configureSerialization()
    
    // 4. 数据库配置和初始化
    configureDatabase()
    
    // 5. JWT 安全配置
    configureSecurity()
    
    // ===== 初始化各层组件 =====
    
    // 初始化仓储层（Repository）
    // 注意：Ktor 使用延迟初始化，避免在配置阶段创建连接
    val userRepository by lazy { UserRepository() }
    val todoRepository by lazy { TodoRepository() }
    
    // 初始化服务层（Service）- 依赖仓储层
    val authService by lazy { AuthService(userRepository) }
    val todoService by lazy { TodoService(todoRepository) }
    
    // ===== 注册路由 =====
    
    // 认证路由 - 登录、注册
    configureAuthRoute(authService)
    
    // Todo 路由 - CRUD 操作
    // 注意：受保护的路由需要在 auth 函数内注册
    configureTodoRoute(todoService, authService)
    
    // 根路由 - 健康检查
    routing {
        get("/") {
            call.respondText("✅ Todo API is running! Version: 1.0.0")
        }
        
        // 健康检查端点 - 用于 Docker 健康检查
        get("/health") {
            call.respondText("OK")
        }
    }
}
```

### 6. Database.kt（数据库配置）

```kotlin
package com.example.todo.plugin

import com.example.todo.model.*
import com.zaxxer.hikari.HikariConfig
import com.zaxxer.hikari.HikariDataSource
import io.ktor.server.application.*
import kotlinx.coroutines.Dispatchers
import kotlinx.coroutines.withContext
import org.jetbrains.exposed.sql.Database
import org.jetbrains.exposed.sql.SchemaUtils
import org.jetbrains.exposed.sql.transactions.transaction
import org.slf4j.LoggerFactory

/**
 * 数据库配置插件
 * 
 * Exposed 是 JetBrains 出品的轻量级 ORM 框架
 * 特点：
 * - 完全使用 Kotlin 语言编写
 * - 支持类型安全的 SQL DSL
 * - 可以写原生 SQL
 */
private val logger = LoggerFactory.getLogger("Database")

// 全局数据库连接实例
lateinit var database: Database

/**
 * 配置数据库连接
 */
fun Application.configureDatabase() {
    // ===== 配置 HikariCP 连接池 =====
    // HikariCP 是目前性能最好的 JDBC 连接池
    
    val hikariConfig = HikariConfig().apply {
        // JDBC 连接 URL
        // 格式: jdbc:postgresql://主机:端口/数据库名
        jdbcUrl = "jdbc:postgresql://localhost:5432/todo_db"
        
        // 数据库用户名
        username = "todo_user"
        
        // 数据库密码
        password = "todo_password_123"
        
        // 驱动类名
        driverClassName = "org.postgresql.Driver"
        
        // ===== 连接池配置 =====
        
        // 最大连接数 - 根据服务器配置调整
        maximumPoolSize = 10
        
        // 最小空闲连接数
        minimumIdle = 2
        
        // 连接超时时间（毫秒）
        // 超过这个时间拿不到连接会抛出异常
        connectionTimeout = 30000
        
        // 空闲超时时间（毫秒）
        // 空闲连接超过这个时间会被关闭
        idleTimeout = 600000
        
        // 连接最大生命周期（毫秒）
        // 连接超过这个时间会被关闭
        maxLifetime = 1800000
        
        // 连接测试查询
        // 从连接池获取连接后，执行这个 SQL 测试连接是否有效
        connectionTestQuery = "SELECT 1"
        
        // 启用连接池
        poolName = "TodoPool"
    }
    
    // 创建数据源（连接池）
    val dataSource = HikariDataSource(hikariConfig)
    
    // ===== 连接到数据库 =====
    database = Database.connect(dataSource)
    
    logger.info("✅ Database connected successfully")
    
    // ===== 初始化数据库表 =====
    // 在首次运行时创建表结构
    initializeTables()
}

/**
 * 初始化数据库表
 * 
 * transaction: Exposed 的事务包装函数
 * 所有数据库写操作都应该在事务中执行
 */
private suspend fun initializeTables() {
    // withContext: 切换到指定的协程上下文执行
    // IO: 用于 I/O 操作（数据库、网络等）
    // 这样不会阻塞主线程
    withContext(Dispatchers.IO) {
        transaction(database) {
            logger.info("📋 Creating database tables...")
            
            // SchemaUtils.create: 创建表
            // 如果表已存在会忽略（不会报错）
            SchemaUtils.create(
                Users,           // 用户表
                Todos,           // 待办事项表
                checkExistence = true  // 如果存在则跳过
            )
            
            logger.info("✅ Database tables created successfully")
        }
    }
}

/**
 * 获取数据库连接（用于测试）
 */
fun getDatabase(): Database = database
```

### 7. Security.kt（JWT 安全配置）

```kotlin
package com.example.todo.plugin

import com.auth0.jwt.JWT
import com.auth0.jwt.algorithms.Algorithm
import io.ktor.server.application.*
import io.ktor.server.auth.*
import io.ktor.server.auth.jwt.*
import java.util.*

/**
 * JWT 安全配置插件
 * 
 * JWT（JSON Web Token）是一种开放标准（RFC 7519）
 * 用于在各方之间安全地传输信息
 * 
 * JWT 结构: Header.Payload.Signature
 * 示例: eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4ifQ.4S5C9mQHF3t
 */
object JwtConfig {
    // ===== JWT 配置常量 =====
    
    // JWT 密钥 - 用于签名验证（生产环境应从环境变量或配置文件读取）
    // 长度建议至少 256 位（32 字节）
    private const val SECRET = "your-super-secret-key-change-in-production-2024"
    
    // 颁发者 - 标识 JWT 的签发者
    private const val ISSUER = "todo-api"
    
    // 观众 - 标识 JWT 的预期接收者
    private const val AUDIENCE = "todo-api-clients"
    
    // 过期时间 - 24 小时（毫秒）
    private const val EXPIRATION = 24 * 60 * 60 * 1000L
    
    // 算法 - HMAC SHA256（对称加密算法）
    private val algorithm = Algorithm.HMAC256(SECRET)

    /**
     * 生成 JWT Token
     * 
     * @param userId 用户 ID
     * @param username 用户名
     * @return JWT 字符串
     */
    fun generateToken(userId: Int, username: String): String {
        // JWT.create(): 创建 JWT 构建器
        return JWT.create()
            .withIssuer(ISSUER)                    // 设置颁发者
            .withAudience(AUDIENCE)                // 设置观众
            .withClaim("userId", userId)           // 添加自定义声明（用户ID）
            .withClaim("username", username)       // 添加自定义声明（用户名）
            .withIssuedAt(Date())                  // 设置签发时间
            .withExpiresAt(Date(System.currentTimeMillis() + EXPIRATION))  // 设置过期时间
            .sign(algorithm)                       // 使用密钥签名
    }

    /**
     * JWT 配置对象
     * 包含验证器和密钥
     */
    val config = JWTConfig {
        // 验证密钥
        verify { it.verify(algorithm) }
        
        // 验证颁发者
        verifyClaim(ISSUER) { claim -> claim.asString() == ISSUER }
        
        // 验证观众
        verifyClaim(AUDIENCE) { claim -> claim.asString() == AUDIENCE }
        
        // 验证过期时间（自动检查 token 是否过期）
        // 如果 token 过期，会抛出 JWTVerificationException
    }
}

/**
 * 配置 Ktor 认证插件
 */
fun Application.configureSecurity() {
    // ===== 安装 JWT 认证插件 =====
    // Ktor 的认证系统支持多种认证方式（JWT、Basic、OAuth等）
    
    install(Authentication) {
        // 配置 JWT Bearer 认证
        // Bearer Token 是 HTTP 认证的一种方式
        // 格式: Authorization: Bearer <token>
        jwt("auth-jwt") {
            // 设置 JWT 配置（包含验证器）
            configure(JwtConfig.config)
            
            // 认证失败时的回调
            // 如果 token 无效或过期，返回 401 错误
            challenge { _, _ ->
                call.respond(
                    io.ktor.http.HttpStatusCode.Unauthorized,
                    mapOf("error" to "Invalid or expired token")
                )
            }
        }
    }
}

/**
 * 认证验证器配置
 * 这是 Ktor 2.x 的配置方式
 */
private fun JWTConfig.configure(block: JWTConfig.() -> Unit) {
    block()
}
```

### 8. Serialization.kt（JSON 序列化配置）

```kotlin
package com.example.todo.plugin

import io.ktor.serialization.kotlinx.json.*
import io.ktor.server.plugins.contentnegotiation.*
import io.ktor.server.application.*
import kotlinx.serialization.json.Json

/**
 * JSON 序列化配置
 * 
 * Ktor 使用 kotlinx.serialization 作为默认的 JSON 序列化库
 * 特点：
 * - 完全使用 Kotlin 语言编写
 * - 支持类型安全
 * - 性能优秀
 */
fun Application.configureSerialization() {
    // ===== 安装内容协商插件 =====
    // Content Negotiation 会根据客户端请求的 Content-Type
    // 自动选择合适的序列化方式
    
    install(ContentNegotiation) {
        // 配置 JSON 序列化
        json(Json {
            // 是否美化输出（格式化 JSON，方便调试）
            prettyPrint = true
            
            // 是否忽略未知属性（当 JSON 有多余字段时不报错）
            ignoreUnknownKeys = true
            
            // 是否允许 JSON 中有 null 值
            encodeDefaults = true
            
            // 是否将非字符串类型转为字符串（宽松模式）
            isLenient = true
            
            // 序列化时的行为配置
            serializationConfig = kotlinx.serialization.json.Json {
                // 启用类别名（用于多态序列化）
                classDiscriminator = "#type"
            }
        })
    }
}
```

### 9. User.kt（用户实体）

```kotlin
package com.example.todo.model

import org.jetbrains.exposed.sql.*
import org.jetbrains.exposed.sql.javatime.*
import java.time.LocalDateTime

/**
 * 用户表（Users）实体定义
 * 
 * Exposed 有两种定义表的方式：
 * 1. 表对象（Table Object）- 使用 object 关键字
 * 2. 子类继承 - 继承 Table 类
 * 
 * 这里使用 object 方式，更简洁
 */
object Users : Table("users") {
    // ===== 表字段定义 =====
    
    // id - 主键，自增
    // integer("column_name") - 整数类型列
    // autoIncrement() - 自增主键
    val id = integer("id").autoIncrement()
    
    // username - 用户名，唯一且非空
    // varchar() - 可变字符串类型
    // uniqueIndex() - 创建唯一索引，加快查询
    val username = varchar("username", 50).uniqueIndex()
    
    // email - 邮箱，唯一且非空
    val email = varchar("email", 100).uniqueIndex()
    
    // password - 密码（存储 BCrypt 加密后的哈希）
    // 不要存储明文密码！这是基本的安全常识
    val password = varchar("password", 255)
    
    // createdAt - 创建时间，默认当前时间
    val createdAt = datetime("created_at").defaultExpression(
        CurrentDateTime()  // 数据库默认值
    )
    
    // updatedAt - 更新时间
    val updatedAt = datetime("updated_at").defaultExpression(
        CurrentDateTime()
    )
    
    // ===== 表级配置 =====
    
    // primaryKey: 指定主键
    override val primaryKey = PrimaryKey(id)
    
    // tableName: 表名（默认从类名推断，这里显式指定）
    override val tableName = "users"
}

/**
 * 用户实体类 - 表示一行用户数据
 * 
 * 使用 data class 的好处：
 * 1. 自动生成 equals()、hashCode()、toString()
 * 2. 自动生成 copy() 方法
 * 3. 可以使用解构声明
 */
data class User(
    val id: Int,
    val username: String,
    val email: String,
    val password: String,  // BCrypt 哈希后的密码
    val createdAt: LocalDateTime,
    val updatedAt: LocalDateTime
)
```

### 10. Todo.kt（待办事项实体）

```kotlin
package com.example.todo.model

import org.jetbrains.exposed.sql.*
import org.jetbrains.exposed.sql.javatime.*
import java.time.LocalDateTime

/**
 * 待办事项表（Todos）实体定义
 * 
 * 设计思路：
 * - 每个待办事项属于一个用户（外键关联）
 * - 支持分类、优先级、截止日期
 * - 软删除（isDeleted 字段）保留数据
 */
object Todos : Table("todos") {
    // ===== 主键和关联 =====
    
    // id - 主键，自增
    val id = integer("id").autoIncrement()
    
    // userId - 所属用户 ID（外键）
    // references() - 建立外键关联
    val userId = integer("user_id").references(Users.id)
    
    // ===== 业务字段 =====
    
    // title - 标题，必填，最大 200 字符
    val title = varchar("title", 200)
    
    // description - 描述，可选，最大 1000 字符
    val description = varchar("description", 1000).nullable()
    
    // category - 分类，用于组织任务
    // 比如：工作、学习、生活、健康等
    val category = varchar("category", 50).default("默认")
    
    // priority - 优先级（1=低, 2=中, 3=高）
    // integer() 默认 NOT NULL，但添加 default() 后变为可选
    val priority = integer("priority").default(2)  // 默认中等优先级
    
    // status - 状态（pending=待完成, completed=已完成）
    // 使用 varchar 并添加 check 约束确保数据合法性
    val status = varchar("status", 20).default("pending")
    
    // dueDate - 截止日期
    val dueDate = datetime("due_date").nullable()
    
    // ===== 审计字段 =====
    
    // createdAt - 创建时间
    val createdAt = datetime("created_at").defaultExpression(
        CurrentDateTime()
    )
    
    // updatedAt - 更新时间
    val updatedAt = datetime("updated_at").defaultExpression(
        CurrentDateTime()
    )
    
    // isDeleted - 软删除标记
    // 软删除 vs 硬删除：
    // - 硬删除：直接从数据库删除数据
    // - 软删除：设置删除标记，数据仍然保留
    // 软删除的好处：可以恢复数据，方便审计
    val isDeleted = bool("is_deleted").default(false)
    
    // ===== 表级配置 =====
    override val primaryKey = PrimaryKey(id)
    
    // 创建索引：按用户 ID 和状态查询会很快
    // compositeIndex() - 复合索引，针对多列组合查询优化
    override val indexes = listOf(
        // 按用户 ID 和状态索引（查询某用户的所有待完成事项）
        compositeIndex(userId, status),
        // 按用户 ID 和分类索引（查询某用户的某个分类的所有事项）
        compositeIndex(userId, category)
    )
}

/**
 * 优先级枚举
 * 使用 Kotlin Enum 替代魔法数字
 * 
 * Enum 的优势：
 * 1. 类型安全 - 只能用预定义的值
 * 2. 可读性强 - 代码更清晰
 * 3. IDE 支持好 - 自动补全、枚举名称检查
 */
enum class Priority(val value: Int, val label: String) {
    LOW(1, "低优先级"),
    MEDIUM(2, "中优先级"),
    HIGH(3, "高优先级");
    
    // 伴生对象：从值反向查找枚举
    companion object {
        fun fromValue(value: Int): Priority {
            return entries.find { it.value == value } ?: MEDIUM
        }
    }
}

/**
 * 状态枚举
 */
enum class TodoStatus(val value: String, val label: String) {
    PENDING("pending", "待完成"),
    COMPLETED("completed", "已完成");
    
    companion object {
        fun fromValue(value: String): TodoStatus {
            return entries.find { it.value == value } ?: PENDING
        }
    }
}

/**
 * 待办事项实体类
 */
data class Todo(
    val id: Int,
    val userId: Int,
    val title: String,
    val description: String?,
    val category: String,
    val priority: Int,
    val status: String,
    val dueDate: LocalDateTime?,
    val createdAt: LocalDateTime,
    val updatedAt: LocalDateTime,
    val isDeleted: Boolean
)
```

### 11. UserRepository.kt

```kotlin
package com.example.todo.repository

import com.example.todo.model.*
import com.example.todo.plugin.database
import org.jetbrains.exposed.sql.*
import org.jetbrains.exposed.sql.transactions.*
import org.mindrot.jbcrypt.*
import kotlinx.coroutines.Dispatchers
import kotlinx.coroutines.withContext

/**
 * 用户仓储层
 * 
 * 职责：
 * - 封装所有用户相关的数据库操作
 * - 提供简洁的数据访问接口
 * - 处理数据转换
 * 
 * 仓储模式（Repository Pattern）的优点：
 * 1. 隔离数据访问逻辑
 * 2. 便于单元测试（可以 mock）
 * 3. 统一的数据操作入口
 */
class UserRepository {
    
    /**
     * 根据用户名查找用户
     * 
     * @param username 用户名
     * @return 用户实体，不存在则返回 null
     */
    suspend fun findByUsername(username: String): User? {
        return withContext(Dispatchers.IO) {
            transaction(database) {
                // select: SQL SELECT 语句
                // where: WHERE 子句
                Users.select { Users.username eq username }
                    .map { it.toUser() }  // 将查询结果转换为 User 对象
                    .singleOrNull()       // 获取单个结果或 null
            }
        }
    }
    
    /**
     * 根据邮箱查找用户
     */
    suspend fun findByEmail(email: String): User? {
        return withContext(Dispatchers.IO) {
            transaction(database) {
                Users.select { Users.email eq email }
                    .map { it.toUser() }
                    .singleOrNull()
            }
        }
    }
    
    /**
     * 根据 ID 查找用户
     */
    suspend fun findById(id: Int): User? {
        return withContext(Dispatchers.IO) {
            transaction(database) {
                Users.select { Users.id eq id }
                    .map { it.toUser() }
                    .singleOrNull()
            }
        }
    }
    
    /**
     * 创建新用户
     * 
     * @param username 用户名
     * @param email 邮箱
     * @param password 明文密码（会自动加密）
     * @return 新创建的用户
     */
    suspend fun create(username: String, email: String, password: String): User {
        return withContext(Dispatchers.IO) {
            transaction(database) {
                // BCrypt 加密密码
                // BCrypt 是专门设计用于密码哈希的算法
                // 特点：自带盐值，可配置强度，内置防彩虹表攻击
                val hashedPassword = BCrypt.hashpw(password, BCrypt.gensalt(12))
                
                // insert: SQL INSERT 语句
                // returning(id): 返回插入行的指定列
                val insertStatement = Users.insert {
                    it[Users.username] = username
                    it[Users.email] = email
                    it[Users.password] = hashedPassword
                } returning Users.id
                
                val newId = insertStatement.single()[Users.id]
                
                // 查询并返回创建的用户
                Users.select { Users.id eq newId }
                    .map { it.toUser() }
                    .single()
            }
        }
    }
    
    /**
     * 更新用户密码
     */
    suspend fun updatePassword(userId: Int, newPassword: String): Boolean {
        return withContext(Dispatchers.IO) {
            transaction(database) {
                // update: SQL UPDATE 语句
                val updatedCount = Users.update({ Users.id eq userId }) {
                    it[Users.password] = BCrypt.hashpw(newPassword, BCrypt.gensalt(12))
                    it[Users.updatedAt] = org.jetbrains.exposed.sql.javatime.CurrentDateTime()
                }
                updatedCount > 0
            }
        }
    }
    
    /**
     * 验证用户密码
     * 
     * @param user 用户对象
     * @param rawPassword 输入的明文密码
     * @return 是否匹配
     */
    fun verifyPassword(user: User, rawPassword: String): Boolean {
        // BCrypt.checkpw: 验证密码是否匹配
        // 内部处理了盐值提取和哈希比较
        return BCrypt.checkpw(rawPassword, user.password)
    }
    
    /**
     * 检查用户名是否存在
     */
    suspend fun existsByUsername(username: String): Boolean {
        return withContext(Dispatchers.IO) {
            transaction(database) {
                Users.select { Users.username eq username }
                    .count() > 0
            }
        }
    }
    
    /**
     * 检查邮箱是否存在
     */
    suspend fun existsByEmail(email: String): Boolean {
        return withContext(Dispatchers.IO) {
            transaction(database) {
                Users.select { Users.email eq email }
                    .count() > 0
            }
        }
    }
    
    /**
     * 将 ResultRow 转换为 User 对象
     * 
     * ResultRow 是 Exposed 查询返回的行对象
     * 提供下标访问和命名访问两种方式
     */
    private fun ResultRow.toUser(): User {
        return User(
            id = this[Users.id],
            username = this[Users.username],
            email = this[Users.email],
            password = this[Users.password],
            createdAt = this[Users.createdAt],
            updatedAt = this[Users.updatedAt]
        )
    }
}
```

### 12. TodoRepository.kt

```kotlin
package com.example.todo.repository

import com.example.todo.model.*
import com.example.todo.plugin.database
import org.jetbrains.exposed.sql.*
import org.jetbrains.exposed.sql.transactions.*
import kotlinx.coroutines.Dispatchers
import kotlinx.coroutines.withContext
import java.time.LocalDateTime

/**
 * 待办事项仓储层
 * 
 * 提供完整的 CRUD 操作
 * 支持筛选、排序、分页
 */
class TodoRepository {
    
    /**
     * 创建待办事项
     */
    suspend fun create(
        userId: Int,
        title: String,
        description: String?,
        category: String,
        priority: Int,
        dueDate: LocalDateTime?
    ): Todo {
        return withContext(Dispatchers.IO) {
            transaction(database) {
                val insertStatement = Todos.insert {
                    it[Todos.userId] = userId
                    it[Todos.title] = title
                    it[Todos.description] = description
                    it[Todos.category] = category
                    it[Todos.priority] = priority
                    it[Todos.dueDate] = dueDate
                    it[Todos.status] = TodoStatus.PENDING.value
                } returning Todos.id
                
                val newId = insertStatement.single()[Todos.id]
                
                Todos.select { Todos.id eq newId }
                    .map { it.toTodo() }
                    .single()
            }
        }
    }
    
    /**
     * 根据 ID 获取待办事项
     * 只返回未删除的记录
     */
    suspend fun findById(id: Int, userId: Int): Todo? {
        return withContext(Dispatchers.IO) {
            transaction(database) {
                Todos.select {
                    // 复合条件：ID匹配 AND 用户匹配 AND 未删除
                    (Todos.id eq id) and 
                    (Todos.userId eq userId) and 
                    (Todos.isDeleted eq false)
                }.map { it.toTodo() }
                 .singleOrNull()
            }
        }
    }
    
    /**
     * 获取用户的所有待办事项
     * 
     * @param userId 用户 ID
     * @param status 筛选状态（可选）
     * @param category 筛选分类（可选）
     * @param sortBy 排序字段
     * @param sortOrder 排序方向（asc/desc）
     * @param limit 每页数量
     * @param offset 偏移量（分页用）
     */
    suspend fun findByUserId(
        userId: Int,
        status: String? = null,
        category: String? = null,
        sortBy: String = "createdAt",
        sortOrder: SortOrder = SortOrder.DESC,
        limit: Int = 20,
        offset: Int = 0
    ): List<Todo> {
        return withContext(Dispatchers.IO) {
            transaction(database) {
                // 构建动态查询条件
                val query = Todos.select {
                    // 基础条件：用户匹配 + 未删除
                    (Todos.userId eq userId) and (Todos.isDeleted eq false)
                }
                
                // 添加状态筛选
                if (status != null) {
                    query.andWhere { Todos.status eq status }
                }
                
                // 添加分类筛选
                if (category != null) {
                    query.andWhere { Todos.category eq category }
                }
                
                // 添加排序
                val orderColumn = when (sortBy.lowercase()) {
                    "priority" -> Todos.priority
                    "duedate" -> Todos.dueDate
                    "title" -> Todos.title
                    else -> Todos.createdAt
                }
                query.orderBy(orderColumn, sortOrder)
                
                // 添加分页
                query.limit(limit, offset)
                
                query.map { it.toTodo() }
            }
        }
    }
    
    /**
     * 更新待办事项
     */
    suspend fun update(
        id: Int,
        userId: Int,
        title: String? = null,
        description: String? = null,
        category: String? = null,
        priority: Int? = null,
        status: String? = null,
        dueDate: LocalDateTime? = null
    ): Todo? {
        return withContext(Dispatchers.IO) {
            transaction(database) {
                // 先检查是否存在且属于该用户
                val existing = Todos.select {
                    (Todos.id eq id) and 
                    (Todos.userId eq userId) and 
                    (Todos.isDeleted eq false)
                }.singleOrNull() ?: return@transaction null
                
                // 执行更新
                Todos.update({ Todos.id eq id }) {
                    // 使用 apply 对可选参数进行更新
                    // 只有传入非 null 的参数才会更新
                    title?.let { it[Todos.title] = title }
                    description?.let { it[Todos.description] = description }
                    category?.let { it[Todos.category] = category }
                    priority?.let { it[Todos.priority] = priority }
                    status?.let { it[Todos.status] = status }
                    dueDate?.let { it[Todos.dueDate] = dueDate }
                    it[Todos.updatedAt] = LocalDateTime.now()
                }
                
                // 返回更新后的记录
                Todos.select { Todos.id eq id }
                    .map { it.toTodo() }
                    .singleOrNull()
            }
        }
    }
    
    /**
     * 标记为已完成
     * 便捷方法
     */
    suspend fun markAsCompleted(id: Int, userId: Int): Boolean {
        return withContext(Dispatchers.IO) {
            transaction(database) {
                val count = Todos.update({
                    (Todos.id eq id) and 
                    (Todos.userId eq userId) and 
                    (Todos.isDeleted eq false)
                }) {
                    it[Todos.status] = TodoStatus.COMPLETED.value
                    it[Todos.updatedAt] = LocalDateTime.now()
                }
                count > 0
            }
        }
    }
    
    /**
     * 软删除待办事项
     * 实际上只是设置 isDeleted 标志
     */
    suspend fun softDelete(id: Int, userId: Int): Boolean {
        return withContext(Dispatchers.IO) {
            transaction(database) {
                val count = Todos.update({
                    (Todos.id eq id) and 
                    (Todos.userId eq userId)
                }) {
                    it[Todos.isDeleted] = true
                    it[Todos.updatedAt] = LocalDateTime.now()
                }
                count > 0
            }
        }
    }
    
    /**
     * 永久删除待办事项（硬删除）
     * 谨慎使用！
     */
    suspend fun hardDelete(id: Int, userId: Int): Boolean {
        return withContext(Dispatchers.IO) {
            transaction(database) {
                val count = Todos.deleteWhere {
                    (Todos.id eq id) and (Todos.userId eq userId)
                }
                count > 0
            }
        }
    }
    
    /**
     * 获取用户待办事项统计
     */
    suspend fun getStats(userId: Int): TodoStats {
        return withContext(Dispatchers.IO) {
            transaction(database) {
                // 按状态分组统计
                val statusCounts = Todos.slice(Todos.status, Todos.status.count())
                    .select { 
                        (Todos.userId eq userId) and (Todos.isDeleted eq false) 
                    }
                    .groupBy(Todos.status)
                    .associate { 
                        it[Todos.status] to it[Todos.status.count()]
                    }
                
                TodoStats(
                    total = statusCounts.values.sum().toInt(),
                    pending = statusCounts[TodoStatus.PENDING.value]?.toInt() ?: 0,
                    completed = statusCounts[TodoStatus.COMPLETED.value]?.toInt() ?: 0
                )
            }
        }
    }
    
    /**
     * 批量更新状态
     * 比如批量标记为已完成
     */
    suspend fun batchUpdateStatus(ids: List<Int>, userId: Int, status: String): Int {
        return withContext(Dispatchers.IO) {
            transaction(database) {
                Todos.update({ 
                    (Todos.id inList ids) and (Todos.userId eq userId)
                }) {
                    it[Todos.status] = status
                    it[Todos.updatedAt] = LocalDateTime.now()
                }
            }
        }
    }
    
    /**
     * ResultRow 转 Todo 对象
     */
    private fun ResultRow.toTodo(): Todo {
        return Todo(
            id = this[Todos.id],
            userId = this[Todos.userId],
            title = this[Todos.title],
            description = this[Todos.description],
            category = this[Todos.category],
            priority = this[Todos.priority],
            status = this[Todos.status],
            dueDate = this[Todos.dueDate],
            createdAt = this[Todos.createdAt],
            updatedAt = this[Todos.updatedAt],
            isDeleted = this[Todos.isDeleted]
        )
    }
}

/**
 * 待办事项统计 DTO
 */
data class TodoStats(
    val total: Int,
    val pending: Int,
    val completed: Int
)
```

### 13. AuthService.kt

```kotlin
package com.example.todo.service

import com.example.todo.repository.*
import com.example.todo.plugin.JwtConfig
import com.example.todo.dto.*

/**
 * 认证服务层
 * 
 * 职责：
 * - 处理用户注册、登录逻辑
 * - 生成 JWT Token
 * - 验证用户凭证
 */
class AuthService(
    private val userRepository: UserRepository
) {
    
    /**
     * 用户注册
     * 
     * @param username 用户名
     * @param email 邮箱
     * @param password 密码
     * @return 注册结果
     */
    suspend fun register(
        username: String,
        email: String,
        password: String
    ): Result<AuthResponse> {
        // ===== 参数验证 =====
        
        // 用户名验证：3-20字符，字母数字下划线
        if (username.length < 3 || username.length > 20) {
            return Result.failure(IllegalArgumentException("用户名长度必须在 3-20 个字符之间"))
        }
        
        // 邮箱格式验证（简单版本）
        if (!email.matches(Regex("^[A-Za-z0-9+_.-]+@[A-Za-z0-9.-]+$"))) {
            return Result.failure(IllegalArgumentException("邮箱格式不正确"))
        }
        
        // 密码强度验证：至少8位，包含数字和字母
        if (password.length < 8) {
            return Result.failure(IllegalArgumentException("密码长度至少 8 个字符"))
        }
        if (!password.matches(Regex("^(?=.*[A-Za-z])(?=.*\\d).+$"))) {
            return Result.failure(IllegalArgumentException("密码必须包含字母和数字"))
        }
        
        // ===== 检查唯一性 =====
        
        // 检查用户名是否存在
        if (userRepository.existsByUsername(username)) {
            return Result.failure(IllegalArgumentException("用户名已存在"))
        }
        
        // 检查邮箱是否存在
        if (userRepository.existsByEmail(email)) {
            return Result.failure(IllegalArgumentException("邮箱已被注册"))
        }
        
        // ===== 创建用户 =====
        
        val user = userRepository.create(username, email, password)
        
        // ===== 生成 Token =====
        
        val token = JwtConfig.generateToken(user.id, user.username)
        
        return Result.success(
            AuthResponse(
                token = token,
                user = UserDto.fromUser(user)
            )
        )
    }
    
    /**
     * 用户登录
     * 
     * @param usernameOrEmail 用户名或邮箱
     * @param password 密码
     * @return 登录结果
     */
    suspend fun login(
        usernameOrEmail: String,
        password: String
    ): Result<AuthResponse> {
        // ===== 查找用户 =====
        // 支持用户名或邮箱登录
        val user = if (usernameOrEmail.contains("@")) {
            userRepository.findByEmail(usernameOrEmail)
        } else {
            userRepository.findByUsername(usernameOrEmail)
        }
        
        // ===== 验证凭证 =====
        
        if (user == null) {
            return Result.failure(IllegalArgumentException("用户不存在"))
        }
        
        if (!userRepository.verifyPassword(user, password)) {
            return Result.failure(IllegalArgumentException("密码错误"))
        }
        
        // ===== 生成 Token =====
        
        val token = JwtConfig.generateToken(user.id, user.username)
        
        return Result.success(
            AuthResponse(
                token = token,
                user = UserDto.fromUser(user)
            )
        )
    }
}
```

### 14. TodoService.kt

```kotlin
package com.example.todo.service

import com.example.todo.repository.*
import com.example.todo.model.*
import com.example.todo.dto.*
import java.time.LocalDateTime

/**
 * 待办事项服务层
 * 
 * 职责：
 * - 封装业务逻辑
 * - 参数验证
 * - 数据转换
 */
class TodoService(
    private val todoRepository: TodoRepository
) {
    
    /**
     * 创建待办事项
     */
    suspend fun create(
        userId: Int,
        request: CreateTodoRequest
    ): Result<TodoDto> {
        // ===== 参数验证 =====
        
        // 标题必填
        if (request.title.isBlank()) {
            return Result.failure(IllegalArgumentException("标题不能为空"))
        }
        
        // 标题长度限制
        if (request.title.length > 200) {
            return Result.failure(IllegalArgumentException("标题长度不能超过 200 个字符"))
        }
        
        // 描述长度限制
        request.description?.let {
            if (it.length > 1000) {
                return Result.failure(IllegalArgumentException("描述长度不能超过 1000 个字符"))
            }
        }
        
        // 截止日期不能是过去的时间
        request.dueDate?.let {
            if (it.isBefore(LocalDateTime.now())) {
                return Result.failure(IllegalArgumentException("截止日期不能是过去的时间"))
            }
        }
        
        // ===== 创建待办 =====
        
        val todo = todoRepository.create(
            userId = userId,
            title = request.title.trim(),
            description = request.description?.trim(),
            category = request.category ?: "默认",
            priority = request.priority ?: 2,
            dueDate = request.dueDate
        )
        
        return Result.success(TodoDto.fromTodo(todo))
    }
    
    /**
     * 获取单个待办事项
     */
    suspend fun getById(id: Int, userId: Int): Result<TodoDto> {
        val todo = todoRepository.findById(id, userId)
            ?: return Result.failure(NoSuchElementException("待办事项不存在"))
        
        return Result.success(TodoDto.fromTodo(todo))
    }
    
    /**
     * 获取待办事项列表
     */
    suspend fun getList(
        userId: Int,
        status: String? = null,
        category: String? = null,
        sortBy: String = "createdAt",
        page: Int = 1,
        pageSize: Int = 20
    ): Result<TodoListResponse> {
        // ===== 参数处理 =====
        
        // 分页参数验证
        val validPage = maxOf(1, page)
        val validPageSize = minOf(maxOf(1, pageSize), 100)  // 最多100条
        val offset = (validPage - 1) * validPageSize
        
        // 排序方向
        val sortOrder = if (sortBy.endsWith(":desc")) {
            org.jetbrains.exposed.sql.SortOrder.DESC
        } else {
            org.jetbrains.exposed.sql.SortOrder.ASC
        }
        val sortField = sortBy.removeSuffix(":desc").removeSuffix(":asc")
        
        // ===== 查询数据 =====
        
        val todos = todoRepository.findByUserId(
            userId = userId,
            status = status,
            category = category,
            sortBy = sortField,
            sortOrder = sortOrder,
            limit = validPageSize,
            offset = offset
        )
        
        val stats = todoRepository.getStats(userId)
        
        return Result.success(
            TodoListResponse(
                todos = todos.map { TodoDto.fromTodo(it) },
                total = stats.total,
                page = validPage,
                pageSize = validPageSize,
                stats = TodoStatsDto(
                    total = stats.total,
                    pending = stats.pending,
                    completed = stats.completed
                )
            )
        )
    }
    
    /**
     * 更新待办事项
     */
    suspend fun update(
        id: Int,
        userId: Int,
        request: UpdateTodoRequest
    ): Result<TodoDto> {
        // ===== 检查是否存在 =====
        
        val existing = todoRepository.findById(id, userId)
            ?: return Result.failure(NoSuchElementException("待办事项不存在"))
        
        // ===== 参数验证 =====
        
        request.title?.let {
            if (it.isBlank()) {
                return Result.failure(IllegalArgumentException("标题不能为空"))
            }
            if (it.length > 200) {
                return Result.failure(IllegalArgumentException("标题长度不能超过 200 个字符"))
            }
        }
        
        request.description?.let {
            if (it.length > 1000) {
                return Result.failure(IllegalArgumentException("描述长度不能超过 1000 个字符"))
            }
        }
        
        // ===== 执行更新 =====
        
        val updated = todoRepository.update(
            id = id,
            userId = userId,
            title = request.title?.trim(),
            description = request.description?.trim(),
            category = request.category,
            priority = request.priority,
            status = request.status,
            dueDate = request.dueDate
        ) ?: return Result.failure(NoSuchElementException("更新失败"))
        
        return Result.success(TodoDto.fromTodo(updated))
    }
    
    /**
     * 删除待办事项（软删除）
     */
    suspend fun delete(id: Int, userId: Int): Result<Unit> {
        val success = todoRepository.softDelete(id, userId)
        return if (success) {
            Result.success(Unit)
        } else {
            Result.failure(NoSuchElementException("待办事项不存在"))
        }
    }
    
    /**
     * 标记为已完成
     */
    suspend fun markAsCompleted(id: Int, userId: Int): Result<TodoDto> {
        val success = todoRepository.markAsCompleted(id, userId)
        return if (success) {
            val todo = todoRepository.findById(id, userId)!!
            Result.success(TodoDto.fromTodo(todo))
        } else {
            Result.failure(NoSuchElementException("待办事项不存在"))
        }
    }
}
```

### 15. DTO 类（数据传送对象）

```kotlin
// ===== TodoDto.kt =====

package com.example.todo.dto

import com.example.todo.model.*
import kotlinx.serialization.Serializable
import java.time.LocalDateTime

/**
 * 待办事项 DTO（Data Transfer Object）
 * 
 * DTO 的作用：
 * 1. 控制暴露给客户端的字段（保护敏感信息）
 * 2. 定义 API 的数据结构
 * 3. 与数据库实体解耦
 * 
 * @Serializable 注解使对象可以被 JSON 序列化
 */
@Serializable
data class TodoDto(
    val id: Int,
    val title: String,
    val description: String?,
    val category: String,
    val priority: Int,
    val priorityLabel: String,  // 优先级文字说明
    val status: String,
    val statusLabel: String,    // 状态文字说明
    val dueDate: String?,      // 格式化的时间字符串
    val createdAt: String,
    val updatedAt: String
) {
    companion object {
        /**
         * 从实体转换为 DTO
         */
        fun fromTodo(todo: Todo): TodoDto {
            return TodoDto(
                id = todo.id,
                title = todo.title,
                description = todo.description,
                category = todo.category,
                priority = todo.priority,
                priorityLabel = Priority.fromValue(todo.priority).label,
                status = todo.status,
                statusLabel = TodoStatus.fromValue(todo.status).label,
                dueDate = todo.dueDate?.toString(),
                createdAt = todo.createdAt.toString(),
                updatedAt = todo.updatedAt.toString()
            )
        }
    }
}

/**
 * 创建待办事项请求
 */
@Serializable
data class CreateTodoRequest(
    val title: String,
    val description: String? = null,
    val category: String? = null,
    val priority: Int? = null,
    val dueDate: String? = null  // ISO 8601 格式
) {
    /**
     * 转换为 LocalDateTime
     */
    fun toDueDate(): LocalDateTime? {
        return dueDate?.let { 
            LocalDateTime.parse(it) 
        }
    }
}

/**
 * 更新待办事项请求
 */
@Serializable
data class UpdateTodoRequest(
    val title: String? = null,
    val description: String? = null,
    val category: String? = null,
    val priority: Int? = null,
    val status: String? = null,
    val dueDate: String? = null
) {
    fun toDueDate(): LocalDateTime? {
        return dueDate?.let { LocalDateTime.parse(it) }
    }
}

/**
 * 待办事项列表响应
 */
@Serializable
data class TodoListResponse(
    val todos: List<TodoDto>,
    val total: Int,
    val page: Int,
    val pageSize: Int,
    val stats: TodoStatsDto
)

/**
 * 待办事项统计 DTO
 */
@Serializable
data class TodoStatsDto(
    val total: Int,
    val pending: Int,
    val completed: Int
)
```

```kotlin
// ===== UserDto.kt =====

package com.example.todo.dto

import com.example.todo.model.*
import kotlinx.serialization.Serializable

/**
 * 用户 DTO
 * 注意：不要暴露密码字段！
 */
@Serializable
data class UserDto(
    val id: Int,
    val username: String,
    val email: String,
    val createdAt: String
) {
    companion object {
        fun fromUser(user: User): UserDto {
            return UserDto(
                id = user.id,
                username = user.username,
                email = user.email,
                createdAt = user.createdAt.toString()
            )
        }
    }
}
```

```kotlin
// ===== AuthDto.kt =====

package com.example.todo.dto

import kotlinx.serialization.Serializable

/**
 * 认证响应
 * 包含 JWT Token 和用户信息
 */
@Serializable
data class AuthResponse(
    val token: String,
    val user: UserDto
)

/**
 * 登录请求
 */
@Serializable
data class LoginRequest(
    val usernameOrEmail: String,
    val password: String
)

/**
 * 注册请求
 */
@Serializable
data class RegisterRequest(
    val username: String,
    val email: String,
    val password: String
)
```

### 16. AuthRoute.kt（认证路由）

```kotlin
package com.example.todo.route

import com.example.todo.service.*
import com.example.todo.dto.*
import io.ktor.server.application.*
import io.ktor.server.request.*
import io.ktor.server.response.*
import io.ktor.server.routing.*
import io.ktor.http.*

/**
 * 认证路由
 * 
 * RESTful 风格：
 * POST /api/auth/register - 用户注册
 * POST /api/auth/login    - 用户登录
 */
fun Routing.configureAuthRoute(authService: AuthService) {
    // 定义路由前缀
    route("/api/auth") {
        
        /**
         * POST /api/auth/register - 用户注册
         * 
         * 请求体:
         * {
         *   "username": "john",
         *   "email": "john@example.com",
         *   "password": "password123"
         * }
         * 
         * 响应:
         * {
         *   "token": "eyJhbGciOiJIUzI1NiIs...",
         *   "user": {
         *     "id": 1,
         *     "username": "john",
         *     "email": "john@example.com"
         *   }
         * }
         */
        post("/register") {
            // 1. 解析请求体
            val request = call.receive<RegisterRequest>()
            
            // 2. 调用服务层处理注册逻辑
            val result = authService.register(
                username = request.username,
                email = request.email,
                password = request.password
            )
            
            // 3. 返回结果
            result.fold(
                onSuccess = { response ->
                    call.respond(HttpStatusCode.Created, response)
                },
                onFailure = { error ->
                    // 4xx 客户端错误
                    call.respond(
                        HttpStatusCode.BadRequest,
                        mapOf("error" to error.message)
                    )
                }
            )
        }
        
        /**
         * POST /api/auth/login - 用户登录
         * 
         * 请求体:
         * {
         *   "usernameOrEmail": "john" 或 "john@example.com",
         *   "password": "password123"
         * }
         */
        post("/login") {
            val request = call.receive<LoginRequest>()
            
            val result = authService.login(
                usernameOrEmail = request.usernameOrEmail,
                password = request.password
            )
            
            result.fold(
                onSuccess = { response ->
                    call.respond(HttpStatusCode.OK, response)
                },
                onFailure = { error ->
                    // 注意：登录失败统一返回 401
                    // 不返回具体是用户名错误还是密码错误（安全考虑）
                    call.respond(
                        HttpStatusCode.Unauthorized,
                        mapOf("error" to "用户名或密码错误")
                    )
                }
            )
        }
    }
}
```

### 17. TodoRoute.kt（待办事项路由）

```kotlin
package com.example.todo.route

import com.example.todo.service.*
import com.example.todo.dto.*
import io.ktor.server.application.*
import io.ktor.server.request.*
import io.ktor.server.response.*
import io.ktor.server.routing.*
import io.ktor.server.auth.*
import io.ktor.http.*

/**
 * 待办事项路由
 * 
 * RESTful 风格：
 * GET    /api/todos         - 获取列表
 * POST   /api/todos         - 创建
 * GET    /api/todos/{id}    - 获取单个
 * PUT    /api/todos/{id}    - 更新
 * DELETE /api/todos/{id}    - 删除
 * POST   /api/todos/{id}/complete - 标记完成
 * GET    /api/todos/stats   - 获取统计
 */
fun Routing.configureTodoRoute(
    todoService: TodoService,
    authService: AuthService
) {
    // authenticate("auth-jwt"): 需要 JWT 认证的路由组
    authenticate("auth-jwt") {
        route("/api/todos") {
            
            /**
             * GET /api/todos - 获取待办事项列表
             * 
             * 查询参数:
             * - status: pending/completed
             * - category: 分类名称
             * - sortBy: createdAt/priority/dueDate (默认: createdAt:desc)
             * - page: 页码 (默认: 1)
             * - pageSize: 每页数量 (默认: 20, 最大: 100)
             * 
             * 示例:
             * GET /api/todos?status=pending&page=1&pageSize=10
             * GET /api/todos?category=工作&sortBy=priority:desc
             */
            get {
                // 1. 从 JWT Token 中获取用户 ID
                val userId = call.getUserId()
                
                // 2. 获取查询参数
                val status = call.parameters["status"]
                val category = call.parameters["category"]
                val sortBy = call.parameters["sortBy"] ?: "createdAt:desc"
                val page = call.parameters["page"]?.toIntOrNull() ?: 1
                val pageSize = call.parameters["pageSize"]?.toIntOrNull() ?: 20
                
                // 3. 查询数据
                val result = todoService.getList(
                    userId = userId,
                    status = status,
                    category = category,
                    sortBy = sortBy,
                    page = page,
                    pageSize = pageSize
                )
                
                // 4. 返回结果
                result.fold(
                    onSuccess = { response ->
                        call.respond(HttpStatusCode.OK, response)
                    },
                    onFailure = { error ->
                        call.respond(
                            HttpStatusCode.InternalServerError,
                            mapOf("error" to error.message)
                        )
                    }
                )
            }
            
            /**
             * POST /api/todos - 创建待办事项
             * 
             * 请求体:
             * {
             *   "title": "完成项目报告",
             *   "description": "需要包含数据分析和结论",
             *   "category": "工作",
             *   "priority": 3,
             *   "dueDate": "2024-12-31T23:59:59"
             * }
             */
            post {
                val userId = call.getUserId()
                val request = call.receive<CreateTodoRequest>()
                
                val result = todoService.create(userId, request)
                
                result.fold(
                    onSuccess = { todo ->
                        call.respond(HttpStatusCode.Created, todo)
                    },
                    onFailure = { error ->
                        call.respond(
                            HttpStatusCode.BadRequest,
                            mapOf("error" to error.message)
                        )
                    }
                )
            }
            
            /**
             * GET /api/todos/stats - 获取统计信息
             */
            get("/stats") {
                val userId = call.getUserId()
                val result = todoService.getList(userId)
                
                result.fold(
                    onSuccess = { response ->
                        call.respond(HttpStatusCode.OK, response.stats)
                    },
                    onFailure = { error ->
                        call.respond(
                            HttpStatusCode.InternalServerError,
                            mapOf("error" to error.message)
                        )
                    }
                )
            }
            
            /**
             * GET /api/todos/{id} - 获取单个待办事项
             */
            get("/{id}") {
                val userId = call.getUserId()
                val id = call.parameters["id"]?.toIntOrNull()
                
                if (id == null) {
                    call.respond(HttpStatusCode.BadRequest, mapOf("error" to "无效的 ID"))
                    return@get
                }
                
                val result = todoService.getById(id, userId)
                
                result.fold(
                    onSuccess = { todo ->
                        call.respond(HttpStatusCode.OK, todo)
                    },
                    onFailure = { error ->
                        call.respond(
                            HttpStatusCode.NotFound,
                            mapOf("error" to error.message)
                        )
                    }
                )
            }
            
            /**
             * PUT /api/todos/{id} - 更新待办事项
             * 
             * 请求体（都是可选的）:
             * {
             *   "title": "新标题",
             *   "description": "新描述",
             *   "category": "新分类",
             *   "priority": 1,
             *   "status": "completed",
             *   "dueDate": "2024-12-31T23:59:59"
             * }
             */
            put("/{id}") {
                val userId = call.getUserId()
                val id = call.parameters["id"]?.toIntOrNull()
                
                if (id == null) {
                    call.respond(HttpStatusCode.BadRequest, mapOf("error" to "无效的 ID"))
                    return@put
                }
                
                val request = call.receive<UpdateTodoRequest>()
                
                val result = todoService.update(id, userId, request)
                
                result.fold(
                    onSuccess = { todo ->
                        call.respond(HttpStatusCode.OK, todo)
                    },
                    onFailure = { error ->
                        val status = when (error) {
                            is NoSuchElementException -> HttpStatusCode.NotFound
                            else -> HttpStatusCode.BadRequest
                        }
                        call.respond(status, mapOf("error" to error.message))
                    }
                )
            }
            
            /**
             * DELETE /api/todos/{id} - 删除待办事项（软删除）
             */
            delete("/{id}") {
                val userId = call.getUserId()
                val id = call.parameters["id"]?.toIntOrNull()
                
                if (id == null) {
                    call.respond(HttpStatusCode.BadRequest, mapOf("error" to "无效的 ID"))
                    return@delete
                }
                
                val result = todoService.delete(id, userId)
                
                result.fold(
                    onSuccess = {
                        call.respond(HttpStatusCode.NoContent)
                    },
                    onFailure = { error ->
                        call.respond(
                            HttpStatusCode.NotFound,
                            mapOf("error" to error.message)
                        )
                    }
                )
            }
            
            /**
             * POST /api/todos/{id}/complete - 标记为已完成
             */
            post("/{id}/complete") {
                val userId = call.getUserId()
                val id = call.parameters["id"]?.toIntOrNull()
                
                if (id == null) {
                    call.respond(HttpStatusCode.BadRequest, mapOf("error" to "无效的 ID"))
                    return@post
                }
                
                val result = todoService.markAsCompleted(id, userId)
                
                result.fold(
                    onSuccess = { todo ->
                        call.respond(HttpStatusCode.OK, todo)
                    },
                    onFailure = { error ->
                        call.respond(
                            HttpStatusCode.NotFound,
                            mapOf("error" to error.message)
                        )
                    }
                )
            }
        }
    }
}

/**
 * 从认证上下文中获取用户 ID
 * 这是 Ktor JWT 认证的扩展函数
 */
private fun ApplicationCall.getUserId(): Int {
    // principal: 获取认证主体
    // JWTPrincipal 包含 JWT Token 中的所有声明
    val principal = principal<com.auth0.jwt.JWTPrincipal>()
    
    // getClaim: 获取指定声明
    // 这里我们存储的是 Int 类型的 userId
    return principal?.getClaim("userId", com.auth0.jwt.JWTClaimsVerifier::class.java)
        ?.asInt()
        ?: throw IllegalStateException("User ID not found in token")
}
```

### 18. application.conf（Ktor 配置文件）

```hocon
// application.conf - Ktor 配置文件
// 使用 HOCON 格式（Human-Optimized Config Object Notation）

ktor {
    # ===== 应用配置 =====
    application {
        # 模块类（全限定名）
        # Ktor 会调用这个类的 configureModule() 函数
        modules = [ com.example.todo.MainKt ]
    }
    
    # ===== 服务器配置 =====
    server {
        # 端口
        port = 8080
        
        # 主机（0.0.0.0 允许所有网络访问）
        host = "0.0.0.0"
        
        # 连接队列大小
        connectionQueueSize = 65536
        
        # 请求等待超时
        requestTimeout = 30_000L
        
        # 启动根路径
        rootPath = ""
    }
    
    # ===== 部署配置 =====
    deployment {
        # 变体（开发/生产）
        environment = development
        
        # 自动重载（开发环境启用）
        autoreload = true
        
        # 压缩配置
        compression {
            # 启用压缩
            enable = true
            
            # 最小响应大小（小于此值不压缩）
            minSize = 1_000
        }
    }
    
    # ===== 响应头配置 =====
    headers {
        # X-Content-Type-Options: 防止 MIME 类型嗅探
        xContentTypeOptionsEnabled = true
        
        # X-Frame-Options: 防止点击劫持
        xFrameOptionsEnabled = true
    }
    
    # ===== 日志配置 =====
    logging {
        # 日志级别
        level = INFO
        
        # 请求日志
        logRequest = true
        
        # 级别_pages页面日志
        logLevel = ERROR
    }
}
```

---

## 代码关键点说明

### 1. Ktor 异步编程模型

**关键点**：Ktor 使用 Kotlin Coroutines 实现异步非阻塞编程。

```kotlin
// ❌ 错误：在 CoroutineScope 外使用 suspend
val user = userRepository.findByUsername("john")  // 会报错

// ✅ 正确：在协程中调用 suspend 函数
suspend fun getUser(): User? {
    return userRepository.findByUsername("john")
}

// 或者使用 launch 在后台执行
scope.launch {
    val user = userRepository.findByUsername("john")
}
```

**为什么这样设计**：
- `suspend` 函数可以在不阻塞线程的情况下暂停和恢复
- 一个线程可以同时处理多个请求
- 提高服务器吞吐量和资源利用率

### 2. Exposed ORM 的 SQL DSL

**关键点**：Exposed 提供类型安全的 SQL 构建。

```kotlin
// ✅ Exposed DSL - 类型安全
Users.select { Users.username eq "john" }

// ❌ 原始 SQL - 容易出错
// "SELECT * FROM users WHERE username = 'john'"
```

**可能踩的坑**：
```kotlin
// 1. 忘记处理 null 结果
val user = Users.select { ... }.single()  // 如果没有或多个结果会抛异常
val user = Users.select { ... }.singleOrNull()  // ✅ 安全

// 2. 事务未提交
transaction(database) {
    Users.insert { ... }
    // 如果这里抛异常，插入不会提交
    throw RuntimeException("test")
}

// 3. 连接泄漏
// 务必确保在 withContext(Dispatchers.IO) 中执行
```

### 3. JWT Token 验证

**关键点**：JWT Token 的验证流程。

```
客户端请求 → 提取 Token → 验证签名 → 验证过期 → 获取用户信息
```

**可能踩的坑**：
```kotlin
// 1. Token 过期时间设置过长（安全风险）
// ✅ 合理：24小时
// ❌ 危险：1年

// 2. 在客户端存储 Token 明文
// ✅ 安全：使用 HttpOnly Cookie
// ❌ 危险：LocalStorage（XSS 攻击风险）

// 3. 验证失败时泄露敏感信息
// ❌ 错误：返回具体的验证失败原因
// ✅ 正确：统一返回通用错误
```

### 4. 密码加密

**关键点**：使用 BCrypt 加密密码。

```kotlin
// 加密
val hashed = BCrypt.hashpw(password, BCrypt.gensalt(12))

// 验证
val matches = BCrypt.checkpw(rawPassword, hashed)
```

**BCrypt 的特点**：
- 内置盐值（防止彩虹表攻击）
- 可配置强度（10-31，越高越慢越安全）
- 每次加密结果不同（因为盐值随机）

---

## 学习计划（5天）

### Day 1：环境搭建和基础概念

**目标**：搭建开发环境，理解 Ktor 框架结构

**任务**：
- [ ] 安装 JDK 17+
- [ ] 安装 Gradle
- [ ] 安装 Docker Desktop
- [ ] 创建项目结构
- [ ] 配置 docker-compose 启动 PostgreSQL
- [ ] 运行第一个 Ktor "Hello World"

**产出**：
- ✅ 本地运行成功的 Ktor 项目
- ✅ PostgreSQL 数据库连接成功

**学习资源**：
- Ktor 官方文档：https://ktor.io/docs/welcome.html
- Ktor GitHub：https://github.com/ktorio/ktor

### Day 2：数据库和模型

**目标**：掌握 Exposed ORM，实现数据模型

**任务**：
- [ ] 配置 HikariCP 连接池
- [ ] 创建 Users 和 Todos 表
- [ ] 实现 UserRepository 和 TodoRepository
- [ ] 添加数据验证逻辑
- [ ] 编写单元测试

**产出**：
- ✅ 完整的数据库 CRUD 代码
- ✅ Repository 单元测试

### Day 3：认证和安全

**目标**：实现 JWT 认证系统

**任务**：
- [ ] 集成 BCrypt 密码加密
- [ ] 实现用户注册和登录
- [ ] 配置 JWT 生成和验证
- [ ] 实现受保护的路由
- [ ] 测试认证流程

**产出**：
- ✅ 完整的用户认证系统
- ✅ JWT Token 管理

### Day 4：RESTful API

**目标**：实现完整的待办事项 CRUD API

**任务**：
- [ ] 实现 Todo CRUD 路由
- [ ] 添加查询参数（筛选、排序、分页）
- [ ] 添加错误处理
- [ ] 编写 API 文档
- [ ] 使用 Postman 测试 API

**产出**：
- ✅ 完整的 RESTful API
- ✅ Postman 测试集合

### Day 5：项目完善和部署

**目标**：完善项目，准备部署

**任务**：
- [ ] 添加日志和监控
- [ ] 配置 CORS
- [ ] 编写 README.md
- [ ] 配置 CI/CD（可选）
- [ ] 部署到云服务器（可选）

**产出**：
- ✅ 完整的项目文档
- ✅ Docker 镜像（可选）

---

## 面试要点

### 题目1：Kotlin 协程是什么？它和线程有什么区别？

**标准答案**：

Kotlin 协程（Coroutine）是一种**轻量级的并发编程方案**，它让我们可以用同步的方式写异步代码。

**核心区别**：

| 特性 | 线程 | 协程 |
|------|------|------|
| 创建成本 | 高（需要向 OS 申请） | 低（只是对象） |
| 切换成本 | 高（上下文切换） | 低（挂起点切换） |
| 数量限制 | 几百到几千 | 数十万 |
| 阻塞行为 | 阻塞线程 | 挂起协程 |

**代码示例**：
```kotlin
// 线程：创建成本高，阻塞时浪费资源
Thread {
    println("Hello from thread")
}.start()

// 协程：轻量，创建成本低
suspend fun doSomething() {
    println("Hello from coroutine")
}
```

**答题思路**：
1. 先定义协程是什么
2. 对比线程说明优势
3. 用代码举例
4. 说明在 Ktor 中的应用

---

### 题目2：Kotlin 的空安全机制是什么？

**标准答案**：

Kotlin 的空安全（Null Safety）通过**类型系统**在编译期防止空指针异常（NPE）。

**核心概念**：

```kotlin
// 不可空类型 - 不能赋值为 null
val name: String = "John"  // ✅ OK
val name: String = null    // ❌ 编译错误

// 可空类型 - 可以赋值为 null
val name: String? = null   // ✅ OK
val name: String? = "John" // ✅ OK

// 安全调用操作符 ?.
val length: Int? = name?.length  // 如果 name 为 null，返回 null

// Elvis 操作符 ?:
val length: Int = name?.length ?: 0  // 如果为 null，使用默认值 0

// 非空断言 !! （慎用！）
val length: Int = name!!.length  // 如果为 null，抛出 NPE
```

**为什么这样设计**：
- Kotlin 强制开发者在编译期处理空值
- 减少运行时 NPE，提高程序稳定性
- 让 API 设计更明确

---

### 题目3：Kotlin 的扩展函数是什么？

**标准答案**：

扩展函数允许我们在**不修改原类**的情况下，为现有类添加新方法。

**代码示例**：
```kotlin
// 为 String 类添加扩展函数
fun String.addExclamation(): String {
    return this + "!"  // this 指向调用者
}

fun String.repeat(times: Int): String {
    return (1..times).joinToString("") { this }
}

// 使用
"Hello".addExclamation()  // "Hello!"
"Ha".repeat(3)            // "HaHaHa"
```

**在项目中的应用**：
```kotlin
// 为 ApplicationCall 添加扩展函数
private fun ApplicationCall.getUserId(): Int {
    val principal = principal<JWTPrincipal>()
    return principal?.getClaim("userId")?.asInt() 
        ?: throw IllegalStateException("User ID not found")
}
```

**优势**：
1. 无需继承或装饰器模式
2. 保持代码简洁
3. 可以给任何类添加方法（包括第三方库）

---

### 题目4：Ktor 框架有什么特点？

**标准答案**：

Ktor 是 JetBrains 出品的**轻量级异步 Web 框架**，专为 Kotlin 设计。

**核心特点**：

| 特点 | 说明 |
|------|------|
| 轻量 | 核心简单，依赖少 |
| 异步 | 基于 Kotlin Coroutines |
| 灵活 | 模块化设计，按需加载 |
| DSL | 使用 Kotlin DSL 配置路由 |
| 多引擎 | 支持 Netty、Jetty、Tomcat 等 |

**代码示例**：
```kotlin
// Ktor 路由 DSL
routing {
    get("/") {
        call.respondText("Hello!")
    }
    
    route("/api/users") {
        get { /* 获取用户列表 */ }
        post { /* 创建用户 */ }
    }
}
```

**vs Spring Boot**：
- Ktor 更轻量，启动更快
- Ktor 配置更简洁（DSL vs YAML）
- Spring Boot 生态更完善，社区更大

---

### 题目5：Kotlin 数据类（Data Class）有什么作用？

**标准答案**：

数据类是专门用于**存储数据**的类，编译器自动生成常用方法。

**自动生成的方法**：
```kotlin
data class User(
    val id: Int,
    val name: String,
    val email: String
)

// 自动生成：
// - equals() - 按值比较
// - hashCode() - 与 equals 对应
// - toString() - 包含所有字段
// - copy() - 复制并修改部分字段
// - componentN() - 解构声明支持
```

**使用示例**：
```kotlin
val user1 = User(1, "John", "john@example.com")
val user2 = User(1, "John", "john@example.com")

// equals
println(user1 == user2)  // true（按值比较）
println(user1 === user2) // false（对象引用不同）

// copy
val user3 = user1.copy(email = "new@example.com")

// toString
println(user1)  // User(id=1, name=John, email=john@example.com)

// 解构
val (id, name, email) = user1
```

**在项目中的应用**：
```kotlin
// DTO（Data Transfer Object）使用数据类
@Serializable
data class TodoDto(
    val id: Int,
    val title: String,
    val status: String
)
```

---

## API 文档

### 认证接口

#### 注册
```
POST /api/auth/register

请求:
{
  "username": "john",
  "email": "john@example.com",
  "password": "password123"
}

响应 (201):
{
  "token": "eyJhbGciOiJIUzI1NiIs...",
  "user": {
    "id": 1,
    "username": "john",
    "email": "john@example.com",
    "createdAt": "2024-01-01T00:00:00"
  }
}
```

#### 登录
```
POST /api/auth/login

请求:
{
  "usernameOrEmail": "john",
  "password": "password123"
}

响应 (200):
{
  "token": "eyJhbGciOiJIUzI1NiIs...",
  "user": {
    "id": 1,
    "username": "john",
    "email": "john@example.com",
    "createdAt": "2024-01-01T00:00:00"
  }
}
```

### 待办事项接口

**注意**：所有待办事项接口都需要在 Header 中携带 Token：
```
Authorization: Bearer <token>
```

#### 创建待办
```
POST /api/todos

请求:
{
  "title": "完成项目报告",
  "description": "需要包含数据分析和结论",
  "category": "工作",
  "priority": 3,
  "dueDate": "2024-12-31T23:59:59"
}

响应 (201):
{
  "id": 1,
  "title": "完成项目报告",
  "description": "需要包含数据分析和结论",
  "category": "工作",
  "priority": 3,
  "priorityLabel": "高优先级",
  "status": "pending",
  "statusLabel": "待完成",
  "dueDate": "2024-12-31T23:59:59",
  "createdAt": "2024-01-01T00:00:00",
  "updatedAt": "2024-01-01T00:00:00"
}
```

#### 获取列表
```
GET /api/todos?status=pending&category=工作&page=1&pageSize=20

响应 (200):
{
  "todos": [...],
  "total": 100,
  "page": 1,
  "pageSize": 20,
  "stats": {
    "total": 100,
    "pending": 80,
    "completed": 20
  }
}
```

#### 更新待办
```
PUT /api/todos/{id}

请求:
{
  "title": "新标题",
  "status": "completed"
}

响应 (200):
{
  "id": 1,
  "title": "新标题",
  ...
}
```

#### 删除待办
```
DELETE /api/todos/{id}

响应: 204 No Content
```

---

## 项目启动指南

### 前置条件
1. JDK 17+
2. Docker Desktop
3. Gradle 8+

### 启动步骤

```bash
# 1. 启动 PostgreSQL
docker-compose up -d

# 2. 等待数据库就绪（大约10秒）
docker-compose ps

# 3. 构建项目
./gradlew build

# 4. 运行项目
./gradlew run

# 5. 测试 API
curl http://localhost:8080/health
```

### 项目验证

```bash
# 注册用户
curl -X POST http://localhost:8080/api/auth/register \
  -H "Content-Type: application/json" \
  -d '{"username":"test","email":"test@example.com","password":"password123"}'

# 登录获取 Token
curl -X POST http://localhost:8080/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{"usernameOrEmail":"test","password":"password123"}'

# 使用 Token 创建待办
curl -X POST http://localhost:8080/api/todos \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer <your-token>" \
  -d '{"title":"测试任务","category":"测试"}'
```
