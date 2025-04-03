# Lua MOD 开发指南

## 基础设置

### 1. MOD 结构
一个标准的 Lua MOD 应该包含以下文件结构：
```
MyMod/
  ├── mod.lua      # 主入口文件
  ├── config.json  # 配置文件
  ├── scripts/     # 脚本目录
  │   ├── init.lua # 初始化脚本
  │   └── ...      # 其他脚本文件
  ├── assets/      # 资源目录
  │   ├── textures/
  │   ├── sounds/
  │   └── ...
  └── localization/ # 本地化目录
      ├── en.json
      ├── zh.json
      └── ...
```

### 2. 入口文件
每个 MOD 必须包含一个 `mod.lua` 文件作为入口点，示例：
```lua
-- mod.lua
local MyMod = {}

-- MOD 元数据
MyMod.Name = "我的第一个MOD"
MyMod.Version = "1.0.0"
MyMod.Author = "开发者名称"
MyMod.Description = "这是一个示例MOD"

-- 初始化函数
function MyMod:Initialize()
    print("MOD已加载!")
    -- 加载其他脚本
    require("scripts/init")
end

-- 注册MOD
RegisterMod(MyMod)
```

## 开发流程

### 1. 创建新MOD
1. 在游戏的 `Mods` 目录下创建新文件夹
2. 创建基本文件结构
3. 实现 `mod.lua` 入口文件
4. 创建配置文件

### 2. 开发与测试
1. 编写MOD逻辑
2. 使用游戏内置的MOD测试工具进行测试
3. 检查日志文件查看错误和警告
4. 迭代改进

### 3. 发布
1. 打包MOD文件
2. 编写详细的安装和使用说明
3. 上传到MOD分享平台

## 核心功能实现

### 1. 游戏事件监听
```lua
-- 监听玩家受伤事件
RegisterEventHandler("OnPlayerDamaged", function(damageInfo)
    print("玩家受到了 " .. damageInfo.Amount .. " 点伤害")
    
    -- 修改伤害值（示例：减少50%伤害）
    damageInfo.Amount = damageInfo.Amount * 0.5
    return damageInfo
end)

-- 监听NPC生成事件
RegisterEventHandler("OnNPCSpawned", function(npc)
    print("NPC已生成: " .. npc:GetName())
    
    -- 自定义NPC行为
    local aiController = npc:GetAIController()
    aiController:SetBehavior("Friendly")
end)
```

### 2. 自定义UI
```lua
-- 创建自定义UI窗口
local myWindow = CreateWindow("我的MOD窗口", 400, 300)

-- 添加按钮
local button = myWindow:AddButton("点击我", 150, 100, 100, 30)
button.OnClick = function()
    print("按钮被点击了!")
end

-- 显示窗口
myWindow:Show()
```

### 3. 命令系统
```lua
-- 注册自定义命令
RegisterCommand("heal", function(args)
    local amount = tonumber(args[1]) or 100
    local player = GetPlayer()
    local stats = player:GetStats()
    
    stats:ModifyValue("Health", amount)
    print("已恢复 " .. amount .. " 点生命值")
end, "恢复生命值，用法: /heal [数量]")
```

### 4. 物品修改
```lua
-- 修改物品属性
RegisterEventHandler("OnItemCreated", function(item)
    if item:GetID() == "Weapon_Sword_01" then
        -- 提升武器伤害
        item:ModifyProperty("Damage", function(value)
            return value * 1.5 -- 增加50%伤害
        end)
    end
end)
```

### 5. 数据持久化
```lua
-- 保存MOD数据
local function SavePlayerProgress()
    local player = GetPlayer()
    local position = player:GetLocation()
    
    -- 保存数据
    SaveModData("LastPosition", {
        x = position.X,
        y = position.Y,
        z = position.Z
    })
end

-- 加载MOD数据
local function LoadPlayerProgress()
    local savedPos = LoadModData("LastPosition")
    if savedPos then
        local player = GetPlayer()
        player:SetLocation(FVector(savedPos.x, savedPos.y, savedPos.z))
        print("已恢复玩家位置")
    end
end
```

## 高级技巧

### 1. 性能优化
```lua
-- 缓存频繁使用的数据
local cachedData = {}

-- 使用定时器批量处理
SetTimer(1.0, function()
    -- 每秒执行一次的批量处理
    ProcessBatchedOperations()
end, true)

-- 避免每帧执行昂贵操作
RegisterEventHandler("OnTick", function(deltaTime)
    -- 使用计数器控制执行频率
    tickCounter = (tickCounter or 0) + 1
    if tickCounter % 60 == 0 then -- 每60帧执行一次
        PerformExpensiveOperation()
    end
end)
```

### 2. 模块化开发
```lua
-- 模块定义 (scripts/combat.lua)
local CombatModule = {}

function CombatModule.CalculateDamage(attacker, defender, baseAmount)
    -- 复杂的伤害计算逻辑
    return finalDamage
end

return CombatModule

-- 在主文件中使用模块
local CombatSystem = require("scripts/combat")
local damage = CombatSystem.CalculateDamage(player, enemy, 100)
```

### 3. 错误处理
```lua
-- 使用pcall安全调用函数
local function SafeExecute(func, ...)
    local success, result = pcall(func, ...)
    if not success then
        print("错误: " .. tostring(result))
        -- 记录错误到日志
        LogError("MOD执行错误: " .. tostring(result))
        return nil
    end
    return result
end

-- 使用示例
SafeExecute(function()
    -- 可能会出错的代码
    local result = SomeRiskyFunction()
    -- 处理结果
end)
```

## 常见问题与解决方案

### 1. MOD冲突
- 使用唯一的命名空间
- 检查其他MOD的兼容性
- 实现冲突检测和解决机制

### 2. 游戏更新兼容性
- 使用版本检查机制
- 实现优雅降级策略
- 保持MOD结构模块化，便于更新

### 3. 性能问题
- 避免在主线程中执行耗时操作
- 使用缓存减少重复计算
- 优化事件处理函数

### 4. 内存泄漏
- 正确清理不再使用的资源
- 避免创建过多的全局变量
- 使用弱引用表管理临时对象

## 调试技巧

### 1. 日志输出
```lua
-- 不同级别的日志
LogDebug("调试信息")
LogInfo("一般信息")
LogWarning("警告信息")
LogError("错误信息")

-- 条件日志
if IsDebugMode() then
    LogDebug("详细调试信息: " .. inspect(complexObject))
end
```

### 2. 调试控制台
- 使用游戏内置的调试控制台
- 实现自定义调试命令
- 使用调试可视化工具

### 3. 性能分析
```lua
-- 简单的性能计时器
local startTime = os.clock()
-- 执行需要测量的代码
local elapsedTime = os.clock() - startTime
print("操作耗时: " .. elapsedTime .. " 秒")
```

## 最佳实践

1. 保持MOD模块化和可扩展
2. 遵循Lua编码规范
3. 编写详细的文档和注释
4. 实现完善的错误处理
5. 尊重游戏原有的平衡性
6. 提供配置选项增强用户体验
7. 定期更新和维护MOD

## 资源与参考

1. Lua 5.3 参考手册
2. 游戏官方MOD API文档
3. 社区MOD开发论坛
4. 示例MOD和教程

更多详细信息请参考：
- [函数接口文档](function_interfaces.md)
- [数据结构文档](data_structures.md)
- [类型系统文档](type_system.md)