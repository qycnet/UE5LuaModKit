# UE5LuaModKit 社区贡献指南

欢迎来到 UE5LuaModKit 社区！我们非常感谢您对项目的兴趣，并期待您的贡献。本指南旨在帮助您了解如何参与到项目中来，包括报告问题、提交改进建议、贡献代码等。

## 目录

1. [行为准则](#行为准则)
2. [如何贡献](#如何贡献)
3. [报告问题](#报告问题)
4. [提交改进建议](#提交改进建议)
5. [贡献代码](#贡献代码)
6. [文档贡献](#文档贡献)
7. [社区互动](#社区互动)

## 行为准则

我们希望 UE5LuaModKit 社区能够成为一个友好、包容和富有成效的环境。请遵守以下准则：

- 尊重每一位贡献者，不论其经验水平如何
- 保持开放和友好的态度
- 以建设性的方式提供反馈
- 专注于项目的改进，避免无谓的争论
- 遵守项目的许可证条款

## 如何贡献

有多种方式可以为 UE5LuaModKit 做出贡献：

1. 报告 bug 和问题
2. 提出新功能或改进建议
3. 改进文档
4. 提交代码修复或新功能
5. 帮助其他用户解决问题
6. 分享您使用 UE5LuaModKit 的经验和案例

## 报告问题

如果您发现了 bug 或遇到了问题，请按照以下步骤报告：

1. 检查是否已有相同或类似的问题报告
2. 如果没有，创建一个新的 issue，包括：
   - 清晰简洁的标题
   - 问题的详细描述
   - 复现步骤
   - 预期行为和实际行为
   - 环境信息（操作系统、UE5 版本、UE5LuaModKit 版本等）
   - 如果可能，附上相关的代码片段、错误日志或截图

示例：

```markdown
标题：[BUG] 使用 GetPlayerLocation() 函数返回错误坐标

描述：
在尝试获取玩家位置时，GetPlayerLocation() 函数返回的坐标与实际游戏中的位置不符。

复现步骤：
1. 创建一个新的 Lua MOD
2. 在 MOD 中使用以下代码：
   ```lua
   local player = GetPlayer()
   local location = player:GetPlayerLocation()
   print("Player location:", location)
   ```
3. 运行游戏并加载 MOD
4. 观察控制台输出的位置信息

预期行为：
输出的位置信息应与游戏中玩家的实际位置相符。

实际行为：
输出的位置信息与玩家的实际位置有显著偏差，通常差距在 100 单位以上。

环境信息：
- 操作系统：Windows 10
- UE5 版本：5.1
- UE5LuaModKit 版本：1.2.3

附加信息：
[附上任何相关的错误日志或截图]
```

## 提交改进建议

如果您有改进 UE5LuaModKit 的想法，我们非常欢迎！请按照以下步骤提交您的建议：

1. 检查是否已有类似的建议
2. 如果没有，创建一个新的 issue，标记为"建议"或"功能请求"，包括：
   - 清晰的标题
   - 详细的功能描述
   - 使用场景和潜在好处
   - 可能的实现方式（如果有想法）

示例：

```markdown
标题：[建议] 添加自动化测试框架支持

描述：
建议为 UE5LuaModKit 添加内置的自动化测试框架，以便 MOD 开发者能够更容易地编写和运行单元测试。

使用场景和好处：
1. MOD 开发者可以为其 Lua 代码编写单元测试，提高代码质量
2. 自动化测试可以帮助开发者在修改代码后快速验证功能是否正常
3. 有助于发现和修复潜在的 bug
4. 可以作为文档的补充，展示 API 的正确使用方法

可能的实现方式：
1. 集成一个轻量级的 Lua 测试框架，如 luaunit
2. 提供一组辅助函数，用于模拟游戏环境和常用操作
3. 添加命令行工具，用于运行测试套件
4. 在文档中添加测试编写指南和最佳实践

示例代码：
```lua
-- 示例测试用例
function TestPlayerMovement()
    local player = MockPlayer.new()
    player:SetLocation(Vector(0, 0, 0))
    player:MoveForward(100)
    
    local newLocation = player:GetLocation()
    AssertEquals(newLocation.x, 100, "Player should move 100 units forward")
    AssertEquals(newLocation.y, 0, "Player should not move sideways")
    AssertEquals(newLocation.z, 0, "Player should not change elevation")
end
```

## 贡献代码

如果您想为 UE5LuaModKit 贡献代码，请遵循以下步骤：

1. Fork 项目仓库
2. 创建一个新的分支，使用描述性的名称（例如：`feature/add-new-api` 或 `bugfix/player-location`）
3. 在新分支上进行修改
4. 确保您的代码符合项目的编码规范
5. 编写或更新相关的测试（如果适用）
6. 提交变更，使用清晰简洁的提交信息
7. 推送您的分支到 GitHub
8. 创建一个 Pull Request（PR），包括：
   - 清晰的标题和描述
   - 相关 issue 的引用（如果有）
   - 变更的简要说明
   - 任何需要审核者特别注意的地方

代码风格指南：

- 遵循 Lua 的官方风格指南
- 使用 4 个空格进行缩进（不使用制表符）
- 变量和函数名使用 camelCase
- 常量使用全大写 SNAKE_CASE
- 模块名和类名使用 PascalCase
- 添加必要的注释，特别是对于复杂的逻辑
- 保持函数简短，遵循单一职责原则

示例代码：

```lua
-- 玩家管理模块
local PlayerManager = {}

-- 最大玩家数量
local MAX_PLAYERS = 64

-- 获取玩家位置
-- @param playerId 玩家ID
-- @return Vector 玩家位置
function PlayerManager.getPlayerLocation(playerId)
    assert(type(playerId) == "number", "Player ID must be a number")
    
    local player = GetPlayerById(playerId)
    if not player then
        error("Player not found: " .. playerId)
    end
    
    return player:GetLocation()
end

return PlayerManager
```

## 文档贡献

文档对于项目的成功至关重要。如果您想改进文档，请：

1. 遵循与代码贡献相同的基本流程
2. 确保文档清晰、准确且易于理解
3. 使用 Markdown 格式
4. 添加示例和说明来阐述复杂的概念
5. 检查拼写和语法错误

## 社区互动

积极参与社区讨论是贡献的重要方式：

1. 回答其他用户在 issue 或论坛中提出的问题
2. 分享您的 MOD 开发经验和技巧
3. 参与功能讨论，提供您的见解
4. 帮助测试新功能和报告反馈
5. 在社交媒体上分享项目，帮助扩大社区

## 致谢

我们真诚地感谢所有贡献者的付出。您的努力使 UE5LuaModKit 变得更好！

如果您有任何问题或需要帮助，请随时联系项目维护者或在社区中寻求帮助。

祝您在 UE5LuaModKit 社区中有愉快的体验！