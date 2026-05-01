# Gin+React 方案二：在线商城 API 项目

## 📚 项目介绍

### 1.1 项目概述

这是一个完整的电商后端 API 系统，使用 Go + Gin 框架构建，提供商品管理、购物车、订单处理、支付集成等核心电商功能。项目采用分层架构设计，代码结构清晰，适合作为面试展示项目。

**核心功能模块：**
- 用户模块：注册、登录、收货地址管理
- 商品模块：商品浏览、搜索、分类筛选、详情查看
- 购物车模块：添加商品、修改数量、删除商品
- 订单模块：创建订单、订单列表、订单状态流转
- 支付模块：集成支付宝/微信支付（沙箱环境）

### 1.2 核心技术亮点

```
技术架构：
├── Go + Gin              // 高性能 Web 框架
├── GORM                  // 优雅的 ORM 库
├── PostgreSQL            // 主数据库
├── Redis                 // 缓存层 + Session 存储
├── JWT                   // 无状态认证
├── 策略模式              // 支付方式切换
├── 乐观锁                // 库存并发控制
└── 雪花算法              // 分布式 ID 生成
```

### 1.3 适合场景

- 展示电商系统设计能力
- 理解复杂业务逻辑（订单状态机、事务处理）
- 掌握 Redis 缓存和数据库事务

---

## 🔧 完整可运行代码

### 2.1 项目结构

```
ecommerce-api/
├── cmd/server/main.go
├── config/config.go
├── internal/
│   ├── handler/         # HTTP 处理器
│   │   ├── auth.go
│   │   ├── product.go
│   │   ├── cart.go
│   │   ├── order.go
│   │   └── payment.go
│   ├── middleware/     # 中间件
│   │   ├── auth.go
│   │   ├── ratelimit.go  # 限流中间件
│   │   └── logger.go
│   ├── model/          # 数据模型
│   │   ├── user.go
│   │   ├── product.go
│   │   ├── cart.go
│   │   ├── order.go
│   │   └── address.go
│   ├── repository/     # 数据访问层
│   │   ├── user.go
│   │   ├── product.go
│   │   ├── cart.go
│   │   └── order.go
│   └── service/        # 业务逻辑层
│       ├── auth.go
│       ├── product.go
│       ├── cart.go
│       ├── order.go
│       └── payment.go
├── pkg/
│   ├── response/       # 统一响应
│   │   └── response.go
│   ├── jwt/
│   │   └── jwt.go
│   ├── idgen/          # ID 生成器
│   │   └── snowflake.go
│   └── errors/         # 自定义错误
│       └── errors.go
├── go.mod
└── .env
```

### 2.2 核心代码实现

#### 2.2.1 主程序入口 main.go

```go
package main

import (
	"context"
	"fmt"
	"log"
	"net/http"
	"os"
	"os/signal"
	"syscall"
	"time"

	"ecommerce-api/config"
	"ecommerce-api/internal/handler"
	"ecommerce-api/internal/middleware"
	"ecommerce-api/internal/model"
	"ecommerce-api/internal/repository"
	"ecommerce-api/internal/service"
	"ecommerce-api/pkg/idgen"

	"github.com/gin-gonic/gin"
	"github.com/joho/godotenv"
	"github.com/redis/go-redis/v9"
	"gorm.io/driver/postgres"
	"gorm.io/gorm"
	"gorm.io/gorm/logger"
)

func main() {
	// ========== 第1步：加载环境变量 ==========
	if err := godotenv.Load(); err != nil {
		log.Printf("⚠️  未找到 .env 文件")
	}

	// ========== 第2步：加载配置 ==========
	cfg := config.Load()

	// ========== 第3步：初始化数据库 ==========
	db, err := initDatabase(cfg)
	if err != nil {
		log.Fatalf("❌ 数据库初始化失败: %v", err)
	}

	// ========== 第4步：初始化 Redis ==========
	rdb := initRedis(cfg)
	ctx := context.Background()
	if err := rdb.Ping(ctx).Err(); err != nil {
		log.Printf("⚠️  Redis 连接失败: %v（部分功能可能受限）", err)
	}

	// ========== 第5步：初始化 ID 生成器 ==========
	// 雪花算法：生成分布式唯一ID
	// 结构：41位时间戳 + 10位机器ID + 12位序列号
	node, err := idgen.NewNode(1)
	if err != nil {
		log.Fatalf("❌ ID生成器初始化失败: %v", err)
	}
	idgen.SetNode(node)

	// ========== 第6步：初始化各层组件 ==========
	repos := initRepositories(db)
	services := initServices(repos, rdb, cfg)
	handlers := initHandlers(services)

	// ========== 第7步：初始化 Gin ==========
	if cfg.IsProduction() {
		gin.SetMode(gin.ReleaseMode)
	}
	r := gin.New()
	r.Use(gin.Logger(), gin.Recovery(), middleware.CORS())

	// ========== 第8步：注册路由 ==========
	registerRoutes(r, handlers, rdb)

	// ========== 第9步：启动服务器（优雅关闭） ==========
	srv := &http.Server{
		Addr:    ":" + cfg.AppPort,
		Handler: r,
	}

	// 启动服务在 goroutine 中
	go func() {
		log.Printf("🚀 服务器启动中，监听端口: %s", cfg.AppPort)
		if err := srv.ListenAndServe(); err != nil && err != http.ErrServerClosed {
			log.Fatalf("❌ 服务器启动失败: %v", err)
		}
	}()

	// ========== 第10步：等待中断信号实现优雅关闭 ==========
	// 收到信号后：
	// 1. 停止接受新连接
	// 2. 等待现有请求处理完成（5秒超时）
	// 3. 关闭数据库连接
	quit := make(chan os.Signal, 1)
	signal.Notify(quit, syscall.SIGINT, syscall.SIGTERM)
	<-quit
	log.Println("🛑 正在关闭服务器...")

	ctx, cancel := context.WithTimeout(context.Background(), 5*time.Second)
	defer cancel()
	if err := srv.Shutdown(ctx); err != nil {
		log.Printf("❌ 服务器强制关闭: %v", err)
	}

	// 关闭数据库
	if sqlDB, err := db.DB(); err == nil {
		sqlDB.Close()
	}
	log.Println("✅ 服务器已关闭")
}

// initDatabase 初始化数据库
func initDatabase(cfg *config.Config) (*gorm.DB, error) {
	dsn := fmt.Sprintf(
		"host=%s user=%s password=%s dbname=%s port=%s sslmode=disable TimeZone=Asia/Shanghai",
		cfg.DBHost, cfg.DBUser, cfg.DBPassword, cfg.DBName, cfg.DBPort,
	)

	db, err := gorm.Open(postgres.Open(dsn), &gorm.Config{
		Logger: logger.Default.LogMode(logger.Info),
	})
	if err != nil {
		return nil, err
	}

	// 自动迁移
	if err := db.AutoMigrate(
		&model.User{},
		&model.Address{},
		&model.Category{},
		&model.Product{},
		&model.CartItem{},
		&model.Order{},
		&model.OrderItem{},
		&model.Payment{},
	); err != nil {
		return nil, err
	}

	// 配置连接池（生产环境必配）
	sqlDB, err := db.DB()
	if err != nil {
		return nil, err
	}
	sqlDB.SetMaxIdleConns(10)                // 最大空闲连接数
	sqlDB.SetMaxOpenConns(100)              // 最大打开连接数
	sqlDB.SetConnMaxLifetime(time.Hour)     // 连接最大生命周期

	return db, nil
}

// initRedis 初始化 Redis 客户端
func initRedis(cfg *config.Config) *redis.Client {
	return redis.NewClient(&redis.Options{
		Addr:     cfg.RedisAddr(),
		Password: cfg.RedisPass,
		DB:       0,
	})
}

// initRepositories 初始化仓库层
func initRepositories(db *gorm.DB) *repository.Repositories {
	return &repository.Repositories{
		User:    repository.NewUserRepository(db),
		Product: repository.NewProductRepository(db),
		Cart:    repository.NewCartRepository(db),
		Order:   repository.NewOrderRepository(db),
	}
}

// initServices 初始化服务层
func initServices(repos *repository.Repositories, rdb *redis.Client, cfg *config.Config) *service.Services {
	return &service.Services{
		Auth:    service.NewAuthService(repos.User, cfg.JWTSecret),
		Product: service.NewProductService(repos.Product, rdb),
		Cart:    service.NewCartService(repos.Cart, repos.Product),
		Order:   service.NewOrderService(repos.Order, repos.Cart, repos.Product),
		Payment: service.NewPaymentService(repos.Order),
	}
}

// initHandlers 初始化处理器
func initHandlers(services *service.Services) *handler.Handlers {
	return &handler.Handlers{
		Auth:    handler.NewAuthHandler(services.Auth),
		Product: handler.NewProductHandler(services.Product),
		Cart:    handler.NewCartHandler(services.Cart),
		Order:   handler.NewOrderHandler(services.Order),
		Payment: handler.NewPaymentHandler(services.Payment),
	}
}

// registerRoutes 注册路由
func registerRoutes(r *gin.Engine, h *handler.Handlers, rdb *redis.Client) {
	// 健康检查
	r.GET("/api/v1/health", func(c *gin.Context) {
		c.JSON(200, gin.H{"status": "healthy", "service": "ecommerce-api"})
	})

	// API v1 分组
	api := r.Group("/api/v1")

	// 公开接口
	public := api.Group("")
	{
		// 认证
		public.POST("/auth/register", h.Auth.Register)
		public.POST("/auth/login", h.Auth.Login)
		
		// 商品（无需登录即可浏览）
		products := public.Group("/products")
		{
			products.GET("", h.Product.List)
			products.GET("/:id", h.Product.Get)
			products.GET("/categories", h.Product.ListCategories)
		}
	}

	// 需要认证的接口
	auth := api.Group("")
	auth.Use(middleware.JWTAuth())
	auth.Use(middleware.RateLimit(rdb, 100, 60)) // 限流：100请求/分钟
	{
		// 用户
		users := auth.Group("/users")
		{
			users.GET("/me", h.Auth.GetCurrentUser)
			users.PUT("/me", h.Auth.UpdateProfile)
			users.POST("/addresses", h.Auth.AddAddress)
			users.GET("/addresses", h.Auth.ListAddresses)
			users.DELETE("/addresses/:id", h.Auth.DeleteAddress)
		}

		// 购物车
		cart := auth.Group("/cart")
		{
			cart.GET("", h.Cart.Get)
			cart.POST("/items", h.Cart.AddItem)
			cart.PUT("/items/:id", h.Cart.UpdateItem)
			cart.DELETE("/items/:id", h.Cart.RemoveItem)
			cart.DELETE("", h.Cart.Clear)
		}

		// 订单
		orders := auth.Group("/orders")
		{
			orders.POST("", h.Order.Create)
			orders.GET("", h.Order.List)
			orders.GET("/:id", h.Order.Get)
			orders.POST("/:id/cancel", h.Order.Cancel)
		}

		// 支付
		payments := auth.Group("/payments")
		{
			payments.POST("/:orderId/pay", h.Payment.Pay)
			payments.POST("/notify", h.Payment.Notify) // 支付回调（通常不需要认证）
		}
	}
}
```

#### 2.2.2 订单服务（事务处理）service/order.go

```go
package service

import (
	"errors"
	"fmt"
	"time"

	"ecommerce-api/internal/model"
	"ecommerce-api/internal/repository"
	"ecommerce-api/pkg/idgen"

	"gorm.io/gorm" // 事务支持
)

// 订单服务 - 展示复杂业务逻辑和事务处理

type OrderService struct {
	orderRepo   *repository.OrderRepository
	cartRepo    *repository.CartRepository
	productRepo *repository.ProductRepository
}

func NewOrderService(
	orderRepo *repository.OrderRepository,
	cartRepo *repository.CartRepository,
	productRepo *repository.ProductRepository,
) *OrderService {
	return &OrderService{
		orderRepo:   orderRepo,
		cartRepo:    cartRepo,
		productRepo: productRepo,
	}
}

// CreateOrderRequest 创建订单请求
type CreateOrderRequest struct {
	AddressID  uint                `json:"addressId" binding:"required"`  // 收货地址ID
	Remark     string              `json:"remark"`                          // 订单备注
	Items      []CreateOrderItemReq `json:"items" binding:"required,min=1"` // 订单商品
	FromCart   bool                `json:"fromCart"`                        // 是否来自购物车
}

// CreateOrderItemReq 订单商品项
type CreateOrderItemReq struct {
	ProductID uint `json:"productId" binding:"required"`  // 商品ID
	Quantity  int  `json:"quantity" binding:"required,min=1"` // 数量
}

// 创建订单
// 核心难点：
// 1. 使用数据库事务保证数据一致性
// 2. 库存扣减要检查库存是否充足
// 3. 订单创建成功后清空购物车（如果来自购物车）
// 4. 任何一步失败都要回滚整个事务
func (s *OrderService) CreateOrder(userID uint, req *CreateOrderRequest) (*model.Order, error) {
	// ========== 第1步：获取商品信息并计算总价 ==========
	// 验证商品存在、库存充足
	var orderItems []model.OrderItem
	var totalAmount float64

	for _, item := range req.Items {
		// 查询商品（使用行级锁 SELECT ... FOR UPDATE）
		// FOR UPDATE 锁定行，防止并发修改
		product, err := s.productRepo.FindByIDForUpdate(item.ProductID)
		if err != nil || product == nil {
			return nil, fmt.Errorf("商品不存在: %d", item.ProductID)
		}

		// 检查商品状态
		if product.Status != 1 {
			return nil, fmt.Errorf("商品已下架: %s", product.Name)
		}

		// 检查库存
		if product.Stock < item.Quantity {
			return nil, fmt.Errorf("商品库存不足: %s（剩余: %d，需要: %d）",
				product.Name, product.Stock, item.Quantity)
		}

		// 计算商品小计
		subtotal := product.Price * float64(item.Quantity)
		totalAmount += subtotal

		// 构建订单商品项
		orderItems = append(orderItems, model.OrderItem{
			ProductID:   product.ID,
			ProductName: product.Name,
			ProductCode: product.Code,
			Price:       product.Price,
			Quantity:    item.Quantity,
			Subtotal:    subtotal,
		})
	}

	// ========== 第2步：获取收货地址 ==========
	// 这里省略地址获取逻辑

	// ========== 第3步：使用事务创建订单 ==========
	// GORM 的 Transaction 方法提供安全的事务支持
	// 如果闭包内返回 error，会自动回滚
	var order *model.Order
	err := s.orderRepo.Transaction(func(txRepo *repository.OrderRepository) error {
		// 生成订单号：时间戳 + 用户ID后4位 + 随机数
		orderNo := generateOrderNo(userID)

		// 创建订单主记录
		order = &model.Order{
			OrderNo:     orderNo,
			UserID:      userID,
			AddressID:   req.AddressID,
			Status:      model.OrderStatusPending, // 待支付
			TotalAmount: totalAmount,
			Remark:      req.Remark,
		}

		// 保存订单（会返回带ID的订单）
		if err := txRepo.Create(order); err != nil {
			return fmt.Errorf("创建订单失败: %w", err)
		}

		// 批量创建订单商品（关联订单ID）
		for i := range orderItems {
			orderItems[i].OrderID = order.ID
		}
		if err := txRepo.CreateOrderItems(orderItems); err != nil {
			return fmt.Errorf("创建订单商品失败: %w", err)
		}

		// 扣减库存（乐观锁方式）
		for _, item := range req.Items {
			// UPDATE products SET stock = stock - ? WHERE id = ? AND stock >= ?
			affected, err := s.productRepo.DecreaseStock(item.ProductID, item.Quantity)
			if err != nil {
				return fmt.Errorf("扣减库存失败: %w", err)
			}
			if affected == 0 {
				return fmt.Errorf("库存不足，订单创建失败")
			}
		}

		// 如果来自购物车，清空用户购物车
		if req.FromCart {
			if err := s.cartRepo.ClearByUserID(userID); err != nil {
				return fmt.Errorf("清空购物车失败: %w", err)
			}
		}

		return nil // 返回 nil 表示提交事务
	})

	if err != nil {
		return nil, err
	}

	// 重新加载订单商品
	order.Items, _ = s.orderRepo.GetOrderItems(order.ID)
	return order, nil
}

// OrderStatus 订单状态
type OrderStatus int

const (
	OrderStatusPending   OrderStatus = 1  // 待支付
	OrderStatusPaid      OrderStatus = 2  // 已支付
	OrderStatusShipped   OrderStatus = 3  // 已发货
	OrderStatusDelivered OrderStatus = 4  // 已收货
	OrderStatusCancelled OrderStatus = 5  // 已取消
	OrderStatusRefunded  OrderStatus = 6  // 已退款
)

// generateOrderNo 生成订单号
// 格式：202401011234560001
// - 20240101: 日期
// - 123456: 用户ID（补零）
// - 0001: 序号
func generateOrderNo(userID uint) string {
	now := time.Now()
	// 使用雪花算法生成唯一ID
	snowID := idgen.GetNode().Generate().Int64()
	return fmt.Sprintf("%s%06d%d",
		now.Format("20060102150405"),
		userID,
		snowID%10000,
	)
}

// GetUserOrders 获取用户订单列表
func (s *OrderService) GetUserOrders(userID uint, page, pageSize int) ([]model.Order, int64, error) {
	return s.orderRepo.ListByUserID(userID, page, pageSize)
}

// GetOrder 获取订单详情
func (s *OrderService) GetOrder(userID, orderID uint) (*model.Order, error) {
	order, err := s.orderRepo.FindByID(orderID)
	if err != nil {
		return nil, err
	}
	// 验证订单所属用户
	if order.UserID != userID {
		return nil, errors.New("无权访问此订单")
	}
	// 加载订单商品
	order.Items, _ = s.orderRepo.GetOrderItems(order.ID)
	return order, nil
}

// CancelOrder 取消订单
func (s *OrderService) CancelOrder(userID, orderID uint) error {
	return s.orderRepo.Transaction(func(txRepo *repository.OrderRepository) error {
		// 查询订单（加锁）
		order, err := txRepo.FindByIDForUpdate(orderID)
		if err != nil {
			return err
		}
		if order == nil {
			return errors.New("订单不存在")
		}
		if order.UserID != userID {
			return errors.New("无权操作此订单")
		}

		// 检查订单状态，只有待支付才能取消
		if order.Status != model.OrderStatusPending {
			return fmt.Errorf("当前订单状态不允许取消: %d", order.Status)
		}

		// 恢复库存
		items, _ := txRepo.GetOrderItems(orderID)
		for _, item := range items {
			if _, err := s.productRepo.IncreaseStock(item.ProductID, item.Quantity); err != nil {
				return err
			}
		}

		// 更新订单状态
		return txRepo.UpdateStatus(orderID, model.OrderStatusCancelled)
	})
}

// UpdateOrderStatus 更新订单状态（内部/管理员使用）
func (s *OrderService) UpdateOrderStatus(orderID uint, status OrderStatus) error {
	return s.orderRepo.UpdateStatus(orderID, int(status))
}
```

#### 2.2.3 雪花算法 ID 生成器 pkg/idgen/snowflake.go

```go
package idgen

import (
	"errors"
	"sync"
	"time"
)

/*
雪花算法（Snowflake）介绍：
- 分布式唯一ID生成算法，由Twitter开源
- 优点：趋势递增、不依赖数据库、高性能
- 结构：64位 = 1位符号 + 41位时间戳 + 10位机器ID + 12位序列号

位分配（从高到低）：
- 1 bit:  符号位，固定为0，表示正数
- 41 bit: 时间戳（毫秒），可用69年
- 10 bit: 机器ID，支持1024个节点
- 12 bit: 序列号，每节点每毫秒支持4096个ID

时间戳起点：2020-01-01 00:00:00（可自定义）
*/

const (
	// 时间戳起始点（2020-01-01 00:00:00）
	epoch int64 = 1577836800000

	// 各部分位数
	timestampBits  = 41 // 时间戳位数
	workerIDBits   = 10 // 机器ID位数
	sequenceBits   = 12 // 序列号位数

	// 各部分最大值
	maxWorkerID    = -1 ^ (-1 << workerIDBits)  // 1023
	maxSequence    = -1 ^ (-1 << sequenceBits) // 4095
)

// Node 雪花算法节点
// 每个服务实例使用唯一的 workerID
type Node struct {
	mu         sync.Mutex           // 互斥锁，保证并发安全
	workerID   int64                // 机器ID（0-1023）
	sequence   int64               // 当前序列号
	lastTime   int64               // 上次生成ID的时间戳
}

// 移位常量
const (
	workerIDShift   = sequenceBits                     // 机器ID左移位数
	timestampShift  = sequenceBits + workerIDBits     // 时间戳左移位数
)

// NewNode 创建新的雪花算法节点
// 参数 workerID: 机器ID，必须在 0-1023 之间
func NewNode(workerID int64) (*Node, error) {
	if workerID < 0 || workerID > maxWorkerID {
		return nil, errors.New("workerID 必须在 0-1023 之间")
	}
	return &Node{
		workerID: workerID,
		sequence: 0,
		lastTime: 0,
	}, nil
}

// 全局节点实例
var node *Node
var nodeOnce sync.Once

// SetNode 设置全局节点
func SetNode(n *Node) {
	node = n
}

// GetNode 获取全局节点
func GetNode() *Node {
	return node
}

// Generate 生成一个新的唯一ID
// 返回 int64 类型的唯一ID
func (n *Node) Generate() int64 {
	// 加锁保证并发安全
	n.mu.Lock()
	defer n.mu.Unlock()

	// 获取当前时间戳（毫秒）
	now := currentTimeMillis()

	// ========== 时间回拨处理 ==========
	// 如果当前时间小于上次生成ID的时间，说明系统时间被回拨了
	// 这时使用上次时间 + 1 毫秒，继续生成
	if now < n.lastTime {
		now = n.lastTime
	}

	// ========== 同一毫秒内序列号递增 ==========
	if now == n.lastTime {
		// 序列号递增，超过最大值则等待下一毫秒
		n.sequence = (n.sequence + 1) & maxSequence
		// 如果序列号归零，说明同一毫秒内生成超过4096个ID，需要等待
		if n.sequence == 0 {
			// 等待到下一毫秒
			for now <= n.lastTime {
				now = currentTimeMillis()
			}
		}
	} else {
		// 时间戳变化，序列号归零
		n.sequence = 0
	}

	// 更新最后时间戳
	n.lastTime = now

	// ========== 组装 ID ==========
	// ID 结构：timestamp | workerID | sequence
	// 使用位运算将各部分组合
	id := (now-epoch)<<timestampShift | // 时间戳部分
		n.workerID<<workerIDShift |      // 机器ID部分
		n.workerID<<sequenceBits |        // 序列号部分
		n.sequence

	return id
}

// currentTimeMillis 获取当前时间戳（毫秒）
func currentTimeMillis() int64 {
	return time.Now().UnixNano() / 1e6
}

// Int64 转换为 int64
func (id *ID) Int64() int64 {
	return int64(*id)
}

// String 转换为字符串
func (id *ID) String() string {
	return string(rune(id.Int64()))
}

// ID 类型别名，便于使用
type ID int64
