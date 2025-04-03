# UE5LuaModKit API 参考文档

本文档提供了 UE5LuaModKit 中所有可用 API 的详细说明。这些 API 涵盖了游戏开发的各个方面，从核心系统到特定功能模块。

## 目录

1. [核心系统 API](#核心系统-api)
2. [游戏对象 API](#游戏对象-api)
3. [UI 系统 API](#ui-系统-api)
4. [物理系统 API](#物理系统-api)
5. [音频系统 API](#音频系统-api)
6. [网络系统 API](#网络系统-api)
7. [工具函数 API](#工具函数-api)
8. [事件系统 API](#事件系统-api)

## 核心系统 API

### GetEngine()
获取游戏引擎实例。

**返回值**:
- `Engine`: 游戏引擎实例

**示例**:
```lua
local engine = GetEngine()
print("当前引擎版本：" .. engine:GetVersion())
```

### LoadModule(moduleName)
加载指定的模块。

**参数**:
- `moduleName` (string): 要加载的模块名称

**返回值**:
- `table`: 加载的模块

**示例**:
```lua
local mathUtils = LoadModule("MathUtils")
local result = mathUtils.CalculateDistance(point1, point2)
```

### RegisterMod(modInfo)
注册一个新的 MOD。

**参数**:
- `modInfo` (table): MOD 信息，包含 name, version, author 等字段

**返回值**:
- `boolean`: 注册是否成功

**示例**:
```lua
local success = RegisterMod({
    name = "Enhanced UI",
    version = "1.0.0",
    author = "John Doe"
})
```

## 游戏对象 API

### GetPlayer()
获取当前玩家对象。

**返回值**:
- `Player`: 当前玩家对象

**示例**:
```lua
local player = GetPlayer()
print("玩家名称：" .. player:GetName())
```

### CreateActor(actorClass, location, rotation)
在指定位置和旋转创建一个 Actor。

**参数**:
- `actorClass` (string): Actor 类名
- `location` (Vector): 创建位置
- `rotation` (Rotator): 创建旋转

**返回值**:
- `Actor`: 创建的 Actor 对象

**示例**:
```lua
local newActor = CreateActor("BP_Enemy", Vector(100, 200, 0), Rotator(0, 90, 0))
```

### DestroyActor(actor)
销毁指定的 Actor。

**参数**:
- `actor` (Actor): 要销毁的 Actor

**返回值**:
- `boolean`: 销毁是否成功

**示例**:
```lua
local success = DestroyActor(someActor)
```

## UI 系统 API

### CreateWidget(widgetClass)
创建一个 UI 控件。

**参数**:
- `widgetClass` (string): 控件类名

**返回值**:
- `Widget`: 创建的控件对象

**示例**:
```lua
local button = CreateWidget("Button")
button:SetText("点击我")
```

### AddWidgetToViewport(widget)
将控件添加到视口。

**参数**:
- `widget` (Widget): 要添加的控件

**返回值**:
- `boolean`: 添加是否成功

**示例**:
```lua
local success = AddWidgetToViewport(myWidget)
```

## 物理系统 API

### RayCast(start, end, traceChannel)
进行射线检测。

**参数**:
- `start` (Vector): 射线起点
- `end` (Vector): 射线终点
- `traceChannel` (string): 检测通道

**返回值**:
- `table`: 检测结果，包含是否命中、命中位置等信息

**示例**:
```lua
local result = RayCast(Vector(0, 0, 100), Vector(0, 0, -100), "Visibility")
if result.bBlockingHit then
    print("命中位置：" .. tostring(result.Location))
end
```

## 音频系统 API

### PlaySound2D(soundAsset)
播放 2D 音效。

**参数**:
- `soundAsset` (string): 音效资源路径

**返回值**:
- `boolean`: 播放是否成功

**示例**:
```lua
local success = PlaySound2D("/Game/Sounds/UI/ButtonClick")
```

### PlaySound3D(soundAsset, location)
在 3D 空间中播放音效。

**参数**:
- `soundAsset` (string): 音效资源路径
- `location` (Vector): 播放位置

**返回值**:
- `boolean`: 播放是否成功

**示例**:
```lua
local success = PlaySound3D("/Game/Sounds/Explosion", Vector(100, 200, 50))
```

## 网络系统 API

### SendNetworkMessage(messageType, data)
发送网络消息。

**参数**:
- `messageType` (string): 消息类型
- `data` (table): 消息数据

**返回值**:
- `boolean`: 发送是否成功

**示例**:
```lua
local success = SendNetworkMessage("PlayerAction", {action = "Jump", position = Vector(100, 200, 300)})
```

### RegisterNetworkHandler(messageType, handler)
注册网络消息处理函数。

**参数**:
- `messageType` (string): 消息类型
- `handler` (function): 处理函数

**返回值**:
- `boolean`: 注册是否成功

**示例**:
```lua
local success = RegisterNetworkHandler("PlayerAction", function(data)
    print("收到玩家动作：" .. data.action)
end)
```

## 工具函数 API

### Vector(x, y, z)
创建一个 3D 向量。

**参数**:
- `x` (number): X 坐标
- `y` (number): Y 坐标
- `z` (number): Z 坐标

**返回值**:
- `Vector`: 创建的向量对象

**示例**:
```lua
local position = Vector(100, 200, 50)
```

### Rotator(pitch, yaw, roll)
创建一个旋转器。

**参数**:
- `pitch` (number): 俯仰角
- `yaw` (number): 偏航角
- `roll` (number): 滚转角

**返回值**:
- `Rotator`: 创建的旋转器对象

**示例**:
```lua
local rotation = Rotator(0, 90, 0)
```

## 事件系统 API

### RegisterEventListener(eventName, listener)
注册事件监听器。

**参数**:
- `eventName` (string): 事件名称
- `listener` (function): 监听器函数

**返回值**:
- `number`: 监听器 ID

**示例**:
```lua
local listenerId = RegisterEventListener("PlayerDeath", function(player)
    print(player:GetName() .. " 已阵亡")
end)
```

### UnregisterEventListener(listenerId)
注销事件监听器。

**参数**:
- `listenerId` (number): 监听器 ID

**返回值**:
- `boolean`: 注销是否成功

**示例**:
```lua
local success = UnregisterEventListener(listenerId)
```

### TriggerEvent(eventName, ...)
触发一个事件。

**参数**:
- `eventName` (string): 事件名称
- `...`: 可变参数，传递给事件监听器的参数

**返回值**:
- `boolean`: 触发是否成功

**示例**:
```lua
local success = TriggerEvent("LevelComplete", 1, 1000)
```

---

这个 API 参考文档涵盖了 UE5LuaModKit 中的主要功能。每个 API 都提供了详细的说明、参数列表、返回值说明和使用示例。在实际开发中，您可以根据需要组合使用这些 API 来实现复杂的游戏功能。如果您需要更多关于特定 API 的信息或使用示例，请参考其他相关文档或联系我们的支持团队。