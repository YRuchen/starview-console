# Observable Frontend Helm Chart

这是 Observable Frontend 应用的 Helm Chart，用于在 Kubernetes 集群中部署前端应用。

## 项目概述

Observable Frontend 是一个基于 Vue 3 + TypeScript + Vite 构建的现代化前端应用，使用 Nginx 作为 Web 服务器。

### 技术栈
- **前端框架**: Vue 3 + TypeScript
- **构建工具**: Vite
- **UI 框架**: Element Plus
- **Web 服务器**: Nginx
- **容器化**: Docker

## 安装指南

### 前置条件
- Kubernetes 1.16+
- Helm 3.0+
- 镜像仓库访问权限

### 安装步骤

1. **添加镜像仓库密钥**（如果需要）:
```bash
kubectl create secret docker-registry docker-registry-secret \
  --docker-server=your-registry.com \
  --docker-username=your-username \
  --docker-password=your-password \
  --docker-email=your-email@example.com
```

2. **安装 Chart**:
```bash
# 开发环境
helm install obs-frontend ./helm-chart -f ./helm-chart/values/values-dev.yaml

# 生产环境
helm install obs-frontend ./helm-chart -f ./helm-chart/values/values-prod.yaml
```

3. **升级 Chart**:
```bash
helm upgrade obs-frontend ./helm-chart -f ./helm-chart/values/values-dev.yaml
```

4. **卸载 Chart**:
```bash
helm uninstall obs-frontend
```

## 配置说明

### 镜像配置
```yaml
image:
  repository: "harbor.gainetics.io/observable/images/obs-frontend-prod"  # 镜像仓库地址
  pullPolicy: IfNotPresent                                                # 镜像拉取策略
  tag: "1.0.0"                                                           # 镜像标签
```

### 环境变量配置
基于 `.env.eks` 文件设计的环境变量：

```yaml
env:
  NODE_ENV: "production"
  VITE_APP_BASE_API: ""
  VITE_APP_BASE_API_METRICS: "https://alert-manager.observe.dev.gainetics.io"
  VITE_APP_BASE_API_AVAILABILITY: "gateway.observe.dev.eks.gainetics.io"
  VITE_APP_BASE_API_DOMAIN: "gateway.observe.dev.eks.gainetics.io/domain/api/v1"
  VITE_APP_BASE_API_AIAGENT: "gateway.observe.dev.eks.gainetics.io"
  VITE_APP_USER_API: "http://observable-user-service.scloud-obs-dev.svc.cluster.local:8000"
  VITE_APP_CORE_API: "http://observable-core-service-dev.scloud-obs-dev.svc.cluster.local:8000"
```

### Ingress 配置
```yaml
ingress:
  enabled: true
  className: "nginx"
  hosts:
    - host: www.observe.dev.eks.gainetics.io
      paths:
        - path: /
          pathType: Prefix
```

### 资源配置
```yaml
resources:
  limits:
    cpu: 500m
    memory: 512Mi
  requests:
    cpu: 100m
    memory: 128Mi
```

## 部署环境

### 开发环境 (values-dev.yaml)
- **副本数**: 1
- **资源配置**: 较低的 CPU 和内存限制
- **镜像标签**: dev
- **自动扩缩**: 禁用

### 生产环境 (values-prod.yaml)
- **副本数**: 3
- **资源配置**: 较高的 CPU 和内存限制
- **镜像标签**: latest
- **自动扩缩**: 启用 (3-10 副本)
- **TLS**: 启用 HTTPS

## 监控和健康检查

### 存活探针 (Liveness Probe)
- **路径**: `/`
- **初始延迟**: 30 秒
- **检查间隔**: 10 秒

### 就绪探针 (Readiness Probe)
- **路径**: `/`
- **初始延迟**: 5 秒
- **检查间隔**: 5 秒

## 故障排除

### 查看 Pod 状态
```bash
kubectl get pods -l app.kubernetes.io/name=obs-frontend
```

### 查看日志
```bash
kubectl logs -l app.kubernetes.io/name=obs-frontend
```

### 查看配置
```bash
kubectl describe configmap obs-frontend-config
kubectl describe configmap obs-frontend-nginx-config
```

### 端口转发调试
```bash
kubectl port-forward svc/obs-frontend 8080:80
```

## 自定义配置

### 修改 Nginx 配置
可以通过修改 `values.yaml` 中的 `nginx.config` 来自定义 Nginx 配置。

### 添加自定义环境变量
在 `values.yaml` 的 `env` 部分添加新的环境变量。

### 配置持久化存储
如果需要持久化存储，可以在 `values.yaml` 中配置 `volumes` 和 `volumeMounts`。

## 安全考虑

1. **镜像安全**: 使用私有镜像仓库并定期更新镜像
2. **网络安全**: 通过 NetworkPolicy 限制网络访问
3. **RBAC**: 使用最小权限原则配置 ServiceAccount
4. **TLS**: 在生产环境中启用 HTTPS

## 版本历史

- **v0.1.0**: 初始版本，支持基本的部署功能

## 支持和贡献

如有问题或建议，请联系 Gainetics 开发团队。
