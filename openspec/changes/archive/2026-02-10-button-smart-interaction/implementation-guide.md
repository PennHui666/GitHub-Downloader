这是一份结构化的最终需求-技术方案，包含所有核心细节。

---

# 🎯 GitHub 仓库下载器 - 智能按钮交互升级
## 最终需求-技术方案文档

---

## 📋 一、核心数据结构

### 1.1 配置对象（CONFIG）

```javascript
const CONFIG = {
    // 吸附阈值（像素）
    SNAP_THRESHOLD: 20,
    
    // 隐藏延迟（毫秒）
    HIDE_DELAY: 400,
    
    // 动画时长（毫秒）
    ANIMATION_DURATION: 200,
    
    // 隐藏比例（0-1）
    HIDE_RATIO: 0.5,
    
    // 按钮尺寸（像素）
    BUTTON_SIZE: 50,
    
    // 面板尺寸（像素）
    PANEL_WIDTH: 300,
    PANEL_HEIGHT: 400,
    
    // 面板与按钮的间距（像素）
    PANEL_GAP: 10,
    
    // 拖拽判定阈值（像素）
    DRAG_THRESHOLD: 3,
    
    // 支持的停靠边缘
    SUPPORTED_EDGES: ['left', 'right'],
    
    // 默认位置
    DEFAULT_RIGHT: 20,
    DEFAULT_BOTTOM: 30
};
```

### 1.2 状态对象（state）

```javascript
const state = {
    // 按钮位置
    x: null,           // 水平位置（left）
    y: null,           // 垂直位置（top）
    
    // 停靠状态
    docked: false,     // 是否停靠
    edge: null,        // 停靠边缘：'left' | 'right'
    
    // 隐藏状态
    hidden: false,     // 是否半隐藏
    
    // 拖拽状态
    isDragging: false,     // 是否正在拖拽
    hasDragged: false,     // 是否发生了有效拖拽（超过阈值）
    dragStartX: 0,         // 拖拽起始鼠标 X 坐标
    dragStartY: 0,         // 拖拽起始鼠标 Y 坐标
    buttonStartX: 0,       // 拖拽起始按钮 X 坐标
    buttonStartY: 0,       // 拖拽起始按钮 Y 坐标
    
    // 鼠标状态
    isMouseOver: false,    // 鼠标是否在按钮上
    
    // 面板状态
    panelOpen: false,      // 面板是否打开
    panelDirection: null,  // 面板展开方向
    
    // 自动隐藏开关
    autoHideEnabled: true, // 是否启用自动隐藏
    
    // 计时器
    hideTimer: null        // 隐藏计时器引用
};
```

### 1.3 存储结构（buttonState）

```javascript
// 保存到 GM_setValue 的数据结构
const buttonState = {
    x: number,              // 按钮水平位置
    y: number,              // 按钮垂直位置
    docked: boolean,        // 是否停靠
    edge: string|null,      // 停靠边缘
    autoHideEnabled: boolean // 自动隐藏开关状态
};
```

---

## 🎯 二、核心功能需求

### 2.1 拖拽移动功能

**需求描述：**
- 用户可以按住按钮任意位置拖动到窗口内的任意位置
- 拖拽过程中按钮实时跟随鼠标，提供视觉反馈
- 限制按钮在可视区域内，不允许拖出浏览器窗口
- 使用位移阈值（3px）区分拖拽和点击行为

**技术实现要点：**
- 事件序列：mousedown → mousemove → mouseup
- 拖拽判定：位移超过 3px 判定为拖拽
- 边界限制：使用 `clampPosition()` 函数
- 视觉反馈：添加 `.dragging` 类，cursor: grabbing，阴影增强，缩放 1.1

---

### 2.2 边缘吸附功能

**需求描述：**
- 当按钮距离边缘 ≤ 20px 时，自动吸附到边缘
- 支持左、右边缘吸附
- 吸附时按钮对齐到边缘（left: 0 或 right: 0）
- 吸附后应用 `.docked` 样式类

**技术实现要点：**
- 实时检测：在 `mousemove` 中调用 `checkEdgeSnap()`
- 距离计算：`leftEdge = x`, `rightEdge = windowWidth - x - BUTTON_SIZE`
- 吸附应用：在 `mouseup` 中调用 `applyDocked()`

---

### 2.3 半隐藏效果

**需求描述：**
- 按钮停靠到边缘后，延迟 400ms 自动隐藏 50%
- 隐藏比例：向边缘方向平移 25px（50% × 50px）
- 隐藏时应用平滑过渡动画（200ms）
- 鼠标悬停在按钮上时，立即完整显示（无延迟）
- 鼠标离开按钮后，如果面板未打开，延迟 400ms 重新隐藏

**技术实现要点：**
- 隐藏方式：使用 `transform: translateX()`
- 计时器管理：`scheduleHide()` 和 `cancelHide()`
- 鼠标事件：`mouseenter` 取消隐藏并显示，`mouseleave` 重新安排隐藏

---

### 2.4 智能面板定位

**需求描述：**
- 面板根据按钮位置自动选择展开方向
- 支持四个方向：up-left, up-right, down-left, down-right
- 计算可用空间，优先选择空间最大的方向
- 确保面板不超出可视区域
- 拖拽按钮时，面板实时更新位置

**技术实现要点：**
- 方向计算：`calculatePanelDirection()` - 四象限判断 + 空间检测
- 位置更新：`updatePanelPosition()` - 计算坐标 + 边界限制
- 实时跟随：在拖拽的 `mousemove` 中调用

---

### 2.5 状态持久化

**需求描述：**
- 保存按钮位置（x, y）
- 保存停靠状态（docked, edge）
- 保存自动隐藏开关状态（autoHideEnabled）
- 页面刷新后自动恢复状态
- 使用 Tampermonkey 的 `GM_setValue` / `GM_getValue`

**技术实现要点：**
- 保存：`GM_setValue('buttonState', JSON.stringify(buttonState))`
- 加载：`GM_getValue('buttonState')` → `JSON.parse()`
- 应用：`applyState()` - 恢复位置和停靠状态
- 触发时机：初始化、拖拽结束、状态变化

---

### 2.6 拖拽与点击区分

**需求描述：**
- 位移 ≤ 3px：视为点击，切换面板显示/隐藏
- 位移 > 3px：视为拖拽，不触发点击事件
- 避免拖拽按钮时意外打开面板

**技术实现要点：**
- 位移检测：在 `mousemove` 中计算 `Math.abs(deltaX/Y)`
- 点击防护：在 `onButtonClick()` 中检查 `state.hasDragged`
- 延迟重置：在 `mouseup` 中延迟重置 `hasDragged` 标志

---

### 2.7 窗口响应式

**需求描述：**
- 窗口大小变化时，检查按钮是否在可视区域内
- 如果按钮超出边界，重置到默认位置
- 更新面板位置

**技术实现要点：**
- 事件监听：`window.addEventListener('resize', onWindowResize)`
- 边界检测：检查 `getBoundingClientRect()` 是否超出视口
- 重置逻辑：`resetButton()` - 恢复默认位置

---

### 2.8 用户控制功能（GM_registerMenuCommand）

**需求描述：**
- 使用 `GM_registerMenuCommand` 注册多个菜单项
- 提供一键恢复到默认位置和状态的功能
- 提供自动隐藏开关控制功能
- 方便调试和恢复默认配置

**技术实现要点：**
- 菜单注册：使用 `GM_registerMenuCommand` API
- 功能实现：重置状态、切换开关、查看状态

---

## 🔧 三、关键函数设计

### 3.1 状态管理函数

```javascript
// 保存状态到存储
function saveState() {
    const stateToSave = {
        x: state.x,
        y: state.y,
        docked: state.docked,
        edge: state.edge,
        autoHideEnabled: state.autoHideEnabled
    };
    GM_setValue('buttonState', JSON.stringify(stateToSave));
}

// 从存储加载状态
function loadState() {
    try {
        const savedState = GM_getValue('buttonState');
        if (savedState) {
            const parsed = JSON.parse(savedState);
            Object.assign(state, parsed);
        }
    } catch (err) {
        error('加载状态失败: ' + err.message);
    }
}

// 应用状态到按钮
function applyState() {
    if (state.x !== null && state.y !== null) {
        toggleBtn.style.left = `${state.x}px`;
        toggleBtn.style.top = `${state.y}px`;
        toggleBtn.style.right = 'auto';
        toggleBtn.style.bottom = 'auto';
    }
    
    if (state.docked && state.edge) {
        applyDocked(state.edge);
    }
}
```

### 3.2 拖拽处理函数

```javascript
// 鼠标按下
function onMouseDown(e) {
    if (e.button !== 0) return; // 只响应左键
    
    state.isDragging = true;
    state.hasDragged = false;
    state.dragStartX = e.clientX;
    state.dragStartY = e.clientY;
    state.buttonStartX = toggleBtn.offsetLeft;
    state.buttonStartY = toggleBtn.offsetTop;
    
    toggleBtn.classList.add('dragging');
    
    // 如果面板打开，更新面板位置
    if (state.panelOpen) {
        updatePanelPosition();
    }
}

// 鼠标移动
function onMouseMove(e) {
    if (!state.isDragging) return;
    
    e.preventDefault();
    
    const deltaX = e.clientX - state.dragStartX;
    const deltaY = e.clientY - state.dragStartY;
    
    // 检测是否发生了有效拖拽
    if (!state.hasDragged) {
        if (Math.abs(deltaX) > CONFIG.DRAG_THRESHOLD || 
            Math.abs(deltaY) > CONFIG.DRAG_THRESHOLD) {
            state.hasDragged = true;
        }
    }
    
    let newX = state.buttonStartX + deltaX;
    let newY = state.buttonStartY + deltaY;
    const clamped = clampPosition(newX, newY);
    
    toggleBtn.style.left = `${clamped.x}px`;
    toggleBtn.style.top = `${clamped.y}px`;
    toggleBtn.style.right = 'auto';
    toggleBtn.style.bottom = 'auto';
    
    state.x = clamped.x;
    state.y = clamped.y;
    
    checkEdgeSnap(clamped.x, clamped.y);
    
    if (state.panelOpen) {
        updatePanelPosition();
    }
}

// 鼠标释放
function onMouseUp(e) {
    if (!state.isDragging) return;
    
    state.isDragging = false;
    toggleBtn.classList.remove('dragging');
    
    if (state.edge) {
        applyDocked(state.edge);
    }
    
    saveState();
    
    setTimeout(() => {
        state.hasDragged = false;
    }, 0);
}
```

### 3.3 边缘吸附函数

```javascript
// 检测边缘吸附
function checkEdgeSnap(x, y) {
    const viewport = {
        width: window.innerWidth,
        height: window.innerHeight
    };
    
    const distances = {
        left: x,
        right: viewport.width - x - CONFIG.BUTTON_SIZE,
        top: y,
        bottom: viewport.height - y - CONFIG.BUTTON_SIZE
    };
    
    let snappedEdge = null;
    for (const edge of CONFIG.SUPPORTED_EDGES) {
        if (distances[edge] <= CONFIG.SNAP_THRESHOLD) {
            snappedEdge = edge;
            break;
        }
    }
    
    state.edge = snappedEdge;
    return snappedEdge;
}

// 应用停靠状态
function applyDocked(edge) {
    state.docked = true;
    state.edge = edge;
    toggleBtn.classList.add('docked');
    
    switch (edge) {
        case 'left':
            toggleBtn.style.left = '0';
            toggleBtn.style.right = 'auto';
            break;
        case 'right':
            toggleBtn.style.right = '0';
            toggleBtn.style.left = 'auto';
            break;
    }
    
    // 延迟启动自动隐藏
    if (state.autoHideEnabled) {
        cancelHide();
        setTimeout(() => {
            if (state.docked && !state.panelOpen) {
                scheduleHide();
            }
        }, 100);
    }
}

// 清除停靠状态
function clearDocked() {
    state.docked = false;
    state.edge = null;
    state.hidden = false;
    
    toggleBtn.classList.remove('docked');
    toggleBtn.style.transform = 'translateX(0) translateY(0)';
    
    cancelHide();
}
```

### 3.4 半隐藏效果函数

```javascript
// 安排隐藏
function scheduleHide() {
    if (!state.autoHideEnabled || !state.docked || state.panelOpen) {
        return;
    }
    
    cancelHide();
    state.hideTimer = setTimeout(() => {
        hideButton();
    }, CONFIG.HIDE_DELAY);
}

// 取消隐藏
function cancelHide() {
    if (state.hideTimer) {
        clearTimeout(state.hideTimer);
        state.hideTimer = null;
    }
}

// 隐藏按钮
function hideButton() {
    if (state.hidden || !state.docked) return;
    
    state.hidden = true;
    toggleBtn.style.transition = `transform ${CONFIG.ANIMATION_DURATION}ms ease`;
    
    switch (state.edge) {
        case 'left':
            toggleBtn.style.transform = `translateX(-${CONFIG.BUTTON_SIZE * CONFIG.HIDE_RATIO}px)`;
            break;
        case 'right':
            toggleBtn.style.transform = `translateX(${CONFIG.BUTTON_SIZE * CONFIG.HIDE_RATIO}px)`;
            break;
    }
}

// 显示按钮
function showButton() {
    if (!state.hidden) return;
    
    state.hidden = false;
    toggleBtn.style.transform = 'translateX(0) translateY(0)';
}
```

### 3.5 智能面板定位函数

```javascript
// 计算面板展开方向
function calculatePanelDirection() {
    const btnRect = toggleBtn.getBoundingClientRect();
    const viewport = {
        width: window.innerWidth,
        height: window.innerHeight
    };
    
    const centerX = btnRect.left + btnRect.width / 2;
    const centerY = btnRect.top + btnRect.height / 2;
    
    const viewCenterX = viewport.width / 2;
    const viewCenterY = viewport.height / 2;
    
    const inLeftHalf = centerX < viewCenterX;
    const inTopHalf = centerY < viewCenterY;
    
    const space = {
        left: btnRect.left,
        right: viewport.width - btnRect.right,
        top: btnRect.top,
        bottom: viewport.height - btnRect.bottom
    };
    
    // 水平方向选择
    let horizontal;
    if (inLeftHalf) {
        horizontal = space.right >= CONFIG.PANEL_WIDTH ? 'right' : 'left';
    } else {
        horizontal = space.left >= CONFIG.PANEL_WIDTH ? 'left' : 'right';
    }
    
    // 垂直方向选择
    let vertical;
    if (inTopHalf) {
        vertical = space.bottom >= CONFIG.PANEL_HEIGHT ? 'down' : 'up';
    } else {
        vertical = space.top >= CONFIG.PANEL_HEIGHT ? 'up' : 'down';
    }
    
    return `${vertical}-${horizontal}`;
}

// 更新面板位置
function updatePanelPosition() {
    if (!state.panelOpen) return;
    
    const direction = calculatePanelDirection();
    const btnRect = toggleBtn.getBoundingClientRect();
    
    let panelLeft, panelTop;
    
    switch (direction) {
        case 'up-left':
            panelLeft = btnRect.left - CONFIG.PANEL_WIDTH - CONFIG.PANEL_GAP;
            panelTop = btnRect.top - CONFIG.PANEL_HEIGHT - CONFIG.PANEL_GAP;
            break;
        case 'up-right':
            panelLeft = btnRect.right + CONFIG.PANEL_GAP;
            panelTop = btnRect.top - CONFIG.PANEL_HEIGHT - CONFIG.PANEL_GAP;
            break;
        case 'down-left':
            panelLeft = btnRect.left - CONFIG.PANEL_WIDTH - CONFIG.PANEL_GAP;
            panelTop = btnRect.bottom + CONFIG.PANEL_GAP;
            break;
        case 'down-right':
            panelLeft = btnRect.right + CONFIG.PANEL_GAP;
            panelTop = btnRect.bottom + CONFIG.PANEL_GAP;
            break;
    }
    
    const clamped = clampPanelPosition(panelLeft, panelTop);
    controlPanel.style.left = `${clamped.x}px`;
    controlPanel.style.top = `${clamped.y}px`;
    
    state.panelDirection = direction;
}
```

### 3.6 边界限制函数

```javascript
// 限制按钮位置在可视区域内
function clampPosition(x, y) {
    const viewport = {
        width: window.innerWidth,
        height: window.innerHeight
    };
    
    return {
        x: Math.max(0, Math.min(x, viewport.width - CONFIG.BUTTON_SIZE)),
        y: Math.max(0, Math.min(y, viewport.height - CONFIG.BUTTON_SIZE))
    };
}

// 限制面板位置在可视区域内
function clampPanelPosition(x, y) {
    const viewport = {
        width: window.innerWidth,
        height: window.innerHeight
    };
    
    return {
        x: Math.max(0, Math.min(x, viewport.width - CONFIG.PANEL_WIDTH)),
        y: Math.max(0, Math.min(y, viewport.height - CONFIG.PANEL_HEIGHT))
    };
}
```

### 3.7 重置和控制函数

```javascript
// 重置按钮到默认位置和状态
function resetButton() {
    // 清除存储
    GM_setValue('buttonState', '');
    
    // 重置状态
    state.x = null;
    state.y = null;
    state.docked = false;
    state.edge = null;
    state.hidden = false;
    
    // 重置按钮样式
    toggleBtn.style.left = 'auto';
    toggleBtn.style.top = 'auto';
    toggleBtn.style.right = `${CONFIG.DEFAULT_RIGHT}px`;
    toggleBtn.style.bottom = `${CONFIG.DEFAULT_BOTTOM}px`;
    toggleBtn.style.transform = 'translateX(0) translateY(0)';
    
    // 关闭面板
    closePanel();
    
    log('按钮已重置到默认位置');
}

// 切换自动隐藏开关
function toggleAutoHide() {
    state.autoHideEnabled = !state.autoHideEnabled;
    
    if (state.autoHideEnabled) {
        alert('✅ 自动隐藏已启用');
        if (state.docked) {
            scheduleHide();
        }
    } else {
        alert('❌ 自动隐藏已禁用');
        cancelHide();
        showButton();
    }
    
    saveState();
}

// 查看当前状态
function showCurrentState() {
    const info = `
📊 当前按钮状态
═══════════════════════════════════

位置:
  X: ${state.x !== null ? state.x + 'px' : '未设置'}
  Y: ${state.y !== null ? state.y + 'px' : '未设置'}

停靠:
  是否停靠: ${state.docked ? '是' : '否'}
  停靠边缘: ${state.edge || '无'}
  是否隐藏: ${state.hidden ? '是' : '否'}

面板:
  是否打开: ${state.panelOpen ? '是' : '否'}
  展开方向: ${state.panelDirection || '-'}

配置:
  自动隐藏: ${state.autoHideEnabled ? '启用' : '禁用'}

═══════════════════════════════════
    `.trim();
    
    alert(info);
}
```

---

## 📡 四、事件处理流程

### 4.1 初始化流程

```
脚本启动
  ↓
检查是否为代码页面 (isCodePage)
  ↓
初始化状态对象
  ↓
加载保存的状态 (loadState)
  ↓
创建控制面板 (createControlPanel)
  ↓
应用状态到按钮 (applyState)
  ↓
绑定事件处理器
  ↓
注册 GM_registerMenuCommand 菜单项
  ↓
初始化完成
```

### 4.2 拖拽流程

```
mousedown (按钮上)
  ↓
state.isDragging = true
state.hasDragged = false
记录起始位置
添加 .dragging 类
  ↓
mousemove (文档上)
  ↓
计算位移
检测是否超过阈值 (3px)
  ├─ 否：继续等待
  └─ 是：state.hasDragged = true
      ↓
      计算新位置
      限制边界 (clampPosition)
      应用位置 (left/top)
      检测边缘 (checkEdgeSnap)
      更新面板 (updatePanelPosition)
  ↓
mouseup (文档上)
  ↓
state.isDragging = false
移除 .dragging 类
  ├─ 检测到边缘 → applyDocked(edge)
  └─ 未检测到边缘 → 保持自由位置
      ↓
      保存状态 (saveState)
      延迟重置 hasDragged 标志
```

### 4.3 点击流程

```
click (按钮上)
  ↓
检查 state.hasDragged
  ├─ true（拖拽）：阻止事件，不打开面板
  └─ false（点击）：切换面板状态
      ├─ 面板关闭 → openPanel()
      └─ 面板打开 → closePanel()
```

### 4.4 面板流程

```
openPanel()
  ↓
state.panelOpen = true
显示面板 (display: block)
取消隐藏计时器
显示按钮 (showButton)
更新面板位置 (updatePanelPosition)
  ↓
用户操作
  ↓
closePanel()
  ↓
state.panelOpen = false
隐藏面板 (display: none)
  ↓
检查是否需要重新隐藏
  ├─ 停靠 + 自动隐藏启用 + 鼠标不在按钮上
  │   → scheduleHide()
  └─ 否 → 保持显示
```

### 4.5 停靠和隐藏流程

```
拖拽到边缘
  ↓
applyDocked(edge)
  ↓
state.docked = true
添加 .docked 类
对齐到边缘 (left: 0 或 right: 0)
  ↓
延迟 100ms
  ↓
scheduleHide()
  ↓
设置计时器 (400ms)
  ↓
计时器触发
  ↓
hideButton()
  ↓
state.hidden = true
应用 transform 隐藏 50%
  ↓
鼠标悬停 (mouseenter)
  ↓
cancelHide()
showButton()
  ↓
鼠标离开 (mouseleave)
  ↓
如果面板未打开 → scheduleHide()
```

### 4.6 窗口响应式流程

```
resize 事件触发
  ↓
onWindowResize()
  ↓
检查按钮边界
  ├─ 按钮在可视区域内 → 继续
  └─ 按钮超出边界 → resetButton()
      ↓
      检查面板状态
      ├─ 面板打开 → updatePanelPosition()
      └─ 面板关闭 → 无操作
```

---

## 🛠️ 五、GM_registerMenuCommand 实现

### 5.1 菜单项设计

```javascript
// 注册右键菜单命令
function registerMenuCommands() {
    // 1. 重置按钮位置
    GM_registerMenuCommand('🔄 重置按钮位置', function() {
        if (confirm('确定要重置按钮到默认位置吗？')) {
            resetButton();
        }
    });
    
    // 2. 切换自动隐藏
    GM_registerMenuCommand('⏱️ 切换自动隐藏', function() {
        toggleAutoHide();
    });
    
    // 3. 查看当前状态
    GM_registerMenuCommand('📊 查看当前状态', function() {
        showCurrentState();
    });
    
    // 4. 清除状态存储
    GM_registerMenuCommand('🗑️ 清除状态存储', function() {
        if (confirm('确定要清除所有状态存储吗？\n\n这将删除按钮位置和设置，按钮将恢复到默认位置。')) {
            resetButton();
            alert('✅ 状态存储已清除');
        }
    });
}
```

### 5.2 菜单功能说明

| 菜单项 | 功能 | 实现函数 | 用户场景 |
|--------|------|----------|----------|
| 🔄 重置按钮位置 | 恢复按钮到默认位置和状态 | `resetButton()` | 按钮位置混乱或需要重新定位时 |
| ⏱️ 切换自动隐藏 | 开启/关闭自动隐藏功能 | `toggleAutoHide()` | 用户不希望按钮自动隐藏时 |
| 📊 查看当前状态 | 显示按钮和面板的当前状态 | `showCurrentState()` | 调试或了解当前配置时 |
| 🗑️ 清除状态存储 | 删除所有保存的状态 | `resetButton()` | 需要完全重置所有设置时 |

### 5.3 菜单使用示例

```
用户操作流程：

1. 在 GitHub 页面上右键点击
2. 选择"用户脚本"菜单
3. 找到"GitHub 仓库下载器"子菜单
4. 选择相应功能：
   - "🔄 重置按钮位置" → 确认 → 按钮回到右下角
   - "⏱️ 切换自动隐藏" → 切换开关 → 弹出提示
   - "📊 查看当前状态" → 弹出状态信息框
   - "🗑️ 清除状态存储" → 确认 → 清除所有设置
```

---

## 🎨 六、CSS 样式类

### 6.1 新增样式类

```css
/* 拖拽状态 */
.toggle-btn.dragging {
    cursor: grabbing;
    box-shadow: 0 8px 20px rgba(0, 0, 0, 0.3);
    transform: scale(1.1);
    transition: none; /* 拖拽时禁用过渡 */
}

/* 停靠状态 */
.toggle-btn.docked {
    transition: transform 0.2s ease;
}

/* 悬停效果 */
.toggle-btn:hover {
    background: #0256c7;
    transform: scale(1.1);
    box-shadow: 0 4px 12px rgba(3, 102, 214, 0.3);
}

/* 拖拽时的悬停效果 */
.toggle-btn.dragging:hover {
    transform: scale(1.1); /* 保持放大 */
}
```

### 6.2 现有样式修改

```javascript
// 修改现有按钮创建代码
const toggleBtn = document.createElement('button');
toggleBtn.id = 'github-zip-toggle-btn';
toggleBtn.textContent = '📦';
toggleBtn.style.cssText = `
    position: fixed;
    /* 初始默认位置 */
    right: ${CONFIG.DEFAULT_RIGHT}px;
    bottom: ${CONFIG.DEFAULT_BOTTOM}px;
    width: ${CONFIG.BUTTON_SIZE}px;
    height: ${CONFIG.BUTTON_SIZE}px;
    border-radius: 50%;
    background: #0366d6;
    color: white;
    border: none;
    cursor: grab; /* 改为 grab */
    font-size: 24px;
    z-index: 9999;
    box-shadow: 0 2px 8px rgba(0, 0, 0, 0.15);
    /* 使用 CSS 变量分离不同场景的样式 */
    --hover-scale: 1;
    transform: scale(var(--hover-scale));
    /* 只过渡颜色、阴影和 hover 缩放，不过渡位置 */
    transition: 
        background 0.3s ease,
        box-shadow 0.3s ease,
        --hover-scale 0.3s ease;
    display: flex;
    align-items: center;
    justify-content: center;
    user-select: none;
`;
```

---

## 📝 七、集成要点

### 7.1 与现有代码的集成点

| 现有代码位置 | 集成方式 | 修改内容 |
|-------------|---------|---------|
| `createControlPanel()` | 扩展 | 添加拖拽事件绑定、样式类 |
| `toggleBtn.onclick` | 替换 | 改为 `onButtonClick()` 以支持拖拽判定 |
| `init()` | 扩展 | 添加状态加载、菜单注册 |
| 面板定位 | 修改 | 从固定位置改为动态计算 |

### 7.2 新增代码位置

```javascript
// 在脚本顶部添加
const CONFIG = { /* ... */ };
const state = { /* ... */ };

// 在 createControlPanel() 之前添加
function saveState() { /* ... */ }
function loadState() { /* ... */ }
function applyState() { /* ... */ }

// 在 init() 中添加
loadState();
registerMenuCommands();

// 修改 createControlPanel() 中的按钮创建
// 添加事件绑定：
toggleBtn.addEventListener('mousedown', onMouseDown);
toggleBtn.addEventListener('mouseenter', onMouseEnter);
toggleBtn.addEventListener('mouseleave', onMouseLeave);

// 添加全局事件
document.addEventListener('mousemove', onMouseMove);
document.addEventListener('mouseup', onMouseUp);
window.addEventListener('resize', onWindowResize);
```

### 7.3 兼容性考虑

1. **GM API 支持**：确保 `GM_setValue` / `GM_getValue` / `GM_registerMenuCommand` 可用
2. **事件兼容**：使用标准 DOM 事件，无需 polyfill
3. **CSS 兼容**：使用标准 CSS3 属性，现代浏览器支持良好
4. **性能优化**：使用 `requestAnimationFrame` 优化频繁更新（可选）

---

## ✅ 八、最终确认清单

### 8.1 核心功能确认

- [x] 拖拽移动（3px 阈值判定）
- [x] 边缘吸附（左右边缘，20px 阈值）
- [x] 半隐藏效果（400ms 延迟，50% 隐藏）
- [x] 鼠标悬停展开（立即展开）
- [x] 智能面板定位（四方向自动选择）
- [x] 状态持久化（GM_setValue/GM_getValue）
- [x] 拖拽与点击区分
- [x] 窗口响应式
- [x] 用户控制（GM_registerMenuCommand）

### 8.2 用户控制功能确认

- [x] 重置按钮位置（一键恢复默认）
- [x] 切换自动隐藏（开关控制）
- [x] 查看当前状态（调试信息）
- [x] 清除状态存储（完全重置）

### 8.3 技术参数确认

- [x] 吸附阈值：20px
- [x] 隐藏延迟：400ms
- [x] 动画时长：200ms
- [x] 隐藏比例：50%
- [x] 拖拽判定阈值：3px
- [x] 停靠边缘：left, right
- [x] 默认位置：right: 20px, bottom: 30px

---

## 🎯 总结

这份最终需求-技术方案文档包含了：

1. **完整的数据结构**：CONFIG、state、buttonState
2. **核心功能需求**：8 个主要功能点的详细说明
3. **关键函数设计**：20+ 个核心函数的实现要点
4. **事件处理流程**：6 个主要流程的详细流程图
5. **GM_registerMenuCommand 实现**：4 个菜单项的具体实现
6. **CSS 样式类**：新增和修改的样式说明
7. **集成要点**：与现有代码的集成方式
8. **确认清单**：所有功能和参数的确认

**请您确认这份方案是否符合您的需求，如果没有问题，我们可以进入实施阶段。** 🚀