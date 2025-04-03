# UE5LuaModKit 类型系统文档

本文档详细说明了 UE5LuaModKit 中的类型系统，包括基本类型、复合类型、类型转换和类型安全建议。

## 目录

1. [基本类型](#基本类型)
2. [复合类型](#复合类型)
3. [枚举类型](#枚举类型)
4. [类型转换](#类型转换)
5. [类型安全](#类型安全)
6. [最佳实践](#最佳实践)

## 基本类型

### 数值类型

#### Number
表示所有数值类型。

```lua
local integer = 42          -- 整数
local float = 3.14          -- 浮点数
local scientific = 1.2e-10  -- 科学计数法
```

**注意事项**:
- 内部使用双精度浮点数
- 整数运算会自动转换为浮点数
- 建议使用显式的类型转换函数确保精度

### 布尔类型

#### Boolean
表示逻辑值。

```lua
local flag = true
local enabled = false
```

### 字符串类型

#### String
表示文本数据。

```lua
local name = "Player"
local description = 'Item description'
local multiline = [[
    多行
    文本
]]
```

## 复合类型

### 数组类型

#### Array<T>
表示同类型元素的有序集合。

```lua
local numbers = {1, 2, 3, 4, 5}
local strings = {"a", "b", "c"}
```

**方法**:
```lua
Array.Insert(index, value)   -- 插入元素
Array.Remove(index)         -- 删除元素
Array.Find(value)          -- 查找元素
Array.Sort(comparator)     -- 排序
Array.Filter(predicate)    -- 过滤
Array.Map(transform)       -- 转换
```

### 表类型

#### Table
表示键值对的集合。

```lua
local player = {
    name = "Hero",
    level = 1,
    health = 100
}
```

### 类类型

#### Class
表示对象的蓝图。

```lua
local Character = {
    -- 静态属性
    MaxLevel = 100,
    
    -- 构造函数
    new = function(self, name)
        local instance = {
            name = name,
            level = 1,
            health = 100
        }
        setmetatable(instance, {__index = self})
        return instance
    end,
    
    -- 方法
    LevelUp = function(self)
        if self.level < self.MaxLevel then
            self.level = self.level + 1
            return true
        end
        return false
    end
}
```

### 函数类型

#### Function
表示可调用的代码块。

```lua
-- 函数声明
local function Add(a: number, b: number): number
    return a + b
end

-- 函数类型变量
local callback: function(sender: string, data: table)

-- 函数类型参数
local function ProcessEvent(handler: function(event: table))
    -- 使用函数参数
    handler({type = "test"})
end
```

## 枚举类型

### Enum
表示一组命名的常量值。

```lua
local ItemType = {
    WEAPON = "weapon",
    ARMOR = "armor",
    CONSUMABLE = "consumable",
    MATERIAL = "material"
}

local ItemRarity = {
    COMMON = 1,
    UNCOMMON = 2,
    RARE = 3,
    EPIC = 4,
    LEGENDARY = 5
}
```

## 类型转换

### 显式转换

```lua
-- 字符串转数字
local number = tonumber("42")
if number then
    print("转换成功：" .. number)
end

-- 数字转字符串
local str = tostring(42)

-- 布尔转换
local bool = not not value  -- 将任何值转换为布尔值
```

### 类型检查

```lua
-- 类型检查函数
local function CheckType(value, expectedType)
    local actualType = type(value)
    if actualType ~= expectedType then
        error(string.format(
            "类型错误：期望 %s，实际为 %s",
            expectedType,
            actualType
        ))
    end
end

-- 使用示例
local function ProcessNumber(n)
    CheckType(n, "number")
    return n * 2
end
```

## 类型安全

### 类型注解
使用注释提供类型信息。

```lua
---@class Player
---@field name string
---@field level number
---@field health number
local Player = {}

---@param damage number
---@return boolean
function Player:TakeDamage(damage)
    self.health = self.health - damage
    return self.health > 0
end
```

### 类型守卫
使用条件检查确保类型安全。

```lua
local function ProcessItem(item)
    -- 类型守卫
    if type(item) ~= "table" then
        error("item must be a table")
    end
    
    if not item.id or type(item.id) ~= "string" then
        error("item must have a string id")
    end
    
    -- 处理 item
end
```

## 最佳实践

### 1. 使用类型注解
为代码添加类型注解，提高可读性和可维护性。

```lua
---@class Weapon
---@field damage number
---@field durability number
local Weapon = {}

---@param target Character
---@return number actual_damage
function Weapon:Attack(target)
    local damage = self.damage
    self.durability = self.durability - 1
    return damage
end
```

### 2. 实现类型检查

```lua
local TypeChecker = {
    IsNumber = function(value)
        return type(value) == "number"
    end,
    
    IsString = function(value)
        return type(value) == "string"
    end,
    
    IsTable = function(value)
        return type(value) == "table"
    end,
    
    IsFunction = function(value)
        return type(value) == "function"
    end,
    
    IsArray = function(value)
        if not TypeChecker.IsTable(value) then
            return false
        end
        local count = 0
        for _ in pairs(value) do
            count = count + 1
            if value[count] == nil then
                return false
            end
        end
        return true
    end
}
```

### 3. 使用工厂函数创建对象

```lua
local ItemFactory = {
    ---@param itemType string
    ---@param data table
    ---@return Item
    CreateItem = function(itemType, data)
        -- 验证参数
        assert(ItemType[itemType], "Invalid item type")
        assert(type(data) == "table", "Data must be a table")
        
        -- 创建基础物品
        local item = {
            id = data.id or GenerateUUID(),
            type = itemType,
            name = data.name or "Unknown Item",
            description = data.description or "",
            stackable = data.stackable or false,
            maxStack = data.maxStack or 1
        }
        
        -- 根据类型添加特定属性
        if itemType == ItemType.WEAPON then
            item.damage = data.damage or 0
            item.range = data.range or 1
        elseif itemType == ItemType.ARMOR then
            item.defense = data.defense or 0
            item.slot = data.slot or "BODY"
        end
        
        return item
    end
}
```

### 4. 使用接口定义

```lua
---@class IStorage
---@field GetItem fun(self: IStorage, id: string): Item
---@field SetItem fun(self: IStorage, id: string, item: Item): boolean
---@field RemoveItem fun(self: IStorage, id: string): boolean
local IStorage = {}

-- 实现接口
---@class Inventory
---@implements IStorage
local Inventory = {
    items = {}
}

function Inventory:GetItem(id)
    return self.items[id]
end

function Inventory:SetItem(id, item)
    self.items[id] = item
    return true
end

function Inventory:RemoveItem(id)
    if self.items[id] then
        self.items[id] = nil
        return true
    end
    return false
end
```

### 5. 使用泛型

```lua
---@generic T
---@param array T[]
---@param predicate fun(item: T): boolean
---@return T[]
local function Filter(array, predicate)
    local result = {}
    for _, item in ipairs(array) do
        if predicate(item) then
            table.insert(result, item)
        end
    end
    return result
end

-- 使用示例
local numbers = {1, 2, 3, 4, 5}
local evenNumbers = Filter(numbers, function(n) return n % 2 == 0 end)
```

### 6. 错误处理

```lua
---@class Result
---@field success boolean
---@field value any
---@field error string|nil
local Result = {}

function Result.Ok(value)
    return {success = true, value = value, error = nil}
end

function Result.Err(error)
    return {success = false, value = nil, error = error}
end

-- 使用示例
---@param id string
---@return Result
local function FindItem(id)
    if not id then
        return Result.Err("Invalid ID")
    end
    
    local item = Database.GetItem(id)
    if not item then
        return Result.Err("Item not found")
    end
    
    return Result.Ok(item)
end

-- 处理结果
local result = FindItem("sword_001")
if result.success then
    ProcessItem(result.value)
else
    LogError(result.error)
end
```

### 7. 常量定义

```lua
local Constants = {
    MAX_INVENTORY_SLOTS = 40,
    MAX_LEVEL = 100,
    MAX_STAT_VALUE = 999,
    
    DAMAGE_TYPES = {
        PHYSICAL = "physical",
        MAGICAL = "magical",
        TRUE = "true"
    },
    
    ITEM_SLOTS = {
        HEAD = "head",
        BODY = "body",
        HANDS = "hands",
        FEET = "feet",
        WEAPON_MAIN = "weapon_main",
        WEAPON_OFF = "weapon_off"
    }
}

-- 冻结常量表防止修改
setmetatable(Constants, {
    __newindex = function()
        error("Attempt to modify read-only table")
    end
})
```

遵循这些最佳实践可以帮助您编写更可靠、可维护的代码，并减少运行时错误。建议在开发过程中始终保持类型安全意识，并适当使用类型系统提供的功能。