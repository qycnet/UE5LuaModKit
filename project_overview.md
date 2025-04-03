# 项目概述

## 项目结构
该项目是基于虚幻引擎5（UE5）的游戏项目，主要包含以下核心组件：

### 1. CppSDK
- 包含游戏的C++ SDK实现
- 主要包括各种游戏类的头文件和实现文件
- 包含基础游戏系统、AI系统、动画系统等核心功能

### 2. Dumpspace
包含关键游戏数据信息：
- ClassesInfo.json - 类信息
- EnumsInfo.json - 枚举信息
- FunctionsInfo.json - 函数信息
- OffsetsInfo.json - 偏移信息
- StructsInfo.json - 结构体信息

### 3. Mappings
包含游戏的映射文件：
- 5.1.1-0+++UE5+Release-5.1-Pal.usmap - UE5映射文件
- IDA映射文件（位于IDAMappings目录）

## 核心系统

### 1. NPC系统
- 基础NPC类 (ABP_NPC_Base)
- NPC变体类 (ABP_NPC_Child)
- 特殊NPC类型 (如DarkTrader)

### 2. 玩家系统
- 基础玩家类 (ABP_Player)
- 玩家外观系统
  - 头部系统 (ABP_Player_Head)
  - 发型系统 (ABP_Player_Hair)

### 3. AI系统
- AI控制器接口
- AI行为树
- 寻路系统

### 4. 动画系统
- 基础动画蓝图
- 动画共享系统
- 动画通知系统

### 5. 音频系统
- AkAudio集成
- 音频分析器
- 音频扩展

## MOD开发注意事项

1. 该项目使用UE5.1.1版本
2. 包含完整的C++ SDK供参考
3. 提供详细的数据结构映射
4. 支持通过Lua进行MOD开发

更多详细信息请参考其他专项文档：
- [数据结构文档](data_structures.md)
- [函数接口文档](function_interfaces.md)
- [Lua MOD开发指南](lua_modding_guide.md)
- [类型系统文档](type_system.md)