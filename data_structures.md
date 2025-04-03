# UE5LuaModKit 数据结构和类型系统指南

本指南详细介绍了 UE5LuaModKit 中可用的数据结构和类型系统，帮助您更好地组织和管理 MOD 数据。

## 目录

1. [基本数据类型](#基本数据类型)
2. [复合数据类型](#复合数据类型)
3. [UE5 特有类型](#ue5-特有类型)
4. [自定义数据结构](#自定义数据结构)
5. [数据序列化](#数据序列化)
6. [类型转换和检查](#类型转换和检查)

## 基本数据类型

### 数值类型

```lua
-- 整数
local level = 1
local maxLevel = 100

-- 浮点数
local health = 100.5
local damage = 15.7

-- 布尔值
local isAlive = true
local canAttack = false

-- 字符串
local playerName = "Hero"
local description = "A brave warrior"
```

### 类型检查

```lua
-- 类型检查函数
function CheckType(value)
    print(string.format("值: %s, 类型: %s", tostring(value), type(value)))
end

-- 使用示例
CheckType(42)        -- 输出: 值: 42, 类型: number
CheckType("test")    -- 输出: 值: test, 类型: string
CheckType(true)      -- 输出: 值: true, 类型: boolean
CheckType({})        -- 输出: 值: table: 0x123, 类型: table
```

## 复合数据类型

### 数组（表）

```lua
-- 简单数组
local items = {"剑", "盾", "药水"}

-- 遍历数组
for i, item in ipairs(items) do
    print(string.format("物品 %d: %s", i, item))
end

-- 数组操作
table.insert(items, "弓箭")    -- 添加元素
table.remove(items, 1)         -- 移除元素
local count = #items           -- 获取长度
```

### 字典（表）

```lua
-- 键值对表
local playerStats = {
    health = 100,
    mana = 50,
    strength = 15,
    agility = 12
}

-- 访问和修改
print(playerStats.health)      -- 使用点号
print(playerStats["mana"])     -- 使用方括号
playerStats.strength = 20      -- 修改值
playerStats["newStat"] = 10    -- 添加新键值对

-- 遍历字典
for key, value in pairs(playerStats) do
    print(string.format("%s: %s", key, value))
end
```

### 嵌套结构

```lua
-- 复杂数据结构
local gameData = {
    player = {
        info = {
            name = "Hero",
            level = 1
        },
        inventory = {
            items = {"剑", "盾"},
            gold = 100
        }
    },
    world = {
        name = "测试世界",
        difficulty = "普通",
        regions = {"森林", "沙漠", "雪原"}
    }
}

-- 访问嵌套数据
print(gameData.player.info.name)           -- 访问玩家名称
print(gameData.world.regions[1])           -- 访问第一个区域
```

## UE5 特有类型

### 向量和旋转

```lua
-- 向量
local location = UE.FVector(100, 200, 300)
local direction = UE.FVector(1, 0, 0)

-- 向量运算
local newLocation = location + direction * 100
local distance = UE.FVector.Distance(location, newLocation)

-- 旋转
local rotation = UE.FRotator(0, 90, 0)  -- Pitch, Yaw, Roll
local forward = rotation:GetForwardVector()
```

### 变换

```lua
-- 变换
local transform = UE.FTransform()
transform:SetLocation(UE.FVector(100, 200, 300))
transform:SetRotation(UE.FQuat.FromRotator(UE.FRotator(0, 90, 0)))
transform:SetScale3D(UE.FVector(1, 1, 1))

-- 应用变换
local actor = GetActor()
actor:SetActorTransform(transform)
```

### 颜色

```lua
-- 颜色定义
local red = UE.FLinearColor(1, 0, 0, 1)       -- RGBA
local green = UE.FLinearColor(0, 1, 0, 0.5)   -- 半透明绿色

-- 颜色转换
local color = UE.FColor(255, 0, 0, 255)       -- 字节格式
local linearColor = color:ReinterpretAsLinear() -- 转换为线性颜色
```

## 自定义数据结构

### 基本类

```lua
-- 简单类定义
local Item = {}
Item.__index = Item

function Item.new(name, type, value)
    local self = setmetatable({}, Item)
    self.name = name
    self.type = type
    self.value = value
    return self
end

function Item:GetDescription()
    return string.format("%s (%s) - 价值: %d金币", self.name, self.type, self.value)
end

-- 使用示例
local sword = Item.new("铁剑", "武器", 100)
print(sword:GetDescription())
```

### 继承

```lua
-- 基类
local Entity = {}
Entity.__index = Entity

function Entity.new(name, health)
    local self = setmetatable({}, Entity)
    self.name = name
    self.health = health
    return self
end

function Entity:TakeDamage(amount)
    self.health = math.max(0, self.health - amount)
    return self.health
end

-- 派生类
local Character = setmetatable({}, {__index = Entity})
Character.__index = Character

function Character.new(name, health, level)
    local self = Entity.new(name, health)
    setmetatable(self, Character)
    self.level = level
    return self
end

function Character:LevelUp()
    self.level = self.level + 1
    self.health = self.health + 10
end

-- 使用示例
local player = Character.new("Hero", 100, 1)
player:TakeDamage(20)
player:LevelUp()
```

### 组合

```lua
-- 组件类
local InventoryComponent = {}
InventoryComponent.__index = InventoryComponent

function InventoryComponent.new()
    local self = setmetatable({}, InventoryComponent)
    self.items = {}
    return self
end

function InventoryComponent:AddItem(item)
    table.insert(self.items, item)
end

-- 状态组件
local StatsComponent = {}
StatsComponent.__index = StatsComponent

function StatsComponent.new()
    local self = setmetatable({}, StatsComponent)
    self.stats = {
        strength = 10,
        agility = 10,
        intelligence = 10
    }
    return self
end

-- 主类
local Player = {}
Player.__index = Player

function Player.new(name)
    local self = setmetatable({}, Player)
    self.name = name
    self.inventory = InventoryComponent.new()
    self.stats = StatsComponent.new()
    return self
end
```

## 数据序列化

### JSON 序列化

```lua
-- 保存数据
function SaveGameData(data)
    local json = require("json")
    local serialized = json.encode(data)
    
    -- 保存到文件
    local file = io.open("save.json", "w")
    if file then
        file:write(serialized)
        file:close()
    end
end

-- 加载数据
function LoadGameData()
    local json = require("json")
    local file = io.open("save.json", "r")
    if file then
        local content = file:read("*all")
        file:close()
        return json.decode(content)
    end
    return nil
end

-- 使用示例
local gameState = {
    player = {
        name = "Hero",
        level = 10,
        inventory = {"剑", "盾"}
    },
    timestamp = os.time()
}

SaveGameData(gameState)
local loaded = LoadGameData()
```

### 自定义序列化

```lua
-- 可序列化对象
local Serializable = {}
Serializable.__index = Serializable

function Serializable:Serialize()
    local data = {}
    for k, v in pairs(self) do
        if type(v) ~= "function" then
            data[k] = v
        end
    end
    return data
end

function Serializable:Deserialize(data)
    for k, v in pairs(data) do
        self[k] = v
    end
end

-- 示例类
local Player = setmetatable({}, {__index = Serializable})
Player.__index = Player

function Player.new(name, level)
    local self = setmetatable({}, Player)
    self.name = name
    self.level = level
    return self
end

-- 使用示例
local player = Player.new("Hero", 10)
local serialized = player:Serialize()
local newPlayer = Player.new()
newPlayer:Deserialize(serialized)
```

## 类型转换和检查

### 类型转换工具

```lua
local TypeConverter = {}

-- 字符串转换
function TypeConverter.ToString(value)
    if type(value) == "table" then
        return TypeConverter.TableToString(value)
    end
    return tostring(value)
end

-- 数值转换
function TypeConverter.ToNumber(value)
    return tonumber(value) or 0
end

-- 布尔转换
function TypeConverter.ToBoolean(value)
    if type(value) == "string" then
        return value:lower() == "true"
    end
    return not not value
end

-- 表转字符串
function TypeConverter.TableToString(t)
    local result = "{"
    for k, v in pairs(t) do
        local key = type(k) == "number" and k or string.format("'%s'", k)
        local value = type(v) == "table" and TypeConverter.TableToString(v) or
                      type(v) == "string" and string.format("'%s'", v) or tostring(v)
        result = result .. string.format("[%s]=%s,", key, value)
    end
    return result .. "}"
end
```

### 类型检查工具

```lua
local TypeChecker = {}

-- 基本类型检查
function TypeChecker.IsNumber(value)
    return type(value) == "number"
end

function TypeChecker.IsString(value)
    return type(value) == "string"
end

function TypeChecker.IsBoolean(value)
    return type(value) == "boolean"
end

function TypeChecker.IsTable(value)
    return type(value) == "table"
end

-- 复杂类型检查
function TypeChecker.IsArray(value)
    if not TypeChecker.IsTable(value) then return false end
    local count = 0
    for _ in pairs(value) do
        count = count + 1
        if value[count] == nil then return false end
    end
    return true
end

function TypeChecker.IsDictionary(value)
    return TypeChecker.IsTable(value) and not TypeChecker.IsArray(value)
end

-- UE5类型检查
function TypeChecker.IsVector(value)
    return TypeChecker.IsTable(value) and value.X and value.Y and value.Z
end

function TypeChecker.IsRotator(value)
    return TypeChecker.IsTable(value) and value.Pitch and value.Yaw and value.Roll
end

-- 类型验证
function TypeChecker.ValidateType(value, expectedType)
    local actualType = type(value)
    if actualType ~= expectedType then
        error(string.format(
            "类型错误: 期望 %s, 实际为 %s",
            expectedType,
            actualType
        ))
    end
end

-- 使用示例
function ProcessVector(vector)
    TypeChecker.ValidateType(vector, "table")
    if not TypeChecker.IsVector(vector) then
        error("参数必须是向量类型")
    end
    -- 处理向量
end
```

这些数据结构和类型系统工具可以帮助您更好地组织和管理 MOD 的数据。通过使用适当的数据结构和类型检查，可以使代码更加健壮和可维护。记住要根据具体需求选择合适的数据结构，并确保正确处理类型转换和验证。