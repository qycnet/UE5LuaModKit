# UE5LuaModKit SDK C++ 类与 Lua 集成指南

本指南展示了如何在 Lua MOD 中使用和扩展 UE5LuaModKit SDK 中的 C++ 类。

## 目录

1. [基本概念](#基本概念)
2. [常用 C++ 类的 Lua 封装](#常用-c-类的-lua-封装)
3. [自定义 C++ 类的 Lua 扩展](#自定义-c-类的-lua-扩展)
4. [高级集成技巧](#高级集成技巧)

## 基本概念

在 UE5LuaModKit 中，C++ 类通过特定的绑定机制暴露给 Lua。这允许 Lua 脚本直接操作 C++ 对象，调用其方法，并访问其属性。

### 示例：基本 C++ 类访问

```lua
-- 假设我们有一个名为 APalCharacter 的 C++ 类
local character = GetPlayerCharacter()  -- 获取玩家角色

-- 调用 C++ 方法
character:Jump()

-- 获取 C++ 属性
local health = character:GetHealth()
print("当前生命值：" .. health)

-- 设置 C++ 属性
character:SetWalkSpeed(300)
```

## 常用 C++ 类的 Lua 封装

### 1. APalCharacter（玩家角色类）

```lua
-- pal_character_extension.lua

local PalCharacter = {}

function PalCharacter:New(character)
    local obj = {}
    setmetatable(obj, self)
    self.__index = self
    obj.character = character
    return obj
end

function PalCharacter:ApplyDamage(damage)
    self.character:TakeDamage(damage)
end

function PalCharacter:Heal(amount)
    local currentHealth = self.character:GetHealth()
    local maxHealth = self.character:GetMaxHealth()
    self.character:SetHealth(math.min(currentHealth + amount, maxHealth))
end

function PalCharacter:GetInventory()
    return self.character:GetInventoryComponent()
end

-- 使用示例
local player = PalCharacter:New(GetPlayerCharacter())
player:ApplyDamage(10)
player:Heal(20)
```

### 2. UPalInventoryComponent（物品栏组件）

```lua
-- pal_inventory_extension.lua

local PalInventory = {}

function PalInventory:New(inventoryComponent)
    local obj = {}
    setmetatable(obj, self)
    self.__index = self
    obj.inventory = inventoryComponent
    return obj
end

function PalInventory:AddItem(itemID, count)
    return self.inventory:AddItem(itemID, count)
end

function PalInventory:RemoveItem(itemID, count)
    return self.inventory:RemoveItem(itemID, count)
end

function PalInventory:GetItemCount(itemID)
    return self.inventory:GetItemNum(itemID)
end

-- 使用示例
local playerInventory = PalInventory:New(GetPlayerCharacter():GetInventoryComponent())
playerInventory:AddItem("Potion", 5)
local potionCount = playerInventory:GetItemCount("Potion")
print("玩家拥有 " .. potionCount .. " 个药水")
```

## 自定义 C++ 类的 Lua 扩展

### 示例：扩展 APalAIController

假设我们想要为 AI 控制器添加一些自定义行为：

```lua
-- pal_ai_controller_extension.lua

local PalAIController = {}

function PalAIController:New(aiController)
    local obj = {}
    setmetatable(obj, self)
    self.__index = self
    obj.controller = aiController
    return obj
end

function PalAIController:SetPatrolPath(waypoints)
    self.waypoints = waypoints
    self.currentWaypointIndex = 1
    self:MoveToNextWaypoint()
end

function PalAIController:MoveToNextWaypoint()
    if not self.waypoints then return end
    
    local nextWaypoint = self.waypoints[self.currentWaypointIndex]
    self.controller:MoveToLocation(nextWaypoint)
    
    -- 使用 C++ 的事件系统
    self.controller:BindOnMoveCompleted(function()
        self.currentWaypointIndex = (self.currentWaypointIndex % #self.waypoints) + 1
        self:MoveToNextWaypoint()
    end)
end

function PalAIController:StartAttacking(target)
    self.controller:SetFocus(target)
    -- 假设 C++ 类中有 StartAttackBehavior 方法
    self.controller:StartAttackBehavior()
end

-- 使用示例
local aiCharacter = GetAICharacter("Guard_01")  -- 假设的函数来获取 AI 角色
local aiController = PalAIController:New(aiCharacter:GetController())

local patrolPath = {
    {x = 100, y = 100, z = 0},
    {x = 200, y = 100, z = 0},
    {x = 200, y = 200, z = 0},
    {x = 100, y = 200, z = 0}
}
aiController:SetPatrolPath(patrolPath)

-- 当玩家进入警戒范围时
local player = GetPlayerCharacter()
aiController:StartAttacking(player)
```

## 高级集成技巧

### 1. 使用 C++ 委托

在 Lua 中响应 C++ 的委托事件：

```lua
-- delegate_example.lua

local function OnHealthChanged(newHealth, oldHealth)
    print("生命值从 " .. oldHealth .. " 变为 " .. newHealth)
end

-- 假设 APalCharacter 有一个 OnHealthChanged 委托
local player = GetPlayerCharacter()
player:BindOnHealthChanged(OnHealthChanged)
```

### 2. 创建自定义 UObject

在 Lua 中创建和使用自定义的 UObject：

```lua
-- custom_object_example.lua

local CustomObject = UObject:extend("CustomObject")

function CustomObject:initialize()
    CustomObject.super.initialize(self)
    self.customValue = 0
end

function CustomObject:SetValue(value)
    self.customValue = value
    -- 触发 C++ 端的事件
    self:BroadcastEvent("OnValueChanged", value)
end

-- 使用示例
local myObject = CustomObject:new()
myObject:SetValue(10)

-- 在 C++ 端可以监听这个事件
```

### 3. 与蓝图交互

通过 Lua 调用和扩展蓝图功能：

```lua
-- blueprint_interaction.lua

local BlueprintLibrary = require("BlueprintLibrary")

-- 调用蓝图函数
local result = BlueprintLibrary.ExecuteBlueprint("BP_CustomLogic", "CalculateDamage", 50, 1.5)

-- 扩展蓝图类
local CustomBlueprintClass = BlueprintLibrary.GetBlueprintClass("BP_CustomActor")
local myActor = CustomBlueprintClass:new()
myActor:SetActorLocation({x = 100, y = 100, z = 100})

-- 重写蓝图事件
function myActor:ReceiveTick(deltaTime)
    -- 自定义每帧逻辑
    self:CustomUpdateLogic(deltaTime)
end
```

这些示例展示了如何在 Lua 中集成和扩展 SDK 中的 C++ 类。通过这种方式，MOD 开发者可以充分利用底层引擎的功能，同时享受 Lua 脚本的灵活性和易用性。在实际开发中，您可能需要根据具体的 SDK 结构和游戏需求来调整这些示例。