## **附录: `admin_cache.html` 迁移至 React 的详细 Prompts 序列 (最小化实现)**

本序列旨在作为将 `admin_cache.html` 迁移至 `AdminCache.tsx` 组件的详细分步指南。

### **阶段 A: 基础架构与数据层搭建**

**Prompt A.1: 创建组件骨架、路由与核心类型定义**

**目标**: 搭建 `AdminCache` 组件的基础文件，配置路由，并根据 `admin_cache.html` 的数据结构预先定义好所有 TypeScript 类型。

**Prompt**:
"我们开始迁移 `admin_cache.html`。
**任务**:
1.  **创建组件文件**: 根据项目现有的扁平化组件结构，在 `src/components/` 目录下创建 `AdminCache.tsx` 和 `AdminCache.css` 文件。`AdminCache.tsx` 暂时只需包含一个返回 `<div>Admin Cache Page</div>` 的基础函数组件。
2.  **配置路由**: 在 `src/App.tsx` 中，为路径 `/admin` 添加一条新的 `<Route>`，使其渲染 `AdminCache` 组件。
3.  **定义 TypeScript 类型**: 为了保持代码的组织性和可维护性，将在 `src/services/` 目录下创建一个新的 `types.ts` 文件来存放与API服务层相关的类型定义。请在 `src/services/types.ts` 中添加以下接口：
    *   `CacheStats`: 包含 `total_entries`, `oldest_entry`, `newest_entry` 等字段。
    *   `PGVectorStats`: 包含 `total_collections`, `total_embeddings` 等字段。
    *   `PGVectorCollection`: 包含 `uuid`, `name`, `document_hash`, `embedding_count`, `created`, `last_modified` 等字段。
    *   `CollectionDetails`: 包含集合的详细信息及其中的 `embeddings` 列表。
    *   `ApiActionResponse`: 用于表示通用操作（如清除、删除）的响应，例如 `{ success: boolean; message: string; }`。"

**Prompt A.2: 构建 API 服务层**

**目标**: 在 `src/services/api.ts` 中创建与后端交互的所有函数，将数据获取逻辑与 UI 组件分离。

**Prompt**:
"现在，我们来构建与 `AdminCache` 组件相关的所有后端 API 调用。
**任务**:
1.  **分析 API 端点**: 根据 `admin_cache.html` 中的 `fetch` 调用，识别出后端端点为 `/admin/cache/stats` (GET) 和 `/admin/cache/actions` (POST)。
2.  **实现 API 函数**: 在 `src/services/api.ts` 中，为每个后端交互创建独立的、类型安全的函数。从 `./types.ts` 导入所需的类型。暂时使用 `mockData.ts` 返回模拟数据。
    *   `getCacheAndPgVectorStats(): Promise<{ cacheStats: CacheStats, pgVectorStats: PGVectorStats }>`
    *   `listPgVectorCollections(): Promise<PGVectorCollection[]>`
    *   `getCollectionDetails(collectionId: string): Promise<CollectionDetails>`
    *   `deletePgVectorCollection(collectionId: string): Promise<ApiActionResponse>`
    *   `clearPgVectorStore(): Promise<ApiActionResponse>`
    *   `cleanOldCacheEntries(maxAgeDays: number, maxEntries: number): Promise<ApiActionResponse>`
    *   `purgeAllCache(): Promise<ApiActionResponse>`
3.  **创建模拟数据**: 在 `src/services/mockData.ts` 中为上述函数提供逼真的模拟数据，以便在没有后端的情况下进行开发。"

---

### **阶段 B: UI 组件实现与交互**

**Prompt B.1: 渲染统计信息卡片**

**目标**: 实现 `AdminCache` 组件，获取并展示顶部的 "Cache Statistics" 和 "PGVector Store" 信息，并处理加载和错误状态。

**Prompt**:
"我们来渲染 `AdminCache` 页面的静态部分。
**任务**:
1.  **状态管理**: 在 `AdminCache.tsx` 中，使用 `useState` 管理 `stats`, `isLoading`, 和 `error` 状态。
2.  **数据获取**: 使用 `useEffect` Hook 在组件首次加载时调用 `api.getCacheAndPgVectorStats()`，并相应地更新状态。
3.  **UI 渲染**:
    *   使用 `react-bootstrap` 的 `Card`, `Row`, `Col` 组件来布局 "Cache Statistics" 和 "PGVector Store" 两个卡片区域。
    *   在 `isLoading` 为 `true` 时，显示通用的 `LoadingSpinner` 组件。
    *   如果请求失败，显示 `ErrorMessage` 组件。
    *   成功获取数据后，将 `stats` 对象中的数据显示在卡片中。"

**Prompt B.2: 实现 PGVector 集合列表**

**目标**: 获取并以表格形式展示 PGVector 集合列表，并实现刷新功能。

**Prompt**:
"接下来，实现页面中最重要的部分：PGVector 集合列表。
**任务**:
1.  **状态管理**: 为集合列表单独管理状态：`collections`, `isCollectionsLoading`, `collectionsError`。
2.  **数据获取与刷新**:
    *   创建一个 `fetchCollections` 函数，用于调用 `api.listPgVectorCollections()` 并更新状态。
    *   在 `useEffect` 中调用一次 `fetchCollections`。
    *   在 "PGVector Collections" 卡片的头部添加一个 "Refresh" `Button`，其 `onClick` 事件调用 `fetchCollections`。
3.  **UI 渲染**:
    *   使用 `react-bootstrap` 的 `Table` 组件来展示集合列表，表头和数据格式应与 `admin_cache.html` 中一致。
    *   在表格的 "Actions" 列中，为每一行添加 "View" 和 "Delete" `Button`。
    *   同样，处理好列表加载过程中的 `LoadingSpinner` 和错误状态的 `ErrorMessage`。"

**Prompt B.3: 实现删除操作与确认模态框**

**目标**: 实现集合的删除功能，并使用 `react-bootstrap` 的 `Modal` 组件进行操作确认，提升用户体验。

**Prompt**:
"现在让删除按钮工作起来。
**任务**:
1.  **状态管理**: 使用 `useState` 来管理删除模态框的显示状态 (`showDeleteModal`) 和待删除的集合 ID (`collectionToDeleteId`)。
2.  **触发模态框**: 当用户点击任一行的 "Delete" 按钮时，更新上述两个状态，以显示确认模态框并传入集合 ID。
3.  **实现模态框 UI**:
    *   使用 `react-bootstrap` 的 `Modal` 组件创建删除确认对话框。
    *   模态框应显示要删除的集合信息，并提供 "Confirm Delete" 和 "Cancel" 按钮。
4.  **执行删除**:
    *   当用户点击 "Confirm Delete" 按钮时，调用 `api.deletePgVectorCollection(collectionToDeleteId)`。
    *   在请求期间，可以禁用模态框中的按钮。
    *   请求完成后，关闭模态框，并调用 `fetchCollections` 刷新列表。
    *   使用全局通知（将在后续步骤中实现）或临时 `Alert` 显示操作结果。"

**Prompt B.4: 实现维护表单功能**

**目标**: 实现 "Cache Maintenance" 卡片中的两个表单："Clean Old Entries" 和 "Purge All Cache"。

**Prompt**:
"我们来完成最后的表单功能。
**任务**:
1.  **"Clean Old Entries" 表单**:
    *   使用 `useState` 管理 `maxAgeDays` 和 `maxEntries` 输入框的状态（受控组件）。
    *   创建一个 `handleCleanSubmit` 函数，在表单提交时调用 `api.cleanOldCacheEntries`。
    *   在请求期间禁用提交按钮。
2.  **"Purge All Cache" 表单**:
    *   创建一个 `handlePurgeSubmit` 函数。由于此操作风险高，在调用 `api.purgeAllCache` 之前，使用 `window.confirm()` 或一个独立的确认模态框进行二次确认。
3.  **用户反馈**: 两个表单在操作成功或失败后，都应向用户提供清晰的反馈信息。"

---

### **阶段 C: 质量保证与文档**

**Prompt C.1: 编写单元与集成测试**

**目标**: 为 `AdminCache` 组件及其子功能编写测试，确保其健壮性和可维护性。

**Prompt**:
"为保证迁移质量，我们需要为 `AdminCache` 组件编写全面的测试。
**任务**:
1.  **设置测试文件**: 在组件旁边创建测试文件 `src/components/AdminCache.test.tsx`。
2.  **模拟 API**: 使用 Mock Service Worker (MSW) 或 `jest.mock` 来拦截 `api.ts` 中的所有函数调用，并提供可控的成功/失败响应。
3.  **编写测试用例**:
    *   **初始渲染**: 测试组件在初始加载统计和集合列表时是否正确显示了加载状态。
    *   **成功渲染**: 测试当 API 成功返回数据时，统计信息和集合表格是否被正确渲染。
    *   **错误处理**: 测试当 API 返回错误时，是否显示了对应的错误消息。
    *   **用户交互**:
        *   使用 `@testing-library/user-event` 模拟用户点击 "Refresh" 按钮，并验证是否触发了 API 调用。
        *   模拟用户点击 "Delete" 按钮，验证确认模态框是否出现。
        *   模拟用户在模态框中点击 "Confirm Delete"，验证删除 API 是否被调用，并且列表是否刷新。
        *   模拟用户填写并提交维护表单，验证对应的 API 调用。"

**Prompt C.2: 更新项目文档**

**目标**: 在项目 `README.md` 或专门的文档中，记录新组件的设计和用法。

**Prompt**:
"最后，请更新项目文档。
**任务**:
1.  **更新 `README.md`**: 在 `README.md` 的组件概览部分，添加对 `AdminCache` 组件的描述。
2.  **说明其功能**: 简要说明该组件的作用（管理后端缓存）、其在应用中的位置 (`/admin` 路由) 以及其包含的主要功能（查看统计、管理集合、执行维护）。
3.  **环境变量**: 如果该组件依赖任何新的环境变量，请在 `env.example` 文件中添加并说明。"

---
---

## **附录: `admin_cache.html` 迁移至 React 的最小化实现 Prompts 序列**

本序列旨在作为将 `admin_cache.html` 快速迁移为一个最小可运行的只读 `AdminCache.tsx` 组件的单步指南。后续的 prompts 将在此基础上添加交互功能。

### **阶段 M: 最小化只读实现**

**Prompt M.1: 创建只读的 AdminCache 组件**

**目标**: 在一个步骤中，创建 `AdminCache` 组件，完成数据获取和核心信息的静态展示，暂时不包含任何写操作（如删除、清理）或复杂交互（如模态框）。

**Prompt**:
"我们开始对 `admin_cache.html` 进行一次最小化的迁移。
**任务**:
1.  **创建基础文件与路由**:
    *   在 `src/components/` 目录下创建 `AdminCache.tsx` 和 `AdminCache.css`。
    *   在 `src/App.tsx` 中，确保已为路径 `/admin` 配置了渲染 `AdminCache` 组件的 `<Route>`。
2.  **定义聚合数据类型**:
    *   在 `src/services/types.ts` (如果不存在则创建) 中，定义渲染页面所需的所有数据类型，包括一个聚合接口 `AdminPageData`，例如：
        ```typescript
        interface AdminPageData {
          cacheStats: CacheStats;
          pgVectorStats: PGVectorStats;
          collections: PGVectorCollection[];
        }
        ```
    *   确保 `CacheStats`, `PGVectorStats`, 和 `PGVectorCollection` 类型也已定义。
3.  **创建聚合 API 函数**:
    *   在 `src/services/api.ts` 中，创建一个单一的 API 调用函数 `getAdminPageData(): Promise<AdminPageData>`。确保从 `./types.ts` 导入 `AdminPageData` 类型。
    *   在 `src/services/mockData.ts` 中为该函数提供完整的模拟数据。
4.  **实现组件的只读视图**:
    *   在 `AdminCache.tsx` 中，使用 `useState` 管理 `data: AdminPageData | null`, `isLoading: boolean`, 和 `error: string | null` 这三个核心状态。
    *   使用 `useEffect` 在组件挂载时调用 `getAdminPageData` 并更新状态。
    *   **UI 渲染**:
        *   在 `isLoading` 时显示一个全局的 `LoadingSpinner`。
        *   在 `error` 时显示 `ErrorMessage`。
        *   成功获取数据后：
            *   使用 `react-bootstrap` 的 `Card` 组件展示 "Cache Statistics" 和 "PGVector Store" 的信息。
            *   使用 `react-bootstrap` 的 `Table` 组件展示 PGVector 集合列表。
            *   **注意**: 表格的 "Actions" 列暂时留空或只显示禁用的 "View" / "Delete" 按钮。所有维护表单的功能也暂时不实现。
"
