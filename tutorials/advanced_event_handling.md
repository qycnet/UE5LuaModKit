# 高级事件处理教程

本教程将介绍一些高级的事件处理技巧，帮助你更好地控制游戏中的各种事件。

## 1. 复杂条件事件处理

有时我们需要在特定条件下才触发事件处理。以下是一个示例，展示如何处理玩家在特定区域受到伤害的情况：

```lua
local dangerZones = {
    {x1 = 100, y1 = 100, x2 = 200, y2 = 200},
    {x1 = 300, y1 = 300, x2 = 400, y2 = 400}
}

RegisterEventHandler("OnPlayerDamaged", function(player, damageInfo)
    local playerPos = player:GetPosition()
    
    for _, zone in ipairs(dangerZones) do
        if playerPos.x >= zone.x1 and playerPos.x <= zone.x2 and
           playerPos.y >= zone.y1 and playerPos.y <= zone.y2 then
            -- 玩家在危险区域内受到伤害
            damageInfo.Amount = damageInfo.Amount * 2  -- 双倍伤害
            ShowWarning(player, "你在危险区域受到了双倍伤害！")
            return damageInfo
        end
    end
    
    -- 玩家不在危险区域，正常伤害
    return damageInfo
end)
```

## 2. 事件链和级联事件

有时一个事件的发生可能会触发一系列相关事件。以下是一个示例，展示如何处理玩家击败Boss后的一系列事件：

```lua
RegisterEventHandler("OnBossDefeated", function(boss, player)
    -- 1. 给予玩家奖励
    GiveReward(player, boss.RewardTable)
    
    -- 2. 解锁新区域
    UnlockNewArea(boss.AssociatedArea)
    
    -- 3. 更新任务状态
    UpdateQuestStatus(player, "DefeatBoss_" .. boss.Name, "Completed")
    
    -- 4. 触发庆祝事件
    TriggerEvent("CelebrationEvent", {
        playerName = player:GetName(),
        bossName = boss.Name,
        location = boss:GetLocation()
    })
end)

-- 庆祝事件处理
RegisterEventHandler("CelebrationEvent", function(eventData)
    -- 播放庆祝音乐
    PlayBackgroundMusic("celebration_theme.ogg")
    
    -- 在击败Boss的位置创建烟花效果
    CreateFireworksEffect(eventData.location)
    
    -- 向所有玩家广播消息
    BroadcastMessage(eventData.playerName .. " 击败了 " .. eventData.bossName .. "!")
    
    -- 给予所有在线玩家一些奖励
    for _, onlinePlayer in ipairs(GetOnlinePlayers()) do
        GiveItem(onlinePlayer, "CelebrationToken", 1)
    end
end)
```

## 3. 动态事件注册和注销

在某些情况下，我们可能需要动态地注册或注销事件处理器。以下是一个示例，展示如何创建一个临时的事件处理器：

```lua
local function CreateTemporaryEventHandler(eventName, handler, duration)
    local handlerId = RegisterEventHandler(eventName, handler)
    
    SetTimer(duration, function()
        UnregisterEventHandler(eventName, handlerId)
        print("临时事件处理器已被移除")
    end, false)  -- false表示这是一个一次性定时器
    
    return handlerId
end

-- 使用示例
CreateTemporaryEventHandler("OnPlayerMove", function(player, newPosition)
    -- 在10秒内跟踪玩家移动
    print(player:GetName() .. " 移动到 " .. tostring(newPosition))
end, 10.0)  -- 10秒后自动注销
```

## 4. 事件优先级

如果你的MOD系统支持事件优先级，你可以使用它来确保某些事件处理器先于其他处理器执行：

```lua
-- 假设RegisterEventHandler支持优先级参数
RegisterEventHandler("OnPlayerAttack", function(player, target)
    -- 检查玩家是否有权限攻击目标
    if not CanPlayerAttack(player, target) then
        print(player:GetName() .. " 无权攻击 " .. target:GetName())
        return false  -- 阻止攻击
    end
end, 100)  -- 高优先级，确保在其他处理器之前执行

RegisterEventHandler("OnPlayerAttack", function(player, target)
    -- 正常的攻击逻辑
    PerformAttack(player, target)
end, 0)  -- 默认优先级
```

## 5. 全局事件监听器

有时你可能需要监听所有类型的事件，以便进行日志记录或调试：

```lua
RegisterGlobalEventListener(function(eventName, ...)
    local args = {...}
    local argsStr = table.concat(args, ", ")
    LogEvent(string.format("事件触发: %s, 参数: %s", eventName, argsStr))
end)

-- 禁用全局事件监听器的函数
local function DisableGlobalEventListener()
    -- 实现代码...
end
```

通过这些高级事件处理技巧，你可以更灵活地控制游戏中的各种情况，创建更复杂和动态的MOD行为。记住要谨慎使用这些技巧，以避免创建过于复杂或难以维护的代码。