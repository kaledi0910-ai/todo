# 系統架構圖

本專案是一個以 FastAPI 為核心的 Todo API，並串接 GitHub Actions、GHCR、Argo CD、Kubernetes(kind) 與自建 Worker Agent，形成從開發、CI/CD 到 GitOps 部署的完整實戰流程。

## 整體架構

```mermaid
flowchart LR
    Dev[開發者 / Codex] -->|修改程式碼| Git[Git Repository]
    Git -->|push / PR| GH[GitHub]

    GH --> CI[GitHub Actions CI]
    CI --> Test[測試 / Codex Review / Gate]
    CI --> Build[Build Docker Image]
    Build --> GHCR[GitHub Container Registry]

    CI -->|更新 manifest PR| Manifest[k8s manifests]
    Manifest --> GH

    GH --> Argo[Argo CD]
    Argo -->|GitOps Sync| K8s[Kubernetes / kind]
    GHCR -->|Pull Image| K8s

    K8s --> API[FastAPI Todo API]
    User[使用者 / Browser / Swagger] --> API

    GH --> Agent[自建 Worker Agent]
    Agent -->|Auto Review / Autofix / 任務執行| GH

    subgraph Repo[Repository 主要內容]
        App[app/ FastAPI 應用]
        K8sFiles[k8s/ 部署設定]
        ArgoFiles[argocd/ Argo CD Application]
        Agents[agents/ Worker Agents]
        Plugins[plugins/ 擴充]
        Hooks[hooks/ 與 .codex/ 規則]
    end

    Git -. contains .-> Repo
```

## CI/CD 與 GitOps 流程

```mermaid
sequenceDiagram
    participant D as 開發者/Codex
    participant G as GitHub
    participant A as GitHub Actions
    participant R as GHCR
    participant C as Argo CD
    participant K as Kubernetes(kind)

    D->>G: Push / Pull Request
    G->>A: 觸發 CI
    A->>A: Test / Codex Review / Gate
    A->>R: Build & Push Image
    A->>G: 更新 k8s manifest / 建立 PR
    G->>C: Git repo 狀態改變
    C->>K: Sync manifests
    K->>R: Pull 新 image
    K-->>D: 新版本 Todo API 可用
```

## 元件職責

| 元件 | 職責 |
|---|---|
| `app/` | FastAPI Todo API 與前端/Swagger 入口 |
| GitHub Actions | CI、測試、Codex review/gate、建置與發佈 image |
| GHCR | 儲存 Todo API Docker image |
| `k8s/` | Kubernetes Deployment / Service 等 manifests |
| `argocd/` | Argo CD Application，持續同步 Git 狀態 |
| kind | 本機 Kubernetes 實驗環境 |
| `agents/` | 自建 Worker Agent、auto-review、autofix 等 |
| `.codex/` / `hooks/` | Codex 規則、hooks 與自動化護欄 |
| `plugins/` | 額外擴充能力 |

## 一句話理解

> Codex 協助開發 → GitHub Actions 驗證與建置 → GHCR 保存映像 → Argo CD 監看 Git → Kubernetes 自動同步部署；Worker Agent 再回頭協助 GitHub 上的 review 與修復流程。
