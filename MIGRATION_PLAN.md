# 旧版 HTML 模板移植到新版 React 前端的工作指引

本指南旨在为将 `contract-analysis-system\contract-migration\templates` 目录下的旧版 HTML 模板移植到新的 `contract-analysis-frontend/` React 应用提供一份工作指引。由于无法直接访问旧版 HTML 文件的具体内容，本指南将侧重于通用迁移步骤和注意事项。

**新版前端技术栈概览：**
*   **框架**: React 19
*   **语言**: TypeScript
*   **UI 框架**: Bootstrap 5, Bootstrap Icons, React-Bootstrap
*   **HTTP 客户端**: Axios
*   **路由**: React Router
*   **构建工具**: Create React App (通过 `react-scripts`)

## 1. 理解旧版 HTML 模板的功能和数据流

在开始移植之前，需要深入理解每个旧版 HTML 模板的功能和其背后依赖的数据。

**旧版 HTML 模板（推测功能）：**
*   `admin_cache.html`: 可能用于管理后端缓存或配置的界面。
*   `highlight_llm_evidence_excel.html`: 可能用于在 Excel 转换的 HTML 中高亮显示 LLM 证据。
*   `highlight_llm_evidence.html`: 可能用于在通用 HTML 文档中高亮显示 LLM 证据。
*   `highlight.html`: 通用高亮显示功能页面。
*   `index.html`: 旧版应用的主入口页面。
*   `kb_documents.html`: 可能用于显示知识库文档列表或详情。
*   `result-detail.html`: 显示单个分析结果的详细信息。
*   `results.html`: 显示分析结果列表。

**关键问题（需要通过后端代码或旧版文档确认）：**
*   每个 HTML 页面展示了哪些数据？
*   这些数据是如何获取的（例如，通过后端模板引擎直接渲染，还是通过内联 JavaScript 发送 AJAX 请求）？
*   页面中是否存在任何表单提交、用户交互逻辑？这些逻辑是如何实现的（例如，纯 JavaScript，jQuery，或其他库）？
*   页面中是否包含任何后端模板语言（如 Jinja2、Flask 的 `{{ }}` 语法）？

## 2. 核心移植工作

### 2.1 UI 组件化 (Componentization)

React 的核心是组件化。每个旧版 HTML 页面或其逻辑块都应该被拆分成独立的、可复用的 React 组件。

*   **创建新组件**: 在 `src/components/` 目录下为每个旧版页面或其主要功能创建新的 TypeScript (TSX) 组件文件，例如：
    *   `src/components/AdminCache.tsx`
    *   `src/components/AnalysisResultsList.tsx` (对应 `results.html`)
    *   `src/components/AnalysisResultDetail.tsx` (对应 `result-detail.html`)
    *   `src/components/DocumentHighlighter.tsx` (整合 `highlight*.html` 的功能)
    *   `src/components/KnowledgeBaseDocuments.tsx` (对应 `kb_documents.html`)
*   **拆分职责**: 将每个组件的 UI、逻辑和数据处理分离。
*   **使用 React-Bootstrap**: 新版前端使用了 `react-bootstrap`，应优先使用其提供的组件（如 `Table`, `Modal`, `Form`, `Button` 等）来构建 UI，而不是直接使用原生 HTML 标签和 Bootstrap CSS 类。

### 2.2 数据流改造 (Data Flow Transformation)

旧版 HTML 可能是服务器端渲染，数据直接嵌入页面。新版 React 应用是客户端渲染，需要通过 API 获取数据。

*   **定义 API 接口**: 根据旧版页面所需的数据，在 `src/services/api.ts` 中定义新的或适配现有的 API 调用函数。例如，如果 `results.html` 显示分析结果列表，你需要一个 `getAnalysisResults()` 函数。
*   **使用 Axios**: 在 React 组件中使用 Axios 来调用后端 API 获取数据。
*   **状态管理**: 使用 React 的 `useState` 和 `useEffect` Hooks 来管理组件内部的数据状态和生命周期。对于更复杂的数据流，可能需要考虑 Redux 或 React Context API。
*   **数据转换**: 后端返回的数据格式可能与前端组件所需的数据格式不完全匹配，需要进行适当的转换。

### 2.3 路由适配 (Routing Adaptation)

旧版 HTML 页面通过 URL 直接访问，新版 React 应用使用客户端路由。

*   **配置 React Router**: 在 `src/App.tsx` 或单独的路由配置文件中，使用 `react-router-dom` 配置新的路由路径，将旧版页面的功能映射到新的 React 组件。
    *   例如，如果旧版 `/results.html` 对应新版 `/results`，则配置 `<Route path="/results" element={<AnalysisResultsList />} />`。
*   **导航更新**: 更新应用内部的所有导航链接，使其指向新的 React Router 路径。

### 2.4 样式和布局 (Styling and Layout)

新版前端使用了 Bootstrap 5 和自定义 CSS。

*   **复用 CSS**: 检查旧版 HTML 中使用的 CSS 样式。如果它们是独立的 CSS 文件，可以尝试将其移植到 `src/App.css` 或为每个组件创建独立的 `.css` 文件（例如 `src/components/AnalysisResults.css`）。
*   **适配 Bootstrap 5**: 确保所有样式与 Bootstrap 5 兼容。可能需要调整一些类名或重写部分 CSS。
*   **响应式设计**: 确保新组件在不同屏幕尺寸下都能良好显示。

### 2.5 交互逻辑 (Interactive Logic)

旧版 HTML 中的 JavaScript 交互逻辑需要用 React 的方式重写。

*   **事件处理**: 将内联事件处理（如 `onclick`）转换为 React 的事件处理函数。
*   **DOM 操作**: 避免直接操作 DOM。所有 UI 更新都应通过 React 的状态管理来完成。
*   **表单处理**: 使用 React 的受控组件或非受控组件模式来处理表单输入。

## 3. 特定功能移植考量

### 3.1 文档高亮显示 (Document Highlighting)

旧版有多个 `highlight*.html` 文件，这表明高亮显示是核心功能。

*   **集成查看器**: 新版前端的 `DocumentViewer.tsx` 已经具备 HTML 文档查看功能。需要将旧版的高亮逻辑（例如，根据 LLM 证据在文档中插入高亮标签）集成到 `DocumentViewer` 或其辅助函数中。
*   **数据格式**: 明确后端如何提供高亮所需的数据（例如，高亮文本的起始/结束位置，或直接提供带有高亮标签的 HTML 片段）。前端需要解析这些数据并在 `DocumentViewer` 中渲染。

### 3.2 文件上传 (File Upload)

新版前端有 `DocumentUpload.tsx`。

*   **适配旧版上传流程**: 如果旧版有特定的文件上传逻辑或文件类型限制，需要确保 `DocumentUpload.tsx` 能够适配或扩展以支持这些功能。
*   **后端接口**: 确认后端文件上传接口是否与新前端的 `api.ts` 定义一致。

### 3.3 管理界面 (Admin Interface)

`admin_cache.html` 可能涉及后端管理操作。

*   **权限控制**: 确保新版管理界面有适当的权限验证。
*   **API 调用**: 为管理操作（如清除缓存、更新配置）定义相应的 API 端点，并在 React 组件中调用。

## 4. 开发与测试

*   **本地开发**: 利用 `npm start` 启动开发服务器，实时查看更改。
*   **模拟数据**: 新版前端支持模拟数据 (`src/services/mockData.ts`)，这对于在后端 API 未完全就绪时进行前端开发非常有用。可以根据旧版数据结构创建模拟数据。
*   **单元测试**: 为新创建的 React 组件编写单元测试 (`.test.tsx` 文件)，确保其功能正确。
*   **集成测试**: 验证不同组件之间以及与后端 API 的集成是否正常。
*   **端到端测试**: 确保整个用户流程（从上传到查看结果）在新版前端中正常工作。

## 5. 部署

*   **构建**: 使用 `npm run build` 生成生产环境的优化代码。
*   **Nginx 配置**: 检查 `nginx.conf` 文件，确保它正确地服务 React 应用的静态文件，并处理 React Router 的路由（`try_files $uri $uri/ /index.html;`）。如果旧版有特定的 Nginx 配置，可能需要合并或调整。
*   **Docker**: 新版前端提供了 `Dockerfile`，可以用于容器化部署。

## 总结

将旧版 HTML 模板移植到 React 前端是一个将服务器端渲染应用转换为客户端单页应用的过程。核心工作包括将 UI 拆分为 React 组件、重构数据获取逻辑为 API 调用、适配路由以及迁移或重写样式和交互。在整个过程中，与后端团队的紧密协作以确保 API 兼容性至关重要。

---

# 后续 Vibe Coding 的 SOTA Prompts 序列

以下是一组为旧版 HTML 模板到新版 React 前端迁移工作设计的、达到 SOTA 水平的可靠 prompts 序列。这些 prompts 旨在引导 AI 在 Vibe Coding 过程中逐步完成任务，确保代码质量、最佳实践和功能完整性。

**通用 Vibe Coding 指导原则：**

*   **逐步实现**: 每次只关注一个组件或一个功能点。
*   **模块化**: 确保组件职责单一，可复用性强。
*   **类型安全**: 严格使用 TypeScript 定义接口和类型。
*   **性能优化**: 考虑使用 `React.memo`, `useCallback`, `useMemo` 等优化手段。
*   **错误处理**: 妥善处理 API 请求错误和组件内部错误。
*   **可访问性**: 遵循 Web 内容可访问性指南 (WCAG)。
*   **代码注释**: 为复杂逻辑和关键部分添加清晰的注释。
*   **测试驱动**: 优先考虑编写单元测试。

---

## Prompts 序列

### 阶段 1: 准备与基础架构

**Prompt 1.1: 初始化项目结构与依赖确认**

**目标**: 验证项目环境，创建新组件的占位符文件，并配置基础路由。

**Prompt**:
"作为一名负责迁移旧版 HTML 应用的资深前端工程师，请开始我们的迁移工作。
**任务**:
1.  **审查 `package.json`**: 确认 `react-bootstrap`, `bootstrap`, `axios`, 和 `react-router-dom` 已被正确安装。
2.  **规划组件文件**: 根据 `MIGRATION_PLAN.md`，在 `contract-analysis-frontend/src/components/` 目录下创建以下新的组件（TSX）和对应的 CSS 模块文件。暂时只需包含基础的函数组件骨架。
    *   `AdminCache/`
        *   `AdminCache.tsx`
        *   `AdminCache.module.css`
    *   `AnalysisResultDetail/`
        *   `AnalysisResultDetail.tsx`
        *   `AnalysisResultDetail.module.css`
    *   `KnowledgeBase/`
        *   `KnowledgeBase.tsx`
        *   `KnowledgeBase.module.css`
    *   `common/` (用于存放共享组件)
        *   `Layout.tsx`
        *   `LoadingSpinner.tsx`
        *   `ErrorMessage.tsx`
3.  **更新主路由**: 修改 `src/App.tsx`，引入 `react-router-dom` 的 `BrowserRouter`, `Routes`, 和 `Route`。设置一个基本布局，并为以下路径配置路由，指向新创建的组件：
    *   `/` -> `DocumentUpload` (已存在)
    *   `/results` -> `AnalysisResults` (已存在)
    *   `/results/:id` -> `AnalysisResultDetail`
    *   `/kb` -> `KnowledgeBase`
    *   `/admin` -> `AdminCache`
    *   `/health` -> `HealthCheck` (已存在)
4.  **创建共享布局**: 实现 `Layout.tsx` 组件，包含一个通用的导航栏（Navbar），导航到上述主要页面。在 `App.tsx` 中使用此布局包裹所有路由。"

---

### 阶段 2: 核心功能迁移

**Prompt 2.1: 迁移 `results.html` -> `AnalysisResults.tsx` (数据获取与列表展示)**

**目标**: 实现分析结果列表页面，包括从 API 获取数据、展示加载状态和错误信息，并以表格形式呈现数据。

**Prompt**:
"我们现在开始迁移第一个核心页面：分析结果列表。
**上下文**:
*   旧页面: `results.html`
*   新组件: `contract-analysis-frontend/src/components/AnalysisResults.tsx`
*   后端将提供一个 API 端点 `/api/results`，返回 `AnalysisResult[]` 数组。
**任务**:
1.  **定义数据类型**: 在 `src/types/index.ts` (如果不存在则创建) 中定义 `AnalysisResult` 接口，应至少包含 `id`, `documentName`, `status`, `analysisDate`, `summary` 字段。
2.  **适配 API 服务**: 在 `src/services/api.ts` 中，实现 `getAnalysisResults(): Promise<AnalysisResult[]>` 函数。暂时可以使用 `src/services/mockData.ts` 中的模拟数据进行开发。
3.  **实现组件逻辑**: 在 `AnalysisResults.tsx` 中：
    *   使用 `useState` 管理 `results`, `isLoading`, 和 `error` 状态。
    *   使用 `useEffect` Hook 在组件挂载时调用 `getAnalysisResults`，并更新状态。
    *   在 `isLoading` 为 `true` 时，显示 `LoadingSpinner` 组件。
    *   如果请求出错，显示 `ErrorMessage` 组件。
4.  **渲染 UI**:
    *   使用 `react-bootstrap` 的 `Table` 组件来展示结果列表。表头应包括文档名称、状态、分析日期和摘要。
    *   每一行应可点击，使用 `react-router-dom` 的 `Link` 组件导航到对应的详情页，路径为 `/results/{id}`。
    *   应用 `AnalysisResults.module.css` 中的样式，确保表格具有良好的可读性和视觉效果。"

**Prompt 2.2: 迁移 `result-detail.html` -> `AnalysisResultDetail.tsx` (动态路由与复杂视图)**

**目标**: 实现分析结果详情页，通过动态路由参数获取特定结果的数据，并集成文档查看器。

**Prompt**:
"接下来，我们处理结果详情页。
**上下文**:
*   旧页面: `result-detail.html`
*   新组件: `contract-analysis-frontend/src/components/AnalysisResultDetail/AnalysisResultDetail.tsx`
*   此页面通过动态路由 `/results/:id` 访问。
**任务**:
1.  **获取动态参数**: 使用 `react-router-dom` 的 `useParams` Hook 来获取 URL 中的 `id`。
2.  **定义数据类型**: 在 `src/types/index.ts` 中定义 `AnalysisResultDetail` 接口，它应继承 `AnalysisResult` 并额外包含 `htmlContent` 和 `highlightData` 字段。
3.  **适配 API 服务**: 在 `src/services/api.ts` 中，实现 `getAnalysisResultById(id: string): Promise<AnalysisResultDetail>` 函数。
4.  **实现组件逻辑**:
    *   与列表页类似，管理 `result`, `isLoading`, `error` 状态。
    *   使用 `useEffect` Hook，在 `id` 变化时调用 `getAnalysisResultById`。
5.  **渲染 UI**:
    *   页面顶部显示结果的元数据（如文档名、状态等）。
    *   集成现有的 `DocumentViewer.tsx` 组件，将获取到的 `htmlContent` 和 `highlightData` 作为 props 传递给它。
    *   确保在数据加载完成前，`DocumentViewer` 不会渲染或显示加载状态。"

**Prompt 2.3: 实现文档高亮核心功能**

**目标**: 根据后端提供的高亮数据，在 `DocumentViewer` 中高亮显示文档内容。

**Prompt**:
"现在，我们来攻克核心的高亮功能。
**上下文**:
*   相关组件: `contract-analysis-frontend/src/components/DocumentViewer.tsx`
*   后端返回的 `highlightData` 可能是两种格式之一：
    1.  一个包含 `startOffset`, `endOffset`, `text`, `className` 的对象数组。
    2.  一段已经预先处理好、包含 `<mark>` 等高亮标签的 HTML 字符串。
**任务**:
1.  **创建高亮工具函数**: 在 `src/utils/highlight.ts` (如果不存在则创建) 中，创建一个名为 `applyHighlights` 的函数。
2.  **实现高亮逻辑**:
    *   `applyHighlights` 函数接收 `htmlContent: string` 和 `highlightData: any` 作为参数。
    *   它需要能够处理上述两种可能的数据格式，并返回一个带有高亮效果的完整 HTML 字符串。
    *   对于偏移量格式，你需要精确地在原始 HTML 中找到位置并插入高亮标签。这可能需要一个健壮的 HTML 解析器或巧妙的字符串操作。
3.  **集成到 `DocumentViewer`**:
    *   修改 `DocumentViewer.tsx`，使其接收 `htmlContent` 和 `highlightData`。
    *   在组件内部调用 `applyHighlights` 来生成最终要渲染的 HTML。
    *   使用 `dangerouslySetInnerHTML` 将处理后的 HTML 渲染到视图中。请在代码注释中解释为什么在这里使用是安全的（例如，我们信任后端返回的内容，或者我们已经进行了清理）。
4.  **添加样式**: 在 `DocumentViewer.module.css` 中为不同的高亮类别（`className`）定义不同的背景颜色和样式。"

---

### 阶段 3: 其他页面与收尾

**Prompt 3.1: 迁移 `admin_cache.html` -> `AdminCache.tsx` (表单与操作)**

**目标**: 创建一个管理界面，允许用户执行如“清除缓存”等操作。

**Prompt**:
"现在我们来构建管理界面。
**上下文**:
*   旧页面: `admin_cache.html`
*   新组件: `contract-analysis-frontend/src/components/AdminCache/AdminCache.tsx`
*   后端提供 `/api/admin/clear-cache` (POST) 端点。
**任务**:
1.  **构建 UI**: 使用 `react-bootstrap` 的 `Card`, `Button`, 和 `Alert` 组件创建一个简单的界面。
2.  **实现操作逻辑**:
    *   当用户点击“清除缓存”按钮时，调用 `api.ts` 中对应的 `clearCache()` 函数。
    *   在请求期间禁用按钮并显示加载指示。
    *   请求成功后，使用 `Alert` 组件显示成功消息。
    *   如果请求失败，显示错误消息。
3.  **权限考虑**: 虽然前端不做硬性权限检查，但在组件文档中注明此组件应由路由守卫或后端进行权限控制。"

**Prompt 3.2: 编写单元测试**

**目标**: 为关键组件和工具函数编写单元测试，确保其可靠性。

**Prompt**:
"为了保证代码质量，我们需要为已完成的功能编写单元测试。
**任务**:
1.  **测试高亮工具**: 在 `src/utils/highlight.test.ts` 中，为 `applyHighlights` 函数编写测试用例。覆盖两种数据格式，以及边缘情况（如空数据、无效 HTML）。
2.  **测试 `AnalysisResults` 组件**: 在 `AnalysisResults.test.tsx` 中：
    *   使用 Mock Service Worker (MSW) 或 `jest.mock` 来模拟 `api.ts` 的 `getAnalysisResults` 函数。
    *   测试组件在加载、成功和错误三种状态下的渲染情况。
    *   使用 `@testing-library/react` 验证表格是否正确渲染了模拟数据。
3.  **测试 `AnalysisResultDetail` 组件**: 类似地，测试详情页组件，确保它能正确解析 URL 参数并发起相应的 API 请求。"

**Prompt 3.3: 最终审查与文档更新**

**目标**: 对整个迁移工作进行代码审查，并更新项目文档。

**Prompt**:
"迁移工作已接近尾声。请进行最后的审查和整理。
**任务**:
1.  **代码审查**: 检查所有新代码，确保遵循了以下原则：
    *   一致的代码风格和命名约定。
    *   没有遗留的 `console.log` 或未使用的变量/导入。
    *   所有组件和函数都有适当的 JSDoc 注释。
    *   CSS 类名遵循 BEM 或 CSS Modules 的最佳实践。
2.  **更新 `README.md`**:
    *   更新项目的 `README.md` 文件，反映新的前端架构。
    *   添加关于如何运行和测试新组件的说明。
    *   描述项目的主要组件及其职责。
3.  **准备部署**: 确认 `Dockerfile` 和 `nginx.conf` 的配置与 React 单页应用兼容，特别是 Nginx 的 `try_files` 指令，以正确处理客户端路由。"

---

### 阶段 4: 高级功能与健壮性

**Prompt 4.1: 迁移 `kb_documents.html` -> `KnowledgeBase.tsx` (搜索与分页)**

**目标**: 实现知识库页面，支持文档搜索和分页功能。

**Prompt**:
"现在我们来处理知识库页面，这需要更复杂的交互。
**上下文**:
*   旧页面: `kb_documents.html`
*   新组件: `contract-analysis-frontend/src/components/KnowledgeBase/KnowledgeBase.tsx`
*   后端 API: `/api/kb/documents`，支持查询参数 `?search=...` 和 `?page=...`。
**任务**:
1.  **定义数据类型**: 在 `src/types/index.ts` 中定义 `KBDocument` 和 `Pagination` 类型。
2.  **适配 API 服务**: 在 `api.ts` 中实现 `getKBDocuments(params: { search?: string; page?: number }): Promise<{ documents: KBDocument[], pagination: Pagination }>`。
3.  **实现组件逻辑**:
    *   使用 `useState` 管理搜索词、当前页码、文档列表、分页信息、加载和错误状态。
    *   使用 `useDebounce` 自定义 Hook (或第三方库) 来防止用户输入搜索词时过于频繁地调用 API。
    *   当搜索词或页码变化时，使用 `useEffect` 调用 API 获取数据。
4.  **渲染 UI**:
    *   使用 `react-bootstrap` 的 `Form.Control` 作为搜索输入框。
    *   使用 `ListGroup` 或 `Table` 展示文档列表。
    *   使用 `Pagination` 组件来处理分页导航。
    *   确保在加载新数据时提供清晰的用户反馈。"

**Prompt 4.2: 增强 `DocumentUpload.tsx` (文件校验与进度)**

**目标**: 改进现有的文件上传组件，增加前端文件类型校验和上传进度显示。

**Prompt**:
"为了提升用户体验和系统稳定性，我们需要增强文件上传功能。
**上下文**:
*   相关组件: `contract-analysis-frontend/src/components/DocumentUpload.tsx`
*   旧系统可能对文件类型有特定要求（如仅限 `.pdf`, `.docx`, `.xlsx`）。
**任务**:
1.  **前端文件校验**:
    *   在用户选择文件后，但在上传前，检查文件扩展名和 MIME 类型。
    *   如果文件类型不符合要求，使用 `Alert` 组件向用户显示明确的错误消息，并阻止上传。
2.  **实现上传进度**:
    *   修改 `api.ts` 中的文件上传函数，利用 Axios 的 `onUploadProgress` 配置项来跟踪上传进度。
    *   `onUploadProgress` 回调函数应更新组件中的一个状态（如 `uploadProgress`）。
3.  **渲染进度 UI**:
    *   在文件上传期间，向用户显示 `react-bootstrap` 的 `ProgressBar` 组件。
    *   根据 `uploadProgress` 状态动态更新进度条。
    *   上传完成后，隐藏进度条并显示成功或失败的消息。"

**Prompt 4.3: 引入全局应用上下文 (Global Context)**

**目标**: 创建一个全局的 React Context，用于管理应用级别状态，例如用户信息或全局通知。

**Prompt**:
"为了更好地管理跨组件共享的状态，我们将引入 React Context API。
**任务**:
1.  **创建 Context**: 在 `src/context/AppContext.tsx` 中：
    *   定义 `AppContext`，并为其创建一个 `AppContextProvider` 组件。
    *   在 Context 中管理一个状态对象，至少包含 `userInfo` 和一个显示/隐藏全局通知的函数。
2.  **包裹应用**: 在 `src/App.tsx` 中，使用 `AppContextProvider` 包裹整个应用，以便所有子组件都能访问全局状态。
3.  **创建自定义 Hook**: 创建一个 `useAppContext` 自定义 Hook，简化在组件中访问 Context 的过程。
4.  **使用 Context**:
    *   改造 `Layout.tsx` 中的导航栏，如果 `userInfo` 存在，则显示用户名。
    *   改造 `AdminCache.tsx`，使其在操作成功或失败时，调用 Context 中的通知函数来显示一个全局的、会自动消失的 `Toast` 通知，而不是在组件内部显示 `Alert`。"

**Prompt 4.4: 引入端到端 (E2E) 测试**

**目标**: 为核心用户流程设置并编写第一个端到端测试，确保应用整体功能的稳定性。

**Prompt**:
"为了确保我们的应用在真实浏览器环境中按预期工作，我们将引入端到端测试。
**任务**:
1.  **选择并安装框架**: 安装 [Cypress](https://www.cypress.io/) 或 [Playwright](https://playwright.dev/) 作为 E2E 测试框架。请在 `package.json` 中添加相应的 `scripts` 命令来运行测试。
2.  **配置测试环境**: 根据所选框架的文档进行初始化配置。
3.  **编写第一个测试用例**:
    *   创建一个测试文件，覆盖核心用户流程：“上传文档 -> 在结果列表中看到该文档 -> 点击进入详情页 -> 验证详情页内容”。
    *   使用 `data-testid` 属性来选择 DOM 元素，使测试更健壮，不易因样式或结构变化而失败。
    *   在 `DocumentUpload.tsx`, `AnalysisResults.tsx`, 和 `AnalysisResultDetail.tsx` 中添加必要的 `data-testid` 属性。
4.  **编写文档**: 在 `README.md` 中添加一节，说明如何运行 E2E 测试。"

---

### 阶段 5: 生产准备与持续集成 (CI/CD)

**Prompt 5.1: 集成测试到 CI/CD 流水线**

**目标**: 将单元测试和端到端测试集成到 GitLab CI/CD 流程中，实现自动化质量保证。

**Prompt**:
"为了实现自动化并保障代码质量，我们需要将测试流程集成到 CI/CD 中。
**上下文**:
*   配置文件: `.gitlab-ci.yml`
*   已编写的测试: 单元测试 (Jest) 和 E2E 测试 (Cypress/Playwright)。
**任务**:
1.  **修改 `.gitlab-ci.yml`**:
    *   在现有的 CI 配置中，添加一个新的 `test` 阶段。
    *   **创建 `unit-test` 作业**:
        *   该作业应运行 `npm test` (或 `npm run test:ci`，如果需要无交互模式)。
        *   确保它能生成测试覆盖率报告，并将其作为产物 (artifact) 保存。
    *   **创建 `e2e-test` 作业**:
        *   该作业需要一个能够运行浏览器的环境。你可能需要使用一个包含浏览器的 Docker 镜像 (例如 `cypress/browsers` 或 `mcr.microsoft.com/playwright`)。
        *   作业需要先在后台启动应用 (`npm start`)，然后运行 E2E 测试命令 (`npm run cypress:run` 或 `npx playwright test`)。
        *   将 E2E 测试的视频和截图报告保存为产物，以便在测试失败时进行调试。
2.  **配置流水线规则**: 确保 `test` 阶段在每次向主分支 (main/master) 或合并请求 (Merge Request) 推送代码时都会自动触发。"

**Prompt 5.2: 应用性能分析与优化**

**目标**: 使用 React DevTools Profiler 对应用进行性能分析，并针对性地进行优化。

**Prompt**:
"现在应用功能已基本完成，我们需要关注其性能，确保流畅的用户体验。
**任务**:
1.  **安装 React DevTools**: 确保你的开发浏览器已安装 React DevTools 扩展。
2.  **性能剖析 (Profile)**:
    *   使用 Profiler 记录核心用户流程的交互，例如：
        *   加载分析结果列表 (`AnalysisResults.tsx`)。
        *   在知识库中进行搜索 (`KnowledgeBase.tsx`)。
        *   打开一个包含大量高亮数据的文档详情页 (`AnalysisResultDetail.tsx`)。
3.  **分析火焰图**:
    *   识别那些渲染时间过长或不必要地重复渲染的组件。
    *   检查是否存在由于 props 引用变化而导致的意外重渲染。
4.  **应用优化策略**:
    *   对于那些 props 不经常变化但父组件频繁重渲染的组件，使用 `React.memo` 进行包裹。
    *   对于在渲染中创建的函数，如果它们被作为 props 传递给子组件，使用 `useCallback` 进行记忆化。
    *   对于复杂的计算，使用 `useMemo` 来记忆化计算结果。
    *   **具体行动**: 请审查 `AnalysisResults.tsx` 和 `AnalysisResultDetail.tsx`，并应用至少一种上述优化策略，并在代码注释中解释为什么这样做是有效的。"

**Prompt 5.3: 可访问性 (A11y) 审计与修复**

**目标**: 确保应用符合基本的 Web 内容可访问性指南 (WCAG)，让所有用户都能无障碍使用。

**Prompt**:
"为了让我们的应用更具包容性，现在进行一次可访问性审计。
**任务**:
1.  **安装测试工具**: 在项目中添加 `axe-core` 和 `@axe-core/react` 依赖。
2.  **集成到开发环境**:
    *   修改 `src/index.tsx`，在开发模式下 (`process.env.NODE_ENV === 'development'`)，使用 `@axe-core/react` 在应用启动时运行可访问性检查，并将问题打印到控制台。
3.  **手动审计**:
    *   使用浏览器开发者工具中的 Lighthouse 或 aXe DevTools 扩展，对以下页面进行审计：
        *   主上传页面
        *   结果列表页
        *   详情页
4.  **修复常见问题**:
    *   **图像**: 确保所有 `<img>` 标签都有 `alt` 属性。
    *   **表单**: 确保所有表单输入 (`<input>`, `<select>`) 都有关联的 `<label>`。
    *   **颜色对比度**: 检查并修复文本与背景颜色对比度不足的问题。
    *   **键盘导航**: 尝试只使用键盘（Tab, Shift+Tab, Enter, Space）来操作整个应用，确保所有交互元素（按钮、链接、表单）都能被聚焦和操作。
    *   **ARIA 属性**: 在需要时，为动态组件（如 `Modal`, `Alert`）添加适当的 ARIA (Accessible Rich Internet Applications) 角色和属性。"

**Prompt 5.4: 状态管理策略复盘与文档化**

**目标**: 总结并文档化项目的状态管理策略，为未来的功能扩展提供指导。

**Prompt**:
"最后，我们来明确并记录下项目的状态管理方法论。
**任务**:
1.  **创建文档**: 在 `docs/` 目录下创建一个新的 Markdown 文件 `STATE_MANAGEMENT.md`。
2.  **定义策略**: 在该文件中，详细阐述我们遵循的状态管理分层策略：
    *   **本地状态 (Local State - `useState`, `useReducer`)**:
        *   **适用场景**: 仅单个组件需要的状态（例如，表单输入的值、一个 `Modal` 的开关状态）。
        *   **原则**: “状态应该尽可能地靠近使用它的地方。”
    *   **共享状态 (Shared State - React Context)**:
        *   **适用场景**: 需要在多个嵌套层级或兄弟组件间共享的、不频繁更新的状态（例如，当前登录用户信息、应用主题）。
        *   **我们项目中的实践**: `AppContext` 用于管理全局用户信息和通知。
    *   **服务端缓存/远程状态 (Server Cache / Remote State - React Query, SWR)**:
        *   **未来展望**: 虽然我们目前直接使用 `axios` 和 `useEffect`，但文档应指出，当应用变得更复杂，特别是需要处理缓存、重新验证、乐观更新等场景时，应考虑引入 `TanStack Query (React Query)` 或 `SWR`。这能极大简化数据获取逻辑。
    *   **复杂客户端状态 (Complex Client State - Zustand, Redux Toolkit)**:
    *   **未来展望**: 如果未来出现复杂的、频繁更新且跨多个组件共享的客户端状态（例如，一个复杂的、多步骤的表单向导），可以考虑引入一个轻量级的状态管理器如 `Zustand`。
3.  **提供决策树**: 在文档中包含一个简单的流程图或决策树，帮助开发者根据场景选择合适的状态管理工具。"

---
---


