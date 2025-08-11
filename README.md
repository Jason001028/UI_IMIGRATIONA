# UI_IMIGRATIONA

# 合同分析系统 - React 前端

[![React Version](https://img.shields.io/badge/React-19-blue.svg)](https://react.dev/)
[![TypeScript](https://img.shields.io/badge/TypeScript-5.x-blue.svg)](https://www.typescriptlang.org/)
[![Bootstrap](https://img.shields.io/badge/Bootstrap-5.3-purple.svg)](https://getbootstrap.com/)
[![CI/CD Status](https://img.shields.io/badge/CI/CD-pending-lightgrey.svg)](#)

一个现代化的 React 单页应用（SPA），用于合同分析、结果展示和知识库管理。

## 🎯 项目目标

本项目的核心任务是将旧有的、基于服务器端渲染的 `HTML` 模板，全面迁移至一个基于 **React 19** 和 **TypeScript** 的现代化前端架构。我们致力于构建一个高性能、可维护、组件化的单页应用（SPA），以提升用户体验和开发效率。

## ✨ 核心功能

*   **📄 文档上传与处理**: 支持多种格式的合同文件上传。
*   **📊 分析结果展示**: 以列表和详情页的形式清晰地展示合同分析结果。
*   **✨ 智能证据高亮**: 在文档视图中动态高亮显示由 LLM (大语言模型) 标记的关键证据，支持多种高亮模式。
*   **📚 知识库管理**: 提供知识库文档的浏览、搜索和分页功能。
*   **⚙️ 管理员面板**: 为管理员提供简单的后台操作界面，如缓存清理。
*   **🔒 客户端路由**: 基于 React Router 的现代化、流畅的页面导航体验。

## 🛠️ 技术栈 (Tech Stack)

*   **框架 (Framework)**: [React 19](https://react.dev/)
*   **语言 (Language)**: [TypeScript](https://www.typescriptlang.org/)
*   **UI 框架 (UI Framework)**: [React-Bootstrap](https://react-bootstrap.github.io/), [Bootstrap 5](https://getbootstrap.com/)
*   **HTTP 客户端 (HTTP Client)**: [Axios](https://axios-http.com/)
*   **路由 (Routing)**: [React Router](https://reactrouter.com/)
*   **构建工具 (Build Tool)**: Create React App (`react-scripts`)
*   **测试 (Testing)**:
    *   [Jest](https://jestjs.io/) & [React Testing Library](https://testing-library.com/docs/react-testing-library/intro/) (单元/集成测试)
    *   [Cypress](https://www.cypress.io/) / [Playwright](https://playwright.dev/) (端到端测试)
*   **部署 (Deployment)**: Docker, Nginx

## 🚀 快速开始 (Getting Started)

请确保您已安装 [Node.js](https://nodejs.org/) (v18 或更高版本) 和 [npm](https://www.npmjs.com/)。

1.  **克隆仓库**
    ```bash
    git clone <your-repository-url>
    cd contract-analysis-frontend
    ```

2.  **安装依赖**
    ```bash
    npm install
    ```

3.  **启动开发服务器**
    ```bash
    npm start
    ```
    应用将在 `http://localhost:3000` 上运行。

4.  **运行测试**
    ```bash
    npm test
    ```

5.  **构建生产版本**
    ```bash
    npm run build
    ```
    构建产物将生成在 `build/` 目录下。

## 🏗️ 迁移策略概览

本次迁移遵循从服务器端渲染 (SSR) 到客户端渲染 (CSR) 的转变，核心工作包括：

1.  **UI 组件化 (Componentization)**: 将旧 HTML 页面的各个部分拆分为独立的、可复用的 React 组件。
2.  **API 驱动的数据流 (API-Driven Data Flow)**: 废弃后端模板注入数据的方式，改为通过 Axios 调用 RESTful API 在客户端获取和管理数据。
3.  **路由适配 (Routing Adaptation)**: 使用 React Router 管理应用内所有视图的导航。
4.  **交互逻辑重写 (Logic Rewrite)**: 使用 React Hooks 和事件处理重写所有原生的 JavaScript/jQuery 交互。

详细的迁移计划和开发指引请参考 [MIGRATION_PLAN.md](./MIGRATION_PLAN.md)。

## 项目总体架构图

<img width="2966" height="3846" alt="Untitled diagram _ Mermaid Chart-2025-08-11-020056" src="https://github.com/user-attachments/assets/13e0f372-0eb6-4544-b67d-c4d56355a4b1" />


## 🤝 贡献

我们欢迎任何形式的贡献！如果您有好的想法或发现了 Bug，请随时提交 Pull Request 或创建 Issue。

## 📄 许可证 (License)

[MIT](./LICENSE)
