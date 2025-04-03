# UE5LuaModKit 事件系统和回调机制详解

本指南详细介绍了 UE5LuaModKit 中的事件系统和回调机制，帮助您理解如何在 Lua MOD 中有效地使用事件驱动编程。

## 目录

1. [事件系统基础](#事件系统基础)
2. [内置事件类型](#内置事件类型)
3. [自定义事件](#自定义事件)
4. [事件优先级和顺序](#事件优先级和顺序)
5. [事件性能优化](#事件性能优化)
6. [高级应用场景](#高级应用场景)

## 事件系统基础

UE5LuaModKit 的事件系统允许 MOD 在特定游戏事件发生时执行自定义代码。这种机制使得 MOD 可以以非侵入式的方式扩展游戏功能。

### 基本用法

```lua
-- 注册事件监听器
function RegisterEventHandler()
    -- 语法: RegisterEventListener(eventName, callback)
    RegisterEventListener("OnGameStart", OnGameStartHandler)
    RegisterEventListener("OnPlayerSpawned", OnPlayerSpawnedHandler)
end

-- 事件处理函数
function OnGameStartHandler()
    print("游戏已启动")
    -- 在游戏启动时执行的代码
end

function OnPlayerSpawnedHandler(player)
    print("玩家已生成: " .. player:GetName())
    -- 玩家生成时执行的代码
end

-- 初始化
function Init()
    RegisterEventHandler()
end
```

### 取消注册事件

```lua
-- 取消注册特定事件的处理函数
function UnregisterSpecificHandler()
    UnregisterEventListener("OnPlayerSpawned", OnPlayerSpawnedHandler)
end

-- 取消注册所有事件处理函数
function CleanupAllHandlers()
    UnregisterAllEventListeners()
end
```

## 内置事件类型

UE5LuaModKit 提供了许多内置事件，涵盖了游戏生命周期的各个方面。

### 游戏生命周期事件

```lua
-- 游戏启动
RegisterEventListener("OnGameStart", function()
    print("游戏已启动")
end)

-- 游戏结束
RegisterEventListener("OnGameEnd", function()
    print("游戏已结束")
end)

-- 关卡加载
RegisterEventListener("OnLevelLoaded", function(levelName)
    print("关卡已加载: " .. levelName)
end)

-- 关卡卸载前
RegisterEventListener("OnLevelUnloading", function(levelName)
    print("关卡即将卸载: " .. levelName)
    -- 执行清理操作
end)
```

### 玩家相关事件

```lua
-- 玩家生成
RegisterEventListener("OnPlayerSpawned", function(player)
    print("玩家已生成: " .. player:GetName())
end)

-- 玩家死亡
RegisterEventListener("OnPlayerDied", function(player, killer)
    print("玩家 " .. player:GetName() .. " 被 " .. (killer and killer:GetName() or "未知") .. " 击杀")
end)

-- 玩家受伤
RegisterEventListener("OnPlayerDamaged", function(player, damage, damageType, instigator)
    print("玩家受到 " .. damage .. " 点伤害")
    -- 可以修改伤害值或执行其他操作
    return damage * 0.9  -- 减少10%伤害
end)

-- 玩家升级
RegisterEventListener("OnPlayerLevelUp", function(player, newLevel)
    print("玩家升级到 " .. newLevel)
    -- 给予额外奖励
end)
```

### 物品和库存事件

```lua
-- 物品获取
RegisterEventListener("OnItemAcquired", function(player, itemID, count)
    print("玩家获得了 " .. count .. " 个 " .. itemID)
end)

-- 物品使用
RegisterEventListener("OnItemUsed", function(player, itemID)
    print("玩家使用了 " .. itemID)
    -- 可以添加额外效果
    if itemID == "MysteryPotion" then
        player:AddBuff("ExtraStrength", 30)  -- 额外添加一个力量增益
    end
end)

-- 物品丢弃
RegisterEventListener("OnItemDropped", function(player, itemID, count)
    print("玩家丢弃了 " .. count .. " 个 " .. itemID)
end)
```

### 环境和世界事件

```lua
-- 时间变化
RegisterEventListener("OnTimeOfDayChanged", function(newTime)
    print("游戏时间变为 " .. newTime)
    if newTime >= 18.0 or newTime <= 6.0 then
        -- 夜间效果
        ApplyNightEffects()
    else
        -- 白天效果
        ApplyDayEffects()
    end
end)

-- 天气变化
RegisterEventListener("OnWeatherChanged", function(newWeather)
    print("天气变为 " .. newWeather)
end)

-- 区域进入
RegisterEventListener("OnAreaEntered", function(player, areaName)
    print("玩家进入区域: " .. areaName)
    -- 显示区域欢迎信息
    ShowWelcomeMessage(areaName)
end)
```

## 自定义事件

除了使用内置事件外，您还可以创建和触发自定义事件。

### 创建自定义事件

```lua
-- 触发自定义事件
function TriggerCustomEvent()
    -- 语法: TriggerEvent(eventName, ...)
    TriggerEvent("OnCustomAction", "参数1", "参数2")
end

-- 监听自定义事件
RegisterEventListener("OnCustomAction", function(arg1, arg2)
    print("自定义事件触发，参数: " .. arg1 .. ", " .. arg2)
end)
```

### 带返回值的自定义事件

```lua
-- 触发带返回值的事件
function CalculateModifiedValue()
    local baseValue = 100
    -- 触发事件并收集返回值
    local modifiedValue = TriggerEventWithReturn("OnValueModification", baseValue)
    print("原始值: " .. baseValue .. ", 修改后: " .. modifiedValue)
    return modifiedValue
end

-- 监听并修改值
RegisterEventListener("OnValueModification", function(value)
    -- 增加50%
    return value * 1.5
end)
```

## 事件优先级和顺序

在复杂的 MOD 系统中，多个 MOD 可能会监听同一个事件，因此理解事件的优先级和执行顺序非常重要。

### 设置事件优先级

```lua
-- 语法: RegisterEventListenerWithPriority(eventName, callback, priority)
-- 优先级越高的处理函数越先执行

-- 高优先级处理函数
RegisterEventListenerWithPriority("OnPlayerDamaged", function(player, damage)
    print("高优先级处理函数")
    return damage * 0.8  -- 减少20%伤害
end, 100)

-- 普通优先级处理函数
RegisterEventListenerWithPriority("OnPlayerDamaged", function(player, damage)
    print("普通优先级处理函数")
    return damage - 5  -- 减少固定5点伤害
end, 50)

-- 低优先级处理函数
RegisterEventListenerWithPriority("OnPlayerDamaged", function(player, damage)
    print("低优先级处理函数")
    -- 最后执行的处理函数
    return damage
end, 10)
```

### 中断事件传播

```lua
RegisterEventListener("OnItemUsed", function(player, itemID)
    if itemID == "ForbiddenItem" then
        print("禁止使用此物品")
        return false  -- 中断事件传播，阻止物品使用
    end
    return true  -- 继续事件传播
end)
```

## 事件性能优化

在使用事件系统时，需要注意性能优化，特别是对于频繁触发的事件。

### 减少事件监听器

```lua
-- 不推荐：在循环中频繁注册和取消注册
function BadPractice()
    for i = 1, 1000 do
        RegisterEventListener("OnFrequentEvent", OnEventHandler)
        -- 其他操作
        UnregisterEventListener("OnFrequentEvent", OnEventHandler)
    end
end

-- 推荐：只注册一次
function GoodPractice()
    RegisterEventListener("OnFrequentEvent", OnEventHandler)
    
    -- 在处理函数中进行条件检查
    function OnEventHandler(...)
        if ShouldProcess() then
            ProcessEvent(...)
        end
    end
end
```

### 使用条件事件监听

```lua
-- 条件事件监听器
function RegisterConditionalEventListener()
    -- 语法: RegisterConditionalEventListener(eventName, condition, callback)
    RegisterConditionalEventListener("OnEntitySpawned", 
        function(entity)
            -- 只有当实体是敌人时才触发回调
            return entity:IsEnemy()
        end,
        function(entity)
            -- 处理敌人生成
            ProcessEnemySpawn(entity)
        end
    )
end
```

### 批量事件处理

```lua
-- 批量处理事件
local pendingItems = {}

RegisterEventListener("OnItemAcquired", function(player, itemID, count)
    -- 将物品添加到待处理列表
    if not pendingItems[itemID] then
        pendingItems[itemID] = 0
    end
    pendingItems[itemID] = pendingItems[itemID] + count
    
    -- 延迟处理
    if not IsProcessingScheduled then
        IsProcessingScheduled = true
        SetTimeout(100, ProcessPendingItems)  -- 100ms后处理
    end
end)

function ProcessPendingItems()
    -- 一次性处理所有待处理物品
    for itemID, count in pairs(pendingItems) do
        ProcessItemAcquisition(itemID, count)
    end
    
    -- 清空列表
    pendingItems = {}
    IsProcessingScheduled = false
end
```

## 高级应用场景

### 事件链

```lua
-- 创建事件链，一个事件触发另一个事件
function SetupEventChain()
    -- 第一个事件
    RegisterEventListener("OnEnemyDefeated", function(enemy, player)
        -- 处理敌人击败逻辑
        local dropItems = CalculateDrops(enemy)
        
        -- 触发物品掉落事件
        TriggerEvent("OnLootDropped", enemy:GetLocation(), dropItems)
    end)
    
    -- 第二个事件
    RegisterEventListener("OnLootDropped", function(location, items)
        -- 创建掉落物
        for itemID, count in pairs(items) do
            SpawnLootItem(location, itemID, count)
        end
        
        -- 触发区域更新事件
        TriggerEvent("OnAreaLootUpdated", location)
    end)
    
    -- 第三个事件
    RegisterEventListener("OnAreaLootUpdated", function(location)
        -- 更新小地图标记
        UpdateMinimapMarkers()
        
        -- 通知附近玩家
        NotifyNearbyPlayers(location, "有新的战利品出现！")
    end)
end
```

### 事件状态管理

```lua
-- 使用事件系统管理游戏状态
local GameStateManager = {
    currentState = "NONE",
    stateTransitions = {
        NONE = {"MENU"},
        MENU = {"PLAYING", "OPTIONS", "QUIT"},
        PLAYING = {"PAUSED", "GAME_OVER"},
        PAUSED = {"PLAYING", "MENU"},
        OPTIONS = {"MENU"},
        GAME_OVER = {"MENU", "QUIT"}
    }
}

function GameStateManager:ChangeState(newState)
    -- 检查状态转换是否有效
    local validTransitions = self.stateTransitions[self.currentState]
    local isValidTransition = false
    
    for _, state in ipairs(validTransitions or {}) do
        if state == newState then
            isValidTransition = true
            break
        end
    end
    
    if not isValidTransition then
        print("无效的状态转换: " .. self.currentState .. " -> " .. newState)
        return false
    end
    
    -- 触发退出当前状态事件
    TriggerEvent("OnExitState_" .. self.currentState)
    
    -- 更新状态
    local oldState = self.currentState
    self.currentState = newState
    
    -- 触发进入新状态事件
    TriggerEvent("OnEnterState_" .. newState)
    
    -- 触发状态变化事件
    TriggerEvent("OnStateChanged", oldState, newState)
    
    return true
end

-- 使用示例
RegisterEventListener("OnEnterState_PLAYING", function()
    print("游戏开始")
    -- 初始化游戏
end)

RegisterEventListener("OnExitState_PLAYING", function()
    print("游戏结束")
    -- 清理资源
end)

-- 切换状态
GameStateManager:ChangeState("MENU")
GameStateManager:ChangeState("PLAYING")
```

### 模块化事件系统

```lua
-- 创建模块化的事件系统
local ModuleEventSystem = {}

-- 创建新的事件模块
function ModuleEventSystem:CreateModule(moduleName)
    local module = {
        name = moduleName,
        eventListeners = {}
    }
    
    -- 注册模块事件
    function module:RegisterEvent(eventName, callback)
        local fullEventName = self.name .. "." .. eventName
        RegisterEventListener(fullEventName, callback)
        
        -- 保存引用以便后续清理
        table.insert(self.eventListeners, {
            event = fullEventName,
            callback = callback
        })
    end
    
    -- 触发模块事件
    function module:TriggerEvent(eventName, ...)
        local fullEventName = self.name .. "." .. eventName
        return TriggerEvent(fullEventName, ...)
    end
    
    -- 清理模块事件
    function module:Cleanup()
        for _, listener in ipairs(self.eventListeners) do
            UnregisterEventListener(listener.event, listener.callback)
        end
        self.eventListeners = {}
    end
    
    return module
end

-- 使用示例
local combatModule = ModuleEventSystem:CreateModule("Combat")

-- 注册模块事件
combatModule:RegisterEvent("OnAttack", function(attacker, target)
    print(attacker:GetName() .. " 攻击了 " .. target:GetName())
end)

-- 触发模块事件
combatModule:TriggerEvent("OnAttack", player, enemy)

-- 清理模块
function OnModUnload()
    combatModule:Cleanup()
end
```

通过这些示例，您可以看到 UE5LuaModKit 的事件系统提供了强大而灵活的机制，可以用于各种复杂的 MOD 开发场景。合理使用事件系统可以使您的 MOD 代码更加模块化、可维护，并且能够与其他 MOD 良好地协同工作。