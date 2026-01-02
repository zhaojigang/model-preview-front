# AI 模型管理器 - 前端项目

## 项目概述

这是一个用于管理 ComfyUI 模型的前端应用，旨在提供小而美的模型管理功能。该应用允许用户直观地查看和管理 ComfyUI 的各类模型（checkpoint、diffusion_model、lora 等），支持展示模型的缩略图、使用说明和下载链接。

### 主要功能

- **模型展示**：以瀑布流形式展示模型缩略图，支持多种模型类型筛选
- **模型搜索**：支持按模型名称搜索
- **模型详情**：点击模型卡片查看详细使用说明和下载链接
- **自动创建说明文件**：首次访问时自动为模型创建使用说明和下载地址文件

### 技术栈

- **前端框架**：Vue 3 (Composition API)
- **构建工具**：Vite
- **UI 组件库**：Element Plus
- **HTTP 客户端**：Axios
- **路由**：Vue Router
- **语言**：TypeScript

### 项目架构

```
src/
├── App.vue                 # 根组件
├── main.ts                 # 应用入口
├── components/
│   └── ModelPreview.vue    # 主模型预览组件
├── router/                 # 路由配置（当前为空）
└── assets/                 # 静态资源
```

## 构建和运行

### 开发环境

```bash
# 安装依赖
npm install

# 启动开发服务器
npm run dev
```

开发服务器默认运行在 `http://localhost:5173/`

### 生产构建

```bash
# 类型检查
npm run type-check

# 构建生产版本
npm run build

# 预览生产构建
npm run preview
```

### 完整构建流程

```bash
# 执行类型检查和构建（并行执行）
npm run build
```

## 开发规范

### 代码风格

- 使用 TypeScript 进行类型检查
- 使用 Vue 3 Composition API (`<script setup>`)
- 遵循 Vue 3 官方风格指南
- 使用 ESLint 进行代码检查（如配置）

### 组件开发

- 使用 `<script setup>` 语法糖
- 使用 TypeScript 类型注解
- 组件样式使用 scoped CSS
- 使用 Element Plus 组件库构建 UI

### API 代理配置

开发环境中，API 请求通过 Vite 代理转发到后端服务：

- 代理路径：`/api`
- 目标地址：`http://localhost:8080`
- 配置文件：`vite.config.ts`

### 路径别名

项目配置了 `@` 别名指向 `src` 目录：

```typescript
import Component from '@/components/Component.vue'
```

### 模型目录结构

后端服务需要按照以下结构组织模型文件：

```
ComfyUI\models\
├── checkpoints/            # Checkpoint 模型
│   ├── flux/
│   ├── sdxl/
│   ├── sd15/
│   └── pony/
├── diffusion_models/       # Diffusion Model
├── loras/                  # Lora 模型
│   ├── flux/
│   ├── sdxl/
│   ├── sd15/
│   └── pony/
├── controlNet/            # ControlNet 模型
├── upscaleModels/         # Upscale 模型
└── vae/                   # VAE 模型
```

### 文件命名规范

- 模型文件：`模型名.safetensors` 或其他格式
- 缩略图：`模型名.png`
- 使用说明：`模型名_使用说明.txt`
- 下载地址：`模型名_下载地址.txt`

## 相关项目

- **后端工程**：https://github.com/zhaojigang/model-preview
- **前端工程**：https://github.com/zhaojigang/model-preview-front（当前项目）

## 注意事项

- 开发前请确保后端服务已启动（默认端口 8080）
- 模型文件必须放置在正确的二级目录中才能被识别
- 首次访问模型管理界面时会自动创建空的说明文件和下载地址文件
- 项目使用 TypeScript，开发时请注意类型安全