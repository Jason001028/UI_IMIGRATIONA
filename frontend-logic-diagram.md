# React 前端逻辑与组件交互

这是一个详细的图表，用于说明 `contract-analysis-frontend` 应用的内部工作原理。它展示了从用户交互到组件渲染、API 调用和状态管理的整个流程。

## 核心概念

- **组件化架构**: 应用被分解为多个可重用的 React 组件（例如 `DocumentUpload`, `AnalysisResults`）。
- **路由**: `react-router-dom` 用于管理不同页面（视图）之间的导航。
- **服务层**: 一个专门的 `api.ts` 文件处理所有与后端 API 的通信，将数据获取逻辑与 UI 组件分离。
- **状态管理**: 组件内部状态主要通过 `useState` 和 `useEffect` 钩子来管理。

## Mermaid 流程图

下面的图表详细描述了应用的结构和数据流。

```mermaid
graph TD
    %% General Style Definitions
    classDef component fill:#e1effd,stroke:#5a96e0,stroke-width:2px,color:#000
    classDef service fill:#d5fada,stroke:#5aa05a,stroke-width:2px,color:#000
    classDef hook fill:#fff2cc,stroke:#d6b656,stroke-width:1px,color:#000,stroke-dasharray: 5 5
    classDef user fill:#f9f,stroke:#333,stroke-width:2px
    classDef api fill:#fadddd,stroke:#c87070,stroke-width:2px,color:#000
    classDef route fill:#e6e0f8,stroke:#9673e8,stroke-width:2px,color:#000
    classDef ui fill:#dcdcdc,stroke:#666,stroke-width:1px,color:#000
    classDef logic fill:#fce8b2,stroke:#e5a743,stroke-width:1px,color:#000

    %% Start of Diagram
    User["/"用户"/"]:::user

    subgraph "浏览器"
        direction LR
        User -- "访问 http://localhost:3000" --> App
        App["App.tsx - 应用入口"]:::component
    end

    subgraph "React 应用核心"
        direction TB
        App -- "初始化路由" --> Router{React Router}:::route

        Router -- "路径: /" --> HomePage{"主页 (/)"}:::route
        Router -- "路径: /results/:id" --> ResultsPage{"结果页 (/results/:id)"}:::route
        Router -- "路径: /admin" --> AdminPage{"管理页 (/admin)"}:::route

        subgraph "主页视图 (Home)"
            HomePage -- "渲染" --> DU["DocumentUpload.tsx"]:::component
            DU -- "包含UI" --> UploadUI["文件拖放区和上传按钮"]:::ui
            UploadUI -- "用户选择/拖放文件" --> HandleUpload["handleUpload() 函数"]:::logic
            HandleUpload -- "1. 设置加载状态" --> UseState1["useState(isLoading)"]:::hook
            HandleUpload -- "2. 调用API服务" --> APIService["api.ts 服务层"]:::service
            APIService -- "调用 api.uploadDocument(file)" --> Backend["后端 API"]:::api
            Backend -- "成功时返回 { success: true, id: '...' }" --> APIService
            APIService -- "Promise<ApiResponse>" --> HandleUpload
            HandleUpload -- "3. 成功后导航" --> NavigateToResults["navigate('/results/' + id)"]:::logic
            NavigateToResults -- "触发路由变更" --> Router
            HandleUpload -- "4. 失败时" --> ShowError["显示错误提示"]:::ui
        end

        subgraph "结果页视图 (Results)"
            ResultsPage -- "渲染" --> AR["AnalysisResults.tsx"]:::component
            AR -- "组件挂载时触发" --> UseEffect1["useEffect()"]:::hook
            UseEffect1 -- "从URL中提取 'id'" --> GetID["useParams() 钩子"]:::hook
            GetID -- "提供ID" --> FetchData["fetchAnalysisData(id) 函数"]:::logic
            FetchData -- "调用API服务" --> APIService
            APIService -- "调用 api.getAnalysis(id)" --> Backend
            Backend -- "返回 AnalysisResult 对象" --> APIService
            APIService -- "Promise<AnalysisResult>" --> FetchData
            FetchData -- "设置结果状态" --> UseState2["useState(analysisResult)"]:::hook
            UseState2 -- "提供数据给" --> RenderResults["渲染逻辑"]:::logic
            RenderResults -- "渲染" --> ResultsUI["结果表格和图表"]:::ui
            RenderResults -- "同时渲染" --> DV["DocumentViewer.tsx"]:::component
            DV -- "接收 'documentUrl' 作为prop" --> DV
            DV -- "渲染" --> DocumentUI["PDF/HTML查看器 (iframe)"]:::ui
        end

        subgraph "管理页视图 (Admin)"
            AdminPage -- "渲染" --> AC["AdminCache.tsx"]:::component
            AC -- "组件挂载时触发" --> UseEffect2["useEffect()"]:::hook
            UseEffect2 -- "调用" --> FetchStats["fetchCacheStats() 函数"]:::logic
            FetchStats -- "调用API服务" --> APIService
            APIService -- "调用 api.getCacheStats()" --> Backend
            Backend -- "返回 CacheStats 对象" --> APIService
            APIService -- "Promise<CacheStats>" --> FetchStats
            FetchStats -- "设置统计状态" --> UseState3["useState(cacheStats)"]:::hook
            UseState3 -- "提供数据给" --> RenderAdminUI["渲染逻辑"]:::logic
            RenderAdminUI -- "渲染" --> StatsUI["统计信息卡片"]:::ui
            RenderAdminUI -- "同时渲染" --> CollectionsUI["缓存集合表格"]:::ui
            CollectionsUI -- "用户点击 '删除'" --> HandleDelete["handleDeleteCollection(name) 函数"]:::logic
            HandleDelete -- "调用API服务" --> APIService
            APIService -- "调用 api.deleteCollection(name)" --> Backend
            Backend -- "成功时返回 { success: true }" --> APIService
            APIService -- "Promise<ApiResponse>" --> HandleDelete
            HandleDelete -- "成功后刷新数据" --> FetchStats
        end
    end

    %% Final Link to Backend
    APIService -- "通过 Axios 发送 HTTP 请求" --> Backend
