# 高级性能优化指南

本指南将介绍一些高级的性能优化技巧，帮助你创建高效的MOD。

## 1. 内存管理

### 1.1 对象池模式

对象池可以显著减少内存分配和垃圾回收：

```lua
-- 创建一个通用对象池
local ObjectPool = {}
ObjectPool.__index = ObjectPool

function ObjectPool.new(factory, resetFunc)
    local self = setmetatable({}, ObjectPool)
    self.objects = {}
    self.factory = factory
    self.resetFunc = resetFunc
    return self
end

function ObjectPool:acquire()
    local obj = table.remove(self.objects) or self.factory()
    return obj
end

function ObjectPool:release(obj)
    if self.resetFunc then
        self.resetFunc(obj)
    end
    table.insert(self.objects, obj)
end

-- 使用示例：粒子效果系统
local particlePool = ObjectPool.new(
    function()
        return {
            x = 0, y = 0,
            velocity = {x = 0, y = 0},
            life = 0,
            active = false
        }
    end,
    function(particle)
        particle.x, particle.y = 0, 0
        particle.velocity.x, particle.velocity.y = 0, 0
        particle.life = 0
        particle.active = false
    end
)

-- 创建粒子效果
function CreateParticleEffect(x, y, count)
    local particles = {}
    for i = 1, count do
        local p = particlePool:acquire()
        p.x, p.y = x, y
        p.velocity.x = math.random(-100, 100)
        p.velocity.y = math.random(-100, 100)
        p.life = 1.0
        p.active = true
        table.insert(particles, p)
    end
    return particles
end
```

### 1.2 弱引用表

使用弱引用表来避免内存泄漏：

```lua
-- 创建缓存系统
local Cache = {
    -- 使用弱引用表存储数据
    data = setmetatable({}, {__mode = "v"}),
    -- 设置缓存超时时间
    timeout = 300  -- 5分钟
}

function Cache:Set(key, value)
    self.data[key] = {
        value = value,
        timestamp = os.time()
    }
end

function Cache:Get(key)
    local entry = self.data[key]
    if entry and (os.time() - entry.timestamp) < self.timeout then
        return entry.value
    end
    -- 自动清理过期数据
    self.data[key] = nil
    return nil
end

-- 定期清理过期数据
SetTimer(60, function()  -- 每分钟执行一次
    local now = os.time()
    for key, entry in pairs(Cache.data) do
        if (now - entry.timestamp) >= Cache.timeout then
            Cache.data[key] = nil
        end
    end
end, true)
```

## 2. 计算优化

### 2.1 空间分区

使用空间分区来优化大世界场景中的对象检测：

```lua
-- 网格系统
local Grid = {
    cellSize = 100,  -- 每个网格的大小
    cells = {}
}

function Grid:GetCellKey(x, y)
    local cellX = math.floor(x / self.cellSize)
    local cellY = math.floor(y / self.cellSize)
    return cellX .. "," .. cellY
end

function Grid:AddObject(obj)
    local key = self:GetCellKey(obj.x, obj.y)
    if not self.cells[key] then
        self.cells[key] = {}
    end
    self.cells[key][obj.id] = obj
end

function Grid:RemoveObject(obj)
    local key = self:GetCellKey(obj.x, obj.y)
    if self.cells[key] then
        self.cells[key][obj.id] = nil
    end
end

function Grid:GetNearbyObjects(x, y, radius)
    local results = {}
    local cellRadius = math.ceil(radius / self.cellSize)
    
    local centerCellX = math.floor(x / self.cellSize)
    local centerCellY = math.floor(y / self.cellSize)
    
    for dx = -cellRadius, cellRadius do
        for dy = -cellRadius, cellRadius do
            local key = (centerCellX + dx) .. "," .. (centerCellY + dy)
            if self.cells[key] then
                for _, obj in pairs(self.cells[key]) do
                    local dx = obj.x - x
                    local dy = obj.y - y
                    if (dx * dx + dy * dy) <= radius * radius then
                        table.insert(results, obj)
                    end
                end
            end
        end
    end
    
    return results
end
```

### 2.2 延迟计算

使用延迟计算来优化性能：

```lua
-- 创建延迟计算包装器
local function CreateLazyValue(computeFunc)
    local lazy = {
        computed = false,
        value = nil,
        compute = computeFunc
    }
    
    return setmetatable(lazy, {
        __index = function(t, k)
            if k == "value" then
                if not t.computed then
                    t.value = t.compute()
                    t.computed = true
                end
                return t.value
            end
        end
    })
end

-- 使用示例
local expensiveCalculation = CreateLazyValue(function()
    -- 假设这是一个耗时的计算
    local result = 0
    for i = 1, 1000000 do
        result = result + i
    end
    return result
end)

-- 只有在实际需要时才会执行计算
print(expensiveCalculation.value)
```

## 3. 渲染优化

### 3.1 视野剔除

实现简单的视野剔除系统：

```lua
local ViewFrustum = {
    x = 0, y = 0,  -- 视点位置
    width = 800,   -- 视野宽度
    height = 600,  -- 视野高度
    range = 1000   -- 视野范围
}

function ViewFrustum:IsVisible(obj)
    -- 快速距离检查
    local dx = obj.x - self.x
    local dy = obj.y - self.y
    if dx * dx + dy * dy > self.range * self.range then
        return false
    end
    
    -- 视野范围检查
    local halfWidth = self.width / 2
    local halfHeight = self.height / 2
    return math.abs(dx) <= halfWidth and math.abs(dy) <= halfHeight
end

-- 优化渲染循环
function RenderScene(objects)
    local visibleObjects = {}
    
    -- 首先收集所有可见对象
    for _, obj in ipairs(objects) do
        if ViewFrustum:IsVisible(obj) then
            table.insert(visibleObjects, obj)
        end
    end
    
    -- 按材质对可见对象进行排序以减少状态切换
    table.sort(visibleObjects, function(a, b)
        return a.material < b.material
    end)
    
    -- 渲染可见对象
    for _, obj in ipairs(visibleObjects) do
        obj:Render()
    end
end
```

### 3.2 LOD系统

实现简单的LOD（细节层次）系统：

```lua
local LODSystem = {
    levels = {
        {distance = 100, detail = "high"},
        {distance = 300, detail = "medium"},
        {distance = 1000, detail = "low"}
    }
}

function LODSystem:GetDetailLevel(distance)
    for _, level in ipairs(self.levels) do
        if distance <= level.distance then
            return level.detail
        end
    end
    return "lowest"
end

function LODSystem:UpdateObject(obj, viewerPos)
    local dx = obj.x - viewerPos.x
    local dy = obj.y - viewerPos.y
    local distance = math.sqrt(dx * dx + dy * dy)
    
    local detail = self:GetDetailLevel(distance)
    if obj.currentDetail ~= detail then
        obj:SetDetailLevel(detail)
        obj.currentDetail = detail
    end
end
```

## 4. 网络优化

### 4.1 数据压缩

实现简单的数据压缩：

```lua
local DataCompression = {
    -- 常用字符串字典
    dictionary = {},
    -- 反向查找表
    reverseDictionary = {}
}

function DataCompression:Initialize()
    -- 初始化常用字符串字典
    local commonStrings = {
        "player", "position", "rotation", "health",
        "damage", "inventory", "equipment", "level"
    }
    
    for i, str in ipairs(commonStrings) do
        self.dictionary[str] = i
        self.reverseDictionary[i] = str
    end
end

function DataCompression:Compress(data)
    local compressed = {}
    
    for key, value in pairs(data) do
        local keyCode = self.dictionary[key] or key
        compressed[keyCode] = value
    end
    
    return compressed
end

function DataCompression:Decompress(compressed)
    local data = {}
    
    for keyCode, value in pairs(compressed) do
        local key = self.reverseDictionary[keyCode] or keyCode
        data[key] = value
    end
    
    return data
end
```

### 4.2 增量更新

实现增量更新系统：

```lua
local DeltaCompression = {
    lastState = {},
    threshold = 0.01  -- 最小变化阈值
}

function DeltaCompression:CalculateDelta(currentState)
    local delta = {}
    local hasChanges = false
    
    for key, value in pairs(currentState) do
        local lastValue = self.lastState[key]
        if not lastValue or math.abs(value - lastValue) > self.threshold then
            delta[key] = value
            hasChanges = true
        end
    end
    
    if hasChanges then
        self.lastState = table.shallow_copy(currentState)
        return delta
    end
    return nil
end

function DeltaCompression:ApplyDelta(delta)
    if not delta then return self.lastState end
    
    local newState = table.shallow_copy(self.lastState)
    for key, value in pairs(delta) do
        newState[key] = value
    end
    
    self.lastState = newState
    return newState
end
```

## 5. 监控与分析

实现性能监控系统：

```lua
local PerformanceMonitor = {
    metrics = {},
    samples = {},
    sampleSize = 100
}

function PerformanceMonitor:StartMeasure(name)
    self.metrics[name] = {
        startTime = os.clock(),
        calls = (self.metrics[name] and self.metrics[name].calls or 0) + 1
    }
end

function PerformanceMonitor:EndMeasure(name)
    local metric = self.metrics[name]
    if metric then
        local duration = os.clock() - metric.startTime
        
        if not self.samples[name] then
            self.samples[name] = {}
        end
        
        table.insert(self.samples[name], duration)
        if #self.samples[name] > self.sampleSize then
            table.remove(self.samples[name], 1)
        end
    end
end

function PerformanceMonitor:GetStats(name)
    local samples = self.samples[name]
    if not samples or #samples == 0 then
        return nil
    end
    
    local sum = 0
    local max = samples[1]
    local min = samples[1]
    
    for _, duration in ipairs(samples) do
        sum = sum + duration
        max = math.max(max, duration)
        min = math.min(min, duration)
    end
    
    return {
        average = sum / #samples,
        max = max,
        min = min,
        calls = self.metrics[name].calls,
        samples = #samples
    }
end

-- 使用示例
function SomeExpensiveFunction()
    PerformanceMonitor:StartMeasure("ExpensiveFunction")
    
    -- 执行一些耗时操作
    local result = 0
    for i = 1, 1000000 do
        result = result + i
    end
    
    PerformanceMonitor:EndMeasure("ExpensiveFunction")
    return result
end

-- 定期输出性能报告
SetTimer(5.0, function()
    local stats = PerformanceMonitor:GetStats("ExpensiveFunction")
    if stats then
        print(string.format(
            "性能报告: 平均耗时=%.4f秒, 最大=%.4f秒, 最小=%.4f秒, 调用次数=%d",
            stats.average, stats.max, stats.min, stats.calls
        ))
    end
end, true)
```

通过实施这些优化技巧，你可以显著提高MOD的性能。记住要根据实际需求和性能瓶颈选择合适的优化策略。使用性能监控工具来识别问题并验证优化效果。