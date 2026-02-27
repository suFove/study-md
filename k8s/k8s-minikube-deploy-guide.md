# K8s Minikube 部署指南 - 本地镜像服务

> 记录：2026-02-27  
> 场景：WSL2 + Minikube + 本地 Docker 镜像

---

## 📋 前置条件

- Minikube 已安装并运行
- Docker 镜像已加载到本地
- kubectl 已配置

---

## 🔍 检查镜像

```bash
# 查看 Minikube 中的镜像列表
minikube image ls | grep -i <镜像名>

# 示例
minikube image ls | grep age
# 输出：docker.io/library/age-viewer-image:latest
```

---

## 🧹 一、清理旧资源

```bash
# 删除 Deployment 和 Service（如果存在）
kubectl delete deployment <服务名> 2>/dev/null
kubectl delete svc <服务名> 2>/dev/null

# 示例
kubectl delete deployment age-k8s 2>/dev/null
kubectl delete svc age-k8s 2>/dev/null
```

---

## 🚀 二、启动部署流程

### 步骤 1：创建 Deployment

```bash
kubectl create deployment <服务名> --image=<完整镜像名>

# 示例
kubectl create deployment age-k8s --image=docker.io/library/age-viewer-image:latest
```

### 步骤 2：设置本地镜像策略（关键！）

```bash
kubectl patch deployment <服务名> --type='json' -p='[{"op": "replace", "path": "/spec/template/spec/containers/0/imagePullPolicy", "value": "Never"}]'

# 示例
kubectl patch deployment age-k8s --type='json' -p='[{"op": "replace", "path": "/spec/template/spec/containers/0/imagePullPolicy", "value": "Never"}]'
```

### 步骤 3：验证 Pod 状态

```bash
kubectl get pods -l app=<服务名>

# 等待 STATUS 变为 Running
# 示例
kubectl get pods -l app=age-k8s
```

### 步骤 4：暴露服务

```bash
kubectl expose deployment <服务名> --type=NodePort --port=<容器端口>

# 示例
kubectl expose deployment age-k8s --type=NodePort --port=3000
```

### 步骤 5：端口转发（让 Windows 浏览器可访问）

```bash
# 后台运行端口转发
kubectl port-forward svc/<服务名> <本地端口>:<容器端口> --address 0.0.0.0 &

# 示例
kubectl port-forward svc/age-k8s 3000:3000 --address 0.0.0.0 &
```

### 步骤 6：获取访问地址

```bash
# 方式 A：使用 localhost（端口转发后）
echo "访问地址：http://localhost:3000"

# 方式 B：使用 Minikube IP
minikube service <服务名> --url
```

---

## 🛑 三、关闭流程

### 方式 A：停止端口转发（保留服务）

```bash
# 查找端口转发进程
ps aux | grep port-forward

# 杀死进程
pkill -f "port-forward"

# 或指定 PID 杀死
kill <PID>
```

### 方式 B：完全删除服务

```bash
# 先停止端口转发
pkill -f "port-forward"

# 删除 Service 和 Deployment
kubectl delete svc <服务名>
kubectl delete deployment <服务名>

# 验证已清理
kubectl get pods
kubectl get svc
kubectl get deployment
```

### 方式 C：缩放副本数为 0（保留配置）

```bash
# 停止所有 Pod，但保留 Deployment 配置
kubectl scale deployment <服务名> --replicas=0

# 恢复时
kubectl scale deployment <服务名> --replicas=1
```

---

## 📝 四、完整示例（age-viewer 服务）

### 启动

```bash
# 1. 清理
kubectl delete deployment age-k8s 2>/dev/null
kubectl delete svc age-k8s 2>/dev/null

# 2. 创建
kubectl create deployment age-k8s --image=docker.io/library/age-viewer-image:latest

# 3. 设置本地镜像
kubectl patch deployment age-k8s --type='json' -p='[{"op": "replace", "path": "/spec/template/spec/containers/0/imagePullPolicy", "value": "Never"}]'

# 4. 暴露
kubectl expose deployment age-k8s --type=NodePort --port=3000

# 5. 端口转发
kubectl port-forward svc/age-k8s 3000:3000 --address 0.0.0.0 &

# 6. 访问
echo "✅ 服务已启动：http://localhost:3000"
```

### 关闭

```bash
# 停止转发
pkill -f "port-forward"

# 删除资源
kubectl delete svc age-k8s
kubectl delete deployment age-k8s

echo "✅ 服务已关闭"
```

---

## 🔧 五、常用命令速查

| 目的 | 命令 |
|------|------|
| 查看 Pod 状态 | `kubectl get pods` |
| 查看 Service | `kubectl get svc` |
| 查看 Deployment | `kubectl get deployment` |
| 查看 Pod 详情 | `kubectl describe pod -l app=<服务名>` |
| 查看 Pod 日志 | `kubectl logs -l app=<服务名>` |
| 查看 Minikube 镜像 | `minikube image ls` |
| 加载镜像到 Minikube | `minikube image load <镜像.tar>` |
| 启动 Minikube | `minikube start` |
| 停止 Minikube | `minikube stop` |

---

## ⚠️ 六、常见问题

### Q1: Pod 状态是 ImagePullBackOff

**原因**：K8s 尝试从远程拉取镜像，但本地才有

**解决**：
```bash
kubectl patch deployment <服务名> --type='json' -p='[{"op": "replace", "path": "/spec/template/spec/containers/0/imagePullPolicy", "value": "Never"}]'
```

### Q2: Windows 浏览器无法访问

**原因**：Minikube 在 WSL2 内，IP 不可达

**解决**：使用端口转发
```bash
kubectl port-forward svc/<服务名> 3000:3000 --address 0.0.0.0 &
```

### Q3: 端口已被占用

**解决**：换一个本地端口
```bash
kubectl port-forward svc/age-k8s 8080:3000 --address 0.0.0.0 &
# 然后访问 http://localhost:8080
```

### Q4: 找不到镜像

**解决**：先加载镜像到 Minikube
```bash
# 从 Docker 导出
docker save <镜像名>:latest -o <镜像名>.tar

# 加载到 Minikube
minikube image load <镜像名>.tar

# 验证
minikube image ls | grep <镜像名>
```

---

## 📚 七、核心概念

| 资源 | 作用 |
|------|------|
| **Pod** | K8s 最小调度单位，包含一个或多个容器 |
| **Deployment** | 管理 Pod 的部署，支持滚动更新、回滚 |
| **ReplicaSet** | 维护 Pod 副本数量（由 Deployment 自动管理） |
| **Service** | 暴露服务，提供稳定的访问入口 |
| **NodePort** | Service 类型之一，在每个节点上开放端口 |

---

## 🎯 关键参数说明

```yaml
imagePullPolicy:
  - Always        # 总是从远程拉取（默认）
  - IfNotPresent  # 本地没有才拉取
  - Never         # 只用本地镜像，不拉取 ← 本地部署用这个
```

---

> 💡 **提示**：本地镜像部署的核心就是 `imagePullPolicy: Never`，确保 K8s 不会尝试联网拉取镜像。
