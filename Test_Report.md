# ArgoCD 實作報告
### 一. 事前準備 (Preparation)
為了構建安全的自動化鏈路，我們完成了以下關鍵基礎建設配置：

1. 雙儲存庫建立：
- Application Repo：存放 Go 原始碼與 CI Workflow。
- Config Repo：存放 Kubernetes Manifests (deployment.yml)，作為環境狀態的唯一真理。

2. 身分與權限串接：
- Docker Hub：生成 Access Token 並配置於 GitHub Secrets，賦予 CI 推送 Image 的權限。
- SSH Deploy Keys：生成 ed25519 金鑰對 
-- 私鑰 (Private Key)：存放在 Application Repo Secrets，供 CI 機器人修改 Config Repo 使用。
-- 公鑰 (Public Key)：配置在 Config Repo 的 Deploy Keys，作為接收變更的「門鎖」。

### 二. 使用服務 (Services Used)
1. 自動更新配置 (Continuous Delivery Handoff)
- CI 的最後階段，Action 讀取 SSH 私鑰，自動 Checkout Config Repo。
- 使用 sed 指令將 deployment.yml 中的 image tag 修改為最新的 Commit SHA。
- 自動 Commit 並 Push 回 Config Repo，完成「版本更新」的動作。

2. 部署 ArgoCD 基礎設施
- 在 K8s 中建立 argocd namespace。
- 透過官方 Manifest 安裝 ArgoCD 核心組件。
- 使用 kubectl port-forward 將服務轉發至本地 8080 端口，並解碼 Secret 取得 admin 初始密碼完成登入。


### 三. 開始實驗 (Experiment Process)

1. 執行 CI 流水線 (Continuous Integration)
- 觸發：在 Application Repo 執行 Workflow (或 Push Code)。
- 建置：GitHub Actions 自動建立支援 linux/amd64 與 linux/arm64 的 Docker Image。
- 推送：將帶有 latest 與 Short SHA 標籤的 Image 推送至 Docker Hub。


2. GitOps 同步 (Deployment)
- ArgoCD 偵測到 Config Repo 的 deployment.yml 發生變動。
- 自動對比 Live State (叢集現狀) 與 Desired State (Git 狀態)。
- 執行 Sync，將 K8s 內的 Pod 更新為新版 Image，完成部署。


### 四. 總結 (Conclusion)
本專案成功實作了符合 Cloud Native 標準的 GitOps 自動化流程，驗證了透過「雙儲存庫 (Dual-Repo)」策略，如何將繁瑣的部署流程轉化為優雅的 Git Commit 操作，並將部署控制權完全交由 ArgoCD 接管。

核心成果：

- 單一真理 (Single Source of Truth)： 徹底落實了 GitOps 核心精神。透過 Config Repo 管理 Manifest，確保了 Git 上的狀態永遠等於 K8s 叢集的預期狀態 (Desired State)，讓每一次的版本追溯與回滾 (Rollback) 都變得有跡可循且絕對可靠。

- 安全架構 (Security by Design)： 完美融合 "Push-based CI" 與 "Pull-based CD" 的黃金架構。CI 僅需負責打包與更新 Image Tag，無需持有 K8s Cluster Admin 憑證；部署權限完全回收至叢集內部的 ArgoCD，大幅降低了資安邊界風險。

- 自動化閉環 (Automated Closed-Loop)： 打通了從 Code Commit -> Image Build (Multi-Arch) -> Manifest Update -> K8s Sync 的最後一哩路。解決了跨平台 (ARM/AMD) 建置痛點，實現了真正的「代碼提交即上線」，全程零人工介入。

心得： 這不僅是一次 CI/CD 的工具串接，更是一種將「人工維運」降至最低的信任機制建立。當基礎設施變成了一行行可被版控的代碼，部署就不再是壓力，而是一種日常。

>***不為了什麼，為了好玩。*** —— 看著 ArgoCD 的 Sync 狀態由黃轉綠，是 DevOps 工程師最治癒的瞬間。