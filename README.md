# temperature
还在考虑如何给BDS添加一个符合现实又高性能的温度系统?来试试temperature,本插件基于Levilamina-lse-nodejs编写

# 注意事项：
- 该插件免费版config.json文件是只读的，不支持编辑。
- 该插件免费版没有API导出,无法通过其他插件调用。
- 该插件购买渠道：
  渠道：
  - 官方QQ群：519653012
  - 作者QQ：3529832433

ps:本文档通过AI做更新内容撰写,存在一定的描述问题,作者在完成大学毕设设计,各位凑合看
### 版本更新历史

#### 版本 0.02 (更新大量可调用API,修复部分bug,调整购买渠道)

- **新增功能**: 新增API接口，支持获取玩家温度、增加/减少温度、设置温度等
- **修复问题**: 修复温度计算逻辑错误，提高稳定性
- **性能优化**: 优化温度计算算法，降低计算复杂度
- **用户体验**: 优化用户交互，增加温度影响状态显示
- **文档更新**: 更新API文档，提供详细使用说明

#### 版本 0.03 (异步优化)

- **核心优化**: 将方块温度计算改为异步分块处理，避免主线程阻塞
- **实现细节**:
  - 坐标预生成与分批次处理（每批50个方块）
  - 使用`await`关键字实现异步调用链
  - 递归`setTimeout`替代`setInterval`避免异步操作重叠
- **性能提升**: 大幅降低高负载场景下的卡顿现象，尤其在多温度影响状态和多玩家环境

#### 版本 1.0.1 (实体温度影响系统)

- **新增功能**: 添加实体温度影响系统，生物和玩家会影响周围温度
- **新增配置**: 可以自由开启loop的显示(开启后可能存在无法堆叠物品的情况)
- **实现特性**:
  - 支持多种生物类型（高温生物、低温生物、中性生物）
  - 可配置的距离衰减曲线（反平方衰减/线性衰减）
  - 实体检测范围可配置（默认10格）
  - 最大实体影响值限制（±30°C）
- **配置优化**: 新增实体温度影响相关配置项
- **性能优化**: 异步实体检测，避免主线程阻塞

# 配置文件说明 (config.json)

## 主要配置项：

```json
{
  // 温度变化平滑系数 (0.1为较平缓变化，值越大变化越快)
  "temperatureCurveFactor": 0.1,

  // 温度极限设置
  "TemperatureLimits": {
    "min": -15,
    "max": 45,
    "warningThreshold": 5 // 警告阈值（默认5度）
  },
  "DamageSettings": {
    "damageInterval": 2000, // 伤害间隔（毫秒）
    "freezeDamage": 4,
    "burnDamage": 4
  },
  "ProtectionSettings": {
    "deathProtectionDuration": 60000, // 死亡保护持续时间（毫秒）
    "initialProtectionDuration": 30000, // 初始进入保护持续时间（毫秒）
    "biomeTemperatureDelay": 5000 // 群系温度延迟生效时间（毫秒）
  },
  "enableInventoryLoop": false, // 是否启用循环遍历背包的功能
  "enableHandItemTemperatureDisplay:": true, // 是否在BOOS栏显示手持物品温度影响值
  "enableLoopOnlyArmor": true, // 是否仅在盔甲上显示温度描述（循环遍历模式下）

  // 温度效果配置
  "TemperatureEffects": {
    "coldEffects": [
      // 低温效果
      {
        "threshold": 10, // 触发阈值(温度≤10°C时生效)
        "effectId": 2, // 效果ID(2=移动加速)
        "amplifier": 1, // 效果等级
        "duration": 60 // 持续时间(秒)
      }
      //...其他低温效果
    ],
    "hotEffects": [
      // 高温效果
      {
        "threshold": 30, // 触发阈值(温度≥30°C时生效)
        "effectId": 17, // 效果ID(17=瞬间治疗)
        "amplifier": 0,
        "duration": 60
      }
      //...其他高温效果
    ]
  },

  // 群系温度修正值
  "BiomeTemperatureModifiers": {
    "desert": 12, // 沙漠+12°C
    "savanna": 8, // 热带草原+8°C
    "snowy_tundra": -15 // 雪原-15°C
    //...其他群系修正
  },

  // 盔甲温度修正
  "ArmorTemperatureModifiers": {
    "minecraft:leather_chestplate": 6, // 皮革胸甲+6°C
    "minecraft:diamond_helmet": -3 // 钻石头盔-3°C
    //...其他盔甲修正
  },

  // 食物温度效果
  "FoodTemperatureEffects": {
    "minecraft:cooked_beef": {
      "tempChange": 8, // 温度变化值
      "duration": 5000 // 持续时间(毫秒)
    },
    "minecraft:melon_slice": {
      "tempChange": -5, // 食用西瓜降低10°C
      "duration": 4000
    }
    //...其他食物效果
  },
  // 方块检测与处理配置
  "blockCheckRange": 5, // 方块检测范围(默认5格)
  "blockEffectCurve": "sqrt", // 方块效果曲线类型(sqrt/log)
  "blockProcessingBatchSize": 50, // 每批处理方块数量(默认50个)
  // 玩家状态温度修正
  "environmentTemperatureModifiers": {
    "inWater": -8, // 水中温度修正
    "inLava": 255, // 岩浆中温度修正
    "inWaterOrRain": -5, // 水中或雨中温度修正
    "inSnow": -10, // 雪中温度修正
    "isOnHotBlock": 15, // 热方块上温度修正
    "daylightDay": 3, // 白天光照温度修正
    "daylightNight": -2 // 夜晚光照温度修正
  },
  // 方块温度修正值
  "BlockTemperatureModifiers": {
    // 冷水/冷源方块
    "minecraft:water": -3,
    "minecraft:ice": -7,
    // 热水/热源方块
    "minecraft:lava": 20,
    "minecraft:fire": 15,
    // 特殊方块
    "minecraft:water_cauldron": -4,
    "minecraft:lava_cauldron": 10,
    "minecraft:respawn_anchor[charges=4]": 15
    //...其他方块修正
  },

  // 实体温度影响系统配置
  "EntityTemperatureModifiers": {
    // 高温生物
    "minecraft:blaze": {
      "baseEffect": 15,
      "range": 8,
      "curve": "inverse_square"
    },
    "minecraft:magma_cube": {
      "baseEffect": 12,
      "range": 6,
      "curve": "inverse_square"
    },
    "minecraft:strider": { "baseEffect": 8, "range": 5, "curve": "linear" },

    // 低温生物
    "minecraft:stray": {
      "baseEffect": -10,
      "range": 7,
      "curve": "inverse_square"
    },
    "minecraft:snow_golem": { "baseEffect": -8, "range": 5, "curve": "linear" },
    "minecraft:polar_bear": {
      "baseEffect": -6,
      "range": 6,
      "curve": "inverse_square"
    },

    // 中性生物（轻微影响）
    "minecraft:cow": { "baseEffect": 1, "range": 3, "curve": "linear" },
    "minecraft:pig": { "baseEffect": 1, "range": 3, "curve": "linear" },
    "minecraft:sheep": { "baseEffect": 1, "range": 3, "curve": "linear" },

    // 特殊生物
    "minecraft:ender_dragon": {
      "baseEffect": 20,
      "range": 15,
      "curve": "inverse_square"
    },
    "minecraft:wither": {
      "baseEffect": 18,
      "range": 12,
      "curve": "inverse_square"
    }
  },

  // 实体检测范围（格）
  "entityCheckRange": 10,

  // 实体影响衰减曲线类型
  "entityEffectCurve": "inverse_square"
}
```

## 温度系统插件 API 文档

### 1. 插件概述

温度系统插件提供了玩家温度管理功能，其他插件可以通过API接口获取和修改玩家温度。

插件名称 : temperature 版本 : 1.0.0 作者 : 伊希娅 简介 : 管理玩家温度及相关效果的系统插件
温度系统插件提供了玩家温度管理功能，其他插件可以通过API接口获取和修改玩家温度。

插件名称 : temperature 版本 : 1.0.0 作者 : 伊希娅 简介 : 管理玩家温度及相关效果的系统插件

### 2. API 接口说明

#### a.获取玩家当前温度

`getPlayerTemperature(player)`  
**参数**：

- player: 玩家对象  
  **返回值**：当前温度值（Number）

#### b.增加玩家温度

`addPlayerTemperature(player, amount)`  
**参数**：

- player: 玩家对象
- amount: 增加的温度值（Number）

#### c.减少玩家温度

`reducePlayerTemperature(player, amount)`  
**参数**：

- player: 玩家对象
- amount: 减少的温度值（Number）

#### d.设置玩家温度

`setPlayerTemperature(player, temperature)`  
 **参数**：

- player: 玩家对象
- temperature: 目标温度值（Number）

#### e.暂停/恢复玩家温度更新

`pauseTemperatureUpdate(player, pauseState)`
参数 :

- player: 玩家对象
- pauseState: true(暂停)/false(恢复)
  返回值 : 无

#### f.检查玩家是否受温度影响

` isAffectedByTemperature(player)`  
 参数：

- player: 玩家对象  
  返回值 : Boolean (true表示正在受温度影响)

#### g.获取基础温度值

` getBaseTemperature(player)`
参数：

- player: 玩家对象  
  返回值 : 计算环境因素后的基础温度值

#### h.维持合适温度范围

`maintainTemperature(player, [min=20], [max=25], [duration=0], [callback])`  
**参数**：

- player: 玩家对象
- min: 最低温度(默认20°C)
- max: 最高温度(默认25°C)
- duration: 持续时间(毫秒，0表示永久)
- callback: 持续时间结束回调函数

#### i.停止维持温度

`stopMaintaining(player)`  
**参数**：

- player: 玩家对象

#### h.维持自定义温度范围

`maintainTemperature(player, [min=20], [max=25], [duration=0], [callback])`  
**参数说明**：

- `min`：最低温度（支持负数，默认20）
- `max`：最高温度（默认25）
- 参数顺序不影响，系统会自动排序区间

### 插件示例:

示例 :

```javascript
const player = ...; // 获取玩家对象
const currentTemp = $Y.i.temperature.getPlayerTemperature(player);
console.log(`玩家当前温度: ${currentTemp}°C`);

// 增加玩家温度5°C
$Y.i.temperature.addPlayerTemperature(player, 5);

// 减少玩家温度5°C
$Y.i.temperature.reducePlayerTemperature(player, 5);

// 将玩家温度设置为25°C
$Y.i.temperature.setPlayerTemperature(player, 25);

// 暂停玩家温度更新
$Y.i.temperature.pauseTemperatureUpdate(player, true);

// 检查温度影响状态
if (!$Y.i.temperature.isAffectedByTemperature(player)) {
   // 设置自定义温度值
   $Y.i.temperature.setPlayerTemperature(player, 25);
}

// 获取环境基础温度
const baseTemp = $Y.i.temperature.getBaseTemperature(player);
console.log(`环境基础温度: ${baseTemp}°C`);

// 定时恢复温度系统
setTimeout(() => {
   $Y.i.temperature.pauseTemperatureUpdate(player, false);
   player.sendToast("温度系统已恢复");
}, 30000);

//维持玩家在22-28°C范围30秒
$Y.i.temperature.maintainTemperature(player, 22, 28, 30000, () => {
   player.sendToast('舒适温度维持结束！');
});

// 手动停止维持
$Y.i.temperature.stopMaintaining(player);

// 维持玩家在-5°C到10°C的低温环境60秒
$Y.i.temperature.maintainTemperature(player, 10, -5, 60000);

// 维持高温环境（50°C到60°C）
$Y.i.temperature.maintainTemperature(player, 60, 50);

// 维持精确温度25°C（设置相同值）
$Y.i.temperature.maintainTemperature(player, 25, 25);
```

## 实体温度影响系统详细说明

### 1. 系统概述

实体温度影响系统是温度系统的重要扩展功能，允许生物实体对玩家温度产生动态影响。系统通过检测玩家周围的生物实体，根据实体类型、距离和配置参数计算温度修正值。

### 2. 核心特性

#### a. 生物类型支持

- **高温生物**：岩浆怪、烈焰人、炽足兽等产生热量的生物
- **低温生物**：流浪者、雪傀儡、北极熊等产生冷气的生物
- **中性生物**：牛、猪、羊等产生轻微体温影响的生物
- **特殊生物**：末影龙、凋灵等拥有强大温度影响的BOSS生物

#### b. 距离衰减系统

- **反平方衰减**：效果随距离平方反比衰减，适合自然物理效果
- **线性衰减**：效果随距离线性衰减，适合简单计算
- **自定义范围**：每种生物可设置独立的影响范围

#### c. 多实体叠加

- 支持多个同类型实体效果叠加
- 支持不同类型实体效果叠加（高温+低温=抵消）
- 总影响值限制在±30°C范围内

### 3. 配置参数详解

#### EntityTemperatureModifiers 配置

```json
"minecraft:blaze": {
  "baseEffect": 15,      // 基础影响值（°C）
  "range": 8,           // 影响范围（格）
  "curve": "inverse_square" // 衰减曲线类型
}
```

**参数说明：**

- `baseEffect`：实体在零距离时的最大温度影响值
- `range`：实体能够影响玩家的最大距离
- `curve`：距离衰减曲线类型（`inverse_square`或`linear`）

#### 全局配置

- `entityCheckRange`：实体检测范围，默认10格
- `entityEffectCurve`：默认衰减曲线类型

### 4. 计算算法

#### 距离衰减公式

**反平方衰减（inverse_square）：**

```
效果值 = baseEffect × (1 - (距离/range)^2)
```

**线性衰减（linear）：**

```
效果值 = baseEffect × (1 - 距离/range)
```

#### 多实体叠加

```
总影响 = Σ(每个实体的效果值)
限制范围：-30°C ≤ 总影响 ≤ +30°C
```

### 5. 性能优化

#### 检测优化

- 每5秒检测一次周围实体（可配置）
- 使用球形检测范围减少计算量
- 批量处理实体检测结果

#### 计算优化

- 距离计算使用曼哈顿距离近似
- 效果值缓存机制
- 无效实体自动跳过

### 6. 调试与日志

系统提供详细的调试信息：

- 实体检测数量统计
- 每个实体的计算效果值
- 总影响值计算过程
- 错误处理和异常捕获

### 7. 使用示例

#### 配置示例

```json
{
  "EntityTemperatureModifiers": {
    "minecraft:blaze": {
      "baseEffect": 15,
      "range": 8,
      "curve": "inverse_square"
    },
    "minecraft:snow_golem": { "baseEffect": -8, "range": 5, "curve": "linear" }
  },
  "entityCheckRange": 10,
  "entityEffectCurve": "inverse_square"
}
```

#### 场景示例

1. **烈焰人农场**：玩家靠近烈焰人时温度逐渐升高
2. **雪傀儡守卫**：雪傀儡为玩家提供降温效果
3. **生物混合区域**：高温和低温生物效果相互抵消

### 8. 兼容性说明

- 与现有方块温度系统完全兼容
- 支持所有原版生物实体
- 可扩展支持模组生物
- 不影响其他温度修正系统

### 9. 故障排除

#### 常见问题

1. **实体不产生效果**：检查实体ID配置是否正确
2. **效果值异常**：验证距离计算和衰减曲线
3. **性能问题**：调整检测频率和范围

#### 调试命令

系统提供调试命令查看实体温度影响详情：

```
/temperature debug entities
```

通过实体温度影响系统，温度系统实现了更加动态和真实的温度环境模拟，为玩家提供更丰富的游戏体验。
