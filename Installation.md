# 安裝與執行指南 (Installation Guide)

本指南將引導您從零開始搭建這套 GitOps 自動化流程。

## ✅ 前置需求 (Prerequisites)

* **Docker Hub** 帳號 (用於存放 Image)。
* **GitHub** 帳號。
* **Kubernetes Cluster** (推薦 Mac 使用者安裝 [OrbStack](https://orbstack.dev/)，輕量且支援 K8s)。
* **ArgoCD CLI** (可選，用於 debug)。

---

## 步驟 1：準備雙 Repo 結構

為了符合 GitOps 最佳實踐，我們需要建立兩個 GitHub Repository：

1.  **App Repo (Source Code)**: 放 Go code。
2.  **Config Repo (Manifests)**: 放 K8s YAML。

### 設定 Config Repo
在 Config Repo 中建立 `deployment.yml`，確保包含以下關鍵設定：
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: go-app-deployment
spec:
  replicas: 2  # 高可用性設定
  template:
    spec:
      containers:
      - name: go-app
        image: <YOUR_DOCKERHUB_USER>/demo-golang:latest # CI 會自動更新這裡