# 端到端联调测试提示词序列 (End-to-End Integration Testing Prompt Sequence)

**目标:** 将 React 前端与 FastAPI 后端服务完全打通，用真实的 API 调用替换所有前端的模拟数据（Mock Data），并确保整个应用流程可以顺畅运行。

---

### 步骤 1: 分析前端 API 需求和数据结构

**目标:** 明确前端当前依赖哪些 API 调用，以及这些调用所期望的数据格式。

**提示词:**
"你是一位资深的软件工程师 Cline。你的首要任务是彻底理解前端应用的 API 通信层。请使用 `read_file` 工具依次阅读并分析以下文件：
1.  `contract-analysis-frontend/src/services/api.ts`
2.  `contract-analysis-frontend/src/services/mockData.ts`
3.  `contract-analysis-frontend/src/services/types.ts`

你的分析应产出以下结论：
-   一个清晰的列表，包含 `api.ts` 中所有当前调用了 `mockData.ts` 的函数。
-   根据 `types.ts` 和 `mockData.ts` 的内容，总结出每个函数所期望返回的数据结构。"

---

### 步骤 2: 调研后端可用 API 端点

**目标:** 掌握后端服务实际提供了哪些 API 端点，以便与前端需求进行匹配。

**提示词:**
"现在，将注意力转向后端。你的任务是调研并记录下所有可用的 API 端点。请使用 `read_file` 工具按以下顺序检查后端代码：
1.  阅读 FastAPI 的主入口文件 `contract-analysis-system/contract-migration/main.py`，了解其路由组织方式。
2.  递归地阅读 `contract-analysis-system/contract-migration/routers/` 目录下的所有文件。

你需要为每个发现的 API 端点记录以下关键信息：
-   完整的 URL 路径。
-   HTTP 请求方法 (例如 GET, POST)。
-   预期的请求体（Request Body）和查询参数（Query Parameters）。
-   响应体（Response Body）的数据结构。"

---

### 步骤 3: 修改前端代码，实现真实 API 调用

**目标:** 这是核心步骤。修改前端代码，使其调用真实的后端 API 而不是模拟数据。

**提示词:**
"现在开始执行最关键的修改。你的任务是重构前端的 API 服务。请使用 `replace_in_file` 工具编辑 `contract-analysis-frontend/src/services/api.ts` 文件。
针对你在步骤 1 中识别出的每一个依赖模拟数据的函数，执行以下操作：
1.  将其内部实现替换为对真实后端 API 的网络调用（推荐使用 `axios` 或 `fetch`）。
2.  确保调用的 URL、HTTP 方法、请求头和请求体与你在步骤 2 中记录的后端端点定义完全一致。
3.  在完成所有函数的替换后，务必删除文件顶部所有对 `mockData.ts` 的 `import` 语句。"

---

### 步骤 4: 优化配置，抽离硬编码 URL

**目标:** 提升代码质量，将硬编码的 API 地址抽离到环境变量中，便于在不同环境（开发、测试、生产）中部署。

**提示词:**
"为了让应用配置更加灵活和专业，我们需要移除代码中的硬编码 URL。请执行以下操作：
1.  使用 `replace_in_file` 工具再次编辑 `contract-analysis-frontend/src/services/api.ts`，将所有 API 调用的基础 URL (例如 `http://localhost:8000`) 替换为从环境变量 `process.env.REACT_APP_API_URL` 读取。
2.  使用 `write_to_file` 工具在 `contract-analysis-frontend/` 目录下创建一个名为 `.env.development.local` 的新文件。
3.  在该新文件中写入一行内容：`REACT_APP_API_URL=http://127.0.0.1:8000` (请根据后端服务的实际监听地址确认)。"

---

### 步骤 5: 验证与清理

**目标:** 确保联调后的应用功能正常，并移除不再需要的模拟数据文件。

**提示词:**
"联调工作已基本完成，现在进入验证阶段。
1.  首先，请使用 `execute_command` 工具，在 `contract-analysis-frontend` 目录下运行 `npm test`，确保你的修改没有破坏任何现有的单元测试或集成测试。
2.  测试通过后，请告知我（用户）启动前端 (`npm start`) 和后端服务进行手动端到端测试。
3.  在我确认所有功能都通过真实 API 正常工作后，请使用 `execute_command` 工具执行 `del contract-analysis-frontend\\src\\services\\mockData.ts` (适用于 Windows) 来删除无用的模拟数据文件，完成清理工作。"
