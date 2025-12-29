# 安裝與執行指南 (Installation Guide)

本指南將從零開始搭建這套 GitOps 自動化流程。

## ✅ 前置需求 (Prerequisites)

* **GitHub** 帳號 *(此專案使用 GitHub Actions 做 CI 範例)*。
* **Docker Hub** 帳號 (用於存放 Image)。
* **Kubernetes Cluster** (推薦 Mac 使用者安裝 [OrbStack](https://orbstack.dev/)，輕量且支援 Kubernetes)。
* **ArgoCD CLI** (不一定需要，用於 debug)。

---
### 步驟 1：準備雙 Repo 結構

為了符合 GitOps 最佳實踐，我們需要建立兩個 GitHub Repository：

1. **Application Repo (Source Code)**: 放 Application 的 code。 *範例 : [Demo-Golang](https://github.com/JeffLin0225/Demo-Golang)*
2. **Config Repo (Manifests)**: 放 K8s YAML。 *範例 : [Kubernetes-ArgoCD](https://github.com/JeffLin0225/Kubernetes-ArgoCD)*

---
### 步驟 2：Application Repo, Config Repo, DockerHub 交互設定

---
### 步驟 3：設定Application Repo 的 CI 配置文件
1. 在專案根目錄下建立 :  `.github/workflows/docker-publish.yml`
2. 填入以下內容: (*配置可以自由選用*)
```yaml
name: (Manual Trigger)Build and Push to Docker Hub

# 觸發方式？
on:
  workflow_dispatch:        #「手動執行」的按鈕
    
# on:
#   push:                   
#     branches: [ "main" ]  # 當有程式碼 push 到 main 分支時

env:
  # 這裡會自動抓 Secret 帳號 + 專案名稱
  IMAGE_NAME: ${{ secrets.DOCKERHUB_USERNAME }}/demo-golang
  
  # ArgoCD 的 Repo 名稱
  CD_REPO_NAME: JeffLin0225/Kubernetes-ArgoCD

jobs:
  build-and-push:
    runs-on: ubuntu-latest
    
    steps:
    # ======================================================
    # 第一階段：CI (打包程式碼 -> Docker Hub)
    # ======================================================
    # 1. 下載程式碼到虛擬機
    - name: Checkout code
      uses: actions/checkout@v4

    # 編譯 ARM64 的 Image
    - name: Set up QEMU
      uses: docker/setup-qemu-action@v3

    # 2. 設定 Docker Buildx 
    - name: Set up Docker Buildx
      uses: docker/setup-buildx-action@v3

    # 3. 登入 Docker Hub
    - name: Log in to Docker Hub
      uses: docker/login-action@v3
      with:
        username: ${{ secrets.DOCKERHUB_USERNAME }}
        password: ${{ secrets.DOCKERHUB_TOKEN }}

    # 這一步是用來產生 "a1b2c3d" 這種短版版號的
    - name: Get Short SHA
      id: vars   # 這個 id 很重要，下面的 steps.vars 就是在叫它
      run: echo "sha_short=$(git rev-parse --short HEAD)" >> $GITHUB_OUTPUT

    # 4. 打包 Image 並推送到 Docker Hub
    - name: Build and push
      uses: docker/build-push-action@v5
      with:
        context: . # 找 Dockerfile 檔案
        push: true
        # 下面這行格式是: 你的帳號/你的專案名:標籤
        # 注意：變數名稱要跟 secret 一樣
        # 指定要同時打包兩種架構
        # latest 用來看， sha用來給CD辨別差異用的
        platforms: linux/amd64,linux/arm64
        tags: |
          ${{ env.IMAGE_NAME }}:latest 
          ${{ env.IMAGE_NAME }}:${{ steps.vars.outputs.sha_short }}

    # ======================================================
    # 第二階段：CD (使用 SSH Key 修改 Repo B)
    # ======================================================
    - name: Checkout ArgoCD Repo
      uses: actions/checkout@v4
      with:
        repository: ${{ env.CD_REPO_NAME }}
        #  這裡就是我們剛剛辛苦設定的 SSH 私鑰
        ssh-key: ${{ secrets.ARGOCD_GITOPS_KEY }}
        path: argocd-repo
    
    - name: Update Image Tag in Deployment
      run: |
          cd argocd-repo
          
          # 1. 設定機器人身分 (Git 需要知道是誰改的)
          git config user.name "GitHub Actions"
          git config user.email "actions@github.com"
          
          # 2. 修改 deployment.yml
          # 使用 sed 指令，把舊的 image tag 換成剛剛打包好的 sha_short
          sed -i "s|image: .*|image: ${{ env.IMAGE_NAME }}:${{ steps.vars.outputs.sha_short }}|g" deployment.yml
          
          # 3. 檢查修改結果 (印在 Log 讓你確認)
          echo "Modified deployment.yml content:"
          cat deployment.yml | grep image:
          
          # 4. 提交並推送 (Push)
          # 因為上面是用 SSH Key checkout，這裡會自動用 SSH 協定推送
          git add deployment.yml
          git commit -m "Auto-update image tag to ${{ steps.vars.outputs.sha_short }}"
          git push origin main
```
---
### 步驟 4：設定 Config Repo 的檔案
1. 在 Config Repo 中建立 `deployment.yml`
2. 確保包含以下關鍵設定：
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: go-app-deployment
spec:
  replicas: 2   # 高可用性設定 (自行決定)
  selector:
    matchLabels:
      app: go-app
  template:
    metadata:
      labels:
        app: go-app
    spec:
      containers:
      - name: go-app
        # 這裡的 tag 之後會被 GitHub Actions 自動改掉
        image: docker-id/go-argocd:latest # <--[超級重點] CI 會自動更新這裡 
        ports:
        - containerPort: 8080
---
apiVersion: v1
kind: Service
metadata:
  name: go-app-service
spec:
  selector:
    app: go-app
  ports:
    - protocol: TCP
      port: 80
      targetPort: 8080
  type: LoadBalancer
```
