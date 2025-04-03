# UE5LuaModKit 数据结构文档

本文档详细说明了 UE5LuaModKit 中的核心数据结构及其使用方法。

## 目录

1. [基础数据类型](#基础数据类型)
2. [游戏对象结构](#游戏对象结构)
3. [UI 相关结构](#ui-相关结构)
4. [配置数据结构](#配置数据结构)
5. [网络数据结构](#网络数据结构)
6. [事件数据结构](#事件数据结构)

## 基础数据类型

### Vector (向量)
表示 3D 空间中的位置或方向。

```lua
local vector = {
    x = 0.0,  -- X 坐标
    y = 0.0,  -- Y 坐标
    z = 0.0   -- Z 坐标
}
```

**示例使用**:
```lua
local position = Vector(100, 200, 300)
local direction = Vector(0, 1, 0)  -- 向前的单位向量
```

### Rotator (旋转器)
表示 3D 空间中的旋转。

```lua
local rotator = {
    pitch = 0.0,  -- 俯仰角
    yaw = 0.0,    -- 偏航角
    roll = 0.0    -- 滚转角
}
```

**示例使用**:
```lua
local rotation = Rotator(0, 90, 0)  -- 向右旋转 90 度
```

### Transform (变换)
组合了位置、旋转和缩放的完整变换信息。

```lua
local transform = {
    location = Vector(0, 0, 0),
    rotation = Rotator(0, 0, 0),
    scale = Vector(1, 1, 1)
}
```

**示例使用**:
```lua
local objectTransform = {
    location = Vector(100, 100, 0),
    rotation = Rotator(0, 45, 0),
    scale = Vector(2, 2, 2)
}
```

## 游戏对象结构

### Actor (游戏对象基类)
所有游戏中可放置对象的基础结构。

```lua
local actor = {
    ID = "",                -- 唯一标识符
    Name = "",             -- 显示名称
    Transform = {},        -- 变换信息
    Tags = {},            -- 标签数组
    Components = {},      -- 组件数组
    IsValid = true,       -- 是否有效
    OwningPlayer = nil    -- 所属玩家
}
```

### Character (角色)
继承自 Actor，表示可控制的角色。

```lua
local character = {
    -- 继承自 Actor 的属性
    ID = "",
    Name = "",
    Transform = {},
    
    -- Character 特有属性
    Health = 100,
    MaxHealth = 100,
    Stamina = 100,
    MaxStamina = 100,
    Level = 1,
    Experience = 0,
    Inventory = {},
    Equipment = {},
    Abilities = {},
    Stats = {
        Strength = 10,
        Agility = 10,
        Intelligence = 10
    }
}
```

### Item (物品)
表示游戏中的物品。

```lua
local item = {
    ID = "",               -- 物品 ID
    Name = "",            -- 显示名称
    Description = "",     -- 描述
    IconPath = "",       -- 图标路径
    Type = "",           -- 物品类型
    Rarity = "Common",   -- 稀有度
    MaxStack = 99,       -- 最大堆叠数量
    Weight = 1.0,        -- 重量
    Value = 0,           -- 价值
    Stats = {},          -- 属性
    Effects = {},        -- 效果
    UseFunction = nil    -- 使用函数
}
```

### Quest (任务)
表示游戏中的任务。

```lua
local quest = {
    ID = "",               -- 任务 ID
    Title = "",           -- 标题
    Description = "",     -- 描述
    Level = 1,           -- 推荐等级
    IsActive = false,    -- 是否激活
    IsCompleted = false, -- 是否完成
    Objectives = {       -- 任务目标
        {
            ID = "",
            Description = "",
            Required = 0,
            Current = 0,
            IsCompleted = false
        }
    },
    Rewards = {          -- 任务奖励
        Experience = 0,
        Gold = 0,
        Items = {}
    },
    PreRequisites = {},  -- 前置条件
    NextQuests = {}      -- 后续任务
}
```

## UI 相关结构

### Window (窗口)
表示 UI 窗口。

```lua
local window = {
    Name = "",           -- 窗口名称
    Title = "",          -- 窗口标题
    Width = 800,         -- 宽度
    Height = 600,        -- 高度
    IsVisible = true,    -- 是否可见
    IsDraggable = true, -- 是否可拖动
    Children = {},       -- 子控件
    Style = {}          -- 样式
}
```

### UIElement (UI 元素基类)
所有 UI 控件的基础结构。

```lua
local uiElement = {
    Name = "",          -- 元素名称
    Type = "",          -- 元素类型
    X = 0,             -- X 坐标
    Y = 0,             -- Y 坐标
    Width = 100,       -- 宽度
    Height = 30,       -- 高度
    IsVisible = true,  -- 是否可见
    IsEnabled = true,  -- 是否启用
    Parent = nil,      -- 父元素
    Style = {}         -- 样式
}
```

### Button (按钮)
继承自 UIElement。

```lua
local button = {
    -- 继承自 UIElement 的属性
    Name = "",
    Type = "Button",
    X = 0,
    Y = 0,
    Width = 100,
    Height = 30,
    
    -- Button 特有属性
    Text = "",           -- 按钮文本
    IconPath = "",       -- 图标路径
    IsPressed = false,   -- 是否按下
    IsHovered = false,   -- 是否悬停
    OnClick = nil,       -- 点击回调
    OnHover = nil        -- 悬停回调
}
```

## 配置数据结构

### ModConfig (MOD 配置)
MOD 的配置信息。

```lua
local modConfig = {
    Info = {
        Name = "",           -- MOD 名称
        Author = "",         -- 作者
        Version = "",        -- 版本
        Description = "",    -- 描述
        Dependencies = {}    -- 依赖项
    },
    Settings = {
        -- 自定义设置
    },
    Data = {
        -- 自定义数据
    }
}
```

### GameSettings (游戏设置)
游戏的通用设置。

```lua
local gameSettings = {
    Graphics = {
        Resolution = {
            Width = 1920,
            Height = 1080
        },
        Quality = "High",
        FullScreen = true,
        VSync = true
    },
    Audio = {
        Master = 1.0,
        Music = 0.8,
        SFX = 0.7,
        Voice = 1.0
    },
    Controls = {
        MouseSensitivity = 1.0,
        InvertY = false,
        KeyBindings = {}
    }
}
```

## 网络数据结构

### NetworkMessage (网络消息)
网络通信的消息结构。

```lua
local networkMessage = {
    Type = "",           -- 消息类型
    Data = {},          -- 消息数据
    Sender = "",        -- 发送者
    Receiver = "",      -- 接收者
    Timestamp = 0,      -- 时间戳
    Priority = 0,       -- 优先级
    Reliable = true     -- 是否可靠传输
}
```

### NetworkPlayer (网络玩家)
网络同步的玩家信息。

```lua
local networkPlayer = {
    ID = "",               -- 玩家 ID
    Name = "",            -- 玩家名称
    Transform = {},       -- 变换信息
    State = "",          -- 状态
    LastUpdate = 0,      -- 最后更新时间
    Ping = 0,           -- 延迟
    IsLocal = false     -- 是否本地玩家
}
```

## 事件数据结构

### EventData (事件数据)
事件系统中传递的数据结构。

```lua
local eventData = {
    Type = "",           -- 事件类型
    Source = nil,        -- 事件源
    Target = nil,        -- 事件目标
    Data = {},          -- 事件数据
    Timestamp = 0,      -- 触发时间
    Handled = false     -- 是否已处理
}
```

### CallbackHandle (回调句柄)
事件回调的句柄结构。

```lua
local callbackHandle = {
    ID = "",            -- 句柄 ID
    EventType = "",     -- 事件类型
    Priority = 0,       -- 优先级
    IsActive = true,    -- 是否激活
    Function = nil      -- 回调函数
}
```

## 使用示例

### 创建和使用物品

```lua
-- 定义物品
local healthPotion = {
    ID = "HealthPotion",
    Name = "生命药水",
    Description = "恢复 50 点生命值",
    IconPath = "/Game/UI/Icons/HealthPotion",
    Type = "Consumable",
    Rarity = "Common",
    MaxStack = 10,
    Value = 100,
    UseFunction = function(player)
        if player.Health < player.MaxHealth then
            player.Health = math.min(player.Health + 50, player.MaxHealth)
            return true  -- 消耗物品
        end
        return false    -- 不消耗物品
    end
}

-- 注册物品
RegisterCustomItem("HealthPotion", healthPotion)

-- 使用物品
local player = GetPlayer()
local item = player:GetInventoryItem("HealthPotion")
if item then
    item:Use(player)
end
```

### 创建和管理任务

```lua
-- 定义任务
local collectionQuest = {
    ID = "HerbCollection",
    Title = "草药收集",
    Description = "为药剂师收集草药",
    Level = 5,
    Objectives = {
        {
            ID = "CollectHerbs",
            Description = "收集 10 个草药",
            Required = 10,
            Current = 0
        }
    },
    Rewards = {
        Experience = 500,
        Gold = 100,
        Items = {
            {ID = "HealthPotion", Count = 3}
        }
    }
}

-- 注册任务
RegisterCustomQuest("HerbCollection", collectionQuest)

-- 更新任务进度
local quest = GetQuest("HerbCollection")
quest:UpdateObjective("CollectHerbs", 1)  -- 收集到一个草药

-- 检查任务完成
if quest:IsCompleted() then
    quest:Reward(GetPlayer())  -- 发放奖励
end
```

### 创建自定义 UI

```lua
-- 创建物品栏窗口
local inventoryWindow = {
    Name = "InventoryWindow",
    Title = "物品栏",
    Width = 400,
    Height = 600,
    IsVisible = false,
    
    Initialize = function(self)
        -- 创建网格布局
        self.Grid = CreateGridLayout(8, 5)  -- 8x5 的物品格子
        
        -- 创建物品格子
        for i = 1, 40 do
            local slot = CreateItemSlot(i)
            self.Grid:AddChild(slot)
        end
        
        -- 创建按钮
        self.SortButton = CreateButton("SortButton", "排序")
        self.SortButton:SetOnClick(function()
            self:SortItems()
        end)
    end,
    
    SortItems = function(self)
        -- 实现物品排序逻辑
    end
}

-- 注册窗口
RegisterCustomWindow("Inventory", inventoryWindow)

-- 显示窗口
ShowWindow("Inventory")
```

### 使用网络功能

```lua
-- 定义网络消息处理
RegisterNetworkMessageHandler("PlayerAction", function(message)
    local player = GetPlayerByID(message.Sender)
    if player then
        if message.Data.actionType == "Jump" then
            player:PlayAnimation("Jump")
        elseif message.Data.actionType == "Attack" then
            player:PerformAttack(message.Data.attackType)
        end
    end
end)

-- 发送网络消息
SendNetworkMessage("PlayerAction", {
    actionType = "Jump",
    position = Vector(100, 200, 300)
})
```

### 使用事件系统

```lua
-- 注册事件监听
local handler = RegisterEventListener("PlayerLevelUp", function(player, newLevel)
    -- 创建事件数据
    local eventData = {
        Type = "PlayerLevelUp",
        Source = player,
        Data = {
            OldLevel = newLevel - 1,
            NewLevel = newLevel,
            BonusPoints = 5
        }
    }
    
    -- 处理升级奖励
    player:AddAttributePoints(eventData.Data.BonusPoints)
    ShowNotification(string.format(
        "恭喜！%s 升级到 %d 级！获得 %d 点属性点。",
        player:GetName(),
        newLevel,
        eventData.Data.BonusPoints
    ))
end)

-- 触发事件
TriggerEvent("PlayerLevelUp", GetPlayer(), 10)

-- 注销事件监听
UnregisterEventListener(handler.ID)
```

这些示例展示了如何使用 UE5LuaModKit 中的各种数据结构来实现游戏功能。每个数据结构都设计用于特定的用途，并可以根据需要进行扩展和修改。在实际开发中，您可以组合使用这些结构来创建复杂的游戏功能。