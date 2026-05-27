# PageBuilder

PageBuilder 是一个基于 Vue 3 + TypeScript 的低代码可视化页面编辑器，面向营销落地页、活动报名页和表单收集页等场景。支持组件拖拽、画布编辑、属性配置、图层管理、撤销重做、实时预览以及 JSON 导入导出。

<img width="1414" height="651" alt="image" src="https://github.com/user-attachments/assets/71a18da8-371e-43a3-8240-a80f7147ab6e" />


当前界面采用低饱和莫兰迪色调，整体风格偏工作台产品：信息密度适中、面板结构清晰、适合长期编辑使用。

## 功能特性

- 可视化拖拽：从左侧组件库拖入文本、图片、按钮、输入框、表单等组件。
- 画布编辑：支持组件移动、缩放、选中态、方向键微调、缩放画布和适配视口。
- 网格与辅助线：支持网格吸附和画布边缘/中心辅助线，便于快速对齐。
- 属性配置：右侧面板可编辑页面信息、页面尺寸、背景色、组件位置、尺寸、样式、内容和事件。
- 图层管理：支持组件上移、下移、置顶、置底，并在左侧展示图层列表。
- 撤销重做：基于命令模式管理操作历史，覆盖组件增删改、图层调整、页面信息和页面样式修改。
- 预览能力：支持在弹窗中实时预览最终页面效果。
- 数据导入导出：页面结构可导出为 JSON，也可从 JSON 文件导入恢复。
- 本地持久化：点击保存后写入 `localStorage`，刷新页面后可继续编辑。

## 技术栈

- Vue 3
- TypeScript
- Vite
- Pinia
- Vue Router
- Element Plus
- HTML5 Drag and Drop API

## 快速开始

项目要求 Node.js 版本满足：

```bash
^20.19.0 || >=22.12.0
```

安装依赖：

```bash
npm install
```

启动开发服务：

```bash
npm run dev
```

生产构建：

```bash
npm run build
```

本地预览构建产物：

```bash
npm run preview
```

代码检查与格式化：

```bash
npm run lint
npm run format
```

## 使用说明

1. 从左侧组件资产库拖拽组件到中间画布。
2. 点击画布中的组件，在右侧配置面板修改位置、尺寸、颜色、文字、事件等属性。
3. 使用画布顶部工具栏调整缩放比例、开启网格吸附或辅助线。
4. 在左侧图层管理中选择组件，或在右侧面板调整图层顺序。
5. 点击顶部预览按钮查看页面最终效果。
6. 点击保存将页面写入本地缓存，点击导出 JSON 可保存页面 schema。

## 项目结构

```text
src/
├── assets/
│   ├── base.css              # 全局设计 token 与基础样式
│   └── main.css              # Element Plus 覆盖样式与应用样式
├── components/
│   ├── Editor.vue            # 编辑器主框架
│   ├── ComponentPanel.vue    # 左侧组件库与图层列表
│   ├── EditorCanvas.vue      # 中间画布、拖拽、缩放、辅助线
│   ├── PropertyPanel.vue     # 右侧页面/组件配置面板
│   └── components/
│       ├── registry.ts       # 组件协议与默认配置
│       ├── TextComponent.vue
│       ├── ImageComponent.vue
│       ├── ButtonComponent.vue
│       ├── InputComponent.vue
│       └── FormComponent.vue
├── stores/
│   ├── editor.ts             # 页面数据、组件操作、持久化
│   └── history.ts            # 撤销/重做命令栈
├── types/
│   └── index.ts              # 页面 schema 与组件类型定义
├── router/
│   └── index.ts
└── main.ts
```

## 核心设计

### 页面 Schema

页面由 `PageData` 描述，包含页面元信息、页面样式和组件数组。每个组件由 `ComponentData` 描述，包含：

- `id`：组件唯一标识
- `type`：组件类型
- `name`：图层展示名称
- `style`：位置、尺寸、旋转、透明度、字体、边框等样式
- `props`：组件业务属性
- `events`：点击事件配置
- `schemaVersion`：schema 版本

### 组件协议

组件默认配置集中在 `src/components/components/registry.ts`。新增组件时主要需要：

1. 在 `ComponentType` 中增加类型。
2. 新增对应渲染组件。
3. 在 `componentRendererMap` 注册渲染器。
4. 在 `componentProtocols` 增加默认样式、默认属性和配置 schema。

### 撤销/重做

历史记录使用命令模式实现。每个可撤销操作都包含：

- `execute()`：执行操作
- `undo()`：回滚操作
- `label`：操作说明

当前已接入历史栈的操作包括新增组件、删除组件、组件样式修改、组件属性修改、组件事件修改、图层调整、页面信息修改和页面样式修改。

### 拖拽与画布坐标

组件拖入画布时会根据 `getBoundingClientRect()` 获取画布视觉坐标，并结合当前缩放比例换算成逻辑坐标。开启网格吸附后，坐标会按 `GRID_SIZE` 对齐。

### 持久化与导入导出

- `persistPage()`：将当前页面写入 `localStorage`
- `loadPersistedPage()`：应用启动时读取本地页面
- `exportPageData()`：导出格式化 JSON
- `importPageData()`：从 JSON 恢复页面，并做基础 schema 归一化

## 视觉主题

项目使用一套低饱和莫兰迪色调，设计 token 位于 `src/assets/base.css`：

```css
--color-background: #eeece6;
--color-surface: #faf8f3;
--color-primary: #6f8583;
--color-primary-strong: #526b69;
--color-accent: #9b7467;
--color-heading: #3f4746;
--color-text: #596261;
--color-border: #d5cec2;
```

如果需要换主题，优先修改这些 token，而不是在业务组件里分散改颜色。

## 常用脚本

| 命令 | 说明 |
| --- | --- |
| `npm run dev` | 启动 Vite 开发服务 |
| `npm run build` | 类型检查并构建生产产物 |
| `npm run preview` | 预览生产构建 |
| `npm run type-check` | 执行 Vue TypeScript 类型检查 |
| `npm run lint` | 执行 oxlint 和 eslint 修复 |
| `npm run format` | 格式化 `src/` 目录 |

## 后续可扩展方向

- 增加组件组合、分组和锁定能力
- 支持拖拽排序图层
- 增加模板市场和页面模板保存
- 增加组件复制、粘贴、对齐分布能力
- 支持更多营销组件，如倒计时、优惠券、轮播图、商品卡片
- 支持导出为 Vue SFC 或静态 HTML

## License

MIT
