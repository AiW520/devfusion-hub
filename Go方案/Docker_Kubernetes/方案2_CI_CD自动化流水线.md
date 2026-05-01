# Docker+Kubernetes 方案二：CI/CD 自动化流水线

## 📚 项目介绍

### 1.1 项目概述

这是一个完整的 CI/CD 自动化流水线系统，使用 GitLab CI/Jenkins/GitHub Actions 实现代码提交到生产部署的全自动化。项目展示了 DevOps 全流程、GitOps 理念、自动化测试、安全扫描等核心能力。

**核心功能：**
- 自动化构建和测试
- 镜像构建和推送
- 多环境部署（dev/staging/production）
- 金丝雀发布和回滚
- 自动化安全扫描
- 通知和报告

### 1.2 核心技术亮点

```
DevOps 工具链：
├── GitHub Actions              # CI/CD 引擎
│   ├── 多平台构建              # Linux/Windows/macOS
│   ├── 矩阵构建               # Go 1.21/1.22
│   ├── 缓存优化                # 依赖缓存
│   └── 并行作业               # 加快构建
├── Docker                      # 容器化
│   ├── BuildKit               # 高效构建
│   ├── 镜像分层缓存           # 加速构建
│   └── 多架构构建             # amd64/arm64
├── Kubernetes                  # 部署
│   ├── ArgoCD                 # GitOps
│   ├── Helm                   # 包管理
│   ├── Kustomize              # 配置差异化
│   └── Istio                  # 服务网格
├── 监控和告警
│   ├── Prometheus             # 指标收集
│   ├── Grafana                # 可视化
│   └── PagerDuty              # 告警通知
└── 安全
    ├── Trivy                  # 漏洞扫描
    ├── SonarQube              # 代码质量
    └── Snyk                   # 依赖安全
```

### 1.3 适合场景

- 展示 DevOps 全栈能力
- 展示 GitOps 实践
- 面试 DevOps/SRE 岗位
- 自动化运维加分项

---

## 🔧 完整可运行代码

### 2.1 GitHub Actions 配置

#### 2.1.1 主工作流 .github/workflows/ci.yml

```yaml
# =============================================================================
# GitHub Actions CI/CD 流水线
# 
# 流水线阶段：
# 1. Lint - 代码检查
# 2. Test - 单元测试
# 3. Build - Docker 镜像构建
# 4. Security - 安全扫描
# 5. Push - 推送镜像
# 6. Deploy - 部署
# =============================================================================

name: CI/CD Pipeline

# =============================================================================
# 触发条件
# =============================================================================
on:
  # 代码推送时触发
  push:
    branches:
      - main                    # 主分支
      - 'release/**'           # 发布分支
      - 'develop'               # 开发分支
    tags:
      - 'v*'                    # 版本标签
    paths-ignore:
      - '*.md'                  # 忽略文档
      - 'docs/**'
  
  # Pull Request 时触发
  pull_request:
    branches:
      - main
      - develop
    paths-ignore:
      - '*.md'
      - 'docs/**'

  # 手动触发
  workflow_dispatch:
    inputs:
      environment:
        description: '部署环境'
        required: true
        default: 'staging'
        type: choice
        options:
          - dev
          - staging
          - production

  # 定时触发（每天凌晨执行）
  schedule:
    - cron: '0 2 * * *'         # UTC 2:00 = 北京时间 10:00

# =============================================================================
# 环境变量
# =============================================================================
env:
  # 镜像仓库
  REGISTRY: ghcr.io
  IMAGE_NAME: ${{ github.repository }}
  
  # Go 版本
  GO_VERSION: '1.21'
  
  # Node 版本（前端构建）
  NODE_VERSION: '20'

# =============================================================================
# 作业定义
# =============================================================================
jobs:
  # =============================================================================
  # 阶段1：Lint - 代码检查
  # =============================================================================
  lint:
    name: Lint
    runs-on: ubuntu-latest
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
        with:
          fetch-depth: 0         # 完整克隆，用于版本计算

      - name: Set up Go
        uses: actions/setup-go@v5
        with:
          go-version: ${{ env.GO_VERSION }}
          cache: true           # 启用依赖缓存

      - name: Run Go Lint
        run: |
          go install golang.org/x/tools/cmd/goimports@latest
          go install github.com/golangci/golangci-lint/cmd/golangci-lint@latest
          
          # 检查格式化
          goimports -l -w .
          git diff --exit-code || exit 1
          
          # 运行 linter
          golangci-lint run --timeout 5m

      - name: Check Commit Messages
        if: github.event_name == 'push'
        run: |
          commit_msg=$(git log --format=%B -n 1 ${{ github.sha }})
          echo "Commit message: $commit_msg"
          
          # 简单检查：必须以 feat/fix/docs/chore 等开头
          if ! echo "$commit_msg" | grep -qE '^(feat|fix|docs|style|refactor|test|chore|release):'; then
            echo "❌ Commit message must start with: feat, fix, docs, style, refactor, test, chore, release"
            exit 1
          fi

  # =============================================================================
  # 阶段2：Test - 单元测试和集成测试
  # =============================================================================
  test:
    name: Test (Go ${{ matrix.go-version }})
    runs-on: ubuntu-latest
    
    # 矩阵构建：测试多个 Go 版本
    strategy:
      matrix:
        go-version: ['1.21', '1.22']
      fail-fast: false          # 一个失败不影响其他

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Set up Go
        uses: actions/setup-go@v5
        with:
          go-version: ${{ matrix.go-version }}
          cache: true

      # =============================================================================
      # 依赖缓存
      # =============================================================================
      - name: Cache Go modules
        uses: actions/cache@v4
        with:
          path: |
            ~/.cache/go-build
            ~/go/pkg/mod
          key: ${{ runner.os }}-go-${{ matrix.go-version }}-${{ hashFiles('**/go.sum') }}
          restore-keys: |
            ${{ runner.os }}-go-${{ matrix.go-version }}-

      - name: Download dependencies
        run: go mod download

      # =============================================================================
      # 单元测试
      # =============================================================================
      - name: Unit Tests
        run: |
          go test -v -race -coverprofile=coverage.out \
            -covermode=atomic \
            -timeout 10m \
            ./...

      # =============================================================================
      # 测试覆盖率
      # =============================================================================
      - name: Coverage Report
        uses:臣击派来的cireport/go-coverage-check-action@v1
        with:
          type: lcov
          path: coverage.out
          min-coverage: 70       # 覆盖率必须 > 70%

      # =============================================================================
      # 上传覆盖率报告
      # =============================================================================
      - name: Upload coverage to Codecov
        uses: codecov/codecov-action@v4
        with:
          file: ./coverage.out
          flags: unittests
          name: codecov-umbrella
          fail_ci_if_error: false

  # =============================================================================
  # 阶段3：Build - Docker 镜像构建
  # =============================================================================
  build:
    name: Build
    runs-on: ubuntu-latest
    needs: [lint, test]
    
    outputs:
      image-tag: ${{ steps.meta.outputs.tags }}
      digest: ${{ steps.build.outputs.digest }}

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      # =============================================================================
      # QEMU - 多架构支持
      # =============================================================================
      - name: Set up QEMU
        uses: docker/setup-qemu-action@v3

      # =============================================================================
      # Docker Buildx - 增强构建
      # =============================================================================
      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3

      # =============================================================================
      # 登录镜像仓库
      # =============================================================================
      - name: Login to GHCR
        uses: docker/login-action@v3
        with:
          registry: ${{ env.REGISTRY }}
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}

      # =============================================================================
      # 生成元数据
      # =============================================================================
      - name: Extract metadata
        id: meta
        uses: docker/metadata-action@v5
        with:
          images: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}
          tags: |
            type=ref,event=branch
            type=ref,event=pr
            type=semver,pattern={{version}}
            type=semver,pattern={{major}}.{{minor}}
            type=raw,value=latest,enable={{is_default_branch}}
            type=sha,prefix=,suffix=,format=short

      # =============================================================================
      # 构建镜像
      # =============================================================================
      - name: Build and push
        id: build
        uses: docker/build-push-action@v6
        with:
          context: .
          platforms: linux/amd64,linux/arm64
          push: true
          tags: ${{ steps.meta.outputs.tags }}
          labels: ${{ steps.meta.outputs.labels }}
          cache-from: type=gha               # GitHub Actions 缓存
          cache-to: type=gha,mode=max        # 缓存所有层
          build-args: |
            VERSION=${{ github.ref_name }}
            BUILD_TIME=${{ github.event.created_at }}
            GO_VERSION=${{ env.GO_VERSION }}

  # =============================================================================
  # 阶段4：Security - 安全扫描
  # =============================================================================
  security:
    name: Security Scan
    runs-on: ubuntu-latest
    needs: [build]
    
    permissions:
      security-events: write
      contents: read

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      # =============================================================================
      # Trivy 漏洞扫描
      # =============================================================================
      - name: Run Trivy vulnerability scanner
        uses: aquasecurity/trivy-action@master
        with:
          image-ref: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:latest
          format: 'sarif'
          output: 'trivy-results.sarif'
          severity: 'CRITICAL,HIGH'
          vuln-type: 'os,library'

      # =============================================================================
      # 上报 GitHub Security
      # =============================================================================
      - name: Upload Trivy results to GitHub Security tab
        uses: github/codeql-action/upload-sarif@v2
        with:
          sarif_file: 'trivy-results.sarif'

      # =============================================================================
      # SonarQube 代码质量扫描
      # =============================================================================
      - name: SonarQube Scan
        uses: sonarsource/sonarqube-scan-action@master
        env:
          SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
          SONAR_HOST_URL: ${{ secrets.SONAR_HOST_URL }}
        with:
          args: >
            -Dsonar.projectKey=user-service
            -Dsonar.go.coverage.reportPaths=coverage.out
            -Dsonar.sources=.
            -Dsonar.exclusions=**/*_test.go,**/vendor/**,**/.git/**

      # =============================================================================
      # 检查质量门禁
      # =============================================================================
      - name: Check Quality Gate
        uses: sonarsource/sonarqube-quality-gate-action@master
        with:
          sonarToken: ${{ secrets.SONAR_TOKEN }}

  # =============================================================================
  # 阶段5：Deploy - 部署
  # =============================================================================
  deploy:
    name: Deploy to ${{ matrix.environment }}
    runs-on: ubuntu-latest
    needs: [build, security]
    
    if: github.event_name != 'pull_request'
    
    strategy:
      matrix:
        include:
          - environment: dev
            kubeconfig: ${{ secrets.KUBECONFIG_DEV }}
          - environment: staging
            kubeconfig: ${{ secrets.KUBECONFIG_STAGING }}
          - environment: production
            kubeconfig: ${{ secrets.KUBECONFIG_PRODUCTION }}
      fail-fast: false

    environment:
      name: ${{ matrix.environment }}
      url: https://${{ matrix.environment }}.example.com

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      # =============================================================================
      # 配置 kubectl
      # =============================================================================
      - name: Configure kubectl
        run: |
          echo "${{ matrix.kubeconfig }}" | base64 -d > kubeconfig
          echo "KUBECONFIG=$(pwd)/kubeconfig" >> $GITHUB_ENV

      - name: Set up Helm
        uses: azure/setup-helm@v4
        with:
          version: '3.14.0'

      # =============================================================================
      # 部署到 Kubernetes
      # =============================================================================
      - name: Deploy to ${{ matrix.environment }}
        run: |
          # 添加 Helm repo
          helm repo add bitnami https://charts.bitnami.com/bitnami
          helm repo update
          
          # 部署/升级应用
          helm upgrade --install user-service ./charts/user-service \
            --namespace ${{ matrix.environment }} \
            --create-namespace \
            --set image.repository=${{ env.REGISTRY }}/${{ env.IMAGE_NAME }} \
            --set image.tag=${{ github.sha }} \
            --set replicaCount=3 \
            --wait --timeout 5m \
            --atomic \
            --cleanup-on-fail

      # =============================================================================
      # 等待就绪
      # =============================================================================
      - name: Wait for deployment
        run: |
          kubectl rollout status deployment/user-service \
            -n ${{ matrix.environment }} \
            --timeout=300s

      # =============================================================================
      # 健康检查
      # =============================================================================
      - name: Health check
        run: |
          kubectl exec -n ${{ matrix.environment }} \
            $(kubectl get pod -n ${{ matrix.environment }} -l app=user-service -o jsonpath='{.items[0].metadata.name}') \
            -- curl -f http://localhost:8080/healthz || exit 1

  # =============================================================================
  # 阶段6：通知
  # =============================================================================
  notify:
    name: Notify
    runs-on: ubuntu-latest
    needs: [deploy]
    if: always()
    
    steps:
      - name: Notify Success
        if: needs.deploy.result == 'success'
        run: |
          echo "🎉 Deployment successful!"

      - name: Notify Failure
        if: needs.deploy.result == 'failure'
        run: |
          echo "❌ Deployment failed!"
          
      # =============================================================================
      # Slack 通知
      # =============================================================================
      - name: Slack Notification
        if: always()
        uses: slackapi/slack-github-action@v1
        with:
          payload: |
            {
              "text": "${{ needs.deploy.result == 'success' && '✅' || '❌' }} Deployment ${{ needs.deploy.result }}",
              "blocks": [
                {
                  "type": "section",
                  "text": {
                    "type": "mrkdwn",
                    "text": "*Deployment Result*\n*Environment:* ${{ matrix.environment || 'unknown' }}\n*Commit:* `${{ github.sha }}`\n*Branch:* `${{ github.ref_name }}`\n*Author:* ${{ github.actor }}"
                  }
                }
              ]
            }
        env:
          SLACK_WEBHOOK_URL: ${{ secrets.SLACK_WEBHOOK_URL }}
          SLACK_WEBHOOK_TYPE: INCOMING_WEBHOOK
```

#### 2.1.2 Helm Chart 结构

```
charts/user-service/
├── Chart.yaml               # Chart 元数据
├── values.yaml              # 默认配置
├── values-dev.yaml          # 开发环境配置
├── values-staging.yaml      # 预发环境配置
├── values-prod.yaml         # 生产环境配置
└── templates/
    ├── NOTES.txt            # 安装说明
    ├── _helpers.tpl         # 辅助模板
    ├── deployment.yaml
    ├── service.yaml
    ├── ingress.yaml
    ├── hpa.yaml
    ├── pdb.yaml
    ├── configmap.yaml
    ├── secret.yaml
    └── serviceaccount.yaml
```

#### 2.1.3 Chart.yaml

```yaml
# =============================================================================
# Helm Chart 定义
# =============================================================================

apiVersion: v2                                        # Chart API 版本
name: user-service                                    # Chart 名称
description: A Helm chart for User Service microservice
type: application

# Chart 版本
version: 1.0.0                                       # Chart 版本
appVersion: "1.0.0"                                  # 应用版本

# 维护者
maintainers:
  - name: DevOps Team
    email: devops@example.com

# 关键字
keywords:
  - user-service
  - microservice
  - golang

# 依赖
dependencies:
  - name: postgresql
    version: "12.x.x"
    repository: "https://charts.bitnami.com/bitnami"
    condition: postgresql.enabled
  - name: redis
    version: "17.x.x"
    repository: "https://charts.bitnami.com/bitnami"
    condition: redis.enabled
```

#### 2.1.4 values.yaml

```yaml
# =============================================================================
# 默认配置
# 这些值会被环境特定配置覆盖
# =============================================================================

# 镜像配置
image:
  repository: ghcr.io/your-org/user-service
  pullPolicy: IfNotPresent
  tag: "latest"

# 副本数
replicaCount: 3

# 服务配置
service:
  type: ClusterIP
  http:
    port: 80
    targetPort: 8080
  grpc:
    port: 90
    targetPort: 9090

# Ingress 配置
ingress:
  enabled: true
  className: "nginx"
  annotations:
    cert-manager.io/cluster-issuer: letsencrypt-prod
  hosts:
    - host: user-service.local
      paths:
        - path: /
          pathType: Prefix
  tls:
    - secretName: user-service-tls
      hosts:
        - user-service.local

# 资源配置
resources:
  limits:
    cpu: 500m
    memory: 512Mi
  requests:
    cpu: 100m
    memory: 128Mi

# 健康检查
livenessProbe:
  httpGet:
    path: /healthz
    port: http
  initialDelaySeconds: 10
  periodSeconds: 10

readinessProbe:
  httpGet:
    path: /readyz
    port: http
  initialDelaySeconds: 5
  periodSeconds: 5

# 自动扩缩容
autoscaling:
  enabled: true
  minReplicas: 2
  maxReplicas: 10
  targetCPUUtilizationPercentage: 70
  targetMemoryUtilizationPercentage: 80

# Pod 中断预算
podDisruptionBudget:
  enabled: true
  minAvailable: 2

# 策略配置
config:
  app:
    port: 8080
    mode: production
    log_level: info
  db:
    host: ""
    port: 5432
    name: users
    max_open_conns: 25
    max_idle_conns: 5
  redis:
    host: ""
    port: 6379
    db: 0

# Secret 配置
secret:
  db_password: ""
  api_key: ""

# 依赖服务
postgresql:
  enabled: true
  auth:
    database: users
    username: postgres
    password: ""

redis:
  enabled: true
  auth:
    password: ""

# 备注
notes: |
  Thank you for installing user-service!
  
  To access the service:
  1. Get the service endpoint: kubectl get svc -n user-service
  2. Check pod status: kubectl get pods -n user-service
```

### 2.2 ArgoCD 配置（GitOps）

#### 2.2.1 Application 定义

```yaml
# =============================================================================
# ArgoCD Application
# 定义需要部署的应用
# =============================================================================

apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: user-service
  namespace: argocd
  labels:
    app: user-service
  annotations:
    # 描述
    description: "User Service Production Deployment"
spec:
  # =============================================================================
  # 源配置
  # =============================================================================
  source:
    repoURL: https://github.com/your-org/k8s-config.git
    targetRevision: main
    path: production/user-service
    
    # Helm 配置
    chart: user-service
    helm:
      valueFiles:
        - values-prod.yaml
      parameters:
        - name: image.tag
          value: $ARGOCD_APP_REVISION

  # =============================================================================
  # 目标配置
  # =============================================================================
  destination:
    server: https://kubernetes.default.svc
    namespace: production

  # =============================================================================
  # 同步策略
  # =============================================================================
  syncPolicy:
    # =============================================================================
    # 自动同步
    # =============================================================================
    automated:
      prune: true                # 自动删除清理资源
      selfHeal: true             # 自动修复差异
      allowEmpty: false          # 不允许空目录

    # =============================================================================
    # 同步选项
    # =============================================================================
    syncOptions:
      - CreateNamespace=true     # 自动创建命名空间
      - PruneLast=true           # 最后执行清理
      - ServerSideApply=true     # 使用服务端 apply

    # =============================================================================
    # 重试策略
    # =============================================================================
    retry:
      limit: 5
      backoff:
        duration: 5s
        factor: 2
        maxDuration: 3m

  # =============================================================================
  # 资源健康检查
  # =============================================================================
  ignoreDifferences:
    - group: apps
      kind: Deployment
      jsonPointers:
        - /spec/replicas

  # =============================================================================
  # 预览环境
  # =============================================================================
  project: default
```

#### 2.2.2 ApplicationSet（批量部署）

```yaml
# =============================================================================
# ApplicationSet
# 用于批量创建 Application
# 支持多集群、多环境、多租户
# =============================================================================

apiVersion: argoproj.io/v1alpha1
kind: ApplicationSet
metadata:
  name: user-service-all
  namespace: argocd
spec:
  # =============================================================================
  # 生成器
  # =============================================================================
  generators:
    # =============================================================================
    # 矩阵生成器
    # =============================================================================
    - matrix:
        generators:
          # Git 目录生成器：从仓库目录结构读取
          - git:
              repoURL: https://github.com/your-org/k8s-config.git
              revision: main
              directories:
                - path: "*/user-service"
                  exclude: true
                - path: "staging/user-service"
                - path: "production/user-service"

          # 静态参数
          - clusters:
              values:
                environmentSuffix: ""
                namespaceSuffix: ""
                replicas: "3"

    # =============================================================================
    # List 生成器
    # =============================================================================
    - list:
        elements:
          - cluster: dev
            url: https://dev.k8s.example.com
            namespace: dev
          - cluster: staging
            url: https://staging.k8s.example.com
            namespace: staging
          - cluster: prod
            url: https://prod.k8s.example.com
            namespace: production

  # =============================================================================
  # 应用模板
  # =============================================================================
  template:
    metadata:
      name: 'user-service-{{path.basename}}'
      labels:
        app: user-service
    spec:
      project: default
      source:
        repoURL: https://github.com/your-org/k8s-config.git
        targetRevision: main
        path: '{{path}}/user-service'
      destination:
        server: '{{url}}'
        namespace: '{{namespace}}'
      syncPolicy:
        automated:
          prune: true
          selfHeal: true
```

### 2.3 环境差异化配置（Kustomize）

#### 2.3.1 kustomization.yaml

```yaml
# =============================================================================
# Kustomize 基础配置
# =============================================================================

apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

# 元数据
namespace: default
namePrefix: user-service-
commonLabels:
  app: user-service

# 资源列表
resources:
  - deployment.yaml
  - service.yaml
  - ingress.yaml
  - configmap.yaml

# 公共配置
configMapGenerator:
  - name: app-config
    literals:
      - LOG_LEVEL=info
      - ENVIRONMENT=development

# 补丁
patchesStrategicMerge:
  - patch-resources.yaml
```

#### 2.3.2 overlay/production/kustomization.yaml

```yaml
# =============================================================================
# 生产环境配置
# =============================================================================

apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

# 继承基础配置
bases:
  - ../../base

# 命名空间
namespace: production

# 环境特定标签
commonLabels:
  environment: production

# 配置覆盖
configMapGenerator:
  - name: app-config
    behavior: replace
    literals:
      - LOG_LEVEL=warn
      - ENVIRONMENT=production

# 副本数
replicas:
  - name: deployment
    count: 5

# 资源配置
patches:
  # 资源限制增强
  - patch: |-
      - op: replace
        path: /spec/template/spec/containers/0/resources/limits/cpu
        value: "1000m"
    target:
      kind: Deployment

  # 副本数
  - patch: |-
      - op: replace
        path: /spec/replicas
        value: 5
    target:
      kind: Deployment

# 图片覆盖
images:
  - name: my-app
    newName: your-registry.com/user-service
    digest: sha256:abc123...
```

---

## 📝 代码关键点说明

### 3.1 GitHub Actions 工作流设计

```
作业依赖关系：
┌──────────┐
│  Lint   │
└────┬─────┘
     │
     ▼
┌──────────┐     ┌──────────┐
│  Test   │────▶│  Build   │
└────┬─────┘     └────┬─────┘
     │                │
     │                ▼
     │           ┌──────────┐
     │           │ Security │
     │           └────┬─────┘
     │                │
     ▼                ▼
┌──────────┐     ┌──────────┐
│  Deploy  │◀────│  Deploy  │
│  (Dev)   │     │(Staging) │
└──────────┘     └────┬─────┘
                     │
                     ▼
                ┌──────────┐
                │  Deploy  │
                │ (Prod)   │
                └──────────┘
```

### 3.2 GitOps 部署流程

```
GitOps 流程：
1. 开发者提交代码
2. CI 流水线构建镜像并推送
3. CI 更新 GitOps 仓库的镜像版本
4. ArgoCD 检测到变更
5. ArgoCD 同步最新配置到 K8s
6. 监控部署状态
7. 通知结果
```

### 3.3 安全扫描集成

```
扫描层次：
1. 代码扫描（SonarQube）
   - 代码质量
   - 安全漏洞
   - 代码覆盖率

2. 依赖扫描（Snyk/Dependabot）
   - 依赖版本漏洞
   - 许可证合规

3. 镜像扫描（Trivy）
   - OS 漏洞
   - 应用漏洞
   - 敏感信息
```

---

## 📅 学习计划（5天）

### Day 1：CI/CD 基础
**目标：** 理解 CI/CD 流程
**任务：**
- [ ] 了解 CI/CD 概念
- [ ] 编写 GitHub Actions 工作流
- [ ] 实现自动化测试
- [ ] 配置 Docker 构建

**产出：** 基本的 CI/CD 流水线

### Day 2：Docker 高级特性
**目标：** 掌握 Docker 进阶
**任务：**
- [ ] 多阶段构建
- [ ] BuildKit 缓存
- [ ] 多架构构建
- [ ] 安全扫描

**产出：** 优化的 Docker 镜像

### Day 3：Kubernetes 部署
**目标：** 自动化部署到 K8s
**任务：**
- [ ] 编写 K8s 清单
- [ ] 配置 Helm Chart
- [ ] 自动化部署
- [ ] 滚动更新

**产出：** 自动化 K8s 部署

### Day 4：GitOps 实践
**目标：** 实现 GitOps
**任务：**
- [ ] 安装 ArgoCD
- [ ] 配置 Application
- [ ] 实现自动同步
- [ ] 金丝雀发布

**产出：** GitOps 部署流水线

### Day 5：监控和告警
**目标：** 完善可观测性
**任务：**
- [ ] 配置 Prometheus
- [ ] 配置 Grafana
- [ ] 设置告警规则
- [ ] 集成通知

**产出：** 完整的可观测性系统

---

## 🎯 面试要点

### 问题 1：什么是 CI/CD？有什么区别？

**参考答案：**
```
CI（持续集成）：
- 频繁将代码合并到主干
- 自动化构建和测试
- 快速发现集成错误

CD（持续交付/部署）：
- 持续交付：代码随时可部署
- 持续部署：自动部署到生产
- 自动化发布流程

关系：
CI 是 CD 的基础
CI 确保代码质量
CD 确保快速交付
```

### 问题 2：如何在 K8s 中实现零宕机部署？

**参考答案：**
```
关键机制：

1. 滚动更新
   - maxSurge: 1
   - maxUnavailable: 0
   - 保证服务不中断

2. 就绪探针
   - 新 Pod 就绪后才接收流量
   - 避免请求发送到未启动的 Pod

3. 优雅关闭
   - preStop hook
   - 等待连接关闭
   - 缓存清理

4. PodDisruptionBudget
   - 维护期间保证最小可用数
```

### 问题 3：GitOps 的优势是什么？

**参考答案：**
```
GitOps 优势：

1. 声明式基础设施
   - 配置即代码
   - 版本控制
   - 可审计

2. 单一事实来源
   - Git 是期望状态
   - K8s 是实际状态
   - 自动同步差异

3. 简化回滚
   - git revert 即可回滚
   - 完整的配置历史
   - 快速恢复

4. 安全
   - 基于 Git 权限
   - 审批流程
   - 变更审计
```

### 问题 4：Docker 镜像构建优化有哪些？

**参考答案：**
```
优化策略：

1. 多阶段构建
   - 分离构建和运行环境
   - 减小镜像体积

2. 减小层数
   - 合并 RUN 指令
   - 按需安装依赖

3. 利用缓存
   - 先复制依赖文件
   - 再复制源码
   - 依赖不变时不重新构建

4. 使用轻量基础镜像
   - alpine
   - distroless
   - scratch

5. .dockerignore
   - 减少构建上下文
```

### 问题 5：Go 协程和线程的区别是什么？

**参考答案：**
```
线程：
- 内核态调度
- 栈大小 1-2MB
- 创建/切换成本高
- 数量有限

Go 协程：
- 用户态调度（GMP）
- 栈大小 2KB（可扩展）
- 创建成本极低
- 支持数十万并发

调度器：
- M 绑定 P 执行 G
- 工作窃取负载均衡
- 非阻塞式系统调用
```

---

## 📂 完整工具链清单

```
┌─────────────────────────────────────────────────────────────────┐
│                        CI/CD 工具链                              │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   代码编写 ──▶ Git ──▶ PR/MR ──▶ CI ──▶ CD ──▶ 监控              │
│                  │           │        │                        │
│                  ▼           ▼        ▼                        │
│               GitHub    GitHub      ArgoCD                      │
│               Actions   Actions     / Jenkins                   │
│                          │                                         │
│                          ▼                                         │
│                     ┌─────────────┐                               │
│                     │   构建      │                               │
│                     │  Docker     │                               │
│                     │  Buildx     │                               │
│                     └─────────────┘                               │
│                          │                                         │
│                          ▼                                         │
│                     ┌─────────────┐                               │
│                     │   安全      │                               │
│                     │  Trivy      │                               │
│                     │  SonarQube  │                               │
│                     └─────────────┘                               │
│                          │                                         │
│                          ▼                                         │
│                     ┌─────────────┐                               │
│                     │   推送      │                               │
│                     │  Registry   │                               │
│                     └─────────────┘                               │
│                          │                                         │
│                          ▼                                         │
│                     ┌─────────────┐                               │
│                     │   部署      │                               │
│                     │  Kubernetes │                               │
│                     │  Helm       │                               │
│                     └─────────────┘                               │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 🎓 总结

这个 CI/CD 自动化流水线项目展示了以下核心能力：

| 技能 | 应用场景 |
|-----|---------|
| GitHub Actions | CI/CD 自动化 |
| Docker BuildKit | 高效镜像构建 |
| Trivy | 安全扫描 |
| Helm | 包管理 |
| ArgoCD | GitOps |
| Kustomize | 环境差异化 |
| 监控集成 | 可观测性 |

**这是 DevOps/SRE 岗位面试的核心项目！**
