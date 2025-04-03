# 数据结构文档

## 核心数据结构

### 1. 基础类型
项目中使用了以下基础类型：
- FString - 虚幻引擎字符串类型
- TArray - 动态数组容器
- TMap - 键值对映射容器
- FVector - 3D向量
- FRotator - 旋转器
- FTransform - 变换矩阵

### 2. 游戏对象类型

#### 2.1 角色相关
- ABP_Player - 玩家基类
- ABP_NPC_Base - NPC基类
- FPalCharacterData - 角色数据结构

#### 2.2 AI相关
- UPalAIController - AI控制器
- FPalBehaviorTreeData - 行为树数据
- FPalAIActionData - AI行为数据

#### 2.3 动画相关
- UAnimInstance - 动画实例基类
- FAnimNodeData - 动画节点数据
- FPalAnimationState - 动画状态数据

### 3. 系统数据结构

#### 3.1 游戏进程数据
- FPalWorldData - 世界数据
- FPalGameState - 游戏状态
- FPalSaveData - 存档数据

#### 3.2 战斗系统
- FPalDamageInfo - 伤害信息
- FPalCombatStats - 战斗状态
- FPalSkillData - 技能数据

#### 3.3 物品系统
- FPalItemData - 物品数据
- FPalInventoryData - 背包数据
- FPalEquipmentData - 装备数据

## Lua MOD开发中的数据结构使用

### 1. 数据访问
```lua
-- 访问玩家数据示例
local player = GetPlayer()
local playerData = player:GetCharacterData()

-- 访问NPC数据示例
local npc = GetNPCByID(npcId)
local npcData = npc:GetNPCData()
```

### 2. 数据修改
```lua
-- 修改玩家属性示例
local stats = player:GetStats()
stats:ModifyValue("Health", 100)

-- 修改NPC行为示例
local aiController = npc:GetAIController()
aiController:SetBehavior("Passive")
```

### 3. 事件监听
```lua
-- 注册事件监听
RegisterEventHandler("OnPlayerDamaged", function(damageInfo)
    -- 处理伤害事件
end)
```

## 重要注意事项

1. 数据持久化
- 所有需要保存的数据必须使用正确的序列化接口
- 避免直接修改关键游戏数据
- 使用提供的API进行数据存取

2. 性能考虑
- 避免频繁访问大型数据结构
- 合理使用缓存机制
- 注意数据更新的时机

3. 安全性
- 验证所有输入数据
- 使用正确的访问权限
- 避免直接修改关键游戏系统

4. 兼容性
- 注意数据结构版本兼容
- 使用标准的数据交换格式
- 做好版本升级的兼容性处理

更多详细信息请参考：
- [函数接口文档](function_interfaces.md)
- [Lua MOD开发指南](lua_modding_guide.md)
- [类型系统文档](type_system.md)