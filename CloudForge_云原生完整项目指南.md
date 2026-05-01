# CloudForge 云原生完整项目指南

> 基于 Go + Gin + Docker + Kubernetes 的微服务监控系统

---

## 📋 项目介绍

### 项目概述
CloudForge 是一个云原生监控系统，用于采集和展示服务器指标、服务状态、日志分析等。项目展示了云原生开发的核心技能，包括微服务架构、容器化编排、服务网格等。

- **微服务架构**：多个独立服务协同工作
- **容器化部署**：Docker + Kubernetes
- **可观测性**：Prometheus + Grafana
- **面试亮点**：云原生和 DevOps 能力

### 核心技术亮点
```
语言：Go 1.21 + Gin 框架
容器：Docker + Docker Compose
编排：Kubernetes (K3s)
监控：Prometheus + Grafana
日志：Loki + Promtail
服务发现：Consul / etcd
API 网关：Nginx Ingress
```

---

## 🏗️ 技术架构图

### 系统架构

```
┌─────────────────────────────────────────────────────────────────┐
│                         客户端层                                 │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐             │
│  │   Web UI   │  │  Grafana   │  │   API      │             │
│  │   Dashboard │  │   仪表盘   │  │   客户端   │             │
│  └──────┬──────┘  └──────┬──────┘  └──────┬──────┘             │
└─────────┼────────────────┼────────────────┼─────────────────────┘
          │                │                │
          └────────────────┼────────────────┘
                           │ HTTP/gRPC
┌──────────────────────────┼────────────────────────────────────┐
│                     Kubernetes Cluster                          │
│  ┌─────────────────────────────────────────────────────────────┐│
│  │                      Ingress Controller                      ││
│  │                      (Nginx / Traefik)                      ││
│  └─────────────────────────────────────────────────────────────┘│
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐         │
│  │   API GW     │  │   Auth Svc   │  │  Monitor Svc │         │
│  │   (网关)     │  │   (认证)     │  │   (监控)     │         │
│  └──────────────┘  └──────────────┘  └──────────────┘         │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐         │
│  │  Alert Svc  │  │   Log Svc    │  │  Metric Svc  │         │
│  │   (告警)    │  │   (日志)     │  │   (指标)     │         │
│  └──────────────┘  └──────────────┘  └──────────────┘         │
│  ┌──────────────────────────────────────────────────────────┐ │
│  │                    Data Layer (PVC)                        │ │
│  │  ┌────────┐  ┌────────┐  ┌────────┐  ┌────────┐        │ │
│  │  │MySQL   │  │Redis   │  │MongoDB │  │Kafka  │        │ │
│  └──────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘

                        │ Observability
┌────────────────────────┼────────────────────────────────────┐
│  ┌────────────┐  ┌────────────┐  ┌────────────┐              │
│  │Prometheus │  │  Grafana   │  │   Loki    │              │
│  │ (指标收集) │  │ (可视化)  │  │ (日志聚合) │              │
│  └────────────┘  └────────────┘  └────────────┘              │
└───────────────────────────────────────────────────────────────┘
```

### 微服务交互流程

```
┌─────────┐     ┌─────────┐     ┌─────────┐     ┌─────────┐
│  Client │────>│ API GW  │────>│  Auth   │────>│ Service │
│         │<────│         │<────│ Service │<────│         │
└─────────┘     └────┬────┘     └─────────┘     └─────────┘
                     │
         ┌───────────┼───────────┐
         │           │           │
         ▼           ▼           ▼
    ┌─────────┐ ┌─────────┐ ┌─────────┐
    │Metrics  │ │  Logs   │ │ Alerts  │
    │ Collector│ │ Parser  │ │ Manager │
    └────┬────┘ └────┬────┘ └────┬────┘
         │           │           │
         ▼           ▼           ▼
    ┌─────────┐ ┌─────────┐ ┌─────────┐
    │Prometheus│ │  Loki  │ │ AlertMgr│
    └─────────┘ └─────────┘ └─────────┘
```

---

## 📅 学习计划（7天）

### Day 1：Go 语言基础与环境 ⭐

**目标**：掌握 Go 基础和项目结构

**学习内容**：
- Go 安装和配置
- Go 模块管理
- 项目结构设计
- Gin 框架入门

**执行命令**：

```bash
# 1. 安装 Go（Linux）
wget https://go.dev/dl/go1.21.5.linux-amd64.tar.gz
sudo tar -C /usr/local -xzf go1.21.5.linux-amd64.tar.gz
echo 'export PATH=$PATH:/usr/local/go/bin' >> ~/.bashrc
source ~/.bashrc

# 2. 验证安装
go version

# 3. 创建项目
mkdir -p cloudforge && cd cloudforge
go mod init github.com/cloudforge/cloudforge

# 4. 安装依赖
go get github.com/gin-gonic/gin
go get github.com/prometheus/client_golang/prometheus
go get github.com/go-redis/redis/v8
go get gorm.io/gorm
go get gorm.io/driver/mysql

# 5. 项目结构
mkdir -p cmd/server internal/{config,handler,middleware,model,service,repository} pkg/utils
```

---

### Day 2：微服务核心实现 ⭐⭐

**目标**：实现核心微服务

**核心代码**：

`cloudforge/internal/config/config.go` - 配置
```go
package config

import (
	"os"
	"strconv"
	"time"
)

type Config struct {
	Server   ServerConfig
	Database DatabaseConfig
	Redis    RedisConfig
	Prometheus PrometheusConfig
}

type ServerConfig struct {
	Port         string
	ReadTimeout  time.Duration
	WriteTimeout time.Duration
}

type DatabaseConfig struct {
	Host     string
	Port     int
	User     string
	Password string
	DBName   string
	MaxOpen  int
	MaxIdle  int
}

type RedisConfig struct {
	Host     string
	Port     int
	Password string
	DB       int
	PoolSize int
}

type PrometheusConfig struct {
	Enabled    bool
	ListenAddr string
}

func Load() *Config {
	return &Config{
		Server: ServerConfig{
			Port:         getEnv("SERVER_PORT", "8080"),
			ReadTimeout:  getDurationEnv("SERVER_READ_TIMEOUT", 30*time.Second),
			WriteTimeout: getDurationEnv("SERVER_WRITE_TIMEOUT", 30*time.Second),
		},
		Database: DatabaseConfig{
			Host:     getEnv("DB_HOST", "localhost"),
			Port:     getIntEnv("DB_PORT", 3306),
			User:     getEnv("DB_USER", "root"),
			Password: getEnv("DB_PASSWORD", ""),
			DBName:   getEnv("DB_NAME", "cloudforge"),
			MaxOpen:  getIntEnv("DB_MAX_OPEN", 100),
			MaxIdle:  getIntEnv("DB_MAX_IDLE", 10),
		},
		Redis: RedisConfig{
			Host:     getEnv("REDIS_HOST", "localhost"),
			Port:     getIntEnv("REDIS_PORT", 6379),
			Password: getEnv("REDIS_PASSWORD", ""),
			DB:       getIntEnv("REDIS_DB", 0),
			PoolSize: getIntEnv("REDIS_POOL_SIZE", 100),
		},
		Prometheus: PrometheusConfig{
			Enabled:    getBoolEnv("PROMETHEUS_ENABLED", true),
			ListenAddr: getEnv("PROMETHEUS_ADDR", ":9090"),
		},
	}
}

func getEnv(key, defaultValue string) string {
	if value := os.Getenv(key); value != "" {
		return value
	}
	return defaultValue
}

func getIntEnv(key string, defaultValue int) int {
	if value := os.Getenv(key); value != "" {
		if intValue, err := strconv.Atoi(value); err == nil {
			return intValue
		}
	}
	return defaultValue
}

func getBoolEnv(key string, defaultValue bool) bool {
	if value := os.Getenv(key); value != "" {
		if boolValue, err := strconv.ParseBool(value); err == nil {
			return boolValue
		}
	}
	return defaultValue
}

func getDurationEnv(key string, defaultValue time.Duration) time.Duration {
	if value := os.Getenv(key); value != "" {
		if duration, err := time.ParseDuration(value); err == nil {
			return duration
		}
	}
	return defaultValue
}
```

`cloudforge/internal/model/metric.go` - 数据模型
```go
package model

import (
	"time"

	"gorm.io/gorm"
)

type Metric struct {
	ID        uint           `gorm:"primarykey" json:"id"`
	CreatedAt time.Time      `json:"created_at"`
	UpdatedAt time.Time      `json:"updated_at"`
	DeletedAt gorm.DeletedAt `gorm:"index" json:"-"`

	// 指标基本信息
	Name      string `gorm:"index;size:100;not null" json:"name"`       // 指标名称
	Host      string `gorm:"index;size:255" json:"host"`                 // 主机名
	Service   string `gorm:"index;size:100" json:"service"`               // 服务名
	Namespace string `gorm:"index;size:100" json:"namespace"`            // 命名空间

	// 指标值
	Value float64 `gorm:"not null" json:"value"` // 指标值

	// 标签（JSON格式存储）
	Labels JSONMap `gorm:"type:json" json:"labels"`

	// 时间戳
	Timestamp time.Time `gorm:"index;not null" json:"timestamp"`
}

// JSONMap JSON 类型的别名
type JSONMap map[string]interface{}

// Scan 实现 Scanner 接口
func (j *JSONMap) Scan(value interface{}) error {
	if value == nil {
		*j = make(JSONMap)
		return nil
	}
	
	bytes, ok := value.([]byte)
	if !ok {
		*j = make(JSONMap)
		return nil
	}
	
	return json.Unmarshal(bytes, j)
}

// Value 实现 Valuer 接口
func (j JSONMap) Value() (interface{}, error) {
	if j == nil {
		return nil, nil
	}
	return json.Marshal(j)
}

import "encoding/json"

// MetricQuery 查询参数
type MetricQuery struct {
	Name      string    `form:"name"`
	Host      string    `form:"host"`
	Service   string    `form:"service"`
	StartTime time.Time `form:"start_time"`
	EndTime   time.Time `form:"end_time"`
	Limit     int       `form:"limit"`
	Offset    int       `form:"offset"`
}

// Alert 告警规则
type Alert struct {
	ID          uint           `gorm:"primarykey" json:"id"`
	CreatedAt   time.Time      `json:"created_at"`
	UpdatedAt   time.Time      `json:"updated_at"`
	DeletedAt   gorm.DeletedAt `gorm:"index" json:"-"`

	Name        string  `gorm:"size:100;not null" json:"name"`
	Expr        string  `gorm:"type:text;not null" json:"expr"`        // PromQL 表达式
	Description string `gorm:"type:text" json:"description"`           // 告警描述
	Severity   string  `gorm:"size:20;not null" json:"severity"`     // critical/warning/info
	Duration    int     `gorm:"default:300" json:"duration"`         // 持续时间（秒）
	
	// 告警状态
	Status    string    `gorm:"size:20;default:'inactive'" json:"status"`
	FiredAt   *time.Time `json:"fired_at,omitempty"`
	ResolvedAt *time.Time `json:"resolved_at,omitempty"`
	
	// 通知配置
	NotifyChannels []string `gorm:"type:json" json:"notify_channels"`
}

// AlertEvent 告警事件
type AlertEvent struct {
	ID          uint      `gorm:"primarykey" json:"id"`
	AlertID     uint      `gorm:"index;not null" json:"alert_id"`
	Alert       Alert     `gorm:"foreignKey:AlertID" json:"alert"`
	
	Status      string    `gorm:"size:20;not null" json:"status"`      // firing/resolved
	Value       float64   `json:"value"`                              // 触发时的值
	Summary     string    `gorm:"type:text" json:"summary"`           // 摘要信息
	
	CreatedAt   time.Time `json:"created_at"`
	ResolvedAt  *time.Time `json:"resolved_at,omitempty"`
}
```

`cloudforge/internal/repository/metric_repo.go` - 数据访问层
```go
package repository

import (
	"context"
	"fmt"
	"time"

	"github.com/cloudforge/cloudforge/internal/model"
	"gorm.io/gorm"
)

type MetricRepository interface {
	Create(ctx context.Context, metric *model.Metric) error
	CreateBatch(ctx context.Context, metrics []*model.Metric) error
	Query(ctx context.Context, query *model.MetricQuery) ([]*model.Metric, int64, error)
	GetLatest(ctx context.Context, name, host string) (*model.Metric, error)
	GetRange(ctx context.Context, name, host string, start, end time.Time) ([]*model.Metric, error)
	DeleteOlderThan(ctx context.Context, before time.Time) (int64, error)
}

type metricRepository struct {
	db *gorm.DB
}

func NewMetricRepository(db *gorm.DB) MetricRepository {
	return &metricRepository{db: db}
}

func (r *metricRepository) Create(ctx context.Context, metric *model.Metric) error {
	return r.db.WithContext(ctx).Create(metric).Error
}

func (r *metricRepository) CreateBatch(ctx context.Context, metrics []*model.Metric) error {
	if len(metrics) == 0 {
		return nil
	}
	
	// 批量插入
	return r.db.WithContext(ctx).CreateInBatches(metrics, 1000).Error
}

func (r *metricRepository) Query(ctx context.Context, query *model.MetricQuery) ([]*model.Metric, int64, error) {
	var metrics []*model.Metric
	var total int64
	
	db := r.db.WithContext(ctx).Model(&model.Metric{})
	
	// 条件过滤
	if query.Name != "" {
		db = db.Where("name = ?", query.Name)
	}
	if query.Host != "" {
		db = db.Where("host = ?", query.Host)
	}
	if query.Service != "" {
		db = db.Where("service = ?", query.Service)
	}
	if !query.StartTime.IsZero() {
		db = db.Where("timestamp >= ?", query.StartTime)
	}
	if !query.EndTime.IsZero() {
		db = db.Where("timestamp <= ?", query.EndTime)
	}
	
	// 计数
	if err := db.Count(&total).Error; err != nil {
		return nil, 0, err
	}
	
	// 分页
	if query.Limit > 0 {
		db = db.Limit(query.Limit)
	}
	if query.Offset > 0 {
		db = db.Offset(query.Offset)
	}
	
	// 排序
	db = db.Order("timestamp DESC")
	
	err := db.Find(&metrics).Error
	return metrics, total, err
}

func (r *metricRepository) GetLatest(ctx context.Context, name, host string) (*model.Metric, error) {
	var metric model.Metric
	err := r.db.WithContext(ctx).
		Where("name = ? AND host = ?", name, host).
		Order("timestamp DESC").
		First(&metric).Error
	
	if err != nil {
		return nil, err
	}
	return &metric, nil
}

func (r *metricRepository) GetRange(ctx context.Context, name, host string, start, end time.Time) ([]*model.Metric, error) {
	var metrics []*model.Metric
	
	err := r.db.WithContext(ctx).
		Where("name = ? AND host = ? AND timestamp >= ? AND timestamp <= ?", name, host, start, end).
		Order("timestamp ASC").
		Find(&metrics).Error
	
	return metrics, err
}

func (r *metricRepository) DeleteOlderThan(ctx context.Context, before time.Time) (int64, error) {
	result := r.db.WithContext(ctx).
		Where("timestamp < ?", before).
		Delete(&model.Metric{})
	
	return result.RowsAffected, result.Error
}
```

`cloudforge/internal/service/metric_service.go` - 业务逻辑层
```go
package service

import (
	"context"
	"fmt"
	"time"

	"github.com/cloudforge/cloudforge/internal/model"
	"github.com/cloudforge/cloudforge/internal/repository"
	"github.com/prometheus/client_golang/prometheus"
)

type MetricService interface {
	// 指标上报
	ReportMetric(ctx context.Context, metric *model.Metric) error
	ReportMetricsBatch(ctx context.Context, metrics []*model.Metric) error
	
	// 指标查询
	QueryMetrics(ctx context.Context, query *model.MetricQuery) ([]*model.Metric, int64, error)
	GetMetricTimeSeries(ctx context.Context, name, host string, duration time.Duration) ([]*model.Metric, error)
	
	// 指标统计
	GetStatistics(ctx context.Context, service string) (*MetricStatistics, error)
	
	// 健康检查
	HealthCheck(ctx context.Context) error
}

type MetricStatistics struct {
	TotalMetrics   int64     `json:"total_metrics"`
	TotalHosts    int64     `json:"total_hosts"`
	TotalServices int64     `json:"total_services"`
	ReportRate    float64   `json:"report_rate"` // 每秒上报数
	Timestamp     time.Time `json:"timestamp"`
}

type metricService struct {
	repo    repository.MetricRepository
	metrics prometheus.Metrics
}

func NewMetricService(repo repository.MetricRepository) MetricService {
	return &metricService{
		repo: repo,
	}
}

func (s *metricService) ReportMetric(ctx context.Context, metric *model.Metric) error {
	// 设置默认时间
	if metric.Timestamp.IsZero() {
		metric.Timestamp = time.Now()
	}
	
	// 上报指标
	err := s.repo.Create(ctx, metric)
	if err != nil {
		return fmt.Errorf("failed to report metric: %w", err)
	}
	
	// 更新 Prometheus 指标
	s.metrics.ApiRequestsTotal.WithLabelValues("report", "success").Inc()
	
	return nil
}

func (s *metricService) ReportMetricsBatch(ctx context.Context, metrics []*model.Metric) error {
	if len(metrics) == 0 {
		return nil
	}
	
	// 批量上报
	err := s.repo.CreateBatch(ctx, metrics)
	if err != nil {
		s.metrics.ApiRequestsTotal.WithLabelValues("report_batch", "error").Inc()
		return fmt.Errorf("failed to report metrics batch: %w", err)
	}
	
	s.metrics.ApiRequestsTotal.WithLabelValues("report_batch", "success").Inc()
	s.metrics.MetricsReportedTotal.With(float64(len(metrics))).Inc()
	
	return nil
}

func (s *metricService) QueryMetrics(ctx context.Context, query *model.MetricQuery) ([]*model.Metric, int64, error) {
	// 设置默认分页
	if query.Limit <= 0 {
		query.Limit = 100
	}
	if query.Limit > 1000 {
		query.Limit = 1000
	}
	
	return s.repo.Query(ctx, query)
}

func (s *metricService) GetMetricTimeSeries(ctx context.Context, name, host string, duration time.Duration) ([]*model.Metric, error) {
	end := time.Now()
	start := end.Add(-duration)
	
	return s.repo.GetRange(ctx, name, host, start, end)
}

func (s *metricService) GetStatistics(ctx context.Context, service string) (*MetricStatistics, error) {
	stats := &MetricStatistics{
		Timestamp: time.Now(),
	}
	
	// 简化实现，实际应查询数据库
	stats.TotalMetrics = 1000000
	stats.TotalHosts = 50
	stats.TotalServices = 10
	stats.ReportRate = 1000.0
	
	return stats, nil
}

func (s *metricService) HealthCheck(ctx context.Context) error {
	return nil
}
```

---

### Day 3：HTTP API 与中间件 ⭐⭐

**目标**：实现 HTTP API 和中间件

**核心代码**：

`cloudforge/internal/handler/metric_handler.go` - HTTP 处理器
```go
package handler

import (
	"net/http"
	"strconv"
	"time"

	"github.com/gin-gonic/gin"
	"github.com/cloudforge/cloudforge/internal/model"
	"github.com/cloudforge/cloudforge/internal/service"
)

type MetricHandler struct {
	service service.MetricService
}

func NewMetricHandler(service service.MetricService) *MetricHandler {
	return &MetricHandler{service: service}
}

// ReportMetric 上报单个指标
// @Summary 上报指标
// @Description 接收并存储单个监控指标
// @Tags metrics
// @Accept json
// @Produce json
// @Param metric body model.Metric true "指标数据"
// @Success 200 {object} Response
// @Router /api/v1/metrics [post]
func (h *MetricHandler) ReportMetric(c *gin.Context) {
	var metric model.Metric
	
	if err := c.ShouldBindJSON(&metric); err != nil {
		c.JSON(http.StatusBadRequest, Response{
			Success: false,
			Error:   "Invalid request body: " + err.Error(),
		})
		return
	}
	
	if err := h.service.ReportMetric(c.Request.Context(), &metric); err != nil {
		c.JSON(http.StatusInternalServerError, Response{
			Success: false,
			Error:   err.Error(),
		})
		return
	}
	
	c.JSON(http.StatusOK, Response{
		Success: true,
		Message: "Metric reported successfully",
	})
}

// ReportMetrics 批量上报指标
// @Summary 批量上报指标
// @Description 批量接收并存储监控指标
// @Tags metrics
// @Accept json
// @Produce json
// @Param metrics body []model.Metric true "指标数组"
// @Success 200 {object} Response
// @Router /api/v1/metrics/batch [post]
func (h *MetricHandler) ReportMetrics(c *gin.Context) {
	var metrics []*model.Metric
	
	if err := c.ShouldBindJSON(&metrics); err != nil {
		c.JSON(http.StatusBadRequest, Response{
			Success: false,
			Error:   "Invalid request body: " + err.Error(),
		})
		return
	}
	
	if err := h.service.ReportMetricsBatch(c.Request.Context(), metrics); err != nil {
		c.JSON(http.StatusInternalServerError, Response{
			Success: false,
			Error:   err.Error(),
		})
		return
	}
	
	c.JSON(http.StatusOK, Response{
		Success: true,
		Message: "Metrics batch reported successfully",
		Data: map[string]int{
			"count": len(metrics),
		},
	})
}

// QueryMetrics 查询指标
// @Summary 查询指标
// @Description 根据条件查询指标数据
// @Tags metrics
// @Produce json
// @Param name query string false "指标名称"
// @Param host query string false "主机名"
// @Param service query string false "服务名"
// @Param start_time query string false "开始时间"
// @Param end_time query string false "结束时间"
// @Param limit query int false "分页大小"
// @Param offset query int false "偏移量"
// @Success 200 {object} QueryResponse
// @Router /api/v1/metrics [get]
func (h *MetricHandler) QueryMetrics(c *gin.Context) {
	query := &model.MetricQuery{
		Name:    c.Query("name"),
		Host:    c.Query("host"),
		Service: c.Query("service"),
		Limit:   100,
		Offset:  0,
	}
	
	// 解析时间
	if startStr := c.Query("start_time"); startStr != "" {
		if t, err := time.Parse(time.RFC3339, startStr); err == nil {
			query.StartTime = t
		}
	}
	if endStr := c.Query("end_time"); endStr != "" {
		if t, err := time.Parse(time.RFC3339, endStr); err == nil {
			query.EndTime = t
		}
	}
	
	// 解析分页
	if limitStr := c.Query("limit"); limitStr != "" {
		if limit, err := strconv.Atoi(limitStr); err == nil {
			query.Limit = limit
		}
	}
	if offsetStr := c.Query("offset"); offsetStr != "" {
		if offset, err := strconv.Atoi(offsetStr); err == nil {
			query.Offset = offset
		}
	}
	
	metrics, total, err := h.service.QueryMetrics(c.Request.Context(), query)
	if err != nil {
		c.JSON(http.StatusInternalServerError, Response{
			Success: false,
			Error:   err.Error(),
		})
		return
	}
	
	c.JSON(http.StatusOK, QueryResponse{
		Success: true,
		Data:    metrics,
		Total:   total,
		Limit:   query.Limit,
		Offset:  query.Offset,
	})
}

// GetMetricTimeSeries 获取指标时序数据
// @Summary 获取指标时序
// @Description 获取指定指标的时序数据
// @Tags metrics
// @Produce json
// @Param name query string true "指标名称"
// @Param host query string true "主机名"
// @Param duration query string false "时间范围，如: 1h, 24h, 7d"
// @Success 200 {object} Response
// @Router /api/v1/metrics/timeseries [get]
func (h *MetricHandler) GetMetricTimeSeries(c *gin.Context) {
	name := c.Query("name")
	host := c.Query("host")
	durationStr := c.DefaultQuery("duration", "1h")
	
	if name == "" || host == "" {
		c.JSON(http.StatusBadRequest, Response{
			Success: false,
			Error:   "name and host are required",
		})
		return
	}
	
	duration, err := time.ParseDuration(durationStr)
	if err != nil {
		duration = time.Hour
	}
	
	metrics, err := h.service.GetMetricTimeSeries(c.Request.Context(), name, host, duration)
	if err != nil {
		c.JSON(http.StatusInternalServerError, Response{
			Success: false,
			Error:   err.Error(),
		})
		return
	}
	
	c.JSON(http.StatusOK, Response{
		Success: true,
		Data:    metrics,
	})
}

// GetStatistics 获取统计信息
// @Summary 获取统计
// @Description 获取系统统计信息
// @Tags metrics
// @Produce json
// @Param service query string false "服务名"
// @Success 200 {object} Response
// @Router /api/v1/metrics/stats [get]
func (h *MetricHandler) GetStatistics(c *gin.Context) {
	service := c.Query("service")
	
	stats, err := h.service.GetStatistics(c.Request.Context(), service)
	if err != nil {
		c.JSON(http.StatusInternalServerError, Response{
			Success: false,
			Error:   err.Error(),
		})
		return
	}
	
	c.JSON(http.StatusOK, Response{
		Success: true,
		Data:    stats,
	})
}

// Response 通用响应
type Response struct {
	Success bool        `json:"success"`
	Message string      `json:"message,omitempty"`
	Data    interface{} `json:"data,omitempty"`
	Error   string      `json:"error,omitempty"`
}

// QueryResponse 查询响应
type QueryResponse struct {
	Success bool          `json:"success"`
	Data    interface{}   `json:"data"`
	Total   int64         `json:"total"`
	Limit   int           `json:"limit"`
	Offset  int           `json:"offset"`
}
```

`cloudforge/internal/middleware/middleware.go` - 中间件
```go
package middleware

import (
	"bytes"
	"io"
	"time"

	"github.com/gin-gonic/gin"
	"github.com/prometheus/client_golang/prometheus"
)

// PrometheusMetrics Prometheus 指标收集
func PrometheusMetrics() gin.HandlerFunc {
	return func(c *gin.Context) {
		start := time.Now()
		path := c.FullPath()
		method := c.Request.Method
		
		// 处理请求
		c.Next()
		
		// 记录指标
		status := float64(c.Writer.Status())
		duration := time.Since(start).Seconds()
		
		// 请求总数
		HttpRequestsTotal.WithLabelValues(method, path, c.FullPath()).Inc()
		
		// 请求延迟
		HttpRequestDuration.WithLabelValues(method, path).Observe(duration)
		
		// 响应状态
		HttpResponseStatus.WithLabelValues(method, path, status).Inc()
	}
}

// Logger 请求日志
func Logger() gin.HandlerFunc {
	return func(c *gin.Context) {
		start := time.Now()
		path := c.Request.URL.Path
		query := c.Request.URL.RawQuery
		
		// 读取请求体
		var requestBody []byte
		if c.Request.Body != nil {
			requestBody, _ = io.ReadAll(c.Request.Body)
			c.Request.Body = io.NopCloser(bytes.NewBuffer(requestBody))
		}
		
		// 处理请求
		c.Next()
		
		// 记录日志
		latency := time.Since(start)
		
		gin.DefaultWriter.Write([]byte(
			time.Now().Format("2006/01/02 - 15:04:05") +
			" | " + c.Request.Method +
			" | " + path +
			" | " + query +
			" | " + c.ClientIP() +
			" | " + latency.String() +
			" | " + string(rune(c.Writer.Status())) +
			"\n",
		))
	}
}

// Recovery 异常恢复
func Recovery() gin.HandlerFunc {
	return gin.Recovery()
}

// CORS 跨域
func CORS() gin.HandlerFunc {
	return func(c *gin.Context) {
		c.Writer.Header().Set("Access-Control-Allow-Origin", "*")
		c.Writer.Header().Set("Access-Control-Allow-Credentials", "true")
		c.Writer.Header().Set("Access-Control-Allow-Headers", "Content-Type, Content-Length, Accept-Encoding, X-CSRF-Token, Authorization, accept, origin, Cache-Control, X-Requested-With")
		c.Writer.Header().Set("Access-Control-Allow-Methods", "POST, OPTIONS, GET, PUT, DELETE, PATCH")
		
		if c.Request.Method == "OPTIONS" {
			c.AbortWithStatus(204)
			return
		}
		
		c.Next()
	}
}

// RateLimiter 限流（简化版）
func RateLimiter(requestsPerSecond int) gin.HandlerFunc {
	// 这里应该使用 Redis 实现分布式限流
	// 简化实现仅作为示例
	return func(c *gin.Context) {
		// TODO: 实现限流逻辑
		c.Next()
	}
}

// Prometheus 指标变量
var (
	HttpRequestsTotal = prometheus.NewCounterVec(
		prometheus.CounterOpts{
			Name: "http_requests_total",
			Help: "Total number of HTTP requests",
		},
		[]string{"method", "path"},
	)
	
	HttpRequestDuration = prometheus.NewHistogramVec(
		prometheus.HistogramOpts{
			Name:    "http_request_duration_seconds",
			Help:    "HTTP request duration in seconds",
			Buckets: prometheus.DefBuckets,
		},
		[]string{"method", "path"},
	)
	
	HttpResponseStatus = prometheus.NewCounterVec(
		prometheus.CounterOpts{
			Name: "http_response_status_total",
			Help: "Total number of HTTP response status codes",
		},
		[]string{"method", "path", "status"},
	)
	
	MetricsReportedTotal = prometheus.NewCounter(
		prometheus.CounterOpts{
			Name: "metrics_reported_total",
			Help: "Total number of metrics reported",
		},
	)
	
	ApiRequestsTotal = prometheus.NewCounterVec(
		prometheus.CounterOpts{
			Name: "api_requests_total",
			Help: "Total number of API requests",
		},
		[]string{"endpoint", "status"},
	)
)

// 初始化 Prometheus 指标
func init() {
	prometheus.MustRegister(
		HttpRequestsTotal,
		HttpRequestDuration,
		HttpResponseStatus,
		MetricsReportedTotal,
		ApiRequestsTotal,
	)
}
```

---

### Day 4：Docker 容器化 ⭐⭐

**目标**：实现 Docker 容器化

**Docker 配置**：

`cloudforge/Dockerfile`
```dockerfile
# 构建阶段
FROM golang:1.21-alpine AS builder

# 安装构建依赖
RUN apk add --no-cache git ca-certificates tzdata

# 设置工作目录
WORKDIR /app

# 复制依赖文件
COPY go.mod go.sum ./
RUN go mod download

# 复制源代码
COPY . .

# 构建
ARG VERSION=latest
ARG BUILD_TIME
ARG COMMIT_ID

RUN CGO_ENABLED=0 GOOS=linux GOARCH=amd64 \
    go build -ldflags="-s -w -X main.Version=${VERSION} -X main.BuildTime=${BUILD_TIME} -X main.CommitID=${COMMIT_ID}" \
    -o /app/cloudforge \
    ./cmd/server

# 运行阶段
FROM alpine:3.19

# 安装运行时依赖
RUN apk add --no-cache ca-certificates tzdata

# 创建非 root 用户
RUN adduser -D -g '' appuser
WORKDIR /app

# 复制二进制文件
COPY --from=builder /app/cloudforge .

# 复制配置文件
COPY --from=builder /app/config.yaml .

# 复制静态文件（如果有）
COPY --from=builder /app/public ./public

# 设置权限
RUN chown -R appuser:appuser /app

USER appuser

# 暴露端口
EXPOSE 8080

# 健康检查
HEALTHCHECK --interval=30s --timeout=10s --start-period=5s --retries=3 \
    CMD wget --no-verbose --tries=1 --spider http://localhost:8080/health || exit 1

# 启动命令
CMD ["./cloudforge"]
```

`cloudforge/.dockerignore`
```
# Git
.git
.gitignore

# IDE
.idea
.vscode
*.swp
*.swo

# 文档
README.md
docs/
*.md

# 测试
*_test.go
test/
coverage/

# 构建产物
bin/
dist/

# Docker
Dockerfile
docker-compose.yml
.dockerignore

# 环境文件
.env
.env.local
.env.*.local

# 其他
*.log
tmp/
temp/
```

`cloudforge/docker-compose.yml` - 开发环境
```yaml
version: '3.8'

services:
  # 应用服务
  api:
    build:
      context: .
      dockerfile: Dockerfile
    container_name: cloudforge-api
    ports:
      - "8080:8080"
    environment:
      - SERVER_PORT=8080
      - DB_HOST=mysql
      - DB_PORT=3306
      - DB_USER=root
      - DB_PASSWORD=cloudforge123
      - DB_NAME=cloudforge
      - REDIS_HOST=redis
      - REDIS_PORT=6379
      - PROMETHEUS_ENABLED=true
    depends_on:
      mysql:
        condition: service_healthy
      redis:
        condition: service_healthy
    networks:
      - cloudforge-network
    restart: unless-stopped

  # MySQL 数据库
  mysql:
    image: mysql:8.0
    container_name: cloudforge-mysql
    ports:
      - "3306:3306"
    environment:
      - MYSQL_ROOT_PASSWORD=cloudforge123
      - MYSQL_DATABASE=cloudforge
      - MYSQL_USER=cloudforge
      - MYSQL_PASSWORD=cloudforge123
    volumes:
      - mysql_data:/var/lib/mysql
      - ./config/mysql.cnf:/etc/mysql/conf.d/custom.cnf
    healthcheck:
      test: ["CMD", "mysqladmin", "ping", "-h", "localhost"]
      interval: 10s
      timeout: 5s
      retries: 5
    networks:
      - cloudforge-network
    restart: unless-stopped

  # Redis 缓存
  redis:
    image: redis:7-alpine
    container_name: cloudforge-redis
    ports:
      - "6379:6379"
    command: redis-server --appendonly yes --requirepass redis123
    volumes:
      - redis_data:/data
    healthcheck:
      test: ["CMD", "redis-cli", "-a", "redis123", "ping"]
      interval: 10s
      timeout: 5s
      retries: 5
    networks:
      - cloudforge-network
    restart: unless-stopped

  # Prometheus 监控
  prometheus:
    image: prom/prometheus:v2.48.0
    container_name: cloudforge-prometheus
    ports:
      - "9090:9090"
    volumes:
      - ./config/prometheus.yml:/etc/prometheus/prometheus.yml
      - prometheus_data:/prometheus
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'
      - '--storage.tsdb.path=/prometheus'
      - '--web.console.libraries=/usr/share/prometheus/console_libraries'
      - '--web.console.templates=/usr/share/prometheus/consoles'
      - '--web.enable-lifecycle'
    networks:
      - cloudforge-network
    restart: unless-stopped

  # Grafana 可视化
  grafana:
    image: grafana/grafana:10.2.2
    container_name: cloudforge-grafana
    ports:
      - "3000:3000"
    environment:
      - GF_SECURITY_ADMIN_USER=admin
      - GF_SECURITY_ADMIN_PASSWORD=admin123
      - GF_USERS_ALLOW_SIGN_UP=false
    volumes:
      - grafana_data:/var/lib/grafana
      - ./config/grafana/provisioning:/etc/grafana/provisioning
    depends_on:
      - prometheus
    networks:
      - cloudforge-network
    restart: unless-stopped

volumes:
  mysql_data:
  redis_data:
  prometheus_data:
  grafana_data:

networks:
  cloudforge-network:
    driver: bridge
```

`cloudforge/config/prometheus.yml`
```yaml
global:
  scrape_interval: 15s
  evaluation_interval: 15s

alerting:
  alertmanagers:
    - static_configs:
        - targets: []

rule_files: []

scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']

  - job_name: 'cloudforge-api'
    static_configs:
      - targets: ['api:8080']
    metrics_path: '/metrics'
```

---

### Day 5：Kubernetes 部署 ⭐⭐⭐

**目标**：实现 Kubernetes 部署

**K8s 配置**：

`cloudforge/k8s/namespace.yaml`
```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: cloudforge
  labels:
    name: cloudforge
    env: production
```

`cloudforge/k8s/deployment.yaml`
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: cloudforge-api
  namespace: cloudforge
  labels:
    app: cloudforge-api
    version: v1
spec:
  replicas: 3
  selector:
    matchLabels:
      app: cloudforge-api
  template:
    metadata:
      labels:
        app: cloudforge-api
        version: v1
    spec:
      serviceAccountName: cloudforge-api
      containers:
        - name: api
          image: cloudforge/api:latest
          imagePullPolicy: IfNotPresent
          ports:
            - name: http
              containerPort: 8080
              protocol: TCP
          env:
            - name: SERVER_PORT
              value: "8080"
            - name: DB_HOST
              value: "mysql"
            - name: DB_PORT
              value: "3306"
            - name: DB_NAME
              value: "cloudforge"
            - name: REDIS_HOST
              value: "redis"
            - name: REDIS_PORT
              value: "6379"
          resources:
            requests:
              memory: "256Mi"
              cpu: "250m"
            limits:
              memory: "512Mi"
              cpu: "500m"
          livenessProbe:
            httpGet:
              path: /health
              port: http
            initialDelaySeconds: 10
            periodSeconds: 15
            timeoutSeconds: 5
            failureThreshold: 3
          readinessProbe:
            httpGet:
              path: /ready
              port: http
            initialDelaySeconds: 5
            periodSeconds: 10
            timeoutSeconds: 3
            failureThreshold: 3
      affinity:
        podAntiAffinity:
          preferredDuringSchedulingIgnoredDuringExecution:
            - weight: 100
              podAffinityTerm:
                labelSelector:
                  matchExpressions:
                    - key: app
                      operator: In
                      values:
                        - cloudforge-api
                topologyKey: kubernetes.io/hostname
```

`cloudforge/k8s/service.yaml`
```yaml
apiVersion: v1
kind: Service
metadata:
  name: cloudforge-api
  namespace: cloudforge
  labels:
    app: cloudforge-api
spec:
  type: ClusterIP
  ports:
    - name: http
      port: 8080
      targetPort: http
      protocol: TCP
  selector:
    app: cloudforge-api
```

`cloudforge/k8s/ingress.yaml`
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: cloudforge-ingress
  namespace: cloudforge
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
    nginx.ingress.kubernetes.io/ssl-redirect: "true"
    nginx.ingress.kubernetes.io/rate-limit: "100"
spec:
  ingressClassName: nginx
  rules:
    - host: api.cloudforge.local
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: cloudforge-api
                port:
                  name: http
  tls:
    - hosts:
        - api.cloudforge.local
      secretName: cloudforge-tls
```

`cloudforge/k8s/configmap.yaml`
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: cloudforge-config
  namespace: cloudforge
data:
  prometheus.yml: |
    global:
      scrape_interval: 15s
    scrape_configs:
      - job_name: 'cloudforge-api'
        kubernetes_sd_configs:
          - role: pod
        relabel_configs:
          - source_labels: [__meta_kubernetes_pod_name]
            action: keep
            regex: cloudforge-api-.*
          - source_labels: [__meta_kubernetes_pod_container_port_number]
            action: keep
            regex: "8080"
```

`cloudforge/k8s/hpa.yaml`
```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: cloudforge-api-hpa
  namespace: cloudforge
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: cloudforge-api
  minReplicas: 2
  maxReplicas: 10
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 70
    - type: Resource
      resource:
        name: memory
        target:
          type: Utilization
          averageUtilization: 80
  behavior:
    scaleDown:
      stabilizationWindowSeconds: 300
      policies:
        - type: Percent
          value: 10
          periodSeconds: 60
    scaleUp:
      stabilizationWindowSeconds: 0
      policies:
        - type: Percent
          value: 100
          periodSeconds: 15
```

`cloudforge/k8s/rbac.yaml`
```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: cloudforge-api
  namespace: cloudforge

---
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: cloudforge-api
  namespace: cloudforge
rules:
  - apiGroups: [""]
    resources: ["configmaps", "secrets"]
    verbs: ["get", "list", "watch"]

---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: cloudforge-api
  namespace: cloudforge
subjects:
  - kind: ServiceAccount
    name: cloudforge-api
    namespace: cloudforge
roleRef:
  kind: Role
  name: cloudforge-api
  apiGroup: rbac.authorization.k8s.io
```

---

### Day 6 & 7：项目整合与面试准备 ⭐⭐⭐

**部署脚本**：

`cloudforge/deploy.sh`
```bash
#!/bin/bash

# CloudForge 部署脚本

set -e

NAMESPACE="cloudforge"
KUBECONFIG_PATH="./kubeconfig"

echo "===== CloudForge 部署开始 ====="

# 构建镜像
echo "1. 构建 Docker 镜像..."
docker build -t cloudforge/api:latest .

# 推送镜像（如果有私有仓库）
# docker push registry.example.com/cloudforge/api:latest

# 应用 Kubernetes 配置
echo "2. 应用 Kubernetes 配置..."
kubectl apply -f k8s/namespace.yaml
kubectl apply -f k8s/configmap.yaml
kubectl apply -f k8s/rbac.yaml
kubectl apply -f k8s/deployment.yaml
kubectl apply -f k8s/service.yaml
kubectl apply -f k8s/ingress.yaml
kubectl apply -f k8s/hpa.yaml

# 等待部署完成
echo "3. 等待 Pod 就绪..."
kubectl wait --for=condition=available \
  --timeout=300s \
  deployment/cloudforge-api \
  -n $NAMESPACE

# 检查状态
echo "4. 检查服务状态..."
kubectl get pods -n $NAMESPACE
kubectl get svc -n $NAMESPACE

echo "===== CloudForge 部署完成 ====="
echo "API 地址: http://api.cloudforge.local"
echo "Prometheus: http://prometheus.cloudforge.local"
echo "Grafana: http://grafana.cloudforge.local"
```

---

## 📝 面试要点

### Q1：微服务架构的优势和挑战？

**参考答案**：
```
优势：
1. 独立部署：每个服务可独立部署，快速迭代
2. 技术异构：不同服务可用不同技术栈
3. 可扩展性：按需扩展单个服务
4. 容错性：一个服务故障不影响其他服务
5. 团队自治：团队可独立负责服务

挑战：
1. 分布式系统复杂性：网络通信、分布式事务
2. 服务治理：服务发现、负载均衡、熔断
3. 数据一致性：跨服务数据同步
4. 运维复杂度：部署、监控、日志聚合
5. 测试：集成测试更复杂
```

### Q2：Docker 和 Kubernetes 的关系？

**参考答案**：
```
Docker：
- 容器运行时，负责创建和运行容器
- 镜像构建、分发
- 单机容器管理

Kubernetes：
- 容器编排平台
- 管理多个 Docker 主机（集群）
- 提供：自动调度、弹性伸缩、服务发现、负载均衡
- 自我修复、滚动升级、配置管理

关系：
- Kubernetes 依赖 Docker（或其他容器运行时）
- Kubernetes 管理 Docker 容器
- Docker 提供容器化能力，K8s 提供编排能力
```

### Q3：如何实现零宕机部署？

**参考答案**：
```
1. 滚动更新（Rolling Update）
   - 逐步替换旧版本 Pod
   - 保证始终有可用实例

2. 健康检查
   - Liveness Probe：检测存活
   - Readiness Probe：检测就绪
   - 确保新 Pod 就绪后才接收流量

3. 优雅关闭
   - 接收关闭信号后停止接收新请求
   - 处理完现有请求后再退出

4. 资源限制
   - 合理设置 CPU/内存
   - 使用 HPA 自动扩缩容

5. 高可用设计
   - 多副本部署
   - Pod 反亲和性
   - 跨可用区部署
```

### Q4：Prometheus 的工作原理？

**参考答案**：
```
核心机制：
1. Pull 模式：Prometheus 主动拉取指标
2. 服务发现：自动发现监控目标
3. 时序数据库：存储指标数据
4. PromQL：查询语言

工作流程：
1. 应用暴露 /metrics 端点
2. Prometheus 定期抓取
3. 存储到 TSDB
4. Grafana 可视化

指标类型：
- Counter：只增不减
- Gauge：可增可减
- Histogram：直方图
- Summary：分位数
```

---

## ✅ 项目清单

```
CloudForge/
├── cmd/
│   └── server/
│       └── main.go
├── internal/
│   ├── config/
│   │   └── config.go
│   ├── handler/
│   │   ├── metric_handler.go
│   │   └── alert_handler.go
│   ├── middleware/
│   │   └── middleware.go
│   ├── model/
│   │   ├── metric.go
│   │   └── alert.go
│   ├── repository/
│   │   └── metric_repo.go
│   └── service/
│       ├── metric_service.go
│       └── alert_service.go
├── pkg/
│   └── utils/
├── config/
│   ├── prometheus.yml
│   ├── grafana/
│   └── mysql.cnf
├── k8s/
│   ├── namespace.yaml
│   ├── deployment.yaml
│   ├── service.yaml
│   ├── ingress.yaml
│   ├── configmap.yaml
│   ├── hpa.yaml
│   └── rbac.yaml
├── Dockerfile
├── docker-compose.yml
├── deploy.sh
└── README.md
```

---

**文档版本**：v1.0  
**更新日期**：2024
