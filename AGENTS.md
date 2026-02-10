# 项目说明：GitHub 仓库下载器

## 项目概述

这是一个 Tampermonkey 用户脚本（油猴脚本），用于在 GitHub 代码页面添加选择性下载功能。用户可以选择单个文件或整个目录，将它们打包成 ZIP 格式下载。该脚本支持递归下载子目录，并提供 GitHub Token 认证以提高 API 速率限制。

**当前版本：v1.1** - 已实现智能按钮交互系统升级

### 主要功能

#### 核心下载功能
- **选择性下载**：用户可以选择单个文件或多个文件进行下载
- **目录递归下载**：支持选择整个目录，自动递归下载所有子目录中的文件
- **ZIP 打包**：使用 JSZip 库将选中的文件打包成单个 ZIP 文件
- **GitHub Token 支持**：支持配置 GitHub Personal Access Token，提高 API 请求速率限制
- **实时刷新**：可以随时刷新文件列表以获取最新状态
- **可视化控制面板**：提供浮动控制面板，方便用户操作

#### 智能按钮交互系统（v1.1 新增）
- **拖拽移动**：用户可以按住按钮拖动到屏幕任意位置，支持 3px 位移阈值区分拖拽和点击
- **边缘吸附**：当按钮距离左右边缘 ≤ 20px 时自动吸附到边缘，对齐到边界
- **自动半隐藏**：按钮停靠到边缘后，延迟 400ms 自动隐藏 50%，鼠标悬停时立即展开
- **智能面板定位**：控制面板根据按钮位置自动选择最佳展开方向（上左、上右、下左、下右）
- **状态持久化**：使用 GM_setValue/GM_getValue 保存按钮位置、停靠状态、用户偏好设置
- **窗口响应式**：浏览器窗口大小改变时自动检查按钮位置有效性，超出边界则重置到默认位置
- **用户控制菜单**：通过 GM_registerMenuCommand 提供重置位置、切换设置、查看状态等功能

### 技术栈

- **JavaScript**：纯 JavaScript (ES5+) 实现，无需构建工具
- **Tampermonkey API**：
  - `GM_download`：文件下载
  - `GM_xmlhttpRequest`：跨域 HTTP 请求
  - `GM_getValue` / `GM_setValue`：持久化存储
  - `GM_registerMenuCommand`：注册右键菜单命令
- **第三方库**：
  - [JSZip 3.9.1](https://cdnjs.cloudflare.com/ajax/libs/jszip/3.9.1/jszip.min.js)：ZIP 文件创建

### 项目结构

```
e:\Tampermonkey\GitHub-Downloader\
├── GitHub_Downloader-1.0.user.js    # 主脚本文件（v1.1）
├── AGENTS.md                        # 本文件
├── .gitignore                       # Git 忽略配置
├── Button Smart Interaction Upgrade Design.md  # 智能按钮交互设计文档
├── button-prototype.html            # 按钮交互原型测试页面
├── log/                             # 日志目录（已忽略）
├── .opencode/                       # OpenCode 配置目录
├── .qoder/                          # Qoder 配置目录
└── openspec/                        # OpenSpec 规范目录
    ├── config.yaml                  # OpenSpec 配置
    └── changes/                     # 变更记录
        └── button-smart-interaction/  # 智能按钮交互变更
            ├── .openspec.yaml
            ├── design.md            # 设计文档
            ├── proposal.md          # 提案文档
            ├── tasks.md             # 任务清单
            └── specs/               # 功能规范
                ├── auto-hiding/
                ├── button-dragging/
                ├── edge-snapping/
                ├── menu-commands/
                ├── smart-panel-positioning/
                └── state-persistence/
```

## 核心代码架构

### 主要模块

#### 1. URL 解析模块 (`parseGitHubUrl`)
- 从当前页面 URL 解析仓库信息（所有者、仓库名、分支、路径）
- 支持从页面 DOM 元素中检测当前分支
- 兼容新版 GitHub 的分支选择器结构

#### 2. 文件列表获取模块 (`getFileListFromPage`)
- 从 GitHub 页面 DOM 中提取文件和目录列表
- 支持新版 GitHub 的 `react-directory-row` 结构
- 处理 `/blob/` 和 `/tree/` 链接

#### 3. GitHub API 模块
- `getFilesFromDirectory`：通过 GitHub API 获取目录内容
- `collectAllFiles`：递归收集目录中的所有文件
- 支持 GitHub Token 认证
- 自动重试机制（main 分支失败时尝试 master）

#### 4. 下载处理模块
- `downloadFileContent`：下载文件内容
- `createAndDownloadZip`：创建并下载 ZIP 文件
- 支持并发下载控制（最大并发数：3）
- 流式处理大文件

#### 5. UI 模块
- `createControlPanel`：创建浮动控制面板
- `renderFileList`：渲染文件选择列表
- Token 管理界面

#### 6. 智能按钮交互系统（v1.1 新增）

**6.1 状态管理模块**
- `saveState()`：保存按钮状态到持久化存储
- `loadState()`：从持久化存储加载按钮状态
- `applyState()`：应用加载的状态到按钮 DOM 元素
- `resetButton()`：重置按钮到默认位置和状态

**6.2 拖拽处理模块**
- `onMouseDown()`：鼠标按下事件处理，开始拖拽
- `onMouseMove()`：鼠标移动事件处理，拖拽中更新位置
- `onMouseUp()`：鼠标释放事件处理，结束拖拽
- `clampPosition()`：限制按钮位置在可视区域内
- 拖拽与点击使用 3px 位移阈值区分

**6.3 边缘吸附模块**
- `checkEdgeSnap()`：检测按钮是否接近边缘（阈值 20px）
- `applyDocked()`：应用停靠状态，对齐到边缘
- `clearDocked()`：清除停靠状态
- 支持左右边缘吸附（不支持上下边缘）

**6.4 自动隐藏模块**
- `scheduleHide()`：安排按钮自动隐藏（延迟 400ms）
- `cancelHide()`：取消待执行的自动隐藏
- `hideButton()`：隐藏按钮（向边缘平移 50%）
- `showButton()`：显示按钮（恢复到完全可见）
- `toggleAutoHide()`：切换自动隐藏功能开关

**6.5 智能面板定位模块**
- `calculatePanelDirection()`：计算面板的最佳展开方向
- `updatePanelPosition()`：更新控制面板的位置
- `clampPanelPosition()`：限制面板位置在可视区域内
- 支持四个方向：up-left, up-right, down-left, down-right

**6.6 菜单命令模块**
- `registerMenuCommands()`：注册所有右键菜单命令
- 提供重置位置、切换自动隐藏、查看状态、清除存储、切换调试模式等功能

**6.7 窗口响应式模块**
- `onWindowResize()`：窗口大小变化事件处理
- 检查按钮是否超出可视区域，超出则重置
- 更新面板位置

### 关键数据结构

#### CONFIG 配置对象
```javascript
const CONFIG = {
    SNAP_THRESHOLD: 20,        // 吸附阈值（像素）
    HIDE_DELAY: 400,           // 隐藏延迟（毫秒）
    ANIMATION_DURATION: 200,   // 动画时长（毫秒）
    HIDE_RATIO: 0.5,           // 隐藏比例（0-1）
    BUTTON_SIZE: 50,           // 按钮尺寸（像素）
    PANEL_WIDTH: 300,          // 面板宽度（像素）
    PANEL_HEIGHT: 400,         // 面板高度（像素）
    PANEL_GAP: 10,             // 面板与按钮间距（像素）
    DRAG_THRESHOLD: 3,         // 拖拽判定阈值（像素）
    SUPPORTED_EDGES: ['left', 'right'],  // 支持的停靠边缘
    DEFAULT_RIGHT: 20,         // 默认右边距（像素）
    DEFAULT_BOTTOM: 30         // 默认底边距（像素）
};
```

#### state 状态对象
```javascript
const state = {
    x: null,                   // 按钮水平位置
    y: null,                   // 按钮垂直位置
    docked: false,             // 是否停靠
    edge: null,                // 停靠边缘：'left' | 'right'
    hidden: false,             // 是否半隐藏
    isDragging: false,         // 是否正在拖拽
    hasDragged: false,         // 是否发生了有效拖拽
    dragStartX: 0,             // 拖拽起始鼠标 X 坐标
    dragStartY: 0,             // 拖拽起始鼠标 Y 坐标
    buttonStartX: 0,           // 拖拽起始按钮 X 坐标
    buttonStartY: 0,           // 拖拽起始按钮 Y 坐标
    isMouseOver: false,        // 鼠标是否在按钮上
    panelOpen: false,          // 面板是否打开
    panelDirection: null,      // 面板展开方向
    autoHideEnabled: true,     // 是否启用自动隐藏
    hideTimer: null            // 隐藏计时器引用
};
```

### 关键设计决策

#### 下载功能设计
- **并发控制**：限制同时下载的文件数量为 3，避免触发 GitHub API 速率限制
- **错误处理**：对单个文件下载失败进行容错处理，不影响其他文件
- **递归深度限制**：最大递归深度为 10，防止无限循环
- **超时保护**：每个请求和 ZIP 生成都有超时机制
- **流式处理**：使用 JSZip 的流式处理功能，减少内存占用

#### 智能按钮交互设计
- **基于 transform 的半隐藏**：使用 `transform: translateX()` 实现半隐藏，性能更好，不会触发重排
- **状态机管理**：使用集中式的 `state` 对象管理所有交互状态，避免竞态条件
- **拖拽与点击区分**：使用 3px 位移阈值区分拖拽和点击操作
- **四象限面板定位**：将屏幕分为四象限，根据按钮位置和空间可用性选择面板展开方向
- **延迟重置标志**：在 mouseup 中使用 `setTimeout(..., 0)` 延迟重置 `hasDragged` 标志，确保 click 事件能检测到拖拽
- **原生菜单控制**：使用 `GM_registerMenuCommand` 提供用户控制，符合用户脚本使用习惯

## 安装和使用

### 安装方式

1. 安装 [Tampermonkey](https://www.tampermonkey.net/) 浏览器扩展
2. 访问 Greasy Fork 脚本页面：https://greasyfork.org/scripts/556352
3. 点击"安装此脚本"按钮

或手动安装：
1. 复制 `GitHub_Downloader-1.0.user.js` 文件内容
2. 在 Tampermonkey 中创建新脚本并粘贴内容
3. 保存脚本

### 使用方法

#### 基本下载功能
1. 访问任意 GitHub 仓库代码页面
2. 页面右下角会出现一个 📦 按钮
3. 点击按钮打开控制面板
4. 在面板中勾选需要下载的文件或目录
5. 点击"下载"按钮，脚本会自动创建并下载 ZIP 文件

#### 智能按钮交互功能（v1.1 新增）

**拖拽移动**
- 按住按钮并拖动到屏幕任意位置
- 拖拽过程中按钮实时跟随鼠标
- 按钮限制在可视区域内，不会拖出浏览器窗口

**边缘吸附**
- 将按钮拖到屏幕左边缘或右边缘
- 距离边缘 ≤ 20px 时自动吸附对齐
- 吸附后按钮对齐到边缘（left: 0 或 right: 0）

**自动半隐藏**
- 按钮停靠到边缘后，延迟 400ms 自动隐藏 50%
- 隐藏时按钮向边缘方向平移 25px
- 鼠标悬停在按钮上时，立即完整显示（无延迟）
- 鼠标离开按钮后，如果面板未打开，延迟 400ms 重新隐藏

**智能面板定位**
- 控制面板根据按钮位置自动选择最佳展开方向
- 支持四个方向：上左、上右、下左、下右
- 确保面板不超出可视区域
- 拖拽按钮时，面板实时更新位置

#### 右键菜单命令功能（v1.1 新增）

在 GitHub 页面上右键点击，选择"用户脚本"菜单，找到"GitHub 仓库下载器"子菜单：

1. **🔄 重置按钮位置**：恢复按钮到默认位置和状态
2. **⏱️ 切换自动隐藏**：开启/关闭自动隐藏功能
3. **📊 查看当前状态**：显示按钮和面板的当前状态信息
4. **🗑️ 清除状态存储**：删除所有保存的状态，按钮恢复到默认位置
5. **🔧 切换调试模式**：开启/关闭调试日志（刷新页面后保持设置）

### 配置 GitHub Token（可选）

为了提高 API 速率限制，建议配置 GitHub Token：

1. 在控制面板中点击"Token 未设置"区域
2. 点击"申请"按钮跳转到 GitHub Token 创建页面
3. 创建具有 `repo` 权限的 Token
4. 将 Token 粘贴到输入框并保存

**注意**：
- 无 Token：60 次/小时
- 有 Token：5000 次/小时

## 调试模式

脚本内置了调试日志功能，可通过以下方式启用：

#### 方式一：使用右键菜单（推荐）
1. 在 GitHub 页面上右键点击
2. 选择"用户脚本" → "GitHub 仓库下载器"
3. 点击"🔧 切换调试模式"
4. 刷新页面后，调试日志将输出到浏览器控制台

#### 方式二：使用控制台（临时）
```javascript
// 启用调试模式（仅当前页面有效）
GM_setValue('debug_enabled', true);

// 禁用调试模式（仅当前页面有效）
GM_setValue('debug_enabled', false);
```

启用后，所有操作日志会输出到浏览器控制台，日志格式：
```
[GitHub Downloader] HH:MM:SS: 日志内容
```

## 开发约定

### 代码风格

- 使用纯 JavaScript (ES5+)，避免使用需要编译的特性
- 使用模板字符串和箭头函数提高可读性
- 严格的错误处理，避免脚本崩溃
- 使用 JSDoc 注释为公共函数添加文档说明

### 命名约定

- 函数名：camelCase（如 `saveState`、`onMouseDown`）
- 常量：UPPER_SNAKE_CASE（如 `CONFIG`、`SNAP_THRESHOLD`）
- 变量名：camelCase（如 `state`、`toggleBtn`）
- CSS 类名：kebab-case（如 `toggle-btn`、`dragging`、`docked`）

### 日志约定

- `log()`：普通信息日志
- `error()`：错误日志
- 所有日志都带有时间戳和前缀 `[GitHub Downloader]`
- 使用 `GM_getValue('debug_enabled', false)` 持久化调试开关

### 状态管理约定

- 使用集中式的 `state` 对象管理所有交互状态
- 使用 `saveState()` 和 `loadState()` 函数实现状态持久化
- 状态变更后立即调用 `saveState()` 保存
- 使用 `GM_setValue('buttonState', JSON.stringify(stateToSave))` 保存状态

### 事件处理约定

- 拖拽事件序列：mousedown → mousemove → mouseup
- 使用 `DRAG_THRESHOLD`（3px）区分拖拽和点击
- 在 `onButtonClick()` 中检查 `state.hasDragged` 防止拖拽时误触面板
- 使用 `setTimeout(..., 0)` 延迟重置 `hasDragged` 标志

### CSS 样式约定

- 使用内联样式设置初始样式（兼容用户脚本环境）
- 使用 CSS 类添加交互状态样式（如 `.dragging`、`.docked`）
- 拖拽时禁用 transition 以确保实时响应
- 使用 `transform` 而非 `left/right` 实现动画效果（性能优化）

### 修改注意事项

1. **页面结构变化**：GitHub 可能会更新 DOM 结构，导致文件列表获取失败。如果发现脚本无法正常工作，需要检查 `getFileListFromPage` 函数中的选择器。

2. **API 变化**：GitHub API 可能会更新端点或参数。如果遇到 API 错误，需要检查 API 调用逻辑。

3. **版本更新**：修改脚本后，需要更新 `@version` 元数据字段。

4. **GM API 依赖**：智能按钮交互系统依赖 `GM_setValue`、`GM_getValue`、`GM_registerMenuCommand`，确保在脚本头部正确声明 `@grant`。

5. **状态兼容性**：修改 `state` 对象结构时，需要在 `loadState()` 中处理旧版本状态的兼容性。

6. **事件冲突**：添加全局事件监听器时，注意与其他脚本或页面原生事件的冲突，合理使用 `preventDefault` 和 `stopPropagation`。

## 已知限制

### 下载功能限制
1. **API 速率限制**：无 Token 时限制为 60 次/小时，大仓库可能无法完全下载
2. **文件大小**：超大文件可能导致 ZIP 生成超时
3. **私有仓库**：需要配置具有相应权限的 GitHub Token
4. **网络环境**：某些网络环境可能无法访问 GitHub API

### 智能按钮交互限制（v1.1）
1. **边缘支持**：仅支持左右边缘吸附，不支持上下边缘
2. **触摸设备**：当前仅实现鼠标事件，不支持触摸设备（移动端）
3. **GM API 依赖**：状态持久化和菜单功能依赖 Tampermonkey API，其他脚本管理器可能不支持
4. **配置固定**：吸附阈值、隐藏延迟等参数为固定值，不可通过界面调整
5. **状态迁移**：修改 `state` 对象结构时，旧版本状态可能无法正确加载

## 相关链接

- 脚本主页：https://greasyfork.org/scripts/556352
- GitHub Token 申请：https://github.com/settings/tokens
- Tampermonkey 官网：https://www.tampermonkey.net/
- JSZip 文档：https://stuk.github.io/jszip/
- 按钮交互原型测试页面：`/button-prototype.html`

## 常见问题

### 下载功能相关

#### Q: 为什么下载失败？

A: 可能的原因：
- API 速率限制（请配置 GitHub Token）
- 网络连接问题
- 文件或目录不存在
- Token 权限不足（对于私有仓库）

#### Q: 如何下载整个仓库？

A: 使用 GitHub 官方的 "Code" -> "Download ZIP" 功能。本脚本适用于选择性下载部分文件或目录。

#### Q: 为什么显示的文件列表为空？

A: 请确保：
- 当前页面是 GitHub 代码页面（不是 issues、pull requests 等）
- GitHub 页面完全加载完成
- 刷新控制面板（点击刷新按钮）

### 智能按钮交互相关（v1.1）

#### Q: 按钮拖拽到边缘后自动隐藏了，如何找回？

A: 有以下几种方式：
- 将鼠标移动到屏幕左边缘或右边缘，按钮会自动展开
- 使用右键菜单："用户脚本" → "GitHub 仓库下载器" → "🔄 重置按钮位置"
- 使用右键菜单："用户脚本" → "GitHub 仓库下载器" → "📊 查看当前状态" 查看按钮位置

#### Q: 如何禁用自动隐藏功能？

A: 使用右键菜单：
1. 在 GitHub 页面上右键点击
2. 选择"用户脚本" → "GitHub 仓库下载器"
3. 点击"⏱️ 切换自动隐藏"

#### Q: 按钮位置混乱或找不到按钮，如何恢复？

A: 使用右键菜单：
1. 在 GitHub 页面上右键点击
2. 选择"用户脚本" → "GitHub 仓库下载器"
3. 点击"🔄 重置按钮位置" 或 "🗑️ 清除状态存储"

#### Q: 为什么拖拽按钮时面板会跟着移动？

A: 这是正常行为。当控制面板打开时，拖拽按钮会实时更新面板位置，确保面板始终可见且跟随按钮。

#### Q: 如何查看调试日志？

A: 使用右键菜单：
1. 在 GitHub 页面上右键点击
2. 选择"用户脚本" → "GitHub 仓库下载器"
3. 点击"🔧 切换调试模式"
4. 刷新页面后，打开浏览器控制台查看日志

#### Q: 脚本在 Greasemonkey/Violentmonkey 上能正常使用吗？

A: 智能按钮交互系统依赖以下 Tampermonkey API：
- `GM_setValue` / `GM_getValue`：状态持久化
- `GM_registerMenuCommand`：右键菜单命令

如果您的脚本管理器不支持这些 API，按钮仍可使用，但无法保存状态和使用菜单命令。

## 更新日志

### v1.1 (智能按钮交互系统升级)
**新增功能：**
- ✨ 拖拽移动功能：支持按住按钮拖动到屏幕任意位置
- ✨ 边缘吸附功能：距离左右边缘 ≤ 20px 时自动吸附对齐
- ✨ 自动半隐藏功能：停靠后延迟 400ms 隐藏 50%，悬停时立即展开
- ✨ 智能面板定位：根据按钮位置自动选择最佳展开方向（四方向）
- ✨ 状态持久化：保存按钮位置、停靠状态、用户偏好设置
- ✨ 窗口响应式：窗口大小变化时自动检查按钮位置有效性
- ✨ 右键菜单命令：提供重置位置、切换设置、查看状态等功能
- ✨ 调试日志改进：使用 GM_setValue 持久化调试开关，刷新页面后保持设置

**技术改进：**
- 🎨 新增 CSS 样式类：`.dragging`、`.docked`、`.docked:hover`
- 🏗️ 新增状态管理模块：`saveState`、`loadState`、`applyState`、`resetButton`
- 🏗️ 新增拖拽处理模块：`onMouseDown`、`onMouseMove`、`onMouseUp`
- 🏗️ 新增边缘吸附模块：`checkEdgeSnap`、`applyDocked`、`clearDocked`
- 🏗️ 新增自动隐藏模块：`scheduleHide`、`cancelHide`、`hideButton`、`showButton`
- 🏗️ 新增智能面板定位模块：`calculatePanelDirection`、`updatePanelPosition`
- 🏗️ 新增菜单命令模块：`registerMenuCommands`、`toggleAutoHide`、`showCurrentState`、`toggleDebug`
- 🏗️ 新增窗口响应式模块：`onWindowResize`
- 📝 添加 JSDoc 注释文档
- 📝 新增设计文档、提案文档、任务清单和功能规范

**配置变更：**
- 添加 `@grant GM_setValue`、`@grant GM_getValue`、`@grant GM_registerMenuCommand`
- 新增 `CONFIG` 配置对象和 `state` 状态对象

### v1.0 (初始版本)
- ✨ 支持选择性下载文件和目录
- ✨ 支持递归下载子目录
- ✨ 支持 GitHub Token 认证
- ✨ 提供可视化控制面板
- ✨ 支持实时刷新文件列表

## OpenSpec 规范

本项目使用 OpenSpec 进行功能规划和文档管理，所有变更记录位于 `openspec/changes/` 目录。

### 智能按钮交互系统变更

**变更 ID：** `button-smart-interaction`

**文档位置：**
- 提案文档：`openspec/changes/button-smart-interaction/proposal.md`
- 设计文档：`openspec/changes/button-smart-interaction/design.md`
- 任务清单：`openspec/changes/button-smart-interaction/tasks.md`

**功能规范：**
- 自动隐藏规范：`openspec/changes/button-smart-interaction/specs/auto-hiding/spec.md`
- 按钮拖拽规范：`openspec/changes/button-smart-interaction/specs/button-dragging/spec.md`
- 边缘吸附规范：`openspec/changes/button-smart-interaction/specs/edge-snapping/spec.md`
- 菜单命令规范：`openspec/changes/button-smart-interaction/specs/menu-commands/spec.md`
- 智能面板定位规范：`openspec/changes/button-smart-interaction/specs/smart-panel-positioning/spec.md`
- 状态持久化规范：`openspec/changes/button-smart-interaction/specs/state-persistence/spec.md`

## 项目文档

### 设计文档
- `Button Smart Interaction Upgrade Design.md`：智能按钮交互系统的详细设计文档，包含完整的技术方案和实现细节

### 原型测试
- `button-prototype.html`：智能按钮交互系统的独立测试页面，用于验证拖拽、吸附、隐藏等交互效果

### OpenSpec 配置
- `openspec/config.yaml`：OpenSpec 规范驱动的项目配置

## 开发工具和环境

### 开发环境
- **脚本管理器**：Tampermonkey（推荐）或 Violentmonkey
- **浏览器**：Chrome、Firefox、Edge 等现代浏览器
- **开发工具**：浏览器开发者工具（F12）

### 测试工具
- **原型测试**：`button-prototype.html` 独立测试页面
- **调试日志**：通过右键菜单启用调试模式，查看详细日志

### 版本控制
- **Git 仓库**：当前目录已初始化 Git 仓库
- **忽略配置**：`.gitignore` 已配置，忽略 AI 工具目录和日志文件

## 贡献指南

如果您想为这个项目做出贡献：

1. **报告问题**：请在 Greasy Fork 脚本页面提交问题报告
2. **功能建议**：欢迎提出新的功能建议
3. **代码贡献**：
   - 遵循现有的代码风格和命名约定
   - 添加必要的 JSDoc 注释
   - 更新相关文档（AGENTS.md、OpenSpec 规范等）
   - 测试所有功能确保向后兼容

## 许可证

本项目采用 MIT 许可证，详见脚本头部 `@license` 元数据。

---

**文档版本**：v1.1  
**最后更新**：2026-02-06  
**维护者**：GitHub Downloader