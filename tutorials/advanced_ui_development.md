# 高级UI开发教程

本教程将介绍如何创建更复杂和专业的MOD用户界面。

## 1. 响应式布局

创建能够适应不同屏幕分辨率的UI界面：

```lua
local function CreateResponsiveWindow()
    local screenWidth, screenHeight = GetScreenDimensions()
    local window = CreateWindow("响应式窗口")
    
    -- 设置窗口大小为屏幕的75%
    local windowWidth = screenWidth * 0.75
    local windowHeight = screenHeight * 0.75
    window:SetSize(windowWidth, windowHeight)
    
    -- 居中显示窗口
    window:SetPosition(
        (screenWidth - windowWidth) / 2,
        (screenHeight - windowHeight) / 2
    )
    
    -- 注册屏幕分辨率改变事件
    RegisterEventHandler("OnResolutionChanged", function(newWidth, newHeight)
        local newWindowWidth = newWidth * 0.75
        local newWindowHeight = newHeight * 0.75
        window:SetSize(newWindowWidth, newWindowHeight)
        window:SetPosition(
            (newWidth - newWindowWidth) / 2,
            (newHeight - newWindowHeight) / 2
        )
    end)
    
    return window
end
```

## 2. 高级控件

### 2.1 自定义列表视图

创建一个可以显示复杂数据的列表视图：

```lua
local function CreateInventoryList(parent, x, y, width, height)
    local listView = parent:AddListView(x, y, width, height)
    
    -- 添加列
    listView:AddColumn("物品", width * 0.4)
    listView:AddColumn("数量", width * 0.2)
    listView:AddColumn("品质", width * 0.2)
    listView:AddColumn("操作", width * 0.2)
    
    -- 添加排序功能
    listView:EnableSorting(true)
    
    -- 添加物品数据
    local function AddItem(item)
        local row = listView:AddRow()
        row:SetCell(1, item.name)
        row:SetCell(2, item.count)
        row:SetCell(3, item.quality)
        
        -- 添加使用按钮
        local useButton = CreateButton("使用")
        useButton.OnClick = function()
            UseItem(item.id)
            RefreshList()
        end
        row:SetCell(4, useButton)
    end
    
    return listView
end
```

### 2.2 拖放功能

实现物品拖放功能：

```lua
local function EnableDragAndDrop(window)
    local isDragging = false
    local draggedItem = nil
    local startX, startY = 0, 0
    
    window:AddEventHandler("OnMouseDown", function(x, y)
        isDragging = true
        startX, startY = x, y
        draggedItem = GetItemAtPosition(x, y)
        if draggedItem then
            CreateDragImage(draggedItem)
        end
    end)
    
    window:AddEventHandler("OnMouseMove", function(x, y)
        if isDragging and draggedItem then
            UpdateDragImage(x - startX, y - startY)
        end
    end)
    
    window:AddEventHandler("OnMouseUp", function(x, y)
        if isDragging and draggedItem then
            local targetSlot = GetSlotAtPosition(x, y)
            if targetSlot and CanDropItem(draggedItem, targetSlot) then
                MoveItem(draggedItem, targetSlot)
            end
            RemoveDragImage()
        end
        isDragging = false
        draggedItem = nil
    end)
end
```

## 3. 动画效果

为UI添加流畅的动画效果：

```lua
local function CreateAnimatedPanel()
    local panel = CreatePanel()
    
    function panel:SlideIn()
        local startX = -panel:GetWidth()
        local targetX = 0
        local duration = 0.5
        local startTime = GetTime()
        
        SetTimer(0.016, function()  -- 约60FPS
            local progress = (GetTime() - startTime) / duration
            if progress >= 1 then
                panel:SetX(targetX)
                return false  -- 停止定时器
            end
            
            -- 使用缓动函数使动画更流畅
            local easedProgress = 1 - (1 - progress) * (1 - progress)
            local currentX = startX + (targetX - startX) * easedProgress
            panel:SetX(currentX)
            return true  -- 继续定时器
        end, true)
    end
    
    function panel:FadeIn()
        panel:SetAlpha(0)
        local duration = 0.3
        local startTime = GetTime()
        
        SetTimer(0.016, function()
            local progress = (GetTime() - startTime) / duration
            if progress >= 1 then
                panel:SetAlpha(1)
                return false
            end
            panel:SetAlpha(progress)
            return true
        end, true)
    end
    
    return panel
end
```

## 4. 主题系统

创建可自定义的UI主题系统：

```lua
local ThemeManager = {
    currentTheme = "default",
    themes = {
        default = {
            windowBackground = {r = 0.2, g = 0.2, b = 0.2, a = 0.9},
            buttonNormal = {r = 0.3, g = 0.3, b = 0.3, a = 1.0},
            buttonHover = {r = 0.4, g = 0.4, b = 0.4, a = 1.0},
            buttonPressed = {r = 0.2, g = 0.2, b = 0.2, a = 1.0},
            textColor = {r = 1.0, g = 1.0, b = 1.0, a = 1.0},
            fontSize = 14,
            padding = 5
        },
        dark = {
            windowBackground = {r = 0.1, g = 0.1, b = 0.1, a = 0.95},
            buttonNormal = {r = 0.2, g = 0.2, b = 0.2, a = 1.0},
            buttonHover = {r = 0.3, g = 0.3, b = 0.3, a = 1.0},
            buttonPressed = {r = 0.15, g = 0.15, b = 0.15, a = 1.0},
            textColor = {r = 0.9, g = 0.9, b = 0.9, a = 1.0},
            fontSize = 14,
            padding = 5
        }
    }
}

function ThemeManager:ApplyTheme(window, themeName)
    local theme = self.themes[themeName] or self.themes.default
    self.currentTheme = themeName
    
    window:SetBackgroundColor(theme.windowBackground)
    
    -- 应用到所有子控件
    local function ApplyThemeToControl(control)
        if control.type == "button" then
            control:SetColors(theme.buttonNormal, theme.buttonHover, theme.buttonPressed)
        elseif control.type == "label" or control.type == "textbox" then
            control:SetTextColor(theme.textColor)
            control:SetFontSize(theme.fontSize)
        end
        
        -- 递归应用到子控件
        for _, child in ipairs(control:GetChildren()) do
            ApplyThemeToControl(child)
        end
    end
    
    ApplyThemeToControl(window)
end
```

## 5. 性能优化

优化UI性能的一些技巧：

```lua
-- 1. 使用对象池减少UI元素的创建和销毁
local UIPool = {
    pools = {}
}

function UIPool:GetOrCreate(type, parent)
    if not self.pools[type] then
        self.pools[type] = {}
    end
    
    local element = table.remove(self.pools[type])
    if not element then
        element = CreateUIElement(type, parent)
    end
    return element
end

function UIPool:Recycle(type, element)
    if not self.pools[type] then
        self.pools[type] = {}
    end
    element:Hide()
    table.insert(self.pools[type], element)
end

-- 2. 延迟加载
local function LazyLoadUI(window)
    local isLoaded = false
    window:AddEventHandler("OnShow", function()
        if not isLoaded then
            LoadUIContent(window)
            isLoaded = true
        end
    end)
end

-- 3. 批量更新
local function BatchUpdate(listView, items)
    listView:BeginUpdate()  -- 暂停布局计算
    for _, item in ipairs(items) do
        AddItemToList(listView, item)
    end
    listView:EndUpdate()    -- 恢复布局计算并刷新一次
end
```

## 6. 调试工具

创建UI调试工具：

```lua
local UIDebugger = {
    enabled = false,
    overlays = {}
}

function UIDebugger:Toggle()
    self.enabled = not self.enabled
    if self.enabled then
        self:ShowDebugOverlays()
    else
        self:HideDebugOverlays()
    end
end

function UIDebugger:ShowDebugOverlays()
    for _, window in ipairs(GetAllWindows()) do
        local overlay = CreateDebugOverlay(window)
        overlay:ShowBounds()
        overlay:ShowHierarchy()
        table.insert(self.overlays, overlay)
    end
end

function UIDebugger:HideDebugOverlays()
    for _, overlay in ipairs(self.overlays) do
        overlay:Destroy()
    end
    self.overlays = {}
end

-- 使用方法
RegisterCommand("toggleuidebug", function()
    UIDebugger:Toggle()
end)
```

这些高级UI开发技巧将帮助你创建更专业、更流畅的用户界面。记住要根据实际需求选择合适的技术，并始终关注性能优化。