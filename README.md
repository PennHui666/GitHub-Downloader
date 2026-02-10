# GitHub 仓库下载器

> 一个强大的 Tampermonkey 用户脚本，为 GitHub 代码页面添加选择性下载功能

[![Version](https://img.shields.io/badge/version-1.1-blue.svg)](https://greasyfork.org/scripts/556352)
[![License](https://img.shields.io/badge/license-MIT-green.svg)](LICENSE)
[![Tampermonkey](https://img.shields.io/badge/Tampermonkey-compatible-orange.svg)](https://www.tampermonkey.net/)

## ✨ 功能特性

### 核心下载功能
- **选择性下载** - 自由选择单个或多个文件进行下载
- **目录递归下载** - 支持选择整个目录，自动递归下载所有子目录
- **ZIP 打包** - 使用 JSZip 库将选中的文件打包成单个 ZIP 文件
- **GitHub Token 支持** - 配置 Token 可将 API 速率限制提升至 5000 次/小时
- **实时刷新** - 随时刷新文件列表以获取最新状态
- **可视化控制面板** - 优雅的浮动控制面板，方便用户操作

### 智能按钮交互系统 (v1.1 新增)
- **拖拽移动** - 按住按钮拖动到屏幕任意位置
- **边缘吸附** - 距离左右边缘 ≤ 20px 时自动吸附对齐
- **自动半隐藏** - 停靠后延迟 400ms 自动隐藏 50%，悬停时立即展开
- **智能面板定位** - 根据按钮位置自动选择最佳展开方向（四方向）
- **状态持久化** - 保存按钮位置、停靠状态、用户偏好设置
- **窗口响应式** - 窗口大小变化时自动检查按钮位置有效性
- **用户控制菜单** - 通过右键菜单提供重置位置、切换设置等功能

## 📦 安装

### 方式一：Greasy Fork（推荐）

1. 安装 [Tampermonkey](https://www.tampermonkey.net/) 浏览器扩展
2. 访问 [脚本主页](https://greasyfork.org/scripts/556352)
3. 点击"安装此脚本"按钮

### 方式二：手动安装

1. 复制 `GitHub_Downloader-1.0.user.js` 文件内容
2. 在 Tampermonkey 中创建新脚本并粘贴内容
3. 保存脚本

## 🚀 使用方法

### 基本下载功能

1. 访问任意 GitHub 仓库代码页面
2. 页面右下角会出现一个 📦 按钮
3. 点击按钮打开控制面板
4. 在面板中勾选需要下载的文件或目录
5. 点击"下载"按钮，脚本会自动创建并下载 ZIP 文件

### 智能按钮交互

| 功能 | 操作说明 |
|------|----------|
| **拖拽移动** | 按住按钮并拖动到屏幕任意位置 |
| **边缘吸附** | 将按钮拖到屏幕左边缘或右边缘（≤ 20px 自动吸附） |
| **自动半隐藏** | 按钮停靠到边缘后，延迟 400ms 自动隐藏 50%，悬停时立即展开 |
| **智能面板定位** | 控制面板根据按钮位置自动选择最佳展开方向 |

### 右键菜单命令

在 GitHub 页面上右键点击，选择"用户脚本" → "GitHub 仓库下载器"：

- 🔄 **重置按钮位置** - 恢复按钮到默认位置和状态
- ⏱️ **切换自动隐藏** - 开启/关闭自动隐藏功能
- 📊 **查看当前状态** - 显示按钮和面板的当前状态信息
- 🗑️ **清除状态存储** - 删除所有保存的状态，按钮恢复到默认位置
- 🔧 **切换调试模式** - 开启/关闭调试日志

### 配置 GitHub Token（可选）

为提高 API 速率限制，建议配置 GitHub Token：

1. 在控制面板中点击"Token 未设置"区域
2. 点击"申请"按钮跳转到 GitHub Token 创建页面
3. 创建具有 `repo` 权限的 Token
4. 将 Token 粘贴到输入框并保存

| 认证状态 | API 速率限制 |
|----------|--------------|
| 无 Token | 60 次/小时 |
| 有 Token | 5000 次/小时 |

## 🔧 技术栈

- **JavaScript** - 纯 JavaScript (ES5+) 实现，无需构建工具
- **Tampermonkey API** - 跨域请求、文件下载、持久化存储
- **JSZip** - ZIP 文件创建（通过 CDN 引入）

## 📂 项目结构

```
e:\Tampermonkey\GitHub-Downloader\
├── GitHub_Downloader-1.0.user.js    # 主脚本文件
├── README.md                         # 本文件
├── AGENTS.md                         # 项目详细说明文档
├── button-prototype.html             # 按钮交互原型测试页面
├── .gitignore                        # Git 忽略配置
└── openspec/                         # OpenSpec 规范目录
    ├── config.yaml
    └── changes/
        └── button-smart-interaction/  # 智能按钮交互变更记录
```

## 📝 调试模式

### 方式一：使用右键菜单（推荐）

1. 在 GitHub 页面上右键点击
2. 选择"用户脚本" → "GitHub 仓库下载器"
3. 点击"🔧 切换调试模式"
4. 刷新页面后，调试日志将输出到浏览器控制台

### 方式二：使用控制台（临时）

```javascript
// 启用调试模式（仅当前页面有效）
GM_setValue('debug_enabled', true);

// 禁用调试模式（仅当前页面有效）
GM_setValue('debug_enabled', false);
```

## ❓ 常见问题

### 下载相关

**Q: 为什么下载失败？**
- API 速率限制（请配置 GitHub Token）
- 网络连接问题
- 文件或目录不存在
- Token 权限不足（对于私有仓库）

**Q: 如何下载整个仓库？**
使用 GitHub 官方的 "Code" → "Download ZIP" 功能。本脚本适用于选择性下载部分文件或目录。

**Q: 为什么显示的文件列表为空？**
- 确保当前页面是 GitHub 代码页面（不是 issues、pull requests 等）
- 确保 GitHub 页面完全加载完成
- 尝试刷新控制面板（点击刷新按钮）

### 智能按钮交互相关

**Q: 按钮拖拽到边缘后自动隐藏了，如何找回？**
- 将鼠标移动到屏幕左边缘或右边缘，按钮会自动展开
- 使用右键菜单："用户脚本" → "GitHub 仓库下载器" → "🔄 重置按钮位置"

**Q: 如何禁用自动隐藏功能？**
使用右键菜单："用户脚本" → "GitHub 仓库下载器" → "⏱️ 切换自动隐藏"

**Q: 按钮位置混乱或找不到按钮，如何恢复？**
使用右键菜单："用户脚本" → "GitHub 仓库下载器" → "🔄 重置按钮位置" 或 "🗑️ 清除状态存储"

## 🔗 相关链接

- [脚本主页](https://greasyfork.org/scripts/556352)
- [GitHub Token 申请](https://github.com/settings/tokens)
- [Tampermonkey 官网](https://www.tampermonkey.net/)
- [JSZip 文档](https://stuk.github.io/jszip/)

## 📄 许可证

本项目采用 [MIT 许可证](LICENSE)

---

**当前版本：v1.1** | **最后更新：2026-02-10**