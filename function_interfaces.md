# UE5LuaModKit 函数接口参考指南

本文档详细介绍了 UE5LuaModKit 中可用的函数接口，包括核心 API、回调函数和工具函数。

## 目录

1. [核心函数接口](#核心函数接口)
2. [玩家相关函数](#玩家相关函数)
3. [游戏世界函数](#游戏世界函数)
4. [UI 相关函数](#ui-相关函数)
5. [数据管理函数](#数据管理函数)
6. [工具函数](#工具函数)
7. [网络相关函数](#网络相关函数)
8. [调试函数](#调试函数)

## 核心函数接口

### MOD 生命周期函数

```lua
-- MOD 初始化
function Init()
    -- 在 MOD 加载时调用
    print("MOD 已初始化")
    RegisterEventHandlers()
    LoadConfigurations()
end

-- MOD 卸载
function OnModUnload()
    -- 在 MOD 卸载时调用
    print("MOD 即将卸载")
    SaveConfigurations()
    UnregisterEventHandlers()
end

-- 配置加载
function LoadConfigurations()
    local success, config = pcall(function()
        return require("config")
    end)
    if success then
        MOD_CONFIG = config
    else
        print("配置加载失败: " .. tostring(config))
        MOD_CONFIG = {}
    end
end

-- 配置保存
function SaveConfigurations()
    -- 保存 MOD 配置
    local configPath = "config.lua"
    local file = io.open(configPath, "w")
    if file then
        file:write("return " .. SerializeTable(MOD_CONFIG))
        file:close()
    end
end
```

### 事件注册函数

```lua
-- 注册事件处理器
function RegisterEventHandlers()
    -- 游戏事件
    RegisterEventListener("OnGameStart", OnGameStart)
    RegisterEventListener("OnGameEnd", OnGameEnd)
    
    -- 玩家事件
    RegisterEventListener("OnPlayerSpawn", OnPlayerSpawn)
    RegisterEventListener("OnPlayerDeath", OnPlayerDeath)
    
    -- 自定义事件
    RegisterEventListener("OnCustomEvent", OnCustomEvent)
end

-- 注销事件处理器
function UnregisterEventHandlers()
    UnregisterEventListener("OnGameStart", OnGameStart)
    UnregisterEventListener("OnGameEnd", OnGameEnd)
    UnregisterEventListener("OnPlayerSpawn", OnPlayerSpawn)
    UnregisterEventListener("OnPlayerDeath", OnPlayerDeath)
    UnregisterEventListener("OnCustomEvent", OnCustomEvent)
end
```

## 玩家相关函数

### 玩家信息和状态

```lua
-- 获取玩家信息
function GetPlayerInfo(player)
    return {
        name = player:GetPlayerName(),
        level = player:GetLevel(),
        health = player:GetHealth(),
        maxHealth = player:GetMaxHealth(),
        location = player:GetLocation(),
        rotation = player:GetRotation()
    }
end

-- 修改玩家状态
function ModifyPlayerState(player, modifications)
    if modifications.health then
        player:SetHealth(modifications.health)
    end
    
    if modifications.location then
        player:SetLocation(modifications.location)
    end
    
    if modifications.rotation then
        player:SetRotation(modifications.rotation)
    end
end

-- 玩家能力检查
function CheckPlayerCapabilities(player)
    return {
        canJump = player:CanJump(),
        canCrouch = player:CanCrouch(),
        canSprint = player:CanSprint(),
        isInCombat = player:IsInCombat()
    }
end
```

### 玩家物品和库存

```lua
-- 物品操作
function ManagePlayerInventory(player)
    local inventory = player:GetInventoryComponent()
    
    -- 添加物品
    function AddItemToPlayer(itemID, count)
        return inventory:AddItem(itemID, count)
    end
    
    -- 移除物品
    function RemoveItemFromPlayer(itemID, count)
        return inventory:RemoveItem(itemID, count)
    end
    
    -- 检查物品
    function HasItem(itemID, count)
        return inventory:HasItem(itemID, count)
    end
    
    -- 获取物品列表
    function GetInventoryItems()
        return inventory:GetAllItems()
    end
    
    return {
        AddItem = AddItemToPlayer,
        RemoveItem = RemoveItemFromPlayer,
        HasItem = HasItem,
        GetItems = GetInventoryItems
    }
end
```

## 游戏世界函数

### 世界操作

```lua
-- 世界对象管理
function WorldObjectManager()
    local world = GetWorld()
    
    -- 生成 Actor
    function SpawnActor(className, location, rotation)
        return world:SpawnActor(className, location or FVector(0,0,0), rotation or FRotator(0,0,0))
    end
    
    -- 查找 Actor
    function FindActor(predicate)
        local actors = world:GetAllActors()
        for _, actor in ipairs(actors) do
            if predicate(actor) then
                return actor
            end
        end
        return nil
    end
    
    -- 删除 Actor
    function DestroyActor(actor)
        if actor and actor:IsValid() then
            actor:Destroy()
            return true
        end
        return false
    end
    
    return {
        Spawn = SpawnActor,
        Find = FindActor,
        Destroy = DestroyActor
    }
end

-- 时间和天气控制
function WorldEnvironmentControl()
    local world = GetWorld()
    
    -- 设置时间
    function SetTimeOfDay(hours)
        world:SetTimeOfDay(hours)
    end
    
    -- 设置天气
    function SetWeather(weatherType)
        world:SetWeatherType(weatherType)
    end
    
    -- 获取当前环境信息
    function GetEnvironmentInfo()
        return {
            time = world:GetTimeOfDay(),
            weather = world:GetCurrentWeather(),
            temperature = world:GetTemperature()
        }
    end
    
    return {
        SetTime = SetTimeOfDay,
        SetWeather = SetWeather,
        GetInfo = GetEnvironmentInfo
    }
end
```

## UI 相关函数

### UI 创建和管理

```lua
-- UI 管理器
local UIManager = {
    activeWindows = {}
}

-- 创建窗口
function UIManager:CreateWindow(windowName, config)
    local window = {
        name = windowName,
        visible = false,
        config = config or {}
    }
    
    -- 显示窗口
    function window:Show()
        self.visible = true
        -- 实现显示逻辑
    end
    
    -- 隐藏窗口
    function window:Hide()
        self.visible = false
        -- 实现隐藏逻辑
    end
    
    -- 更新窗口内容
    function window:Update(data)
        -- 实现更新逻辑
    end
    
    self.activeWindows[windowName] = window
    return window
end

-- 通知系统
function UIManager:ShowNotification(message, type, duration)
    -- 实现通知显示逻辑
end

-- 对话框系统
function UIManager:ShowDialog(title, message, buttons)
    -- 实现对话框显示逻辑
end
```

### HUD 元素

```lua
-- HUD 管理器
local HUDManager = {
    elements = {}
}

-- 添加 HUD 元素
function HUDManager:AddElement(elementID, config)
    local element = {
        id = elementID,
        visible = true,
        config = config or {}
    }
    
    function element:SetVisibility(visible)
        self.visible = visible
        -- 实现可见性切换逻辑
    end
    
    function element:UpdatePosition(x, y)
        -- 实现位置更新逻辑
    end
    
    self.elements[elementID] = element
    return element
end

-- 更新 HUD
function HUDManager:Update()
    for _, element in pairs(self.elements) do
        if element.visible then
            -- 更新元素状态
        end
    end
end
```

## 数据管理函数

### 数据存储和读取

```lua
-- 数据管理器
local DataManager = {
    cache = {}
}

-- 保存数据
function DataManager:SaveData(key, value)
    self.cache[key] = value
    
    -- 序列化并保存到文件
    local success, err = pcall(function()
        local file = io.open(key .. ".dat", "w")
        if file then
            file:write(SerializeData(value))
            file:close()
            return true
        end
        return false
    end)
    
    return success
end

-- 加载数据
function DataManager:LoadData(key)
    -- 先检查缓存
    if self.cache[key] then
        return self.cache[key]
    end
    
    -- 从文件加载
    local success, data = pcall(function()
        local file = io.open(key .. ".dat", "r")
        if file then
            local content = file:read("*all")
            file:close()
            return DeserializeData(content)
        end
        return nil
    end)
    
    if success and data then
        self.cache[key] = data
        return data
    end
    
    return nil
end

-- 清除缓存
function DataManager:ClearCache(key)
    if key then
        self.cache[key] = nil
    else
        self.cache = {}
    end
end
```

## 工具函数

### 数学和计算工具

```lua
-- 数学工具
local MathUtils = {}

-- 插值计算
function MathUtils.Lerp(a, b, t)
    return a + (b - a) * t
end

-- 夹值
function MathUtils.Clamp(value, min, max)
    return math.max(min, math.min(max, value))
end

-- 平滑步进
function MathUtils.SmoothStep(edge0, edge1, x)
    local t = MathUtils.Clamp((x - edge0) / (edge1 - edge0), 0, 1)
    return t * t * (3 - 2 * t)
end

-- 随机工具
local RandomUtils = {}

-- 范围内随机数
function RandomUtils.Range(min, max)
    return min + math.random() * (max - min)
end

-- 随机选择
function RandomUtils.Choose(array)
    if #array == 0 then return nil end
    return array[math.random(1, #array)]
end

-- 随机布尔值
function RandomUtils.Boolean(probability)
    return math.random() < (probability or 0.5)
end
```

### 字符串工具

```lua
-- 字符串工具
local StringUtils = {}

-- 分割字符串
function StringUtils.Split(str, delimiter)
    local result = {}
    for match in (str .. delimiter):gmatch("(.-)" .. delimiter) do
        table.insert(result, match)
    end
    return result
end

-- 去除空白
function StringUtils.Trim(str)
    return str:match("^%s*(.-)%s*$")
end

-- 格式化字符串
function StringUtils.Format(str, ...)
    return string.format(str, ...)
end
```

## 网络相关函数

### 网络通信

```lua
-- 网络管理器
local NetworkManager = {
    handlers = {}
}

-- 注册消息处理器
function NetworkManager:RegisterHandler(messageType, handler)
    self.handlers[messageType] = handler
end

-- 发送消息
function NetworkManager:SendMessage(messageType, data, target)
    -- 实现消息发送逻辑
end

-- 广播消息
function NetworkManager:Broadcast(messageType, data)
    -- 实现消息广播逻辑
end

-- 处理接收到的消息
function NetworkManager:HandleMessage(messageType, data, sender)
    local handler = self.handlers[messageType]
    if handler then
        handler(data, sender)
    end
end
```

## 调试函数

### 调试和日志

```lua
-- 调试工具
local DebugUtils = {
    enabled = true,
    logLevel = "INFO"
}

-- 日志级别
local LOG_LEVELS = {
    DEBUG = 1,
    INFO = 2,
    WARN = 3,
    ERROR = 4
}

-- 设置日志级别
function DebugUtils:SetLogLevel(level)
    self.logLevel = level
end

-- 日志输出
function DebugUtils:Log(level, message, ...)
    if not self.enabled then return end
    if LOG_LEVELS[level] >= LOG_LEVELS[self.logLevel] then
        local formatted = string.format(message, ...)
        print(string.format("[%s] %s", level, formatted))
    end
end

-- 性能分析
function DebugUtils:ProfileFunction(func, name)
    return function(...)
        local startTime = os.clock()
        local results = {func(...)}
        local endTime = os.clock()
        
        self:Log("DEBUG", "%s took %.4f seconds", name, endTime - startTime)
        return table.unpack(results)
    end
end

-- 变量监视
function DebugUtils:WatchVariable(name, getValue)
    local lastValue = getValue()
    
    return function()
        local currentValue = getValue()
        if currentValue ~= lastValue then
            self:Log("DEBUG", "%s changed from %s to %s", 
                name, tostring(lastValue), tostring(currentValue))
            lastValue = currentValue
        end
        return currentValue
    end
end
```

这些函数接口提供了丰富的功能支持，可以帮助 MOD 开发者更好地实现他们的想法。记住要根据具体需求选择合适的接口，并确保正确处理错误和异常情况。在使用这些接口时，建议参考相关的示例代码和最佳实践指南。