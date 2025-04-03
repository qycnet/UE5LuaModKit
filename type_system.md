# 类型系统文档

## 基础类型系统

### 1. 虚幻引擎原生类型

#### 1.1 基础数据类型
| 类型名称 | 描述 | Lua中的表示 |
|---------|------|------------|
| bool | 布尔值 | boolean |
| int32 | 32位整数 | number |
| float | 浮点数 | number |
| double | 双精度浮点数 | number |
| FString | 字符串 | string |
| FName | 名称标识符 | string |
| FText | 本地化文本 | string |

#### 1.2 容器类型
| 类型名称 | 描述 | Lua中的表示 |
|---------|------|------------|
| TArray<T> | 动态数组 | table (数组) |
| TMap<K, V> | 键值映射 | table (字典) |
| TSet<T> | 唯一元素集合 | table (集合) |

#### 1.3 数学类型
| 类型名称 | 描述 | Lua中的表示 |
|---------|------|------------|
| FVector | 3D向量 | table {X=x, Y=y, Z=z} |
| FRotator | 旋转器 | table {Pitch=p, Yaw=y, Roll=r} |
| FQuat | 四元数 | table {X=x, Y=y, Z=z, W=w} |
| FTransform | 变换 | table {Position=vector, Rotation=rotator, Scale=vector} |

### 2. 游戏特定类型

#### 2.1 角色相关
| 类型名称 | 描述 | 主要属性 |
|---------|------|---------|
| FPalCharacterData | 角色数据 | ID, Level, Stats, Skills |
| FPalNPCData | NPC数据 | ID, Type, Behavior, Dialog |
| FPalPlayerData | 玩家数据 | CharacterData, Inventory, Quests |

#### 2.2 战斗相关
| 类型名称 | 描述 | 主要属性 |
|---------|------|---------|
| FPalDamageInfo | 伤害信息 | Amount, Type, Source, Target |
| FPalCombatStats | 战斗状态 | Health, Stamina, Attack, Defense |
| FPalSkillData | 技能数据 | ID, Level, Cooldown, Effects |

#### 2.3 物品相关
| 类型名称 | 描述 | 主要属性 |
|---------|------|---------|
| FPalItemData | 物品数据 | ID, Count, Quality, Stats |
| FPalInventoryData | 背包数据 | Slots, Items, Capacity |
| FPalEquipmentData | 装备数据 | Slots, EquippedItems |

## 类型转换与处理

### 1. Lua与C++类型转换

#### 1.1 基础类型转换
```lua
-- 数值转换
local intValue = 42          -- Lua number -> C++ int
local floatValue = 3.14      -- Lua number -> C++ float

-- 字符串转换
local stringValue = "Hello"  -- Lua string -> C++ FString

-- 布尔转换
local boolValue = true       -- Lua boolean -> C++ bool
```

#### 1.2 复杂类型转换
```lua
-- 向量转换
local vector = {X=1, Y=2, Z=3}  -- Lua table -> C++ FVector

-- 数组转换
local array = {1, 2, 3, 4, 5}   -- Lua table -> C++ TArray<int32>

-- 映射转换
local map = {key1="value1", key2="value2"}  -- Lua table -> C++ TMap<FString, FString>
```

### 2. 类型检查与验证

```lua
-- 检查参数类型
function SafeFunction(param)
    if type(param) ~= "table" then
        error("Expected table, got " .. type(param))
    end
    
    -- 检查表中的字段
    if not param.X or type(param.X) ~= "number" then
        error("Expected param.X to be a number")
    end
    
    -- 处理参数
    -- ...
end
```

## 枚举类型

### 1. 常用游戏枚举

#### 1.1 EPalDamageType (伤害类型)
| 枚举值 | 描述 |
|-------|------|
| Physical | 物理伤害 |
| Fire | 火焰伤害 |
| Water | 水系伤害 |
| Electric | 电系伤害 |
| Earth | 土系伤害 |
| Wind | 风系伤害 |
| Dark | 暗系伤害 |
| Light | 光系伤害 |

#### 1.2 EPalCharacterState (角色状态)
| 枚举值 | 描述 |
|-------|------|
| Normal | 正常状态 |
| Battle | 战斗状态 |
| Dead | 死亡状态 |
| Stunned | 眩晕状态 |
| Sleeping | 睡眠状态 |
| Poisoned | 中毒状态 |
| Frozen | 冰冻状态 |
| Burning | 燃烧状态 |

#### 1.3 EPalWeatherType (天气类型)
| 枚举值 | 描述 |
|-------|------|
| Clear | 晴天 |
| Cloudy | 多云 |
| Rain | 雨天 |
| Storm | 暴风雨 |
| Snow | 雪天 |
| Sandstorm | 沙尘暴 |
| Foggy | 雾天 |

### 2. 在Lua中使用枚举

```lua
-- 使用伤害类型枚举
local damageInfo = {
    Type = EPalDamageType.Fire,  -- 使用枚举值
    Amount = 50
}

-- 检查角色状态
local character = GetPlayer()
if character:GetState() == EPalCharacterState.Battle then
    print("玩家正在战斗中!")
end

-- 根据天气类型执行不同操作
local currentWeather = GetCurrentWeather()
if currentWeather == EPalWeatherType.Rain then
    -- 雨天特定逻辑
    ApplyRainEffects()
elseif currentWeather == EPalWeatherType.Clear then
    -- 晴天特定逻辑
    ApplyClearWeatherEffects()
end
```

## 结构体与类

### 1. 常用结构体

#### 1.1 FPalDateTime (游戏内日期时间)
```lua
-- 创建日期时间
local gameTime = FPalDateTime.new(2023, 5, 15, 14, 30, 0)  -- 年、月、日、时、分、秒

-- 获取当前游戏时间
local currentTime = GetWorldTime()
local hour = currentTime:GetHour()
local isNight = hour >= 20 or hour < 6
```

#### 1.2 FPalDamageResult (伤害结果)
```lua
-- 伤害结果结构体
local damageResult = {
    ActualDamage = 45,           -- 实际造成的伤害
    IsCritical = true,           -- 是否暴击
    AbsorbedAmount = 5,          -- 被吸收的伤害
    TargetRemainingHealth = 55   -- 目标剩余生命值
}
```

### 2. 类继承关系

#### 2.1 Actor继承体系
```
AActor
  ├── APawn
  │     ├── ACharacter
  │           ├── APalCharacter
  │                 ├── APalPlayerCharacter
  │                 └── APalNPCCharacter
  ├── AController
  │     ├── APlayerController
  │     └── AAIController
  │           └── APalAIController
  └── AItem
        ├── AWeapon
        ├── AArmor
        └── AConsumable
```

#### 2.2 组件继承体系
```
UActorComponent
  ├── USceneComponent
  │     ├── UPrimitiveComponent
  │     │     ├── UMeshComponent
  │     │     │     ├── UStaticMeshComponent
  │     │     │     └── USkeletalMeshComponent
  │     └── UCameraComponent
  ├── UPalCharacterComponent
  ├── UPalInventoryComponent
  └── UPalCombatComponent
```

## 类型安全与最佳实践

### 1. 类型安全编程
- 始终检查函数参数类型
- 使用明确的类型转换
- 避免隐式类型转换导致的错误

### 2. 数据验证
```lua
-- 验证数值范围
function ValidateHealth(health)
    if type(health) ~= "number" then
        return 0  -- 默认值
    end
    
    -- 限制范围
    return math.max(0, math.min(health, 100))
end

-- 验证复杂对象
function ValidateCharacterData(data)
    if type(data) ~= "table" then
        error("Character data must be a table")
    end
    
    -- 检查必要字段
    local requiredFields = {"ID", "Name", "Level"}
    for _, field in ipairs(requiredFields) do
        if data[field] == nil then
            error("Missing required field: " .. field)
        end
    end
    
    -- 验证数值字段
    data.Level = math.max(1, math.min(data.Level, 100))
    
    return data  -- 返回验证后的数据
end
```

### 3. 错误处理
```lua
-- 使用pcall处理类型错误
local function SafeCall(func, ...)
    local success, result = pcall(func, ...)
    if not success then
        -- 记录错误
        LogError("Type error: " .. tostring(result))
        return nil
    end
    return result
end

-- 使用示例
local result = SafeCall(function()
    return ProcessVector({X="not a number", Y=2, Z=3})  -- 会导致类型错误
end)
```

## 高级类型技术

### 1. 元表与面向对象编程
```lua
-- 创建类
local Character = {}
Character.__index = Character

function Character.new(id, name, level)
    local self = setmetatable({}, Character)
    self.ID = id
    self.Name = name
    self.Level = level
    self.Health = 100
    return self
end

function Character:TakeDamage(amount)
    self.Health = math.max(0, self.Health - amount)
    return self.Health
end

-- 创建实例
local player = Character.new("player1", "Hero", 10)
player:TakeDamage(25)
```

### 2. 泛型编程模拟
```lua
-- 泛型容器模拟
local function CreateArray(valueType)
    local array = {}
    
    -- 添加元素，带类型检查
    function array:Add(value)
        if type(value) ~= valueType then
            error("Type mismatch: expected " .. valueType .. ", got " .. type(value))
        end
        table.insert(self, value)
    end
    
    -- 其他方法...
    
    return array
end

-- 使用示例
local numberArray = CreateArray("number")
numberArray:Add(1)  -- 正确
numberArray:Add(2)  -- 正确
-- numberArray:Add("string")  -- 错误：类型不匹配
```

## 参考资料

1. Lua 5.3 参考手册 - 类型与值
2. 虚幻引擎类型系统文档
3. 游戏API类型参考

更多详细信息请参考：
- [函数接口文档](function_interfaces.md)
- [数据结构文档](data_structures.md)
- [Lua MOD开发指南](lua_modding_guide.md)