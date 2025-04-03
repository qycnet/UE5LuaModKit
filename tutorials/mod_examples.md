# UE5LuaModKit MOD 开发实例教程

本文档提供了一系列实际的 MOD 开发案例，帮助您更好地理解如何使用 UE5LuaModKit 开发各类 MOD。

## 基础 MOD 示例

### 1. Hello World MOD
```lua
-- mod_entry.lua
function Init()
    print("Hello from UE5LuaModKit!")
    -- 注册基本事件监听
    RegisterEventListener("GameStart", OnGameStart)
end

function OnGameStart()
    -- 显示欢迎消息
    ShowNotification("欢迎使用 UE5LuaModKit!")
end
```

### 2. 简单物品 MOD
```lua
-- custom_item.lua
local MyItem = {
    Name = "神秘药水",
    Description = "一瓶具有神秘效果的药水",
    IconPath = "/Game/UI/Icons/Potion",
    MaxStack = 10
}

function MyItem:OnUse(character)
    -- 使用物品时的效果
    if character then
        character:AddBuff("Heal", 10.0, 5)  -- 添加治疗效果
        return true  -- 消耗物品
    end
    return false
end

-- 注册物品到游戏系统
RegisterCustomItem("MysteryPotion", MyItem)
```

## 中级 MOD 示例

### 1. 自定义 NPC MOD
```lua
-- custom_npc.lua
local MyNPC = {
    DisplayName = "智者",
    ModelPath = "/Game/Characters/Wise_Man",
    DialogueTree = {
        Welcome = {
            Text = "欢迎，旅行者。",
            Options = {
                {
                    Text = "我需要建议",
                    NextNode = "Advice"
                },
                {
                    Text = "再见",
                    NextNode = "Goodbye"
                }
            }
        },
        Advice = {
            Text = "记住，真正的力量来自内心。",
            Options = {
                {
                    Text = "谢谢",
                    NextNode = "Goodbye"
                }
            }
        },
        Goodbye = {
            Text = "愿你的旅程平安。",
            Options = {}
        }
    }
}

function MyNPC:OnInteract(player)
    self:StartDialogue("Welcome")
end

-- 注册 NPC 到游戏系统
RegisterCustomNPC("WiseMan", MyNPC)
```

### 2. 自定义任务 MOD
```lua
-- custom_quest.lua
local MyQuest = {
    Title = "智者的考验",
    Description = "完成智者给予的特殊任务",
    Objectives = {
        {
            ID = "FIND_ITEMS",
            Description = "收集 3 个神秘药水",
            Required = 3,
            Current = 0
        },
        {
            ID = "RETURN",
            Description = "回到智者处",
            Completed = false
        }
    }
}

function MyQuest:OnStart()
    -- 注册事件监听
    RegisterEventListener("ItemCollected", function(itemID)
        if itemID == "MysteryPotion" then
            self:UpdateObjective("FIND_ITEMS", 1)
        end
    end)
end

function MyQuest:UpdateObjective(objectiveID, progress)
    local objective = self:GetObjective(objectiveID)
    if objective then
        objective.Current = objective.Current + progress
        if objective.Current >= objective.Required then
            self:CompleteObjective(objectiveID)
        end
    end
end

-- 注册任务到游戏系统
RegisterCustomQuest("WiseManTest", MyQuest)
```

## 高级 MOD 示例

### 1. 自定义游戏机制 MOD
```lua
-- custom_mechanics.lua
local WeatherSystem = {
    CurrentWeather = "Sunny",
    WeatherEffects = {
        Sunny = {
            ParticleEffect = nil,
            LightIntensity = 1.0,
            BuffName = "SunBlessing"
        },
        Rainy = {
            ParticleEffect = "/Game/Effects/Rain",
            LightIntensity = 0.7,
            BuffName = "WetDebuff"
        },
        Storm = {
            ParticleEffect = "/Game/Effects/Storm",
            LightIntensity = 0.4,
            BuffName = "StormDebuff"
        }
    }
}

function WeatherSystem:Initialize()
    -- 设置天气变化计时器
    self.WeatherTimer = CreateTimer(300.0, function()
        self:ChangeWeather()
    end, true)
end

function WeatherSystem:ChangeWeather()
    local weathers = {"Sunny", "Rainy", "Storm"}
    local newWeather = weathers[math.random(#weathers)]
    
    if newWeather ~= self.CurrentWeather then
        self:ApplyWeatherEffects(newWeather)
        self.CurrentWeather = newWeather
    end
end

function WeatherSystem:ApplyWeatherEffects(weather)
    local effects = self.WeatherEffects[weather]
    if effects then
        -- 应用天气效果
        SetGlobalLightIntensity(effects.LightIntensity)
        if effects.ParticleEffect then
            SpawnWorldParticleSystem(effects.ParticleEffect)
        end
        -- 对所有玩家应用效果
        for _, player in ipairs(GetAllPlayers()) do
            player:AddBuff(effects.BuffName)
        end
    end
end

-- 注册天气系统
RegisterGameSystem("WeatherSystem", WeatherSystem)
```

### 2. 高级 UI MOD
```lua
-- custom_ui.lua
local AdvancedUI = {
    WindowName = "CustomInventoryWindow",
    Width = 800,
    Height = 600
}

function AdvancedUI:Initialize()
    -- 创建主窗口
    self.Window = CreateWindow(self.WindowName, self.Width, self.Height)
    
    -- 添加物品格子
    self.ItemSlots = {}
    for i = 1, 40 do
        local slot = self:CreateItemSlot(i)
        table.insert(self.ItemSlots, slot)
    end
    
    -- 添加分类标签
    self.Categories = {
        self:CreateCategory("武器", 1),
        self:CreateCategory("防具", 2),
        self:CreateCategory("消耗品", 3),
        self:CreateCategory("材料", 4)
    }
    
    -- 注册拖放事件
    self:RegisterDragDropEvents()
end

function AdvancedUI:CreateItemSlot(index)
    local slot = {
        Index = index,
        Item = nil,
        Button = CreateButton(
            string.format("Slot_%d", index),
            (index - 1) % 8 * 100,
            math.floor((index - 1) / 8) * 100,
            90,
            90
        )
    }
    
    -- 添加鼠标悬停提示
    slot.Button:SetTooltipFunction(function()
        if slot.Item then
            return self:GenerateItemTooltip(slot.Item)
        end
        return ""
    end)
    
    return slot
end

function AdvancedUI:CreateCategory(name, index)
    local category = {
        Name = name,
        Index = index,
        Button = CreateButton(
            string.format("Category_%s", name),
            index * 200 - 200,
            0,
            190,
            40
        )
    }
    
    category.Button:SetText(name)
    category.Button:SetOnClick(function()
        self:FilterItems(name)
    end)
    
    return category
end

function AdvancedUI:RegisterDragDropEvents()
    for _, slot in ipairs(self.ItemSlots) do
        slot.Button:SetDraggable(true)
        slot.Button:SetOnDragStart(function()
            if slot.Item then
                return self:CreateDragVisual(slot.Item)
            end
        end)
        
        slot.Button:SetOnDrop(function(droppedItem)
            if self:CanAcceptItem(droppedItem, slot) then
                self:MoveItem(droppedItem, slot)
            end
        end)
    end
end

function AdvancedUI:GenerateItemTooltip(item)
    local tooltip = string.format(
        [[
%s
品质：%s
类型：%s
%s
        ]],
        item.Name,
        item.Quality,
        item.Type,
        item.Description
    )
    return tooltip
end

-- 注册 UI 到游戏系统
RegisterCustomUI("AdvancedInventory", AdvancedUI)
```

## 工具和辅助函数示例

### 1. 调试工具
```lua
-- debug_tools.lua
local DebugTools = {
    IsEnabled = false,
    LogLevel = "INFO"  -- "INFO", "WARNING", "ERROR"
}

function DebugTools:Log(level, message, ...)
    if not self.IsEnabled then return end
    
    local levels = {
        INFO = 1,
        WARNING = 2,
        ERROR = 3
    }
    
    if levels[level] >= levels[self.LogLevel] then
        local formatted = string.format(message, ...)
        local timestamp = os.date("%Y-%m-%d %H:%M:%S")
        print(string.format("[%s][%s] %s", timestamp, level, formatted))
    end
end

function DebugTools:StartProfiling(name)
    if not self.IsEnabled then return end
    self.ProfilingStart = os.clock()
    self:Log("INFO", "开始性能分析: %s", name)
end

function DebugTools:EndProfiling(name)
    if not self.IsEnabled then return end
    local duration = os.clock() - (self.ProfilingStart or 0)
    self:Log("INFO", "性能分析结果 - %s: %.4f 秒", name, duration)
end

-- 注册调试工具
_G.DebugTools = DebugTools
```

### 2. 工具函数库
```lua
-- utility_functions.lua
local Utils = {}

-- 表格深拷贝
function Utils.DeepCopy(orig)
    local copy
    if type(orig) == 'table' then
        copy = {}
        for k, v in next, orig, nil do
            copy[Utils.DeepCopy(k)] = Utils.DeepCopy(v)
        end
        setmetatable(copy, Utils.DeepCopy(getmetatable(orig)))
    else
        copy = orig
    end
    return copy
end

-- 表格合并
function Utils.MergeTables(t1, t2)
    local result = Utils.DeepCopy(t1)
    for k, v in pairs(t2) do
        if type(v) == "table" and type(result[k]) == "table" then
            result[k] = Utils.MergeTables(result[k], v)
        else
            result[k] = v
        end
    end
    return result
end

-- 字符串分割
function Utils.Split(str, sep)
    local fields = {}
    local pattern = string.format("([^%s]+)", sep)
    str:gsub(pattern, function(c) fields[#fields + 1] = c end)
    return fields
end

-- 向量操作
Utils.Vector = {
    Add = function(v1, v2)
        return {
            x = v1.x + v2.x,
            y = v1.y + v2.y,
            z = v1.z + v2.z
        }
    end,
    
    Subtract = function(v1, v2)
        return {
            x = v1.x - v2.x,
            y = v1.y - v2.y,
            z = v1.z - v2.z
        }
    end,
    
    Length = function(v)
        return math.sqrt(v.x * v.x + v.y * v.y + v.z * v.z)
    end,
    
    Normalize = function(v)
        local len = Utils.Vector.Length(v)
        return {
            x = v.x / len,
            y = v.y / len,
            z = v.z / len
        }
    end
}

-- 注册工具函数库
_G.Utils = Utils
```

## 配置文件示例

### 1. MOD 配置文件
```lua
-- config.lua
return {
    ModInfo = {
        Name = "高级物品系统",
        Author = "YourName",
        Version = "1.0.0",
        Description = "添加了新的物品系统和界面",
        Dependencies = {
            "BaseGame >= 1.0.0",
            "InventorySystem >= 2.0.0"
        }
    },
    
    Settings = {
        EnableDebug = false,
        MaxInventorySlots = 40,
        DefaultCategory = "其他",
        RefreshRate = 0.5,
        
        Categories = {
            "武器",
            "防具",
            "消耗品",
            "材料",
            "其他"
        },
        
        ItemQuality = {
            Common = {
                Color = "#FFFFFF",
                Multiplier = 1.0
            },
            Rare = {
                Color = "#0070DD",
                Multiplier = 1.5
            },
            Epic = {
                Color = "#A335EE",
                Multiplier = 2.0
            },
            Legendary = {
                Color = "#FF8000",
                Multiplier = 3.0
            }
        }
    },
    
    Hotkeys = {
        ToggleInventory = "I",
        QuickSort = "R",
        StackAll = "T"
    }
}
```

这些示例展示了从基础到高级的各种 MOD 开发场景，包括：
1. 基础功能实现
2. 自定义物品和 NPC
3. 任务系统开发
4. 高级游戏机制
5. 复杂 UI 系统
6. 调试工具
7. 实用工具函数
8. 配置系统

每个示例都包含了详细的代码注释和实现说明，可以作为开发参考。