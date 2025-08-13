## **合同分析系统前端迁移至 React 的详细 Prompts 序列**

本序列旨在指导将 `contract-analysis-system\contract-migration\templates\index.html`, `contract-analysis-system\contract-migration\templates\results.html`, `contract-analysis-system\contract-migration\templates\result-detail.html` 迁移至 React 组件，并打通前后端正常显示。

### **阶段 A: 核心服务层与通用组件**

**Prompt A.1: 定义通用类型与 API 接口**

**目标**: 统一管理前端与后端交互的数据结构，并为所有分析相关的 API 定义接口。

**Prompt**:
"我们开始合同分析系统前端的 React 迁移。
**任务**:
1.  **定义通用 TypeScript 类型**: 在 `contract-analysis-frontend/src/services/types.ts` 中，添加以下接口，这些类型将用于 `index`, `results`, 和 `result-detail` 页面：
    *   `AnalysisResult`: 包含 `analysis_response` (分析结果详情，如分类、准则、证据等) 和 `uploaded_files` (上传文件信息) 等字段。
    *   `AnalysisRecordDetails`: 包含 `id`, `filename`, `created_at`, `result_overview` 等字段，用于历史记录列表和详情。
    *   `PaginatedResults<T>`: 一个泛型接口，用于表示分页数据，包含 `results: T[]`, `page`, `per_page`, `total`, `total_pages` 等字段。
    *   `CriteriaItem`: 包含 `name`, `question`, `met`, `confidence`, `explanation`, `evidences` 等字段。
    *   `EvidenceItem`: 包含 `evidence`, `sources` 等字段。
    *   `SourceDocument`: 包含 `hash`, `page`, `type`, `source` 等字段。
    *   `UploadedFile`: 包含 `original_name`, `document_hash`, `html_s3_key` 等字段。
    *   `ClassificationResult`: 包含 `final_classification`, `confidence`, `rules_applied`, `criteria` 等字段。
    *   `TimingInfo`: 包含 `cache_status`, `steps` 等字段。
    *   `LeaseTermConsistency`: 包含 `conflicts`, `sources` 等字段。
    *   `LeaseCriteria`: 包含 `lease_term_criteria`, `pv_ratio_criteria` 等字段。
    *   `AnalysisResponse`: 包含 `results`, `pdf_content_hash`, `excel_content_hash`, `pdf_original_filenames`, `excel_original_filenames`, `analysis_results`, `timing_info` 等字段。
2.  **定义 API 接口**: 在 `contract-analysis-frontend/src/services/api.ts` 中，添加以下 API 函数的接口定义，暂时可以返回 `any` 或 `Promise<any>`，后续会细化类型：
    *   `uploadAndAnalyze(formData: FormData): Promise<AnalysisResult>` (对应 `POST /analysis/analyze`)
    *   `getHistoricalResults(params: { page: number; per_page: number; filename?: string; date_from?: string; date_to?: string; }): Promise<PaginatedResults<AnalysisRecordDetails>>` (对应 `POST /api/results`)
    *   `getResultDetail(id: string): Promise<{ results: AnalysisResult; record_details: AnalysisRecordDetails }>` (对应 `GET /api/result?id={id}`)
3.  **创建模拟数据**: 在 `contract-analysis-frontend/src/services/mockData.ts` 中，为上述 API 函数提供逼真的模拟数据，以便在没有后端的情况下进行开发。确保模拟数据结构与定义的类型一致。

**注意**: 在完成此步骤后，请使用 Cline 原生的文件阅读编辑功能检查 `contract-analysis-frontend/src/services/types.ts`, `contract-analysis-frontend/src/services/api.ts`, 和 `contract-analysis-frontend/src/services/mockData.ts` 的内容，确保类型和接口定义正确，并且模拟数据能够匹配。

"

**Prompt A.2: 创建通用结果展示组件**

**目标**: 识别 `index.html` 和 `result-detail.html` 中重复的结果展示逻辑，并将其抽象为一个可复用的 React 组件。

**Prompt**:
"为了避免代码重复，我们将 `index.html` 和 `result-detail.html` 中展示分析结果的逻辑抽象为一个通用组件。
**任务**:
1.  **创建组件文件**: 在 `contract-analysis-frontend/src/components/` 目录下创建 `AnalysisResultDisplay.tsx` 和 `AnalysisResultDisplay.css`。
2.  **定义组件 Props**: `AnalysisResultDisplay.tsx` 组件应接受一个 `analysisData: AnalysisResult['analysis_response']` 和 `uploadedFiles: AnalysisResult['uploaded_files']` 作为 props。
3.  **迁移展示逻辑**: 将 `index.html` 中 `displayResults` 函数内部用于渲染 `Analysis Summary`, `Lease Classification Results`, `Buyback Classification Results`, `Dealer Classification Results`, 和 `Complex Deal Flagging Result` 的 HTML 结构和 JavaScript 逻辑迁移到 `AnalysisResultDisplay.tsx` 组件中。
    *   确保所有动态数据显示（如分类结果、置信度、准则评估、证据链接等）都通过 props 传入的数据进行渲染。
    *   **特别注意**: `navigateToViewer` 函数及其依赖的 `window.hashToFilename`, `window.hash_to_s3_key`, `window.highlight_doc_options` 等全局变量需要被妥善处理。建议将这些数据作为 props 传递给 `AnalysisResultDisplay`，或者在父组件中处理这些全局变量的初始化，并通过回调函数将 `navigateToViewer` 逻辑传递给子组件。
    *   保留 `feedbackModal` 相关的逻辑，但暂时不实现其功能，只保留按钮和事件绑定。
4.  **样式迁移**: 将 `index.html` 中与结果展示相关的 CSS 样式（如 `.criteria-item`, `.criteria-met`, `.criteria-not-met`, `.criteria-unclear`, `.page-number`, `.pdf-highlight` 等）迁移到 `AnalysisResultDisplay.css`。

**注意**: 在完成此步骤后，请使用 Cline 原生的文件阅读编辑功能检查 `contract-analysis-frontend/src/components/AnalysisResultDisplay.tsx` 和 `contract-analysis-frontend/src/components/AnalysisResultDisplay.css` 的内容，确保组件能够正确接收和渲染数据，并且样式已正确应用。

"

### **阶段 B: `index.html` (文件上传与分析) 迁移**

**Prompt B.1: 在 `DocumentUpload` 组件中实现 UI 布局**

**目标**: 基于 `index.html` 的布局，在现有的 `DocumentUpload.tsx` 组件中构建文件上传的用户界面。

**Prompt**:
"我们继续 `index.html` 的迁移。文件 `contract-analysis-frontend/src/components/DocumentUpload.tsx` 已经存在。
**任务**:
1.  **查阅并迁移 HTML 结构**: 打开 `contract-analysis-system/contract-migration/templates/index.html`，将其文件上传区域 (`.upload-area`) 的核心 HTML 结构迁移到 `contract-analysis-frontend/src/components/DocumentUpload.tsx` 组件的 JSX 中。
2.  **迁移基础样式**: 将 `index.html` 中与文件上传区域、按钮、加载动画等相关的 CSS 样式，合并到 `contract-analysis-frontend/src/components/DocumentUpload.css` 中。确保样式规则不与全局样式冲突。
3.  **确认组件渲染**: 在 `contract-analysis-frontend/src/App.tsx` 中，确认根路径 `/` 的 `<Route>` 正确渲染 `DocumentUpload` 组件。

**注意**: 此阶段只关注静态 UI 的搭建，暂时不需要实现任何交互逻辑。完成后，请检查页面是否能正确渲染出文件上传的表单和按钮。
"

**Prompt B.2: 实现文件上传与分析 API 交互**

**目标**: 为 `DocumentUpload` 组件添加状态管理和交互逻辑，实现文件选择、拖放，并调用后端 API 进行分析。

**Prompt**:
"现在为 `DocumentUpload` 组件添加交互功能。
**任务**:
1.  **查阅 API 和类型定义 (关键步骤)**: 首先，仔细阅读 `src/services/api.ts` 和 `src/services/types.ts`，以完全理解 `uploadAndAnalyze` 函数的签名、其期望的 `FormData` 结构，以及它返回的 `AnalysisResult` 类型的具体字段。这是实现后续逻辑的基础。
2.  **状态管理**: 在 `DocumentUpload.tsx` 中，使用 `useState` 管理以下状态：
    *   `costModelFile`, `leaseContractFiles`, `miscFiles`, `originalContractsFiles` (用于存储选中的文件对象)。
    *   `isLoading` (布尔值，表示分析是否正在进行)。
    *   `analysisResult` (用于存储 API 返回的 `AnalysisResult` 类型的数据)。
    *   `error` (用于存储错误信息)。
3.  **实现文件处理与表单提交**:
    *   实现处理文件选择、文件列表更新和文件拖放的函数。
    *   实现 `handleSubmit` 函数。在此函数中，构建一个 `FormData` 对象，将所有文件和 `analysis_type` (固定为 `['lease', 'buyback', 'dealer']`) 添加进去。
    *   调用在 `api.ts` 中定义的 `uploadAndAnalyze(formData)` 函数。
    *   在请求期间，设置 `isLoading`为 `true` 以显示加载指示器；请求结束后，根据结果更新 `analysisResult` 或 `error` 状态，并设置 `isLoading` 为 `false`。
4.  **UI 联动**: 根据 `isLoading` 状态，条件渲染加载动画 (`.spinner-border`)。

**注意**: 完成后，请检查文件选择、拖放功能是否正常，以及提交表单时是否能正确调用 API 并更新组件状态。
"

**Prompt B.3: 整合 `AnalysisResultDisplay` 组件以展示结果**

**目标**: 在 `DocumentUpload` 组件中，使用已有的 `AnalysisResultDisplay` 通用组件来显示分析结果。

**Prompt**:
"最后一步是将分析结果展示出来。我们将复用在 **阶段 A.2** 中创建的 `AnalysisResultDisplay` 组件。
**任务**:
1.  **导入并使用通用组件**: 在 `DocumentUpload.tsx` 中导入 `AnalysisResultDisplay` 组件。
2.  **条件渲染**: 当 `analysisResult` 状态中包含有效数据时，渲染 `<AnalysisResultDisplay />` 组件。
3.  **传递 Props**:
    *   回顾 `AnalysisResultDisplay` 组件所需的 props（`analysisData` 和 `uploadedFiles`）。
    *   将 `DocumentUpload` 组件 state 中的 `analysisResult.analysis_response` 和 `analysisResult.uploaded_files` 作为 props 传递给 `AnalysisResultDisplay`。
4.  **处理 `navigateToViewer` 逻辑**: 确保 `AnalysisResultDisplay` 内部的 `navigateToViewer` 函数能够正常工作。在 `DocumentUpload` 组件中，从 `analysisResult.uploaded_files` 初始化 `window.hashToFilename`, `window.hash_to_s3_key` 等全局变量，或将这些数据和导航逻辑作为 props 传递给 `AnalysisResultDisplay`。
5.  **清理旧结果**: 在每次发起新的分析请求前，确保清空上一次的 `analysisResult` 和 `error` 状态，以避免显示过时的信息。

**注意**: 完成后，请进行一次完整的文件上传和分析流程，确认分析结果能通过 `AnalysisResultDisplay` 组件正确地显示在页面上。
"

### **阶段 C: `results.html` (历史结果列表) 迁移**

**Prompt C.1: 创建 `HistoricalResults` 组件骨架与路由**

**目标**: 搭建 `results.html` 对应 React 组件的基础文件，并配置路由。

**Prompt**:
"现在我们开始迁移 `results.html`。
**任务**:
1.  **创建组件文件**: 在 `contract-analysis-frontend/src/components/` 目录下创建 `HistoricalResults.tsx` 和 `HistoricalResults.css` 文件。`HistoricalResults.tsx` 暂时只需包含一个返回 `<div>Historical Analysis Results</div>` 的基础函数组件。
2.  **配置路由**: 在 `contract-analysis-frontend/src/App.tsx` 中，为路径 `/results` 添加一条新的 `<Route>`，使其渲染 `HistoricalResults` 组件。
3.  **样式迁移**: 将 `results.html` 中与表格 (`.table-container`, `.table th`, `.table td`), 分页 (`.pagination`), 搜索表单等相关的 CSS 样式迁移到 `HistoricalResults.css`。

**注意**: 在完成此步骤后，请使用 Cline 原生的文件阅读编辑功能检查 `contract-analysis-frontend/src/components/HistoricalResults.tsx`, `contract-analysis-frontend/src/components/HistoricalResults.css`, 和 `contract-analysis-frontend/src/App.tsx` 的内容，确保组件骨架和路由配置正确。

"

**Prompt C.2: 实现历史结果列表 API 交互与搜索/分页**

**目标**: 实现 `HistoricalResults` 组件中的数据获取、搜索过滤和分页功能。

**Prompt**:
"我们来为 `HistoricalResults` 组件实现历史结果列表的 API 交互、搜索和分页功能。
**任务**:
1.  **查阅 API 和类型定义 (关键步骤)**: 首先，仔细阅读 `src/services/api.ts` 和 `src/services/types.ts`，以完全理解 `getHistoricalResults` 函数的签名、其参数，以及它返回的 `PaginatedResults<AnalysisRecordDetails>` 类型的具体字段。这是实现后续逻辑的基础。
2.  **状态管理**: 在 `HistoricalResults.tsx` 中，使用 `useState` 管理以下状态：
    *   `results` (存储 `PaginatedResults<AnalysisRecordDetails>` 类型的数据)。
    *   `currentPage`, `itemsPerPage` (用于分页)。
    *   `filenameSearch`, `dateFrom`, `dateTo` (用于搜索过滤)。
    *   `isLoading`, `error`。
3.  **数据获取函数**: 实现 `loadResults(page: number)` 异步函数：
    *   根据 `currentPage`, `itemsPerPage` 和搜索过滤条件 (`filenameSearch`, `dateFrom`, `dateTo`) 构建请求参数。
    *   调用 `api.getHistoricalResults()` 函数。
    *   处理 API 响应，更新 `results`, `isLoading`, `error` 状态。
4.  **分页逻辑**: 实现 `updatePagination(currentPage: number, totalPages: number)` 函数，用于动态生成分页按钮。
5.  **搜索表单**: 绑定搜索表单的 `submit` 事件和清除按钮的 `click` 事件，触发 `loadResults(1)`。
6.  **初始加载**: 使用 `useEffect` Hook 在组件首次加载时调用 `loadResults(1)`。
7.  **UI 渲染**:
    *   根据 `isLoading` 状态显示或隐藏加载动画。
    *   渲染表格骨架和分页容器，但暂时不填充数据。

**注意**: 在完成此步骤后，请使用 Cline 原生的文件阅读编辑功能检查 `contract-analysis-frontend/src/components/HistoricalResults.tsx` 的内容，确保数据获取、搜索和分页逻辑正确。

"

**Prompt C.3: 渲染结果列表**

**目标**: 在 `HistoricalResults` 组件中填充表格数据，并实现 "View Details" 按钮的导航功能。

**Prompt**:
"现在我们来渲染 `HistoricalResults` 组件中的表格数据，并实现导航到详情页的功能。
**任务**:
1.  **表格数据渲染**: 在 `HistoricalResults.tsx` 中，使用 `results.results` 数据填充表格的 `<tbody>`。
    *   确保 `filename`, `result_overview`, `created_at` 等字段正确显示。
    *   `result_overview` 可能包含换行符，需要处理为 `<br>`。
2.  **"View Details" 按钮**: 为每行添加一个 "View Details" 按钮，其 `onClick` 事件应使用 `react-router-dom` 的 `useNavigate` Hook 导航到 `/result-detail/:id` 路径，其中 `:id` 是当前行的 `item.id`。
3.  **分页按钮渲染**: 根据 `updatePagination` 函数的逻辑，动态渲染分页按钮，并确保点击时调用 `changePage` 函数。

**注意**: 在完成此步骤后，请使用 Cline 原生的文件阅读编辑功能检查 `contract-analysis-frontend/src/components/HistoricalResults.tsx` 的内容，确保表格数据和分页显示正确，并且 "View Details" 按钮能够正确导航。

"

### **阶段 D: `result-detail.html` (结果详情) 迁移**

**Prompt D.1: 创建 `ResultDetail` 组件骨架与路由**

**目标**: 搭建 `result-detail.html` 对应 React 组件的基础文件，并配置路由。

**Prompt**:
"现在我们开始迁移 `result-detail.html`。
**任务**:
1.  **创建组件文件**: 在 `contract-analysis-frontend/src/components/` 目录下创建 `ResultDetail.tsx` 和 `ResultDetail.css` 文件。`ResultDetail.tsx` 暂时只需包含一个返回 `<div>Analysis Result Detail</div>` 的基础函数组件。
2.  **配置路由**: 在 `contract-analysis-frontend/src/App.tsx` 中，为路径 `/result-detail/:id` 添加一条新的 `<Route>`，使其渲染 `ResultDetail` 组件。使用 `useParams` Hook 获取 `:id`。
3.  **样式迁移**: 将 `result-detail.html` 中与详情页布局、加载动画、PDF Viewer Modal 等相关的 CSS 样式迁移到 `ResultDetail.css`。

**注意**: 在完成此步骤后，请使用 Cline 原生的文件阅读编辑功能检查 `contract-analysis-frontend/src/components/ResultDetail.tsx`, `contract-analysis-frontend/src/components/ResultDetail.css`, 和 `contract-analysis-frontend/src/App.tsx` 的内容，确保组件骨架和路由配置正确。

"

**Prompt D.2: 实现结果详情 API 交互**

**目标**: 实现 `ResultDetail` 组件中的数据获取逻辑。

**Prompt**:
"我们来为 `ResultDetail` 组件实现结果详情的数据获取功能。
**任务**:
1.  **查阅 API 和类型定义 (关键步骤)**: 首先，仔细阅读 `src/services/api.ts` 和 `src/services/types.ts`，以完全理解 `getResultDetail` 函数的签名及其返回的数据结构。
2.  **状态管理**: 在 `ResultDetail.tsx` 中，使用 `useState` 管理以下状态：
    *   `analysisData` (存储 `AnalysisResult['analysis_response']` 类型的数据)。
    *   `uploadedFiles` (存储 `AnalysisResult['uploaded_files']` 类型的数据)。
    *   `recordDetails` (存储 `AnalysisRecordDetails` 类型的数据)。
    *   `isLoading`, `error`。
3.  **数据获取**: 使用 `useEffect` Hook 在组件首次加载时，通过 `useParams` 获取 `id`，然后调用 `api.getResultDetail(id)` 函数。
    *   处理 API 响应，更新 `analysisData`, `uploadedFiles`, `recordDetails`, `isLoading`, `error` 状态。
4.  **UI 渲染**:
    *   根据 `isLoading` 状态显示或隐藏加载动画。
    *   渲染 `Record Details` 卡片，显示 `filename` 和 `created_at`。
    *   暂时不渲染详细分析结果。

**注意**: 在完成此步骤后，请使用 Cline 原生的文件阅读编辑功能检查 `contract-analysis-frontend/src/components/ResultDetail.tsx` 的内容，确保数据获取和基本详情显示正确。

"

**Prompt D.3: 整合通用结果展示**

**目标**: 在 `ResultDetail` 组件中整合 `AnalysisResultDisplay` 组件，以显示详细分析结果。

**Prompt**:
"现在我们将 `AnalysisResultDisplay` 组件整合到 `ResultDetail` 中，以显示详细分析结果。
**任务**:
1.  **导入并使用 `AnalysisResultDisplay`**: 在 `ResultDetail.tsx` 中导入 `AnalysisResultDisplay` 组件。
2.  **条件渲染**: 当 `analysisData` 和 `uploadedFiles` 状态有数据时，条件渲染 `AnalysisResultDisplay` 组件，并将它们作为 props 传递。
3.  **处理 `navigateToViewer`**: 确保 `AnalysisResultDisplay` 中 `navigateToViewer` 函数所需的 `window.hashToFilename`, `window.hash_to_s3_key`, `window.highlight_doc_options` 等数据在 `ResultDetail` 组件中被正确初始化，并作为 props 传递给 `AnalysisResultDisplay`，或者通过上下文（Context API）提供。
4.  **PDF Viewer Modal**: 迁移 `result-detail.html` 中的 PDF Viewer Modal 逻辑和 UI 到 `ResultDetail.tsx`。这可能需要一个独立的 `PdfViewerModal` 组件，并处理 PDF.js 的集成。确保 `navigateToViewer` 函数能够正确触发此模态框。

**注意**: 在完成此步骤后，请使用 Cline 原生的文件阅读编辑功能检查 `contract-analysis-frontend/src/components/ResultDetail.tsx` 的内容，确保详细分析结果和 PDF Viewer 能够正确显示和工作。

"

### **阶段 E: 质量保证与文档**

**Prompt E.1: 编写单元与集成测试**

**目标**: 为所有新迁移的组件编写测试，确保其健壮性和可维护性。

**Prompt**:
"为保证迁移质量，我们需要为新迁移的组件编写全面的测试。
**任务**:
1.  **设置测试文件**: 在每个组件旁边创建测试文件，例如 `DocumentUpload.test.tsx`, `HistoricalResults.test.tsx`, `ResultDetail.test.tsx`, `AnalysisResultDisplay.test.tsx`。
2.  **模拟 API**: 使用 Mock Service Worker (MSW) 或 `jest.mock` 来拦截 `api.ts` 中的所有函数调用，并提供可控的成功/失败响应。
3.  **编写测试用例**:
    *   **初始渲染**: 测试组件在初始加载时是否正确显示了加载状态。
    *   **成功渲染**: 测试当 API 成功返回数据时，组件是否被正确渲染，包括文件列表、分析结果、历史列表、分页等。
    *   **错误处理**: 测试当 API 返回错误时，是否显示了对应的错误消息。
    *   **用户交互**:
        *   模拟文件选择和拖放，验证文件状态是否更新。
        *   模拟表单提交，验证 `uploadAndAnalyze` API 是否被调用。
        *   模拟搜索过滤和分页点击，验证 `getHistoricalResults` API 是否被调用，并且列表是否刷新。
        *   模拟 "View Details" 按钮点击，验证是否正确导航到详情页。
        *   模拟证据链接点击，验证 PDF Viewer Modal 是否出现并尝试加载文档。

**注意**: 在完成此步骤后，请使用 Cline 原生的文件阅读编辑功能检查所有测试文件，确保测试用例覆盖了关键功能。

"

**Prompt E.2: 更新项目文档**

**目标**: 在项目 `README.md` 或专门的文档中，记录新组件的设计和用法。

**Prompt**:
"最后，请更新项目文档。
**任务**:
1.  **更新 `README.md`**: 在 `contract-analysis-frontend/README.md` 的组件概览部分，添加对 `DocumentUpload`, `HistoricalResults`, `ResultDetail`, 和 `AnalysisResultDisplay` 组件的描述。
2.  **说明其功能**: 简要说明每个组件的作用、其在应用中的位置 (路由) 以及其包含的主要功能。
3.  **API 复用说明**: 强调 `AnalysisResultDisplay` 组件的复用性，以及后端 API 在不同页面间的复用模式。
4.  **环境变量**: 如果这些组件依赖任何新的环境变量，请在 `contract-analysis-frontend/env.example` 文件中添加并说明。

**注意**: 在完成此步骤后，请使用 Cline 原生的文件阅读编辑功能检查 `contract-analysis-frontend/README.md` 和 `contract-analysis-frontend/env.example` 的内容，确保文档清晰准确。

"
