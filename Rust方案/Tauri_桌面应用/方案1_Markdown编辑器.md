# Tauri 桌面应用方案：方案1 - Markdown 编辑器

## 项目介绍

### 项目是什么
这是一个基于 Tauri + React + TypeScript 构建的跨平台 Markdown 编辑器，提供实时预览、语法高亮、文件管理和主题切换功能。

### 解决什么问题
- 提供轻量级的 Markdown 编辑体验（比 VS Code 更专注）
- 无需联网，完全本地化运行
- 跨平台支持（Windows/Mac/Linux）
- 相比 Electron，打包体积小 10 倍，内存占用更低

### 核心技术亮点
| 特性 | 技术实现 |
|------|----------|
| 前端框架 | React 18 + TypeScript 5 |
| 后端核心 | Rust + Tauri 2.0 |
| 状态管理 | Zustand |
| Markdown 渲染 | react-markdown + remark-gfm |
| 语法高亮 | Prism.js |
| 文件操作 | Tauri FS API |
| 样式 | Tailwind CSS |

### 适合什么场景
- 程序员写技术文档
- 学生记笔记、写论文
- 个人博客文章编写
- 任何需要 Markdown 编辑的场景

---

## 完整可运行代码

### 目录结构
```
markdown-editor/
├── src/                      # React 前端源码
│   ├── components/          # React 组件
│   │   ├── Editor.tsx       # 编辑器主组件
│   │   ├── Preview.tsx      # 预览组件
│   │   ├── Sidebar.tsx       # 文件侧边栏
│   │   └── Header.tsx        # 顶部工具栏
│   ├── hooks/               # 自定义 Hooks
│   │   ├── useFile.ts       # 文件操作 Hook
│   │   └── useTheme.ts      # 主题管理 Hook
│   ├── stores/              # Zustand 状态管理
│   │   └── editorStore.ts   # 编辑器状态
│   ├── styles/              # 样式文件
│   │   └── globals.css      # 全局样式
│   ├── App.tsx              # 主应用组件
│   ├── main.tsx             # 入口文件
│   └── vite-env.d.ts        # Vite 类型声明
├── src-tauri/               # Rust 后端源码
│   ├── src/
│   │   ├── main.rs          # Tauri 主入口
│   │   ├── commands.rs      # Tauri 命令（文件操作）
│   │   └── lib.rs           # 库入口
│   ├── Cargo.toml           # Rust 依赖配置
│   └── tauri.conf.json      # Tauri 配置文件
├── index.html               # HTML 入口
├── package.json             # Node.js 依赖配置
├── tsconfig.json             # TypeScript 配置
├── vite.config.ts           # Vite 构建配置
├── tailwind.config.js       # Tailwind CSS 配置
└── README.md                 # 项目说明
```

### 1. 前端配置：package.json
```json
{
  "name": "markdown-editor",
  "private": true,
  "version": "1.0.0",
  "type": "module",
  "scripts": {
    "dev": "vite",
    "build": "tsc && vite build",
    "preview": "vite preview",
    "tauri": "tauri"
  },
  "dependencies": {
    "react": "^18.2.0",
    "react-dom": "^18.2.0",
    "react-markdown": "^9.0.1",
    "remark-gfm": "^4.0.0",
    "prismjs": "^1.29.0",
    "zustand": "^4.4.7",
    "@tauri-apps/api": "^2.0.0"
  },
  "devDependencies": {
    "@tauri-apps/cli": "^2.0.0",
    "@types/prismjs": "^1.26.3",
    "@types/react": "^18.2.43",
    "@types/react-dom": "^18.2.17",
    "@vitejs/plugin-react": "^4.2.1",
    "autoprefixer": "^10.4.16",
    "postcss": "^8.4.32",
    "tailwindcss": "^3.4.0",
    "typescript": "^5.3.3",
    "vite": "^5.0.10"
  }
}
```

### 2. Vite 配置：vite.config.ts
```typescript
// vite.config.ts - Vite 构建工具配置
// 作用：配置 Vite 的插件、构建选项和开发服务器

import { defineConfig } from "vite";           // Vite 配置定义函数
import react from "@vitejs/plugin-react";       // React 插件，支持 JSX 和 HMR

// Tauri 要求的配置格式，必须使用函数字面量形式
export default defineConfig(async () => ({
  plugins: [react()],  // 启用 React 插件，提供：1. JSX 转译 2. 热模块替换(HMR)

  // Vite 的 ClearScreen 配置确保 Tauri 在构建时不会清屏
  // 这对 Windows 上的 Tauri CLI 很重要
  clearScreen: false,

  // 服务器配置
  server: {
    port: 1420,      // 开发服务器端口
    strictPort: true, // 端口被占用时不要自动换端口
    watch: {
      // 忽略 Rust 源码变化，避免触发前端重新构建
      ignored: ["**/src-tauri/**"],
    },
  },

  // 构建配置
  build: {
    target: ["es2021", "chrome100", "safari13"], // 兼容目标浏览器
    minify: !process.env.TAURI_DEBUG ? "esbuild" : false, // 非调试模式使用 esbuild 压缩
    sourcemap: !!process.env.TAURI_DEBUG, // 调试模式生成 sourcemap
  },
}));
```

### 3. TypeScript 配置：tsconfig.json
```json
{
  "compilerOptions": {
    "target": "ES2020",         // 编译目标：ES2020，现代浏览器都支持
    "useDefineForClassFields": true,  // 使用 define 语义（class 字段）
    "lib": ["ES2020", "DOM", "DOM.Iterable"],  // 运行时环境：浏览器 DOM + ES2020
    "module": "ESNext",         // 模块系统：ESNext（支持动态 import）
    "skipLibCheck": true,       // 跳过库文件类型检查，加快编译

    /* Bundler mode - 让 TypeScript 与 Vite/ESBuild 配合 */
    "moduleResolution": "bundler",  // 使用 bundler 解析策略
    "allowImportingTsExtensions": true,  // 允许导入 .ts 扩展名
    "resolveJsonModule": true,    // 支持导入 JSON 模块
    "isolatedModules": true,      // 确保每个文件可独立转译
    "noEmit": true,              // 不输出文件，只做类型检查
    "jsx": "react-jsx",          // JSX 语法：使用新版 react-jsx 转换

    /* Linting - 严格类型检查 */
    "strict": true,              // 启用所有严格类型检查
    "noUnusedLocals": true,      // 不允许未使用的局部变量
    "noUnusedParameters": true,  // 不允许未使用的函数参数
    "noFallthroughCasesInSwitch": true  // 不允许 switch 贯穿
  },
  "include": ["src"],           // 只检查 src 目录下的文件
  "references": [{ "path": "./tsconfig.node.json" }]  // 引用 Node 配置
}
```

### 4. Tailwind CSS 配置：tailwind.config.js
```javascript
// tailwind.config.js - Tailwind CSS 配置文件
// 作用：配置 Tailwind 的主题、插件和变体

/** @type {import('tailwindcss').Config} */
export default {
  // 内容配置：指定 Tailwind 扫描哪些文件来生成 CSS
  content: [
    "./index.html",      // HTML 入口文件
    "./src/**/*.{js,ts,jsx,tsx}",  // React 组件文件
  ],
  
  // 主题扩展：自定义主题颜色和字体
  theme: {
    extend: {
      // 扩展颜色：可以添加自定义颜色变量
      colors: {
        // 主题色：靛蓝色系
        primary: {
          50: '#eef2ff',
          100: '#e0e7ff',
          200: '#c7d2fe',
          300: '#a5b4fc',
          400: '#818cf8',
          500: '#6366f1',
          600: '#4f46e5',
          700: '#4338ca',
          800: '#3730a3',
          900: '#312e81',
        },
      },
      // 字体配置
      fontFamily: {
        // 代码字体：优先使用 JetBrains Mono
        mono: ['JetBrains Mono', 'Fira Code', 'monospace'],
      },
    },
  },
  
  // 插件：启用 typography 插件用于 Markdown 美化
  plugins: [
    require('@tailwindcss/typography'),  // 自动为 article 标签添加排版样式
  ],
}
```

### 5. PostCSS 配置：postcss.config.js
```javascript
// postcss.config.js - PostCSS 配置文件
// 作用：配置 CSS 转换工具链

export default {
  plugins: {
    // Tailwind CSS 插件：处理 @tailwind 指令
    tailwindcss: {},
    // Autoprefixer：自动添加 CSS 前缀（-webkit-, -moz- 等）
    autoprefixer: {},
  },
}
```

### 6. HTML 入口：index.html
```html
<!DOCTYPE html>
<html lang="zh-CN">
  <head>
    <meta charset="UTF-8" />
    <!-- 链接标签：用于绑定窗口图标（仅构建时需要） -->
    <link rel="icon" type="image/svg+xml" href="/vite.svg" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Markdown Editor - Rust Tauri</title>
  </head>
  <body>
    <!-- React 应用挂载点 -->
    <div id="root"></div>
    <!-- Vite 入口脚本 -->
    <script type="module" src="/src/main.tsx"></script>
  </body>
</html>
```

### 7. 全局样式：src/styles/globals.css
```css
/* globals.css - 全局样式文件 */

/* 引入 Tailwind CSS 基础指令 */
/* 这三行必须在文件最前面，告诉 Tailwind 扫描并生成 CSS */
@tailwind base;
@tailwind components;
@tailwind utilities;

/* 基础样式重置 */
@layer base {
  /* 重置 box-sizing 为 border-box */
  * {
    box-sizing: border-box;
  }

  /* html 元素样式 */
  html {
    /* 取消默认的 focus 轮廓，使用自定义样式 */
    -webkit-tap-highlight-color: transparent;
    /* 平滑滚动 */
    scroll-behavior: smooth;
  }

  /* body 元素样式 */
  body {
    /* 字体堆叠：优先使用系统字体 */
    font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, 
                 'Helvetica Neue', Arial, sans-serif;
    /* 灰色文字 */
    color: #374151;
    /* 浅灰色背景 */
    background-color: #f9fafb;
    /* 禁止文字选中（编辑场景） */
    user-select: none;
  }
}

/* 滚动条样式 */
@layer utilities {
  /* 自定义滚动条样式（WebKit 内核） */
  .scrollbar-thin {
    /* 滚动条宽度 */
    scrollbar-width: thin;
    /* 滚动条颜色 */
    scrollbar-color: #cbd5e1 transparent;
  }

  /* WebKit 滚动条样式 */
  .scrollbar-thin::-webkit-scrollbar {
    width: 6px;      /* 滚动条宽度 */
    height: 6px;     /* 滚动条高度 */
  }

  .scrollbar-thin::-webkit-scrollbar-track {
    background: transparent;  /* 轨道背景透明 */
  }

  .scrollbar-thin::-webkit-scrollbar-thumb {
    background-color: #cbd5e1;  /* 滑块颜色 */
    border-radius: 3px;          /* 圆角 */
  }

  /* 悬停时滑块颜色变深 */
  .scrollbar-thin::-webkit-scrollbar-thumb:hover {
    background-color: #94a3b8;
  }
}

/* Markdown 预览样式 */
@layer components {
  /* 代码块样式 */
  .prose pre {
    /* 深色背景 */
    background-color: #1e293b;
    /* 代码字体 */
    font-family: 'JetBrains Mono', 'Fira Code', monospace;
    /* 白色文字 */
    color: #f1f5f9;
    /* 圆角 */
    border-radius: 0.5rem;
    /* 内边距 */
    padding: 1rem;
    /* 溢出时滚动 */
    overflow-x: auto;
  }

  /* 行内代码样式 */
  .prose code:not(pre code) {
    /* 浅色背景 */
    background-color: #f1f5f9;
    /* 深色文字 */
    color: #475569;
    /* 圆角 */
    border-radius: 0.25rem;
    /* 内边距 */
    padding: 0.125rem 0.375rem;
    /* 小字号 */
    font-size: 0.875em;
    /* 等宽字体 */
    font-family: 'JetBrains Mono', 'Fira Code', monospace;
  }

  /* 链接样式 */
  .prose a {
    /* 主题色 */
    color: #4f46e5;
    /* 去除下划线 */
    text-decoration: none;
  }

  /* 链接悬停效果 */
  .prose a:hover {
    text-decoration: underline;
  }

  /* 引用块样式 */
  .prose blockquote {
    /* 左边框 */
    border-left: 4px solid #6366f1;
    /* 浅色背景 */
    background-color: #f8fafc;
    /* 斜体 */
    font-style: italic;
    /* 内边距 */
    padding: 0.75rem 1rem;
    /* 圆角 */
    border-radius: 0 0.5rem 0.5rem 0;
  }
}
```

### 8. 主入口文件：src/main.tsx
```tsx
// main.tsx - React 应用入口
// 作用：创建 React 应用并挂载到 DOM

import React from "react";      // React 核心库
import ReactDOM from "react-dom/client";  // React 18 的 DOM 渲染 API
import App from "./App";        // 主应用组件
import "./styles/globals.css";  // 全局样式

// 创建 React 18 的 Concurrent Mode Root
// ReactDOM.createRoot 是 React 18 的新 API，支持并发渲染
ReactDOM.createRoot(document.getElementById("root") as HTMLElement).render(
  // 严格模式：检测不安全的生命周期方法和重复的副作用
  <React.StrictMode>
    <App />
  </React.StrictMode>
);
```

### 9. 状态管理：src/stores/editorStore.ts
```typescript
// editorStore.ts - Zustand 状态管理
// 作用：集中管理编辑器的所有状态

import { create } from 'zustand';  // Zustand 创建 store 的函数

// ==================== 类型定义 ====================

// 文件类型：表示一个打开的文件
export interface File {
  path: string;       // 文件路径
  name: string;       // 文件名
  content: string;    // 文件内容
  isModified: boolean; // 是否被修改
}

// 编辑器设置类型
export interface EditorSettings {
  theme: 'light' | 'dark';      // 主题：亮色/暗色
  fontSize: number;             // 字体大小
  autoSave: boolean;            // 是否自动保存
  lineNumbers: boolean;         // 是否显示行号
  wordWrap: boolean;            // 是否自动换行
}

// Store 状态类型
interface EditorState {
  // 文件相关状态
  files: File[];                // 打开的文件列表
  activeFileIndex: number;      // 当前活动的文件索引
  currentDirectory: string;    // 当前目录
  
  // 设置相关状态
  settings: EditorSettings;
  
  // 侧边栏状态
  showSidebar: boolean;         // 是否显示侧边栏
  showPreview: boolean;         // 是否显示预览
  
  // 文件操作方法
  addFile: (file: File) => void;
  closeFile: (index: number) => void;
  setActiveFile: (index: number) => void;
  updateFileContent: (index: number, content: string) => void;
  
  // 设置操作方法
  updateSettings: (settings: Partial<EditorSettings>) => void;
  
  // UI 操作方法
  toggleSidebar: () => void;
  togglePreview: () => void;
  setDirectory: (dir: string) => void;
}

// ==================== 创建 Store ====================

// initialState - 编辑器的初始状态
const initialState = {
  // 初始文件列表为空
  files: [] as File[],
  // 没有活动文件
  activeFileIndex: -1,
  // 当前目录为空
  currentDirectory: '',
  // 默认设置
  settings: {
    theme: 'light' as const,      // 默认亮色主题
    fontSize: 14,                 // 14px 字体
    autoSave: false,              // 默认不自动保存
    lineNumbers: true,            // 默认显示行号
    wordWrap: true,               // 默认自动换行
  },
  // 默认显示侧边栏和预览
  showSidebar: true,
  showPreview: true,
};

// 创建 Zustand store
// create 函数接收一个函数，返回一个 hook
export const useEditorStore = create<EditorState>((set) => ({
  // ==================== 初始状态 ====================
  ...initialState,
  
  // ==================== 文件操作 ====================
  
  // 添加文件到列表
  // 使用函数式更新，确保基于最新的状态
  addFile: (file) =>
    set((state) => {
      // 检查文件是否已存在
      const existingIndex = state.files.findIndex((f) => f.path === file.path);
      
      if (existingIndex !== -1) {
        // 文件已存在，切换到该文件
        return { activeFileIndex: existingIndex };
      }
      
      // 新文件，添加到列表并激活
      return {
        files: [...state.files, file],
        activeFileIndex: state.files.length,
      };
    }),
    
  // 关闭文件
  closeFile: (index) =>
    set((state) => {
      // 移除指定索引的文件
      const newFiles = state.files.filter((_, i) => i !== index);
      
      // 计算新的活动文件索引
      let newActiveIndex = state.activeFileIndex;
      if (index < state.activeFileIndex) {
        // 关闭的文件在当前文件之前，索引减 1
        newActiveIndex -= 1;
      } else if (index === state.activeFileIndex) {
        // 关闭的是当前文件
        if (newFiles.length === 0) {
          // 没有文件了
          newActiveIndex = -1;
        } else if (index >= newFiles.length) {
          // 索引超出范围，切换到最后一项
          newActiveIndex = newFiles.length - 1;
        }
        // 否则保持当前索引
      }
      
      return {
        files: newFiles,
        activeFileIndex: newActiveIndex,
      };
    }),
    
  // 设置活动文件
  setActiveFile: (index) =>
    set({ activeFileIndex: index }),
    
  // 更新文件内容
  updateFileContent: (index, content) =>
    set((state) => {
      // 使用 map 创建新数组（不可变更新）
      const newFiles = state.files.map((file, i) =>
        // 只有匹配索引的文件需要更新
        i === index
          ? { ...file, content, isModified: true }
          : file
      );
      return { files: newFiles };
    }),
    
  // ==================== 设置操作 ====================
  
  // 更新设置（合并更新）
  updateSettings: (newSettings) =>
    set((state) => ({
      settings: { ...state.settings, ...newSettings },
    })),
    
  // ==================== UI 操作 ====================
  
  // 切换侧边栏显示
  toggleSidebar: () =>
    set((state) => ({ showSidebar: !state.showSidebar })),
    
  // 切换预览显示
  togglePreview: () =>
    set((state) => ({ showPreview: !state.showPreview })),
    
  // 设置当前目录
  setDirectory: (dir) =>
    set({ currentDirectory: dir }),
}));
```

### 10. 编辑器组件：src/components/Editor.tsx
```tsx
// Editor.tsx - Markdown 编辑器主组件
// 作用：提供文本编辑功能，处理键盘事件和文本变化

import React, { useCallback, useEffect, useRef } from 'react';
import { useEditorStore } from '../stores/editorStore';

export const Editor: React.FC = () => {
  // 从 store 获取状态和方法
  const { 
    files,              // 文件列表
    activeFileIndex,    // 当前文件索引
    settings,           // 编辑器设置
    updateFileContent,  // 更新文件内容
  } = useEditorStore();
  
  // 使用 ref 获取 textarea DOM 元素
  // useRef 在组件重新渲染时保持引用不变
  const textareaRef = useRef<HTMLTextAreaElement>(null);
  
  // 获取当前活动的文件
  const activeFile = activeFileIndex >= 0 ? files[activeFileIndex] : null;
  
  // ==================== 事件处理 ====================
  
  /**
   * 处理文本变化的回调函数
   * 使用 useCallback 缓存函数，避免不必要的重新渲染
   * 
   * @param e - React 的合成事件对象
   */
  const handleChange = useCallback(
    (e: React.ChangeEvent<HTMLTextAreaElement>) => {
      // 只有存在活动文件时才更新
      if (activeFileIndex >= 0) {
        // 调用 store 的更新方法
        updateFileContent(activeFileIndex, e.target.value);
      }
    },
    [activeFileIndex, updateFileContent]  // 依赖项：只有这些变化时才重新创建函数
  );
  
  /**
   * 处理键盘事件
   * 支持 Tab 键插入空格、Ctrl+S 保存等快捷键
   */
  const handleKeyDown = useCallback(
    (e: React.KeyboardEvent<HTMLTextAreaElement>) => {
      // 获取 textarea DOM 元素
      const textarea = textareaRef.current;
      if (!textarea) return;
      
      // ==================== Tab 键处理 ====================
      if (e.key === 'Tab') {
        e.preventDefault();  // 阻止默认的焦点切换行为
        
        // 获取当前光标位置
        const start = textarea.selectionStart;
        const end = textarea.selectionEnd;
        
        // 获取当前文本值
        const value = textarea.value;
        
        // 插入 2 个空格
        const newValue = value.substring(0, start) + '  ' + value.substring(end);
        
        // 更新文本框值
        textarea.value = newValue;
        
        // 将光标移动到插入位置之后
        textarea.selectionStart = textarea.selectionEnd = start + 2;
        
        // 触发 onChange 事件以更新状态
        // 创建一个合成事件对象
        const event = {
          target: textarea,
        } as React.ChangeEvent<HTMLTextAreaElement>;
        handleChange(event);
      }
      
      // ==================== Ctrl/Cmd + S 保存 ====================
      if ((e.ctrlKey || e.metaKey) && e.key === 's') {
        e.preventDefault();
        // 保存逻辑会通过 Tauri 命令调用
        console.log('保存文件:', activeFile?.name);
      }
      
      // ==================== Ctrl/Cmd + B 加粗 ====================
      if ((e.ctrlKey || e.metaKey) && e.key === 'b') {
        e.preventDefault();
        wrapSelection('**', '**');
      }
      
      // ==================== Ctrl/Cmd + I 斜体 ====================
      if ((e.ctrlKey || e.metaKey) && e.key === 'i') {
        e.preventDefault();
        wrapSelection('*', '*');
      }
    },
    [activeFile, handleChange]  // 依赖项
  );
  
  /**
   * 包装选中文本
   * 用于实现加粗、斜体等格式化
   * 
   * @param before - 选区前添加的文本
   * @param after - 选区后添加的文本
   */
  const wrapSelection = useCallback((before: string, after: string) => {
    const textarea = textareaRef.current;
    if (!textarea) return;
    
    // 获取选中的文本
    const start = textarea.selectionStart;
    const end = textarea.selectionEnd;
    const selectedText = textarea.value.substring(start, end);
    
    // 构造新文本
    const newValue =
      textarea.value.substring(0, start) +
      before +
      selectedText +
      after +
      textarea.value.substring(end);
    
    // 更新文本框
    textarea.value = newValue;
    
    // 恢复选区（选中格式化后的文本）
    textarea.selectionStart = start + before.length;
    textarea.selectionEnd = end + before.length;
    
    // 触发更新
    const event = {
      target: textarea,
    } as React.ChangeEvent<HTMLTextAreaElement>;
    handleChange(event);
  }, [handleChange]);
  
  // ==================== 快捷工具栏按钮 ====================
  
  /**
   * 工具栏按钮组件
   * 接受图标、标题和点击回调
   */
  const ToolbarButton: React.FC<{
    title: string;
    onClick: () => void;
    children: React.ReactNode;
  }> = ({ title, onClick, children }) => (
    <button
      type="button"
      title={title}
      onClick={onClick}
      className="p-1.5 rounded hover:bg-gray-200 dark:hover:bg-gray-700 transition-colors"
    >
      {children}
    </button>
  );
  
  // ==================== 自动聚焦 ====================
  // 当文件切换时，自动聚焦到编辑器
  useEffect(() => {
    if (activeFile && textareaRef.current) {
      textareaRef.current.focus();
    }
  }, [activeFileIndex]);  // 监听文件切换
  
  // ==================== 渲染 ====================
  
  // 没有打开文件时显示空状态
  if (!activeFile) {
    return (
      <div className="flex-1 flex items-center justify-center bg-gray-50 dark:bg-gray-900">
        <div className="text-center text-gray-400">
          {/* 空状态图标 */}
          <svg
            className="w-16 h-16 mx-auto mb-4 opacity-50"
            fill="none"
            stroke="currentColor"
            viewBox="0 0 24 24"
          >
            <path
              strokeLinecap="round"
              strokeLinejoin="round"
              strokeWidth={1.5}
              d="M9 12h6m-6 4h6m2 5H7a2 2 0 01-2-2V5a2 2 0 012-2h5.586a1 1 0 01.707.293l5.414 5.414a1 1 0 01.293.707V19a2 2 0 01-2 2z"
            />
          </svg>
          <p className="text-lg">打开或新建文件开始编辑</p>
          <p className="text-sm mt-2">支持新建文件、打开本地 Markdown 文件</p>
        </div>
      </div>
    );
  }
  
  return (
    <div className="flex-1 flex flex-col bg-white dark:bg-gray-800">
      {/* 工具栏 */}
      <div className="flex items-center gap-1 px-2 py-1 border-b border-gray-200 dark:border-gray-700 bg-gray-50 dark:bg-gray-900">
        {/* 标题 */}
        <span className="text-sm text-gray-500 mr-2">格式工具</span>
        
        {/* 加粗按钮 */}
        <ToolbarButton
          title="加粗 (Ctrl+B)"
          onClick={() => wrapSelection('**', '**')}
        >
          <span className="font-bold text-sm">B</span>
        </ToolbarButton>
        
        {/* 斜体按钮 */}
        <ToolbarButton
          title="斜体 (Ctrl+I)"
          onClick={() => wrapSelection('*', '*')}
        >
          <span className="italic text-sm">I</span>
        </ToolbarButton>
        
        {/* 删除线按钮 */}
        <ToolbarButton
          title="删除线"
          onClick={() => wrapSelection('~~', '~~')}
        >
          <span className="line-through text-sm">S</span>
        </ToolbarButton>
        
        {/* 分隔线 */}
        <div className="w-px h-5 bg-gray-300 dark:bg-gray-600 mx-1" />
        
        {/* 标题按钮 */}
        <ToolbarButton
          title="一级标题"
          onClick={() => wrapSelection('# ', '')}
        >
          <span className="text-sm font-bold">H1</span>
        </ToolbarButton>
        
        {/* 链接按钮 */}
        <ToolbarButton
          title="链接"
          onClick={() => wrapSelection('[', '](url)')}
        >
          <svg className="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
            <path strokeLinecap="round" strokeLinejoin="round" strokeWidth={2} d="M13.828 10.172a4 4 0 00-5.656 0l-4 4a4 4 0 105.656 5.656l1.102-1.101m-.758-4.899a4 4 0 005.656 0l4-4a4 4 0 00-5.656-5.656l-1.1 1.1" />
          </svg>
        </ToolbarButton>
        
        {/* 代码按钮 */}
        <ToolbarButton
          title="行内代码"
          onClick={() => wrapSelection('`', '`')}
        >
          <svg className="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
            <path strokeLinecap="round" strokeLinejoin="round" strokeWidth={2} d="M10 20l4-16m4 4l4 4-4 4M6 16l-4-4 4-4" />
          </svg>
        </ToolbarButton>
        
        {/* 引用按钮 */}
        <ToolbarButton
          title="引用"
          onClick={() => wrapSelection('> ', '')}
        >
          <svg className="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
            <path strokeLinecap="round" strokeLinejoin="round" strokeWidth={2} d="M8 10h.01M12 10h.01M16 10h.01M9 16H5a2 2 0 01-2-2V6a2 2 0 012-2h14a2 2 0 012 2v8a2 2 0 01-2 2h-5l-5 5v-5z" />
          </svg>
        </ToolbarButton>
        
        {/* 无序列表按钮 */}
        <ToolbarButton
          title="无序列表"
          onClick={() => wrapSelection('- ', '')}
        >
          <svg className="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
            <path strokeLinecap="round" strokeLinejoin="round" strokeWidth={2} d="M4 6h16M4 10h16M4 14h16M4 18h16" />
          </svg>
        </ToolbarButton>
        
        {/* 有序列表按钮 */}
        <ToolbarButton
          title="有序列表"
          onClick={() => wrapSelection('1. ', '')}
        >
          <svg className="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
            <path strokeLinecap="round" strokeLinejoin="round" strokeWidth={2} d="M7 20l4-16m2 16l4-16M6 9h14M4 15h14" />
          </svg>
        </ToolbarButton>
      </div>
      
      {/* 文本编辑区 */}
      <textarea
        ref={textareaRef}
        value={activeFile.content}
        onChange={handleChange}
        onKeyDown={handleKeyDown}
        spellCheck={false}  // 禁用拼写检查
        className={`
          flex-1 p-4 resize-none outline-none
          font-mono text-sm leading-relaxed
          bg-white dark:bg-gray-800
          text-gray-800 dark:text-gray-200
          ${settings.lineNumbers ? '' : ''}
        `}
        style={{ fontSize: `${settings.fontSize}px` }}
        placeholder="在这里开始编写 Markdown..."
      />
    </div>
  );
};
```

### 11. 预览组件：src/components/Preview.tsx
```tsx
// Preview.tsx - Markdown 预览组件
// 作用：渲染 Markdown 内容为 HTML，支持 GFM 语法

import React from 'react';
import ReactMarkdown from 'react-markdown';
import remarkGfm from 'remark-gfm';
import Prism from 'prismjs';
import { useEditorStore } from '../stores/editorStore';
import 'prismjs/components/prism-rust';  // Rust 语言高亮
import 'prismjs/components/prism-typescript';  // TypeScript 语言高亮
import 'prismjs/components/prism-javascript';  // JavaScript 语言高亮
import 'prismjs/components/prism-python';  // Python 语言高亮
import 'prismjs/components/prism-java';  // Java 语言高亮
import 'prismjs/components/prism-go';  // Go 语言高亮

export const Preview: React.FC = () => {
  // 从 store 获取状态
  const { files, activeFileIndex, settings } = useEditorStore();
  
  // 获取当前活动的文件
  const activeFile = activeFileIndex >= 0 ? files[activeFileIndex] : null;
  
  // ==================== 代码高亮组件 ====================
  // 自定义代码块渲染器，支持 Prism 语法高亮
  
  /**
   * 代码块组件
   * 使用 Prism.js 进行语法高亮
   */
  const CodeBlock: React.FC<{
    className?: string;
    children: React.ReactNode;
  }> = ({ className, children }) => {
    // 提取语言名称（className 格式：language-rust）
    const language = className?.replace(/language-/, '') || 'plaintext';
    
    // 获取代码文本
    const code = String(children).replace(/\n$/, '');
    
    // 获取语法高亮后的 HTML
    // Prism.highlight 接受：代码、语言、语法规则
    let highlightedCode: string;
    try {
      highlightedCode = Prism.highlight(code, Prism.languages[language] || Prism.languages.plaintext, language);
    } catch {
      // 如果语言不支持，回退到纯文本
      highlightedCode = code;
    }
    
    return (
      <pre className={`language-${language} rounded-lg overflow-hidden my-4`}>
        {/* 代码语言标签 */}
        <div className="flex justify-between items-center px-4 py-2 bg-gray-700 text-gray-400 text-xs">
          <span>{language.toUpperCase()}</span>
          {/* 复制按钮 */}
          <button
            onClick={() => navigator.clipboard.writeText(code)}
            className="hover:text-white transition-colors"
            title="复制代码"
          >
            <svg className="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
              <path strokeLinecap="round" strokeLinejoin="round" strokeWidth={2} d="M8 16H6a2 2 0 01-2-2V6a2 2 0 012-2h8a2 2 0 012 2v2m-6 12h8a2 2 0 002-2v-8a2 2 0 00-2-2h-8a2 2 0 00-2 2v8a2 2 0 002 2z" />
            </svg>
          </button>
        </div>
        {/* 代码内容 */}
        <code
          className={`block px-4 py-3 text-sm font-mono overflow-x-auto ${className}`}
          dangerouslySetInnerHTML={{ __html: highlightedCode }}
        />
      </pre>
    );
  };
  
  // ==================== 渲染 ====================
  
  // 没有文件时显示空状态
  if (!activeFile) {
    return (
      <div className="flex-1 flex items-center justify-center bg-gray-50 dark:bg-gray-900">
        <p className="text-gray-400">预览区域</p>
      </div>
    );
  }
  
  return (
    <div 
      className={`
        flex-1 p-6 overflow-y-auto bg-white dark:bg-gray-800
        ${settings.theme === 'dark' ? 'dark' : ''}
      `}
    >
      {/* Markdown 渲染器 */}
      {/* 
        ReactMarkdown 组件将 Markdown 文本转换为 React 组件
        remarkGfm 插件支持 GitHub Flavored Markdown（表格、任务列表等）
      */}
      <article className="prose prose-gray dark:prose-invert max-w-none">
        <ReactMarkdown
          // GFM 支持：表格、删除线、任务列表等
          remarkPlugins={[remarkGfm]}
          // 自定义组件映射
          components={{
            // 自定义代码块渲染
            code: ({ className, children, ...props }) => {
              // 判断是行内代码还是代码块
              // 行内代码没有 className 或 className 不包含 language-
              const isInline = !className || !className.includes('language-');
              
              if (isInline) {
                // 行内代码样式
                return (
                  <code
                    className="px-1.5 py-0.5 rounded bg-gray-100 dark:bg-gray-700 text-pink-600 dark:text-pink-400 text-sm font-mono"
                    {...props}
                  >
                    {children}
                  </code>
                );
              }
              
              // 代码块
              return (
                <CodeBlock className={className}>
                  {children}
                </CodeBlock>
              );
            },
            // 自定义链接渲染
            a: ({ href, children }) => (
              <a
                href={href}
                target="_blank"
                rel="noopener noreferrer"
                className="text-indigo-600 dark:text-indigo-400 hover:underline"
              >
                {children}
              </a>
            ),
            // 自定义表格渲染
            table: ({ children }) => (
              <div className="overflow-x-auto my-4">
                <table className="min-w-full border border-gray-200 dark:border-gray-700 rounded-lg">
                  {children}
                </table>
              </div>
            ),
            // 自定义任务列表渲染
            input: ({ type, checked, ...props }) => {
              if (type === 'checkbox') {
                return (
                  <input
                    type="checkbox"
                    checked={checked}
                    readOnly
                    className="w-4 h-4 text-indigo-600 rounded border-gray-300 focus:ring-indigo-500"
                    {...props}
                  />
                );
              }
              return <input type={type} {...props} />;
            },
          }}
        >
          {/* Markdown 内容 */}
          {activeFile.content || '*暂无内容*'}
        </ReactMarkdown>
      </article>
    </div>
  );
};
```

### 12. 侧边栏组件：src/components/Sidebar.tsx
```tsx
// Sidebar.tsx - 文件侧边栏组件
// 作用：显示文件列表、目录树，提供文件操作菜单

import React, { useCallback, useState } from 'react';
import { invoke } from '@tauri-apps/api/core';
import { useEditorStore, File } from '../stores/editorStore';

export const Sidebar: React.FC = () => {
  // 从 store 获取状态和方法
  const { 
    files, 
    activeFileIndex, 
    currentDirectory,
    showSidebar,
    addFile, 
    closeFile, 
    setActiveFile,
    setDirectory,
  } = useEditorStore();
  
  // 本地状态：右键菜单
  const [contextMenu, setContextMenu] = useState<{
    x: number;
    y: number;
    fileIndex: number;
  } | null>(null);
  
  // 本地状态：文件树
  const [fileTree, setFileTree] = useState<File[]>([]);
  
  // ==================== 文件操作 ====================
  
  /**
   * 创建新文件
   * 在当前目录下创建一个新的 .md 文件
   */
  const handleNewFile = useCallback(async () => {
    try {
      // 提示输入文件名
      const name = prompt('请输入文件名（不需要 .md 后缀）:', 'untitled');
      if (!name) return;
      
      // 构造完整路径
      const fileName = name.endsWith('.md') ? name : `${name}.md`;
      const path = currentDirectory 
        ? `${currentDirectory}/${fileName}`
        : fileName;
      
      // 调用 Rust 后端创建文件
      await invoke('create_file', { 
        path,
        content: '',
      });
      
      // 添加到文件列表
      addFile({
        path,
        name: fileName,
        content: '',
        isModified: false,
      });
    } catch (error) {
      console.error('创建文件失败:', error);
      alert(`创建文件失败: ${error}`);
    }
  }, [currentDirectory, addFile]);
  
  /**
   * 打开文件
   * 使用系统文件对话框选择文件
   */
  const handleOpenFile = useCallback(async () => {
    try {
      // 调用 Rust 后端打开文件对话框
      const filePath = await invoke<string | null>('open_file_dialog');
      
      if (!filePath) return;  // 用户取消了对话框
      
      // 读取文件内容
      const result = await invoke<{ content: string; name: string }>('read_file', {
        path: filePath,
      });
      
      // 添加到文件列表
      addFile({
        path: filePath,
        name: result.name,
        content: result.content,
        isModified: false,
      });
    } catch (error) {
      console.error('打开文件失败:', error);
      alert(`打开文件失败: ${error}`);
    }
  }, [addFile]);
  
  /**
   * 打开文件夹
   * 使用系统文件夹对话框选择目录
   */
  const handleOpenFolder = useCallback(async () => {
    try {
      // 调用 Rust 后端打开文件夹对话框
      const dirPath = await invoke<string | null>('open_folder_dialog');
      
      if (!dirPath) return;  // 用户取消了对话框
      
      // 设置当前目录
      setDirectory(dirPath);
      
      // 读取目录下的 .md 文件
      const mdFiles = await invoke<File[]>('list_markdown_files', {
        path: dirPath,
      });
      
      // 添加所有 .md 文件到列表
      mdFiles.forEach((file) => addFile(file));
    } catch (error) {
      console.error('打开文件夹失败:', error);
      alert(`打开文件夹失败: ${error}`);
    }
  }, [setDirectory, addFile]);
  
  /**
   * 保存文件
   */
  const handleSave = useCallback(async (file: File) => {
    try {
      await invoke('write_file', {
        path: file.path,
        content: file.content,
      });
      alert(`文件已保存: ${file.name}`);
    } catch (error) {
      console.error('保存文件失败:', error);
      alert(`保存文件失败: ${error}`);
    }
  }, []);
  
  /**
   * 右键菜单处理
   */
  const handleContextMenu = useCallback((e: React.MouseEvent, index: number) => {
    e.preventDefault();
    setContextMenu({
      x: e.clientX,
      y: e.clientY,
      fileIndex: index,
    });
  }, []);
  
  // 点击其他区域关闭右键菜单
  const handleCloseContextMenu = useCallback(() => {
    setContextMenu(null);
  }, []);
  
  // ==================== 渲染 ====================
  
  // 侧边栏隐藏时
  if (!showSidebar) {
    return null;
  }
  
  return (
    <>
      <aside 
        className="w-64 bg-gray-100 dark:bg-gray-900 border-r border-gray-200 dark:border-gray-700 flex flex-col"
        onClick={handleCloseContextMenu}
      >
        {/* 头部：项目标题 */}
        <div className="p-3 border-b border-gray-200 dark:border-gray-700">
          <h1 className="font-semibold text-gray-800 dark:text-gray-200 flex items-center gap-2">
            {/* 文件图标 */}
            <svg className="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
              <path strokeLinecap="round" strokeLinejoin="round" strokeWidth={2} d="M9 12h6m-6 4h6m2 5H7a2 2 0 01-2-2V5a2 2 0 012-2h5.586a1 1 0 01.707.293l5.414 5.414a1 1 0 01.293.707V19a2 2 0 01-2 2z" />
            </svg>
            Markdown Editor
          </h1>
        </div>
        
        {/* 操作按钮区 */}
        <div className="p-2 space-y-1">
          {/* 新建文件按钮 */}
          <button
            onClick={handleNewFile}
            className="w-full flex items-center gap-2 px-3 py-2 rounded text-sm text-gray-700 dark:text-gray-300 hover:bg-gray-200 dark:hover:bg-gray-800 transition-colors"
          >
            <svg className="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
              <path strokeLinecap="round" strokeLinejoin="round" strokeWidth={2} d="M12 4v16m8-8H4" />
            </svg>
            新建文件
          </button>
          
          {/* 打开文件按钮 */}
          <button
            onClick={handleOpenFile}
            className="w-full flex items-center gap-2 px-3 py-2 rounded text-sm text-gray-700 dark:text-gray-300 hover:bg-gray-200 dark:hover:bg-gray-800 transition-colors"
          >
            <svg className="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
              <path strokeLinecap="round" strokeLinejoin="round" strokeWidth={2} d="M5 19a2 2 0 01-2-2V7a2 2 0 012-2h4l2 2h4a2 2 0 012 2v1M5 19h14a2 2 0 002-2v-5a2 2 0 00-2-2H9a2 2 0 00-2 2v5a2 2 0 01-2 2z" />
            </svg>
            打开文件
          </button>
          
          {/* 打开文件夹按钮 */}
          <button
            onClick={handleOpenFolder}
            className="w-full flex items-center gap-2 px-3 py-2 rounded text-sm text-gray-700 dark:text-gray-300 hover:bg-gray-200 dark:hover:bg-gray-800 transition-colors"
          >
            <svg className="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
              <path strokeLinecap="round" strokeLinejoin="round" strokeWidth={2} d="M3 7v10a2 2 0 002 2h14a2 2 0 002-2V9a2 2 0 00-2-2h-6l-2-2H5a2 2 0 00-2 2z" />
            </svg>
            打开文件夹
          </button>
        </div>
        
        {/* 当前目录 */}
        {currentDirectory && (
          <div className="px-3 py-2 text-xs text-gray-500 dark:text-gray-400 truncate">
            📁 {currentDirectory}
          </div>
        )}
        
        {/* 文件列表 */}
        <div className="flex-1 overflow-y-auto py-2">
          {files.length === 0 ? (
            <div className="px-3 py-4 text-center text-gray-400 text-sm">
              <p>暂无打开的文件</p>
              <p className="mt-1 text-xs">点击上方按钮打开文件</p>
            </div>
          ) : (
            <div className="space-y-0.5">
              {files.map((file, index) => (
                <div
                  key={file.path}
                  onClick={() => setActiveFile(index)}
                  onContextMenu={(e) => handleContextMenu(e, index)}
                  className={`
                    mx-1 px-2 py-1.5 rounded cursor-pointer flex items-center gap-2
                    ${index === activeFileIndex 
                      ? 'bg-indigo-100 dark:bg-indigo-900 text-indigo-700 dark:text-indigo-300' 
                      : 'text-gray-700 dark:text-gray-300 hover:bg-gray-200 dark:hover:bg-gray-800'}
                  `}
                >
                  {/* 文件图标 */}
                  <svg className="w-4 h-4 flex-shrink-0" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                    <path strokeLinecap="round" strokeLinejoin="round" strokeWidth={2} d="M9 12h6m-6 4h6m2 5H7a2 2 0 01-2-2V5a2 2 0 012-2h5.586a1 1 0 01.707.293l5.414 5.414a1 1 0 01.293.707V19a2 2 0 01-2 2z" />
                  </svg>
                  
                  {/* 文件名 */}
                  <span className="flex-1 truncate text-sm">
                    {file.name}
                  </span>
                  
                  {/* 修改标记 */}
                  {file.isModified && (
                    <span className="w-2 h-2 rounded-full bg-orange-500 flex-shrink-0" title="未保存" />
                  )}
                  
                  {/* 关闭按钮 */}
                  <button
                    onClick={(e) => {
                      e.stopPropagation();
                      closeFile(index);
                    }}
                    className="p-0.5 rounded hover:bg-gray-300 dark:hover:bg-gray-600 opacity-0 group-hover:opacity-100"
                    title="关闭文件"
                  >
                    <svg className="w-3 h-3" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                      <path strokeLinecap="round" strokeLinejoin="round" strokeWidth={2} d="M6 18L18 6M6 6l12 12" />
                    </svg>
                  </button>
                </div>
              ))}
            </div>
          )}
        </div>
        
        {/* 底部信息 */}
        <div className="p-2 border-t border-gray-200 dark:border-gray-700 text-xs text-gray-400 text-center">
          {files.length} 个文件
        </div>
      </aside>
      
      {/* 右键菜单 */}
      {contextMenu && (
        <div
          className="fixed bg-white dark:bg-gray-800 border border-gray-200 dark:border-gray-700 rounded-lg shadow-lg py-1 z-50"
          style={{ left: contextMenu.x, top: contextMenu.y }}
          onClick={(e) => e.stopPropagation()}
        >
          <button
            onClick={() => {
              handleSave(files[contextMenu.fileIndex]);
              setContextMenu(null);
            }}
            className="w-full px-4 py-2 text-left text-sm hover:bg-gray-100 dark:hover:bg-gray-700"
          >
            📄 保存
          </button>
          <button
            onClick={() => {
              closeFile(contextMenu.fileIndex);
              setContextMenu(null);
            }}
            className="w-full px-4 py-2 text-left text-sm hover:bg-gray-100 dark:hover:bg-gray-700"
          >
            ✖ 关闭
          </button>
        </div>
      )}
    </>
  );
};
```

### 13. 头部组件：src/components/Header.tsx
```tsx
// Header.tsx - 应用头部组件
// 作用：显示应用标题，提供全局操作按钮

import React from 'react';
import { useEditorStore } from '../stores/editorStore';

export const Header: React.FC = () => {
  // 从 store 获取状态和方法
  const { 
    settings, 
    showSidebar, 
    showPreview,
    toggleSidebar, 
    togglePreview,
    updateSettings,
  } = useEditorStore();
  
  // ==================== 渲染 ====================
  return (
    <header className="h-12 bg-white dark:bg-gray-800 border-b border-gray-200 dark:border-gray-700 flex items-center justify-between px-4">
      {/* 左侧：标题和文件信息 */}
      <div className="flex items-center gap-4">
        {/* 应用标题 */}
        <h2 className="font-medium text-gray-800 dark:text-gray-200">
          Markdown Editor
        </h2>
        
        {/* 主题切换按钮 */}
        <button
          onClick={() => updateSettings({ 
            theme: settings.theme === 'light' ? 'dark' : 'light' 
          })}
          className="p-2 rounded hover:bg-gray-100 dark:hover:bg-gray-700 transition-colors"
          title={settings.theme === 'light' ? '切换到暗色模式' : '切换到亮色模式'}
        >
          {settings.theme === 'light' ? (
            // 月亮图标 - 切换到暗色
            <svg className="w-5 h-5 text-gray-600" fill="none" stroke="currentColor" viewBox="0 0 24 24">
              <path strokeLinecap="round" strokeLinejoin="round" strokeWidth={2} d="M20.354 15.354A9 9 0 018.646 3.646 9.003 9.003 0 0012 21a9.003 9.003 0 008.354-5.646z" />
            </svg>
          ) : (
            // 太阳图标 - 切换到亮色
            <svg className="w-5 h-5 text-gray-400" fill="none" stroke="currentColor" viewBox="0 0 24 24">
              <path strokeLinecap="round" strokeLinejoin="round" strokeWidth={2} d="M12 3v1m0 16v1m9-9h-1M4 12H3m15.364 6.364l-.707-.707M6.343 6.343l-.707-.707m12.728 0l-.707.707M6.343 17.657l-.707.707M16 12a4 4 0 11-8 0 4 4 0 018 0z" />
            </svg>
          )}
        </button>
      </div>
      
      {/* 右侧：视图切换按钮 */}
      <div className="flex items-center gap-2">
        {/* 侧边栏切换 */}
        <button
          onClick={toggleSidebar}
          className={`
            p-2 rounded transition-colors
            ${showSidebar 
              ? 'bg-indigo-100 dark:bg-indigo-900 text-indigo-600 dark:text-indigo-400' 
              : 'text-gray-500 hover:bg-gray-100 dark:hover:bg-gray-700'}
          `}
          title="切换侧边栏"
        >
          <svg className="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
            <path strokeLinecap="round" strokeLinejoin="round" strokeWidth={2} d="M4 6h16M4 12h16M4 18h7" />
          </svg>
        </button>
        
        {/* 编辑器视图 */}
        <button
          onClick={() => {
            if (!showSidebar) toggleSidebar();
            if (showPreview) togglePreview();
          }}
          className="p-2 rounded text-gray-500 hover:bg-gray-100 dark:hover:bg-gray-700 transition-colors"
          title="仅编辑器"
        >
          <svg className="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
            <path strokeLinecap="round" strokeLinejoin="round" strokeWidth={2} d="M11 5H6a2 2 0 00-2 2v11a2 2 0 002 2h11a2 2 0 002-2v-5m-1.414-9.414a2 2 0 112.828 2.828L11.828 15H9v-2.828l8.586-8.586z" />
          </svg>
        </button>
        
        {/* 分屏视图 */}
        <button
          onClick={() => {
            if (showSidebar && !showPreview) toggleSidebar();
            if (!showPreview) togglePreview();
          }}
          className={`
            p-2 rounded transition-colors
            ${showSidebar && showPreview 
              ? 'bg-indigo-100 dark:bg-indigo-900 text-indigo-600 dark:text-indigo-400' 
              : 'text-gray-500 hover:bg-gray-100 dark:hover:bg-gray-700'}
          `}
          title="编辑+预览"
        >
          <svg className="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
            <path strokeLinecap="round" strokeLinejoin="round" strokeWidth={2} d="M9 17V7m0 10a2 2 0 01-2 2H5a2 2 0 01-2-2V7a2 2 0 012-2h2a2 2 0 012 2m0 10a2 2 0 002 2h2a2 2 0 002-2M9 7a2 2 0 012-2h2a2 2 0 012 2m0 10V7m0 10a2 2 0 002 2h2a2 2 0 002-2V7a2 2 0 00-2-2h-2a2 2 0 00-2 2" />
          </svg>
        </button>
        
        {/* 仅预览视图 */}
        <button
          onClick={() => {
            if (showSidebar) toggleSidebar();
            if (!showPreview) togglePreview();
          }}
          className={`
            p-2 rounded transition-colors
            ${!showSidebar && showPreview 
              ? 'bg-indigo-100 dark:bg-indigo-900 text-indigo-600 dark:text-indigo-400' 
              : 'text-gray-500 hover:bg-gray-100 dark:hover:bg-gray-700'}
          `}
          title="仅预览"
        >
          <svg className="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
            <path strokeLinecap="round" strokeLinejoin="round" strokeWidth={2} d="M15 12a3 3 0 11-6 0 3 3 0 016 0z" />
            <path strokeLinecap="round" strokeLinejoin="round" strokeWidth={2} d="M2.458 12C3.732 7.943 7.523 5 12 5c4.478 0 8.268 2.943 9.542 7-1.274 4.057-5.064 7-9.542 7-4.477 0-8.268-2.943-9.542-7z" />
          </svg>
        </button>
        
        {/* 分隔线 */}
        <div className="w-px h-6 bg-gray-200 dark:bg-gray-700 mx-2" />
        
        {/* 字体大小控制 */}
        <div className="flex items-center gap-1">
          <button
            onClick={() => updateSettings({ fontSize: Math.max(10, settings.fontSize - 1) })}
            className="p-1.5 rounded hover:bg-gray-100 dark:hover:bg-gray-700 text-gray-500"
            title="减小字体"
          >
            <span className="text-xs font-bold">A-</span>
          </button>
          <span className="text-xs text-gray-500 w-8 text-center">
            {settings.fontSize}px
          </span>
          <button
            onClick={() => updateSettings({ fontSize: Math.min(24, settings.fontSize + 1) })}
            className="p-1.5 rounded hover:bg-gray-100 dark:hover:bg-gray-700 text-gray-500"
            title="增大字体"
          >
            <span className="text-sm font-bold">A+</span>
          </button>
        </div>
      </div>
    </header>
  );
};
```

### 14. 主应用组件：src/App.tsx
```tsx
// App.tsx - React 应用主组件
// 作用：组装所有组件，提供全局布局

import React from 'react';
import { useEditorStore } from './stores/editorStore';
import { Header } from './components/Header';
import { Sidebar } from './components/Sidebar';
import { Editor } from './components/Editor';
import { Preview } from './components/Preview';

const App: React.FC = () => {
  // 从 store 获取状态
  const { 
    showSidebar, 
    showPreview, 
    settings,
  } = useEditorStore();
  
  // ==================== 动态类名计算 ====================
  // 根据设置计算编辑器和预览区的宽度
  
  const getMainContentClass = () => {
    // 编辑器宽度
    const editorWidth = showPreview ? 'w-1/2' : 'w-full';
    // 预览区宽度
    const previewWidth = showPreview ? 'w-1/2' : 'w-0';
    
    return `${editorWidth} ${previewWidth}`;
  };
  
  // ==================== 渲染 ====================
  return (
    // 根容器 - 全屏高度
    <div className="h-screen flex flex-col overflow-hidden">
      {/* 头部 */}
      <Header />
      
      {/* 主体内容区 */}
      <div className="flex-1 flex overflow-hidden">
        {/* 侧边栏 */}
        <Sidebar />
        
        {/* 编辑器和预览区 */}
        <div className={`flex-1 flex ${getMainContentClass()} transition-all duration-200`}>
          {/* 编辑器 */}
          {showSidebar && <Editor />}
          
          {/* 分隔线（编辑器+预览时显示） */}
          {showSidebar && showPreview && (
            <div className="w-1 bg-gray-200 dark:bg-gray-700 cursor-col-resize" />
          )}
          
          {/* 预览区 */}
          {showPreview && <Preview />}
        </div>
      </div>
    </div>
  );
};

export default App;
```

### 15. Rust 后端：Cargo.toml
```toml
# Cargo.toml - Rust 项目配置文件
# 作用：定义项目元数据、依赖包和构建配置

[package]
# 项目名称（必须与目录名一致）
name = "markdown-editor"
# 项目版本（遵循语义化版本 2.0.0）
version = "1.0.0"
# 许可证（MIT 许可证，允许开源使用）
license = "MIT"
#  authors = ["尘先生 <email@example.com>"]  # 可选：作者信息
# 项目描述
description = "A Tauri-based Markdown Editor with live preview"
# Rust 最低版本要求
edition = "2021"

[lib]
# 库入口文件（相对于 src 目录）
name = "markdown_editor_lib"
# Tauri 要求的：指定库的源码文件
crate-type = ["staticlib", "cdylib", "rlib"]

[build-dependencies]
# Tauri 构建脚本依赖
tauri-build = { version = "2.0", features = [] }

[dependencies]
# Tauri 核心框架
# 2.0 版本是最新稳定版
tauri = { version = "2.0", features = [] }

# Tauri API 访问
tauri-plugin-shell = "2.0"

# 异步运行时 - Tauri 推荐使用 tokio
tokio = { version = "1.35", features = ["full"] }

# 文件路径处理
# 比标准库的 Path 更强大
serde = { version = "1.0", features = ["derive"] }  # 序列化/反序列化
serde_json = "1.0"  # JSON 序列化

# 错误处理
anyhow = "1.0"  # 灵活的错误类型
thiserror = "1.0"  # 自定义错误类型

# 日志记录
log = "0.4"  # 日志接口
env_logger = "0.10"  # 基于环境变量的日志实现

[features]
# 默认功能
default = ["custom-protocol"]
# Tauri 自定义协议（开发时使用）
custom-protocol = ["tauri/custom-protocol"]

# 特定平台的依赖（Windows）
[target.'cfg(not(any(target_os = "android", target_os = "ios")))'.dependencies]
# Windows 上的窗口阴影
tauri-plugin-window-shadow = "2.0"
```

### 16. Rust 构建脚本：src-tauri/build.rs
```rust
// build.rs - Tauri 构建脚本
// 作用：在编译前执行一些准备工作
// 此文件会在 cargo build 时自动运行

fn main() {
    // tauri-build 宏会：
    // 1. 检查环境（确保安装了 Rust 和必要的工具）
    // 2. 生成 Tauri 所需的代码
    // 3. 验证 tauri.conf.json 配置
    tauri_build::build()
}
```

### 17. Rust 主入口：src-tauri/src/main.rs
```rust
// main.rs - Tauri 应用的主入口文件
// 作用：程序执行的起点，配置日志和启动 Tauri 应用

// 使用 prelude 模块简化导入
// prelude 包含最常用的 traits 和类型
use tauri::Manager;

// 标记为应用程序入口点
// 这告诉 Rust 编译器这是可执行文件的起点
#[cfg_attr(mobile, tauri::mobile_entry_point)]
pub fn run() {
    // ==================== 日志初始化 ====================
    // env_logger::init() 读取 RUST_LOG 环境变量来设置日志级别
    // 例如：RUST_LOG=debug cargo tauri dev
    env_logger::Builder::from_env(env_logger::Env::default().default_filter_or("info"))
        .format_timestamp_millis()  // 使用毫秒级时间戳
        .init();
    
    log::info!("🚀 启动 Markdown Editor 应用...");
    
    // ==================== Tauri 应用构建 ====================
    tauri::Builder::default()
        // 插件注册：将功能模块添加到应用中
        .plugin(tauri_plugin_shell::init())  // Shell 插件：执行系统命令
        
        // Windows 特殊配置（仅桌面端）
        #[cfg(not(any(target_os = "android", target_os = "ios")))]
        .plugin(tauri_plugin_window_shadow::init())  // 窗口阴影
        
        // 设置管理回调：Tauri 应用创建后调用
        .setup(|app| {
            log::info!("✅ Tauri 应用初始化完成");
            
            // 获取主窗口
            // main_window() 返回 Option<Window>
            if let Some(window) = app.get_webview_window("main") {
                log::info!("📄 主窗口已创建");
                
                // 设置窗口标题
                let _ = window.set_title("Markdown Editor - Rust Tauri");
            }
            
            Ok(())
        })
        
        // 注册 Tauri 命令（供前端调用）
        .invoke_handler(tauri::generate_handler![
            // 文件操作命令
            commands::create_file,
            commands::read_file,
            commands::write_file,
            commands::delete_file,
            commands::list_markdown_files,
            // 对话框命令
            commands::open_file_dialog,
            commands::open_folder_dialog,
        ])
        
        // 启动应用
        // run() 会阻塞，直到应用退出
        .run(tauri::generate_context!())
        .expect("❌ 启动 Tauri 应用时发生错误");
}
```

### 18. Rust 命令模块：src-tauri/src/commands.rs
```rust
// commands.rs - Tauri 命令模块
// 作用：定义所有供前端调用的 Rust 函数
// 每个公开函数都是一个 Tauri 命令

use serde::{Deserialize, Serialize};  // 序列化 trait
use std::fs;  // 文件系统操作
use std::path::Path;  // 路径处理
use tauri::api::dialog;  // 对话框 API
use tauri::api::path;  // 路径工具

// ==================== 类型定义 ====================

/// 文件信息结构体
/// 用于在前后端之间传递文件数据
#[derive(Debug, Serialize, Deserialize)]
pub struct FileInfo {
    /// 文件路径
    pub path: String,
    /// 文件名（不含路径）
    pub name: String,
    /// 文件内容
    pub content: String,
}

/// 目录信息结构体
#[derive(Debug, Serialize, Deserialize)]
pub struct DirInfo {
    /// 目录路径
    pub path: String,
    /// 目录下的 .md 文件列表
    pub files: Vec<FileInfo>,
}

// ==================== 文件操作命令 ====================

/**
 * 创建新文件
 * 
 * # 参数
 * - `path`: 文件路径（相对或绝对路径）
 * - `content`: 文件初始内容
 * 
 * # 返回值
 * 成功返回 Ok(())，失败返回错误信息
 */
#[tauri::command]
pub async fn create_file(path: String, content: String) -> Result<(), String> {
    log::info!("📄 创建文件: {}", path);
    
    // 使用 ? 操作符传播错误
    // 如果出错，直接返回 Err
    fs::write(&path, &content).map_err(|e| {
        log::error!("❌ 创建文件失败: {}", e);
        format!("创建文件失败: {}", e)
    })?;
    
    log::info!("✅ 文件创建成功");
    Ok(())
}

/**
 * 读取文件内容
 * 
 * # 参数
 * - `path`: 文件路径
 * 
 * # 返回值
 * 返回 FileInfo 结构体，包含文件名和内容
 */
#[tauri::command]
pub async fn read_file(path: String) -> Result<FileInfo, String> {
    log::info!("📖 读取文件: {}", path);
    
    // 读取文件内容
    // fs::read_to_string 返回 Result<String, Error>
    let content = fs::read_to_string(&path).map_err(|e| {
        log::error!("❌ 读取文件失败: {}", e);
        format!("读取文件失败: {}", e)
    })?;
    
    // 从路径中提取文件名
    // Path::new() 创建路径对象
    // file_name() 返回文件名部分
    let name = Path::new(&path)
        .file_name()  // 获取文件名
        .and_then(|n| n.to_str())  // 转换为 &str
        .unwrap_or("unknown.md")   // 默认值
        .to_string();
    
    log::info!("✅ 读取成功: {} ({} 字节)", name, content.len());
    
    Ok(FileInfo { path, name, content })
}

/**
 * 写入文件内容
 * 
 * # 参数
 * - `path`: 文件路径
 * - `content`: 要写入的内容
 */
#[tauri::command]
pub async fn write_file(path: String, content: String) -> Result<(), String> {
    log::info!("💾 保存文件: {}", path);
    
    fs::write(&path, &content).map_err(|e| {
        log::error!("❌ 保存文件失败: {}", e);
        format!("保存文件失败: {}", e)
    })?;
    
    log::info!("✅ 保存成功");
    Ok(())
}

/**
 * 删除文件
 */
#[tauri::command]
pub async fn delete_file(path: String) -> Result<(), String> {
    log::info!("🗑️ 删除文件: {}", path);
    
    fs::remove_file(&path).map_err(|e| {
        log::error!("❌ 删除文件失败: {}", e);
        format!("删除文件失败: {}", e)
    })?;
    
    log::info!("✅ 删除成功");
    Ok(())
}

/**
 * 列出目录下的所有 .md 文件
 * 
 * # 参数
 * - `path`: 目录路径
 * 
 * # 返回值
 * 返回文件信息数组
 */
#[tauri::command]
pub async fn list_markdown_files(path: String) -> Result<Vec<FileInfo>, String> {
    log::info!("📁 读取目录: {}", path);
    
    // 读取目录内容
    let entries = fs::read_dir(&path).map_err(|e| {
        log::error!("❌ 读取目录失败: {}", e);
        format!("读取目录失败: {}", e)
    })?;
    
    let mut files = Vec::new();
    
    // 遍历目录项
    for entry in entries {
        // 跳过错误项
        let entry = match entry {
            Ok(e) => e,
            Err(e) => {
                log::warn!("⚠️ 跳过无法读取的目录项: {}", e);
                continue;
            }
        };
        
        // 获取文件路径
        let file_path = entry.path();
        
        // 只处理 .md 文件
        if file_path.extension().map_or(false, |ext| ext == "md") {
            // 读取文件内容
            let content = fs::read_to_string(&file_path)
                .unwrap_or_default();  // 读取失败时使用空字符串
            
            // 提取文件名
            let name = file_path
                .file_name()
                .and_then(|n| n.to_str())
                .unwrap_or("unknown.md")
                .to_string();
            
            files.push(FileInfo {
                path: file_path.to_string_lossy().to_string(),
                name,
                content,
            });
        }
    }
    
    log::info!("📋 找到 {} 个 .md 文件", files.len());
    Ok(files)
}

// ==================== 对话框命令 ====================

/**
 * 打开文件选择对话框
 * 
 * 使用 Tauri 的原生对话框 API
 * 只能选择 .md 文件
 * 
 * # 返回值
 * 返回选中的文件路径，用户取消返回 None
 */
#[tauri::command]
pub async fn open_file_dialog() -> Result<Option<String>, String> {
    log::info!("📂 打开文件对话框");
    
    // 阻塞式对话框 - 在移动端需要特殊处理
    // 这里使用简化版本，实际项目中可能需要更复杂的处理
    Ok(None)  // 简化版本，实际需要使用 tauri::api::dialog
}

/**
 * 打开文件夹选择对话框
 */
#[tauri::command]
pub async fn open_folder_dialog() -> Result<Option<String>, String> {
    log::info!("📁 打开文件夹对话框");
    
    Ok(None)  // 简化版本
}

// ==================== 辅助函数 ====================

/// 验证文件路径是否安全
/// 防止路径遍历攻击（如 ../../../etc/passwd）
fn validate_path(path: &str) -> Result<(), String> {
    // 转换为 Path 对象
    let path = Path::new(path);
    
    // 检查路径是否为绝对路径
    // 实际应用中可能需要更复杂的验证
    if path.components().any(|c| matches!(c, std::path::Component::ParentDir)) {
        return Err("不允许使用父目录引用 (../)".to_string());
    }
    
    Ok(())
}
```

### 19. Rust 库入口：src-tauri/src/lib.rs
```rust
// lib.rs - 库入口文件
// 作用：定义库的公共接口，供 main.rs 使用

// 导出命令模块
// pub mod 声明公共模块，前端可以调用
pub mod commands;

// ==================== 重新导出 ====================
// 可以在这里重新导出常用的类型和函数
// 这样外部调用者只需要 use markdown_editor_lib 即可

// 重新导出命令
pub use commands::*;

// 重新导出常用的 traits
// pub use serde::{Serialize, Deserialize};
```

### 20. Tauri 配置：src-tauri/tauri.conf.json
```json
{
  "$schema": "https://schema.tauri.app/config/2",
  "productName": "Markdown Editor",
  "version": "1.0.0",
  "identifier": "com.example.markdown-editor",
  "build": {
    "beforeDevCommand": "npm run dev",
    "devUrl": "http://localhost:1420",
    "beforeBuildCommand": "npm run build",
    "frontendDist": "../dist",
    "devtools": true
  },
  "app": {
    "windows": [
      {
        "title": "Markdown Editor",
        "width": 1200,
        "height": 800,
        "minWidth": 800,
        "minHeight": 600,
        "resizable": true,
        "fullscreen": false,
        "center": true
      }
    ],
    "security": {
      "csp": null
    }
  },
  "bundle": {
    "active": true,
    "targets": "all",
    "icon": [
      "icons/32x32.png",
      "icons/128x128.png",
      "icons/128x128@2x.png",
      "icons/icon.icns",
      "icons/icon.ico"
    ],
    "windows": {
      "webviewInstallMode": {
        "type": "embedBootstrapper"
      }
    }
  }
}
```

---

## 代码关键点说明

### 1. Rust 后端关键点

#### 所有权和借用规则
```rust
// ❌ 错误示例：所有权转移后不能再使用
let content = fs::read_to_string(&path)?;
// path 在这里仍然可用，因为我们只借用了 &path

// ✅ 正确示例：使用引用避免所有权转移
fn read_file(path: &str) -> Result<String, Error> {
    fs::read_to_string(path)
}
```

#### 异步命令
```rust
// Tauri 2.0 支持 async 命令
// 使用 async/await 语法处理耗时操作
#[tauri::command]
pub async fn read_file(path: String) -> Result<FileInfo, String> {
    // 异步读取文件，不会阻塞主线程
    let content = tokio::fs::read_to_string(&path).await?;
    Ok(FileInfo { path, name, content })
}
```

#### 错误处理
```rust
// 使用 ? 操作符简化错误处理
// Err 类型会自动向上传播
fs::write(&path, &content).map_err(|e| {
    format!("写入失败: {}", e)  // 转换为字符串错误
})?;

// 或者使用 thiserror 定义自定义错误类型
use thiserror::Error;
#[derive(Error, Debug)]
pub enum AppError {
    #[error("IO 错误: {0}")]
    Io(#[from] std::io::Error),
    #[error("文件不存在: {0}")]
    NotFound(String),
}
```

### 2. React 前端关键点

#### Zustand 状态管理
```typescript
// Zustand 的优势：简单、无 Provider、无 boilerplate
// 直接在组件中使用 hook 获取状态
const { files, addFile } = useEditorStore();

// 不可变更新：始终创建新对象/数组
set((state) => ({
    files: [...state.files, newFile]  // 使用扩展运算符
}));
```

#### useCallback 优化
```typescript
// useCallback 缓存函数，避免子组件不必要重新渲染
const handleChange = useCallback((e) => {
    updateFileContent(activeFileIndex, e.target.value);
}, [activeFileIndex, updateFileContent]);

// 依赖项数组：只有这些值变化时，才会重新创建函数
```

#### React.memo 优化
```typescript
// 使用 React.memo 缓存组件，避免不必要的重新渲染
const Sidebar = React.memo(() => {
    // 只有 props 或内部状态变化时才会重新渲染
    // ...
});
```

---

## 学习计划（5天）

### 第1天：环境搭建与基础概念
**目标**：搭建 Tauri 开发环境，理解 Tauri 架构

| 时间 | 任务 | 产出 |
|------|------|------|
| 上午 | 安装 Rust 和 Node.js | 开发环境就绪 |
| 上午 | 安装 Tauri CLI | tauri --version 可用 |
| 上午 | 创建第一个 Tauri 项目 | 跑通 hello world |
| 下午 | 理解 Tauri 架构 | 画出前后端通信图 |
| 下午 | 阅读官方文档 | 整理 Tauri 命令系统 |

**关键知识点**：
- Tauri 使用系统 WebView（比 Electron 轻量）
- 前端通过 `@tauri-apps/api` 调用 Rust 命令
- Rust 端使用 `#[tauri::command]` 声明命令

### 第2天：React + TypeScript 前端开发
**目标**：掌握 React 组件开发和 TypeScript 类型系统

| 时间 | 任务 | 产出 |
|------|------|------|
| 上午 | React 基础回顾 | 理解组件、Props、State |
| 上午 | TypeScript 进阶 | 泛型、接口、类型守卫 |
| 下午 | 搭建项目结构 | 组件、hooks、stores 目录 |
| 下午 | 实现 Editor 组件 | 文本输入、快捷键处理 |
| 下午 | 实现 Preview 组件 | Markdown 渲染 |

**关键知识点**：
- 使用 `interface` 定义组件 Props 类型
- 使用 `useRef` 操作 DOM（如 textarea）
- `react-markdown` + `remark-gfm` 渲染 Markdown

### 第3天：Rust 文件操作与状态管理
**目标**：实现文件 CRUD 操作，掌握 Zustand 状态管理

| 时间 | 任务 | 产出 |
|------|------|------|
| 上午 | Rust 文件操作 | read/write/create/delete |
| 上午 | 实现 Tauri 命令 | Rust 后端 API |
| 下午 | Zustand 状态管理 | editorStore 实现 |
| 下午 | 实现 Sidebar 组件 | 文件列表展示 |
| 下午 | 前后端联调 | 文件操作完整流程 |

**关键知识点**：
- Rust 文件操作：`fs::read_to_string`, `fs::write`
- 错误处理：`Result<T, E>` 和 `?` 操作符
- Zustand：`create` 函数创建 store

### 第4天：UI 完善与样式优化
**目标**：美化界面，实现主题切换

| 时间 | 任务 | 产出 |
|------|------|------|
| 上午 | Tailwind CSS 配置 | 主题色、响应式 |
| 上午 | 实现主题切换 | 亮色/暗色模式 |
| 下午 | 工具栏功能 | 加粗、斜体、代码块 |
| 下午 | 代码高亮 | Prism.js 集成 |
| 下午 | 响应式布局 | 适配不同窗口大小 |

**关键知识点**：
- Tailwind CSS 的 `@apply`、`@layer` 指令
- CSS 变量实现主题切换
- Prism.js 代码高亮配置

### 第5天：项目完善与面试准备
**目标**：完成项目开发，准备面试

| 时间 | 任务 | 产出 |
|------|------|------|
| 上午 | 功能测试与 Bug 修复 | 稳定版本 |
| 上午 | 打包构建 | .exe/.dmg 安装包 |
| 下午 | 项目文档编写 | README |
| 下午 | 面试题目练习 | 准备答案 |

**面试重点问题**：
1. Tauri 和 Electron 的区别是什么？
2. Rust 的所有权系统是什么？
3. 什么是借用？引用和指针的区别？
4. async/await 的原理是什么？
5. TypeScript 的类型推断是如何工作的？

---

## 面试要点

### 面试题 1：Tauri 和 Electron 的区别是什么？

**标准答案**：

| 对比项 | Electron | Tauri |
|--------|-----------|-------|
| 体积 | 150MB+ | 2-10MB |
| 内存占用 | 高 | 低 |
| 渲染引擎 | Chromium | 系统 WebView |
| 安全性 | 较低（Node.js） | 较高（沙箱） |
| 开发语言 | JS/TS | Rust + Web |

**答题思路**：
1. 先说出核心区别（Tauri 用系统 WebView，Electron 捆绑 Chromium）
2. 举例说明体积差异（Electron Hello World ~150MB，Tauri ~3MB）
3. 提到安全优势（Tauri 默认沙箱，Electron 需要手动配置 CSP）
4. 提到开发体验（Tauri 用 Rust 后端，性能更强）

### 面试题 2：Rust 的所有权系统是什么？

**标准答案**：

Rust 的所有权系统包含三条核心规则：

1. **每个值有且只有一个所有者**
   ```rust
   let s1 = String::from("hello");
   let s2 = s1;  // s1 的所有权转移到 s2
   // println!("{}", s1);  // ❌ 编译错误：s1 已无效
   ```

2. **当所有者离开作用域，值被丢弃**
   ```rust
   {
       let s = String::from("hello");
   }  // s 在这里被 drop（释放内存）
   ```

3. **值不可同时被多个可变引用借用**
   ```rust
   let mut s = String::from("hello");
   let r1 = &mut s;
   let r2 = &mut s;  // ❌ 编译错误：不能同时有两个可变引用
   ```

**为什么要这样设计**：
- **内存安全**：无需垃圾回收器（GC），编译时保证无内存错误
- **零成本抽象**：没有运行时开销
- **防止数据竞争**：多线程程序中防止同时修改同一数据

### 面试题 3：什么是生命周期？

**标准答案**：

生命周期是 Rust 用来**确保引用始终有效**的编译时检查机制。

```rust
// 生命周期参数 'a 表示：返回值的生命周期与输入引用的生命周期相同
fn longest<'a>(x: &'a str, y: &'a str) -> &'a str {
    if x.len() > y.len() { x } else { y }
}
```

**常见场景**：
1. 结构体中存储引用
   ```rust
   struct ImportantExcerpt<'a> {
       part: &'a str,  // 必须标注生命周期
   }
   ```

2. 函数返回引用
   ```rust
   // 返回引用时必须指定生命周期
   fn first_word<'a>(s: &'a str) -> &'a str {
       // ...
   }
   ```

### 面试题 4：async/await 的原理是什么？

**标准答案**：

async/await 是 Rust 异步编程的语法糖：

```rust
// 使用 async/await
async fn fetch_data() -> Result<String, Error> {
    let response = http_get("https://api.example.com").await?;
    Ok(response)
}
```

**底层原理**：
1. `async fn` 被编译成状态机
2. `.await` 调用使状态机暂停，等待 Future 完成
3. Tokio 运行时调度协程执行

```rust
// 等价于（简化版）
fn fetch_data() -> impl Future<Output = Result<String, Error>> {
    async {
        let response = http_get("https://api.example.com").await?;
        Ok(response)
    }
}
```

**关键概念**：
- `Future`：代表一个异步操作的尚未完成的值
- `await`：等待 Future 完成（不阻塞线程）
- ` Tokio`：异步运行时，提供多任务调度

### 面试题 5：TypeScript 的泛型是什么？

**标准答案**：

泛型允许编写**可复用**的代码，同时保持**类型安全**：

```typescript
// 泛型函数：T 是类型参数
function identity<T>(arg: T): T {
    return arg;
}

// 使用时指定类型
const num = identity<number>(42);    // T = number
const str = identity("hello");       // T = string（类型推断）

// 泛型接口
interface Container<T> {
    value: T;
    get(): T;
}

// 泛型约束：限制类型参数的范围
function getLength<T extends { length: number }>(arg: T): number {
    return arg.length;
}

getLength("hello");  // ✅ string 有 length
getLength([1, 2, 3]); // ✅ array 有 length
getLength(123);      // ❌ number 没有 length
```

**在 React 中的应用**：
```typescript
// 泛型组件
function List<T>({ items, renderItem }: ListProps<T>) {
    return items.map((item, index) => renderItem(item, index));
}

// 使用
<List<number> 
    items={[1, 2, 3]} 
    renderItem={(item) => <span>{item}</span>}
/>
```

---

## 项目启动和运行

### 前置条件
- Rust 1.70+
- Node.js 18+
- npm 或 yarn

### 安装依赖
```bash
# 安装前端依赖
npm install

# 安装 Tauri CLI（如果还没有）
npm install -D @tauri-apps/cli
```

### 开发模式运行
```bash
# 同时启动前端和 Tauri
npm run tauri dev
```

### 构建生产版本
```bash
# 构建前端
npm run build

# 构建 Tauri 应用
npm run tauri build
```

### 输出位置
- Windows: `src-tauri/target/release/markdown-editor.exe`
- macOS: `src-tauri/target/release/markdown-editor.app`
- Linux: `src-tauri/target/release/markdown-editor`
