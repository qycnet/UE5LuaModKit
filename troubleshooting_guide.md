# UE5LuaModKit 故障排除指南

本指南提供了在使用 UE5LuaModKit 开发 MOD 时可能遇到的常见问题及其解决方案。

## 目录

1. [常见错误和解决方案](#常见错误和解决方案)
2. [性能问题诊断](#性能问题诊断)
3. [内存相关问题](#内存相关问题)
4. [调试技巧](#调试技巧)
5. [环境问题](#环境问题)
6. [常见问题解答（FAQ）](#常见问题解答)

## 常见错误和解决方案

### 1. Lua 脚本加载错误

#### 症状
- MOD 无法加载
- 控制台显示 Lua 语法错误
- 游戏启动时报错

#### 解决方案
1. 检查语法错误
```lua
-- 常见语法错误示例
-- 错误：缺少 end 关键字
function MyFunction()
    if something then
        -- 处理逻辑
    -- 缺少 end

-- 正确写法
function MyFunction()
    if something then
        -- 处理逻辑
    end
end
```

2. 检查文件编码
- 确保使用 UTF-8 编码
- 避免使用特殊字符
- 检查文件 BOM 头

3. 检查路径问题
```lua
-- 错误：使用反斜杠
require("mods\\mymod\\main")

-- 正确：使用正斜杠
require("mods/mymod/main")
```

### 2. C++ SDK 集成问题

#### 症状
- 无法访问 C++ 类
- 类型转换错误
- 空指针异常

#### 解决方案
1. 检查类型声明
```lua
-- 错误：直接使用 C++ 类
local myActor = UMyActor()

-- 正确：使用正确的构造方法
local myActor = UMyActor.Create()
```

2. 处理空值检查
```lua
-- 不安全的代码
function HandleActor(actor)
    actor:DoSomething() -- 可能崩溃

-- 安全的代码
function HandleActor(actor)
    if actor and actor:IsValid() then
        actor:DoSomething()
    end
end
```

### 3. 事件系统问题

#### 症状
- 事件未触发
- 事件处理器重复注册
- 内存泄漏

#### 解决方案
1. 事件注册检查
```lua
-- 问题代码：重复注册
function OnInit()
    RegisterEventHandler("GameStart", OnGameStart)
    RegisterEventHandler("GameStart", OnGameStart) -- 重复！

-- 解决方案：使用标记或清理
local eventRegistered = false
function OnInit()
    if not eventRegistered then
        RegisterEventHandler("GameStart", OnGameStart)
        eventRegistered = true
    end
end
```

2. 事件清理
```lua
-- 正确的事件清理
function OnModUnload()
    UnregisterEventHandler("GameStart", OnGameStart)
end
```

## 性能问题诊断

### 1. 性能分析工具

```lua
-- 简单的性能分析器
local Profiler = {
    startTime = {},
    results = {}
}

function Profiler:Start(name)
    self.startTime[name] = os.clock()
end

function Profiler:End(name)
    if self.startTime[name] then
        local duration = os.clock() - self.startTime[name]
        self.results[name] = (self.results[name] or 0) + duration
        self.startTime[name] = nil
    end
end

function Profiler:Report()
    for name, time in pairs(self.results) do
        print(string.format("%s: %.4f seconds", name, time))
    end
end
```

### 2. 常见性能问题

#### 问题：频繁的垃圾回收
```lua
-- 问题代码
function Update()
    local temp = {} -- 每帧创建新表
    -- ...
end

-- 优化后的代码
local temp = {} -- 重用表
function Update()
    table.clear(temp)
    -- ...
end
```

#### 问题：不必要的字符串连接
```lua
-- 问题代码
local str = ""
for i = 1, 1000 do
    str = str .. i -- 每次连接创建新字符串
end

-- 优化后的代码
local t = {}
for i = 1, 1000 do
    t[i] = tostring(i)
end
local str = table.concat(t)
```

## 内存相关问题

### 1. 内存泄漏检测

```lua
-- 内存使用监控
local MemoryMonitor = {
    snapshots = {}
}

function MemoryMonitor:TakeSnapshot(name)
    local mem = collectgarbage("count")
    self.snapshots[name] = mem
    return mem
end

function MemoryMonitor:Compare(name1, name2)
    local diff = self.snapshots[name2] - self.snapshots[name1]
    print(string.format("内存差异 (%s -> %s): %.2f KB", name1, name2, diff))
end
```

### 2. 循环引用检测

```lua
-- 循环引用检测器
local function DetectCycles(t, seen)
    seen = seen or {}
    if type(t) ~= "table" then return false end
    seen[t] = true
    
    for k, v in pairs(t) do
        if type(v) == "table" then
            if seen[v] or DetectCycles(v, seen) then
                print("检测到循环引用:", k)
                return true
            end
        end
    end
    
    return false
end
```

## 调试技巧

### 1. 日志系统

```lua
-- 分级日志系统
local Logger = {
    LEVEL_DEBUG = 1,
    LEVEL_INFO = 2,
    LEVEL_WARN = 3,
    LEVEL_ERROR = 4,
    currentLevel = 1
}

function Logger:Log(level, message, ...)
    if level >= self.currentLevel then
        local levelName = "DEBUG"
        if level == 2 then levelName = "INFO"
        elseif level == 3 then levelName = "WARN"
        elseif level == 4 then levelName = "ERROR"
        end
        
        print(string.format("[%s] %s", levelName, string.format(message, ...)))
    end
end

-- 使用示例
Logger:Log(Logger.LEVEL_DEBUG, "调试信息: %s", "测试")
Logger:Log(Logger.LEVEL_ERROR, "错误: %s", "发生错误")
```

### 2. 变量监视器

```lua
-- 变量监视器
local function WatchVariable(name, getValue)
    local lastValue = getValue()
    return function()
        local currentValue = getValue()
        if currentValue ~= lastValue then
            print(string.format("[监视] %s: %s -> %s",
                name,
                tostring(lastValue),
                tostring(currentValue)))
            lastValue = currentValue
        end
    end
end

-- 使用示例
local health = 100
local watchHealth = WatchVariable("生命值", function() return health end)
```

## 环境问题

### 1. 路径问题

```lua
-- 路径规范化
local function NormalizePath(path)
    -- 转换反斜杠为正斜杠
    path = string.gsub(path, "\\", "/")
    -- 移除重复的斜杠
    path = string.gsub(path, "//+", "/")
    return path
end

-- 检查文件存在
local function FileExists(path)
    local file = io.open(path, "r")
    if file then
        file:close()
        return true
    end
    return false
end
```

### 2. 依赖检查

```lua
-- 依赖检查器
local function CheckDependencies(dependencies)
    local missing = {}
    for _, dep in ipairs(dependencies) do
        if not pcall(require, dep) then
            table.insert(missing, dep)
        end
    end
    
    if #missing > 0 then
        error("缺少依赖模块: " .. table.concat(missing, ", "))
    end
end
```

## 常见问题解答

### Q1: MOD 加载顺序问题
**问题**: 我的 MOD 依赖于其他 MOD，如何确保正确的加载顺序？

**解决方案**:
1. 在 MOD 配置中声明依赖：
```lua
-- mod_config.lua
return {
    name = "MyMod",
    dependencies = {
        "OtherMod1",
        "OtherMod2"
    }
}
```

2. 实现依赖检查：
```lua
function CheckModDependencies()
    local config = require("mod_config")
    for _, dep in ipairs(config.dependencies) do
        if not IsModLoaded(dep) then
            error(string.format("缺少依赖 MOD: %s", dep))
        end
    end
end
```

### Q2: 热重载问题
**问题**: 在开发过程中如何实现 MOD 的热重载？

**解决方案**:
1. 实现重载函数：
```lua
function ReloadMod()
    -- 保存状态
    local savedState = SaveModState()
    
    -- 清理资源
    CleanupResources()
    
    -- 重新加载脚本
    package.loaded["mymod"] = nil
    require("mymod")
    
    -- 恢复状态
    RestoreModState(savedState)
end
```

### Q3: 版本兼容性问题
**问题**: 如何处理不同游戏版本的兼容性？

**解决方案**:
1. 版本检查：
```lua
function CheckGameVersion()
    local currentVersion = GetGameVersion()
    local requiredVersion = "1.2.3"
    
    if not IsVersionCompatible(currentVersion, requiredVersion) then
        error(string.format(
            "MOD 需要游戏版本 %s 或更高，当前版本为 %s",
            requiredVersion,
            currentVersion
        ))
    end
end
```

### Q4: 资源清理问题
**问题**: 如何确保 MOD 卸载时正确清理资源？

**解决方案**:
1. 实现完整的清理函数：
```lua
function CleanupMod()
    -- 清理事件监听器
    UnregisterAllEventHandlers()
    
    -- 清理 UI 元素
    CleanupUI()
    
    -- 清理定时器
    CancelAllTimers()
    
    -- 保存数据
    SaveModData()
    
    -- 清理内存
    collectgarbage("collect")
end
```

### Q5: 错误报告问题
**问题**: 如何收集和报告 MOD 运行时的错误？

**解决方案**:
1. 实现错误处理器：
```lua
local function ErrorHandler(err)
    -- 获取堆栈跟踪
    local trace = debug.traceback(err, 2)
    
    -- 记录错误
    Logger:Log(Logger.LEVEL_ERROR, "MOD 错误:\n%s", trace)
    
    -- 保存错误日志
    local file = io.open("mod_error.log", "a")
    if file then
        file:write(string.format("[%s] %s\n",
            os.date("%Y-%m-%d %H:%M:%S"),
            trace))
        file:close()
    end
    
    return err
end

-- 使用错误处理器
function SafeExecute(func, ...)
    return xpcall(func, ErrorHandler, ...)
end
```

记住，这些解决方案可能需要根据具体情况进行调整。如果问题持续存在或遇到新的问题，建议：

1. 查看最新的文档和更新日志
2. 在官方论坛或 Discord 寻求帮助
3. 提交详细的错误报告，包括：
   - 错误信息和堆栈跟踪
   - MOD 和游戏版本信息
   - 复现步骤
   - 相关的代码片段

保持代码整洁和模块化不仅有助于排查问题，也能让其他开发者更容易理解和维护你的 MOD。