# 函数接口文档

## 核心系统接口

### 1. 玩家系统

#### GetPlayer()
- 描述：获取当前玩家实例
- 返回：ABP_Player 实例

#### ABP_Player:GetCharacterData()
- 描述：获取玩家角色数据
- 返回：FPalCharacterData 结构

#### ABP_Player:GetInventory()
- 描述：获取玩家背包
- 返回：FPalInventoryData 结构

#### ABP_Player:GetEquipment()
- 描述：获取玩家装备
- 返回：FPalEquipmentData 结构

### 2. NPC系统

#### GetNPCByID(npcId)
- 描述：通过ID获取NPC实例
- 参数：npcId (string) - NPC的唯一标识符
- 返回：ABP_NPC_Base 实例

#### ABP_NPC_Base:GetNPCData()
- 描述：获取NPC数据
- 返回：FPalNPCData 结构

#### ABP_NPC_Base:GetAIController()
- 描述：获取NPC的AI控制器
- 返回：UPalAIController 实例

### 3. 战斗系统

#### ApplyDamage(target, damageInfo)
- 描述：对目标应用伤害
- 参数：
  - target (AActor) - 受伤目标
  - damageInfo (FPalDamageInfo) - 伤害信息
- 返回：实际造成的伤害值 (number)

#### GetCombatStats(character)
- 描述：获取角色的战斗状态
- 参数：character (AActor) - 目标角色
- 返回：FPalCombatStats 结构

### 4. 物品系统

#### CreateItem(itemId, count)
- 描述：创建指定数量的物品
- 参数：
  - itemId (string) - 物品ID
  - count (number) - 物品数量
- 返回：FPalItemData 结构

#### AddItemToInventory(inventory, item)
- 描述：将物品添加到背包
- 参数：
  - inventory (FPalInventoryData) - 目标背包
  - item (FPalItemData) - 要添加的物品
- 返回：是否成功添加 (boolean)

### 5. 世界系统

#### GetWorldTime()
- 描述：获取当前游戏世界时间
- 返回：FDateTime 结构

#### SetWorldTime(newTime)
- 描述：设置游戏世界时间
- 参数：newTime (FDateTime) - 新的时间
- 返回：是否成功设置 (boolean)

#### GetCurrentWeather()
- 描述：获取当前天气状态
- 返回：EPalWeatherType 枚举

## Lua MOD 开发接口

### 1. 事件系统

#### RegisterEventHandler(eventName, handler)
- 描述：注册事件处理函数
- 参数：
  - eventName (string) - 事件名称
  - handler (function) - 处理函数
- 返回：无

#### TriggerEvent(eventName, ...)
- 描述：触发指定事件
- 参数：
  - eventName (string) - 事件名称
  - ... - 可变参数，传递给事件处理函数
- 返回：无

### 2. 定时器

#### SetTimer(delay, callback, repeat)
- 描述：设置定时器
- 参数：
  - delay (number) - 延迟时间（秒）
  - callback (function) - 回调函数
  - repeat (boolean) - 是否重复
- 返回：定时器ID (number)

#### ClearTimer(timerId)
- 描述：清除定时器
- 参数：timerId (number) - 定时器ID
- 返回：是否成功清除 (boolean)

### 3. 数据持久化

#### SaveModData(key, value)
- 描述：保存MOD数据
- 参数：
  - key (string) - 数据键
  - value (any) - 要保存的值
- 返回：是否成功保存 (boolean)

#### LoadModData(key)
- 描述：加载MOD数据
- 参数：key (string) - 数据键
- 返回：保存的值 (any) 或 nil（如果不存在）

## 注意事项

1. 错误处理
- 所有函数调用应该进行错误检查
- 使用 pcall 包装可能抛出异常的函数调用

2. 性能优化
- 避免在频繁调用的函数中执行昂贵的操作
- 合理使用缓存机制减少重复计算

3. 线程安全
- 注意在多线程环境下的数据访问和修改
- 使用提供的同步机制确保线程安全

4. API版本兼容性
- 注意检查API版本，确保兼容性
- 使用废弃API时注意替代方案

5. 模块化和封装
- 将相关功能封装到模块中
- 避免全局变量污染

更多详细信息请参考：
- [Lua MOD开发指南](lua_modding_guide.md)
- [类型系统文档](type_system.md)
- [数据结构文档](data_structures.md)