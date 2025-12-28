# Go-K8s-GitOps-Demo 🚀

![CI Status](https://img.shields.io/badge/CI-GitHub_Actions-blue?logo=github-actions)
![CD Status](https://img.shields.io/badge/CD-ArgoCD-orange?logo=argo)
![Go Version](https://img.shields.io/badge/Go-1.21+-00ADD8?logo=go)
![K8s](https://img.shields.io/badge/Kubernetes-Ready-326ce5?logo=kubernetes)

>一個完整依照 Cloud Native **GitOps** 的實作專案。
展示如何將 Golang 應用程式透過 **GitHub Actions 進行 CI (持續整合)**
並使用 **ArgoCD 實踐 CD (持續部署) 到 Kubernetes 叢集**。

---
###### ✨ 特色 (Features)

* **全自動化 CI/CD**：從 Code Commit 到上線完全無需人工介入。
* **GitOps 最佳實踐**：採用「雙 Repo」策略（源程式碼與CD分離），確保 **Git 是唯一的真理 (Single Source of Truth)**。
* **多架構支援 (Multi-Arch)**：自動構建支援 `linux/amd64` 與 `linux/arm64` (Apple Silicon) 的 Docker Image。
* **自我修復 (Self-Healing)**：ArgoCD 自動監控並修正任何非預期的手動變更 (Configuration Drift)。
* **零停機更新**：利用 Kubernetes Rolling Update 實現平滑版更。

---
###### 🛠 專案結構 (Repositories) & 技術堆疊 (Tech Stack)
- 本專案分為兩個儲存庫：
1.  **Source Code Repo (本專案)**: 包含 Go 程式碼、Dockerfile 與 GitHub Actions Workflow。
2.  **CD Repo (Kubernetes Manifests)**: 包含 K8s YAML 設定檔 (`deployment.yml`, `service.yml`)。

| 類別 | 工具 | 用途 |
| :--- | :--- | :--- |
| **語言** | Golang (Gin Framework) | 後端應用程式 |
| **容器化** | Docker | 應用封裝 |
| **CI 工具** | GitHub Actions | 自動化構建、測試、推送 Image |
| **CD 工具** | ArgoCD | GitOps 同步與部署管理 |
| **基礎設施** | Kubernetes | 容器編排與管理 |
| **環境** | OrbStack | 本地 Kubernetes 模擬環境 |

---
###### 🚀 實作影片 (Experimental Video)

---
###### 🚀 架系統架構 (Architecture)

```mermaid
graph LR
    classDef plain fill:#fff,stroke:#333,stroke-width:1px;
    classDef k8s fill:#e1f5fe,stroke:#0277bd,stroke-width:2px;
    classDef git fill:#f3e5f5,stroke:#7b1fa2,stroke-width:2px;

    User["Developer 👨‍💻"]
    
    subgraph GitHub ["GitHub Cloud ☁️"]
        direction TB
        RepoA["1. Repo A<br>(Source Code)"]:::git
        Action["2. GitHub Actions<br>(CI Pipeline)"]:::plain
        RepoB["4. Repo B<br>(Config/Manifests)"]:::git
    end

    DockerHub["Docker Hub 🐳"]:::plain

    subgraph K8s ["Kubernetes Cluster ☸️"]
        direction TB
        ArgoCD["ArgoCD Controller 🐙"]:::k8s
        App["Application <br>(Pod)"]:::k8s
    end

    %% 流程連線
    User -->|"1. Push Code"| RepoA
    RepoA -->|"2. Trigger CI"| Action
    
    Action -->|"3. Build & Push Image"| DockerHub
    Action -->|"4. Update Image Tag"| RepoB
    
    ArgoCD -->|"5. Watch & Pull Config"| RepoB
    ArgoCD -->|"6. Sync/Deploy"| App
    
    DockerHub -.->|"7. Pull Image"| App

    RepoB ~~~ DockerHub
```


---
## 🚀 執行指南 (Getting Started)

為了確保環境設定正確，請嚴格依照以下順序閱讀並執行文件：
 [INSTALL.md](./INSTALL.md)。
https://deep-wedelia-d0a.notion.site/2ca488f98401801aa42ec3972c6d14ed