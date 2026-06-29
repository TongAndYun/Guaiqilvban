# 怪奇旅伴 (Bizarre Brigade) - Code Wiki

> **当前版本**: v1.2 | **作者**: 一个大伟丘&ZZZ

## 目录

1. [项目概述](#1-项目概述)
2. [项目架构](#2-项目架构)
3. [核心模块说明](#3-核心模块说明)
4. [关键数据结构](#4-关键数据结构)
5. [关键函数说明](#5-关键函数说明)
6. [游戏流程详解](#6-游戏流程详解)
7. [依赖关系](#7-依赖关系)
8. [项目运行方式](#8-项目运行方式)
9. [移动端适配](#9-移动端适配)

---

## 1. 项目概述

### 1.1 项目简介

**怪奇旅伴 (Bizarre Brigade)** 是一款梦境生存肉鸽 (Roguelite) 游戏，采用纯 HTML5 Canvas + JavaScript 单文件架构开发。玩家在梦境中控制角色与怪物战斗，收集道具、招募旅伴，挑战越来越强的敌人波次。

### 1.2 技术栈

| 技术 | 说明 |
|------|------|
| HTML5 Canvas | 游戏画面渲染 |
| Vanilla JavaScript (ES6+) | 游戏逻辑 |
| Web Audio API | 音效与背景音乐 |
| LocalStorage | 进度存档（解锁内容、音效设置） |
| CSS3 | UI 样式与动画 |

### 1.3 游戏特性

- **6 种难度**：简单、普通、困难、噩梦、地狱、无尽
- **4 个可玩角色**：忍者喵、法师兔、铁甲龟、金币鼠
- **3 种旅伴类型**：剑士（近战）、弓手（远程）、冲锋车（冲撞）
- **39 种遗物道具**：提供各种增益与负面效果
- **40 个成就**：包含角色专属成就和通用成就
- **商店系统**：每波结束可购买道具和旅伴
- **旅伴升星系统**：相同星级旅伴可合成升级（最高 3 星）
- **旅伴管理**：支持手动部署/请离，场上+库存统一合成
- **自动攻击**：玩家自动锁定最近敌人攻击
- **进度解锁**：通关解锁更高难度和新角色
- **分享功能**：可分享游戏给朋友
- **开发者模式**：Ctrl+Shift+D 开启，支持作弊和状态查看
- **移动端适配**：支持手机/平板触屏操作

---

## 2. 项目架构

### 2.1 目录结构

```
app_1790jfmbrps/
├── src/
│   └── index.html          # 单文件：HTML + CSS + JS 全部内联
├── .spark/
│   └── meta.json           # 项目元数据（平台配置）
├── package.json            # 项目依赖与脚本
├── package-lock.json       # 依赖锁定文件
├── .npmrc                  # npm 配置
├── .gitignore              # Git 忽略配置
└── README.md               # 项目说明
```

### 2.2 架构模式

项目采用**单文件单体架构**，所有代码（HTML 结构、CSS 样式、JavaScript 逻辑）都内联在一个 `index.html` 文件中。这种架构的特点：

- **零构建复杂度**：build 阶段仅做文件拷贝
- **易于部署**：产物即源码，可直接在浏览器运行
- **快速开发**：HMR 热更新由 dev server 提供
- **适合小型游戏项目**：代码量可控，单文件便于迭代

### 2.3 代码分层

虽然是单文件，但代码在逻辑上可分为以下层次：

```
┌─────────────────────────────────────────┐
│              UI 层 (CSS/HTML)           │
│  开始菜单、HUD、商店、暂停、设置、结算  │
├─────────────────────────────────────────┤
│              音效层 (Audio)             │
│     Web Audio API 合成音效与音乐        │
├─────────────────────────────────────────┤
│              游戏逻辑层 (Game Logic)    │
│  玩家、敌人、旅伴、道具、碰撞、波次     │
├─────────────────────────────────────────┤
│              渲染层 (Canvas Render)     │
│     粒子、投射物、实体绘制              │
├─────────────────────────────────────────┤
│              核心循环 (Game Loop)       │
│     requestAnimationFrame 驱动          │
└─────────────────────────────────────────┘
```

---

## 3. 核心模块说明

### 3.1 音效系统 (Audio System)

**文件位置**：[index.html](file:///d:/游戏开发/怪奇旅伴/app_1790jfmbrps/src/index.html#L1154-L1301)

基于 Web Audio API 的程序化音效系统，所有音效和音乐都通过振荡器实时合成，无需音频文件。

**核心组件**：
- `audioCtx` - AudioContext 上下文
- `musicGain` - 音乐音量控制节点
- `sfxGain` - 音效音量控制节点
- `musicOsc` - 当前音乐振荡器
- `musicNotes` / `musicDurations` - 背景音乐旋律序列

**音效类型**（`playSFX` 函数）：
| 类型 | 说明 |
|------|------|
| `shoot` | 射击音效（双音下降） |
| `hit` | 命中音效（锯齿波） |
| `kill` | 击杀音效（上升三音） |
| `gold` | 金币音效（双音上升） |
| `click` | 点击音效 |
| `wave` | 波次开始音效（三音上升） |
| `unlock` | 解锁音效（五音阶上升） |

### 3.2 设置面板 (Settings Panel)

**文件位置**：[index.html](file:///d:/游戏开发/怪奇旅伴/app_1790jfmbrps/src/index.html#L1303-L1349)

三个标签页：
1. **🔊 音效** - 音乐/音效音量调节滑块
2. **👤 关于** - 游戏信息与作者
3. **💬 作者的话** - 点赞按钮

**彩蛋**：在关于页连续点击头像 10 次可解锁全部难度和角色（后门功能）。

### 3.3 暂停系统 (Pause System)

**文件位置**：[index.html](file:///d:/游戏开发/怪奇旅伴/app_1790jfmbrps/src/index.html#L1351-L1393)

- 按 `ESC` 键暂停/继续游戏
- 暂停菜单提供：继续游戏、重新开始、返回主界面
- 通过 `gameState` 状态机管理暂停状态

### 3.4 开发者模式 (Developer Mode)

**文件位置**：[index.html](file:///d:/游戏开发/怪奇旅伴/app_1790jfmbrps/src/index.html#L2175-L2273)

按 `Ctrl+Shift+D` 快捷键打开开发者面板（游戏中可用）。

**功能**：
- 查看当前状态（生命、攻击、金币、梦境核心）
- 梦境核心 +5
- 金币 +1000
- 恢复满血
- 攻击 +10
- 护甲 +5
- 移速 +0.5
- 生命 +50
- 满额道具（所有道具各1个）
- 强化旅伴（旅伴伤害 x2）
- 解锁全部（难度+角色全解锁）

**特性**：
- 打开时暂停游戏（gameState = 'paused'）
- 敌人生成继续进行（spawnEnemy、spawnBoss 支持 paused 状态）
- 关闭时恢复游戏状态

**作弊函数**：
| 函数 | 说明 |
|------|------|
| `cheatAddCore()` | 增加梦境核心 |
| `cheatAddGold()` | 增加金币 |
| `cheatFullHp()` | 恢复满血 |
| `cheatAddAtk()` | 增加攻击 |
| `cheatAddArmor()` | 增加护甲 |
| `cheatAddSpeed()` | 增加移速 |
| `cheatAddHp()` | 增加生命 |
| `cheatFullRelic()` | 给予所有道具 |
| `cheatBuffCompanion()` | 旅伴伤害翻倍 |
| `cheatUnlockAll()` | 解锁全部内容 |

### 3.4 解锁系统 (Unlock System)

**文件位置**：[index.html](file:///d:/游戏开发/怪奇旅伴/app_1790jfmbrps/src/index.html#L1397-L1435)

- 难度解锁：通关当前难度解锁下一难度
- 角色解锁：通关普通难度解锁"金币鼠"
- 存档位置：`localStorage['bizarreBrigade_unlocks']`

### 3.5 信息面板 (Info Panel)

**文件位置**：[index.html](file:///d:/游戏开发/怪奇旅伴/app_1790jfmbrps/src/index.html#L1699-L1819)

按 `K` 键打开，三个标签页：
1. **🦸 角色** - 基础信息、战斗属性、特殊属性、成长率
2. **💎 道具** - 已购买的遗物列表
3. **👥 旅伴** - 已部署和待部署的旅伴（支持部署/请离/合成）

### 3.6 商店系统 (Shop System)

**文件位置**：[index.html](file:///d:/游戏开发/怪奇旅伴/app_1790jfmbrps/src/index.html#L2107-L2198)

每波结束后自动开启商店：
- 3 个旅伴商品（随机类型）
- 3 个遗物商品（从遗物池随机抽取）
- 刷新功能（费用翻倍递增）
- 价格随波次增长（`getWavePriceMult`）
- 金币鼠角色享受 8 折优惠

### 3.7 旅伴系统 (Companion System)

**文件位置**：[index.html](file:///d:/游戏开发/怪奇旅伴/app_1790jfmbrps/src/index.html#L2200-L2270)

- 最多同时部署 **5 个旅伴**（`MAX_COMPANIONS = 5`）
- 旅伴围绕玩家轨道运行
- **三星合成机制**：场上 + 库存中相同类型星级的旅伴总数 >= 3 时自动合成
  - 3 个 1 星 → 1 个 2 星
  - 3 个 2 星 → 1 个 3 星
- **手动部署/请离**：点击旅伴图标进行操作
- 升星提升伤害、攻击范围等属性

**旅伴类型**：
| 类型 | 攻击方式 | 特点 |
|------|---------|------|
| 剑士 ⚔️ | 近战范围伤害 | 高伤害、近距离 |
| 弓手 🏹 | 远程投射物 | 安全距离、中等伤害 |
| 冲锋车 🚗 | 冲撞直线伤害 | 爆发伤害、冷却长 |

**旅伴管理**：
- **主界面旅伴栏**：点击已部署旅伴请离，点击库存旅伴部署（支持合成）
- **角色详情面板（K键）**：显示"部署"和"请离"按钮，支持合成

**相关函数**：
| 函数 | 说明 |
|------|------|
| `addCompanion(type, star)` | 购买旅伴，自动部署或入库存 |
| `deployCompanionFromInventory(idx)` | 从库存部署旅伴，支持合成 |
| `recallCompanion(idx)` | 请离旅伴回库存 |
| `spawnCompanion(type, star)` | 生成旅伴到战场 |
| `updateCompanionOrbits()` | 更新轨道分布 |
| `updateCompanionUI()` | 更新旅伴栏UI |
| `renderCompsInfo()` | 渲染角色面板的旅伴列表 |
| `deployCompanionFromInfo(idx)` | 从角色面板部署旅伴 |
| `recallCompanionFromInfo(idx)` | 从角色面板请离旅伴 |

### 3.9 渲染系统 (Render System)

**文件位置**：[index.html](file:///d:/游戏开发/怪奇旅伴/app_1790jfmbrps/src/index.html#L2604-L2751)

`draw()` 函数负责每帧渲染，绘制顺序（从底层到顶层）：
1. 背景网格 + 无尽模式滤镜
2. 掉落物（金币、血包）
3. 敌人（含血条、BOSS 标识）
4. 旅伴（含光环效果）
5. 玩家（含攻击范围指示、无敌闪烁）
6. 投射物（含拖尾效果）
7. 粒子效果（伤害数字、爆炸粒子、范围光环）

---

## 4. 关键数据结构

### 4.1 游戏状态 (gameState)

```javascript
// 状态机取值
'menu'       // 主菜单
'playing'    // 游戏进行中
'paused'     // 暂停
'shop'       // 商店界面
'info'       // 信息面板（K键）
'gameover'   // 游戏结束
```

### 4.2 难度配置 (difficulties)

**文件位置**：[index.html](file:///d:/游戏开发/怪奇旅伴/app_1790jfmbrps/src/index.html#L1397-L1404)

| 索引 | 名称 | 波数 | 时间 | HP倍率 | 伤害倍率 | 金币倍率 | 得分倍率 |
|------|------|------|------|--------|----------|----------|----------|
| 0 | 简单 🌱 | 5 | 60s | 0.8 | 0.8 | 1.2 | 1 |
| 1 | 普通 🌿 | 7 | 55s | 1.0 | 1.0 | 1.0 | 1.5 |
| 2 | 困难 🌳 | 10 | 50s | 1.3 | 1.2 | 0.9 | 2 |
| 3 | 噩梦 🔥 | 15 | 45s | 1.8 | 1.5 | 0.8 | 3 |
| 4 | 地狱 💀 | 20 | 40s | 2.5 | 2.0 | 0.7 | 5 |
| 5 | 无尽 🌌 | ∞ | 35s | 3.0 | 2.5 | 0.6 | 8 |

### 4.3 角色配置 (characters)

**文件位置**：[index.html](file:///d:/游戏开发/怪奇旅伴/app_1790jfmbrps/src/index.html#L1437-L1442)

| 索引 | 角色 | 生命 | 攻击 | 移速 | 护甲 | 金币加成 | 旅伴伤害 |
|------|------|------|------|------|------|----------|----------|
| 0 | 忍者喵 🥷 | 100 | 1.1x | 3.0 | 0 | 1.0x | 1.0x |
| 1 | 法师兔 🧙 | 80 | 1.0x | 2.8 | 0 | 1.0x | 1.2x |
| 2 | 铁甲龟 🛡️ | 150 | 0.9x | 2.0 | 5 | 1.0x | 1.0x |
| 3 | 金币鼠 💰 | 85 | 0.95x | 3.2 | 0 | 1.3x | 1.0x |

### 4.4 玩家对象 (player)

**文件位置**：[index.html](file:///d:/游戏开发/怪奇旅伴/app_1790jfmbrps/src/index.html#L1444-L1460)

```javascript
player = {
    x, y,                  // 位置坐标
    r,                     // 碰撞半径
    hp, maxHp,             // 当前/最大生命值
    speed,                 // 移动速度
    atk,                   // 攻击力
    atkRange,              // 攻击范围
    atkSpeed,              // 攻击速度（次/秒）
    atkTimer,              // 攻击冷却计时器
    armor,                 // 护甲（减伤）
    goldBonus,             // 金币获取倍率
    companionDmg,          // 旅伴伤害倍率
    invincible,            // 无敌剩余时间
    thorns,                // 荆棘反伤
    lifesteal,             // 吸血比例
    invincibleDuration     // 受伤后无敌时长
}
```

### 4.5 敌人类型 (enemies)

| 类型 | 占比 | 颜色 | 特点 |
|------|------|------|------|
| normal | 60% | #55efc4 绿 | 普通敌人，均衡属性 |
| fast | 25% | #fd79a8 粉 | 速度 1.8x，HP 0.6x，伤害 0.8x |
| tank | 15% | #a29bfe 紫 | HP 2.5x，速度 0.6x，伤害 1.5x |
| boss | 每5波 | #e74c3c 红 | 大体型，高血量高伤害 |

### 4.6 遗物池 (relicPool)

**文件位置**：[index.html](file:///d:/游戏开发/怪奇旅伴/app_1790jfmbrps/src/index.html#L1683-L2039)

共 39 种遗物，每件都有正面效果和负面效果（双刃剑设计）：

| ID | 名称 | 正面效果 | 负面效果 | 价格 |
|----|------|---------|----------|------|
| atk1 | 锋利之刃 🗡️ | 攻击+20% | 移速-5% | 20 |
| hp1 | 生命之心 ❤️ | 最大生命+30 | 护甲-3 | 25 |
| spd1 | 疾风之靴 👟 | 移速+20% | 最大生命-10% | 20 |
| arm1 | 铁甲护符 🛡️ | 护甲+3 | 攻击-10% | 25 |
| gold1 | 金币磁铁 🧲 | 金币+25% | 攻击-5% | 30 |
| range1 | 远视之镜 🔭 | 攻击范围+30% | 攻速-10% | 25 |
| speed1 | 狂暴药剂 ⚡ | 攻速+30% | 最大生命-15% | 30 |
| heal1 | 治愈之心 💖 | 恢复50生命 | 护甲-2 | 15 |
| comp1 | 旅伴强化 ⭐ | 旅伴伤害+25% | 玩家攻击-10% | 35 |
| time1 | 时间沙漏 ⏳ | 下波时间+10秒 | 移速-5% | 25 |
| score1 | 幸运四叶草 🍀 | 得分+50% | 金币-10% | 40 |
| shadow1 | 暗影斗篷 🌑 | 无敌时间+0.3s | 最大生命-10% | 30 |
| thorn1 | 荆棘之甲 🌵 | 反弹5点伤害 | 护甲-3 | 28 |
| vamp1 | 吸血之牙 🦇 | 击杀回血15% | 最大生命-20% | 35 |
| berserk1 | 狂战士之斧 🪓 | 攻击+40% | 护甲-5 | 35 |
| sage1 | 贤者之石 💎 | 生命+20 & 护甲+2 | 攻速-15% | 30 |
| bolt1 | 闪电之靴 👢 | 移速+30% | 护甲-3 | 25 |
| guard1 | 守护之盾 🔰 | 护甲+8 | 移速-20% | 35 |
| greed1 | 贪婪之戒 💍 | 金币+50% | 最大生命-15% | 40 |
| ultra_atk | 战神之剑 ⚔️ | 攻击+60% | 护甲-5 | 50 |
| ultra_spd | 闪电之翼 🦅 | 攻速+60% | 最大生命-20% | 45 |
| ultra_hp | 巨人心脏 💪 | 生命+80 | 移速-15% | 45 |
| ultra_armor | 龙鳞铠甲 🐉 | 护甲+12 | 金币-20% | 50 |
| ultra_range | 鹰眼之镜 🦅 | 范围+50% | 移速-10% | 40 |
| crit1 | 暴击利刃 🎯 | 攻击+35% | 护甲-3 | 40 |
| swift1 | 疾风药剂 💨 | 移速+40% | 范围-20% | 35 |
| fury1 | 狂暴之心 ❤️‍🔥 | 攻速+40% & 移速+15% | 护甲-4 | 50 |
| stone1 | 磐石护符 🪨 | 护甲+6 & 生命+30 | 金币-15% | 40 |
| rich1 | 黄金罗盘 🧭 | 金币+80% | 得分-20% | 55 |
| glory1 | 荣耀之证 🏅 | 得分+80% | 金币-25% | 55 |
| invincible1 | 无敌披风 🧥 | 无敌+0.8秒 | 生命-15% | 45 |
| thorns2 | 刺猬战甲 🦔 | 荆棘+8 | 移速-15% | 40 |
| vamp2 | 血族之牙 🧛 | 吸血+20% | 护甲-5 | 50 |
| combo1 | 连击手套 🥊 | 攻速+50% & 攻击+20% | 生命-25% | 55 |
| time2 | 永恒沙漏 ⏰ | 下波时间+20秒 | 金币-10% | 40 |
| comp2 | 旅伴圣典 📖 | 旅伴伤害+50% | 玩家攻击-15% | 55 |
| survivor | 幸存者徽章 🎖️ | 生命+50 & 护甲+4 | 金币-20% | 50 |
| assassin | 刺客匕首 🗡️ | 攻击+30% & 攻速+25% | 生命-20 | 45 |

### 4.7 粒子系统 (particles)

粒子类型：
- `circle` - 圆形粒子（爆炸效果）
- `text` - 文字粒子（伤害数字、治疗数字）
- `ring` - 光环粒子（近战范围攻击指示）

---

## 5. 关键函数说明

### 5.1 核心循环

**`gameLoop(timestamp)`**  
**文件位置**：[index.html](file:///d:/游戏开发/怪奇旅伴/app_1790jfmbrps/src/index.html#L2753-L2765)

游戏主循环，由 `requestAnimationFrame` 驱动。每帧执行：
1. 计算时间增量 `dt`（限制最大值 0.1 秒防止跳帧）
2. 调用 `update(dt)` 更新游戏逻辑
3. 播放时更新音乐
4. 调用 `draw()` 渲染画面

### 5.2 更新函数

**`update(dt)`**  
**文件位置**：[index.html](file:///d:/游戏开发/怪奇旅伴/app_1790jfmbrps/src/index.html#L2306-L2476)

每帧游戏逻辑更新，按顺序处理：
1. 波次倒计时
2. 玩家移动输入
3. 玩家自动攻击
4. 旅伴 AI 与攻击
5. 敌人移动与碰撞伤害
6. 投射物更新与命中检测
7. 粒子更新
8. 掉落物磁吸与拾取
9. 波次清场检测（触发商店）

### 5.3 敌人生成

**`spawnWave()`**  
**文件位置**：[index.html](file:///d:/游戏开发/怪奇旅伴/app_1790jfmbrps/src/index.html#L1968-L1998)

生成一波敌人：
- 波次公告动画
- 重置波次时间
- 敌人数量 = 5 + wave * 2
- 每 5 波为 BOSS 波（敌人减半，加一个 BOSS）
- 敌人分批生成（间隔 300ms）

**`spawnEnemy(isBoss)`**  
**文件位置**：[index.html](file:///d:/游戏开发/怪奇旅伴/app_1790jfmbrps/src/index.html#L2000-L2026)

从屏幕四边随机位置生成单个敌人，随机类型（普通/快速/坦克）。

**`spawnBoss()`**  
**文件位置**：[index.html](file:///d:/游戏开发/怪奇旅伴/app_1790jfmbrps/src/index.html#L2028-L2046)

生成 BOSS 敌人，从屏幕顶部中央出现。

### 5.4 战斗相关

**`damageEnemy(e, dmg)`**  
**文件位置**：[index.html](file:///d:/游戏开发/怪奇旅伴/app_1790jfmbrps/src/index.html#L2478-L2496)

对敌人造成伤害，显示伤害数字，血量归零则击杀。

**`killEnemy(e)`**  
**文件位置**：[index.html](file:///d:/游戏开发/怪奇旅伴/app_1790jfmbrps/src/index.html#L2498-L2549)

击杀敌人处理：
- 增加击杀数、得分
- 吸血效果
- 掉落金币（100%）
- 掉落血包（10% 概率）
- 死亡粒子效果

**`shootProjectile(x, y, target, dmg, color)`**  
**文件位置**：[index.html](file:///d:/游戏开发/怪奇旅伴/app_1790jfmbrps/src/index.html#L2551-L2563)

发射一枚追踪投射物。

**`findNearestEnemy(x, y, range)`**  
**文件位置**：[index.html](file:///d:/游戏开发/怪奇旅伴/app_1790jfmbrps/src/index.html#L2565-L2576)

查找范围内最近的敌人（自动索敌）。

### 5.5 商店相关

**`generateShopItems()`**  
**文件位置**：[index.html](file:///d:/游戏开发/怪奇旅伴/app_1790jfmbrps/src/index.html#L2121-L2147)

生成商店商品：3 个旅伴 + 3 个随机遗物。

**`buyItem(idx)`**  
**文件位置**：[index.html](file:///d:/游戏开发/怪奇旅伴/app_1790jfmbrps/src/index.html#L2166-L2189)

购买商品，扣除金币，应用效果。

**`refreshShop()`**  
**文件位置**：[index.html](file:///d:/游戏开发/怪奇旅伴/app_1790jfmbrps/src/index.html#L2191-L2198)

刷新商店，费用翻倍递增。

### 5.6 旅伴相关

**`addCompanion(type, star)`**  
**文件位置**：[index.html](file:///d:/游戏开发/怪奇旅伴/app_1790jfmbrps/src/index.html#L2200-L2221)

购买旅伴，自动部署或进入库存，自动处理合成逻辑。

**`deployCompanionFromInventory(idx)`**  
**文件位置**：[index.html](file:///d:/游戏开发/怪奇旅伴/app_1790jfmbrps/src/index.html#L3840-L3879)

从库存部署旅伴到战场。检查场上+库存中相同类型星级的总数 >= 3 时自动合成。

**`recallCompanion(idx)`**  
**文件位置**：[index.html](file:///d:/游戏开发/怪奇旅伴/app_1790jfmbrps/src/index.html#L3911-L3917)

将已部署的旅伴请离回库存。

**`spawnCompanion(type, star)`**  
**文件位置**：[index.html](file:///d:/游戏开发/怪奇旅伴/app_1790jfmbrps/src/index.html#L3852-L3878)

实例化一个旅伴并加入战场，设置伤害、范围、轨道参数。

**`updateCompanionOrbits()`**  
**文件位置**：[index.html](file:///d:/游戏开发/怪奇旅伴/app_1790jfmbrps/src/index.html#L3870-L3875)

重新计算所有旅伴的轨道参数（均匀分布）。

**`updateCompanionUI()`**  
**文件位置**：[index.html](file:///d:/游戏开发/怪奇旅伴/app_1790jfmbrps/src/index.html#L3919-L3938)

更新主界面旅伴栏UI，已部署旅伴可点击请离，库存旅伴可点击部署。

**`renderCompsInfo()`**  
**文件位置**：[index.html](file:///d:/游戏开发/怪奇旅伴/app_1790jfmbrps/src/index.html#L3268-L3295)

渲染角色详情面板（K键）的旅伴列表，添加部署/请离按钮。

### 5.7 游戏状态管理

**`startGame()`**  
**文件位置**：[index.html](file:///d:/游戏开发/怪奇旅伴/app_1790jfmbrps/src/index.html#L1904-L1954)

开始新游戏，初始化所有状态。

**`gameOver()`**  
**文件位置**：[index.html](file:///d:/游戏开发/怪奇旅伴/app_1790jfmbrps/src/index.html#L2582-L2602)

玩家死亡，显示结算界面。

**`victory()`**  
**文件位置**：[index.html](file:///d:/游戏开发/怪奇旅伴/app_1790jfmbrps/src/index.html#L2071-L2105)

通关所有波次，显示胜利界面，处理解锁。

---

## 6. 游戏流程详解

### 6.1 整体流程图

```
┌──────────┐
│  主菜单  │ 选择难度 + 选择角色
└────┬─────┘
     │ 点击开始
     ▼
┌──────────┐
│ 波次开始 │ 公告动画 + 生成敌人
└────┬─────┘
     │
     ▼
┌──────────┐
│  战斗中  │ WASD移动 + 自动攻击
└────┬─────┘
     ├─ 时间耗尽 ──► 清场进入商店
     └─ 击杀全部 ──► 清场进入商店
     │
     ▼
┌──────────┐
│  商店    │ 购买遗物/旅伴 + 刷新
└────┬─────┘
     │ 下一波
     ▼
     重复波次...
     │
     ├─ 通关所有波次 ──► 胜利结算
     └─ 玩家死亡 ──────► 失败结算
```

### 6.2 波次循环

每波的时间线：
1. **波次开始**（0s）：公告动画播放，敌人开始生成
2. **战斗阶段**（持续到时间结束或敌人全灭）：玩家移动、自动攻击、旅伴作战
3. **清场奖励**：剩余时间转换为分数奖励
4. **商店阶段**：购买道具和旅伴，准备下一波

### 6.3 碰撞与伤害计算

**玩家受到伤害**：
```
实际伤害 = max(1, 敌人伤害 - 玩家护甲)
受伤后获得无敌时间（默认 0.5 秒，可通过遗物延长）
```

**敌人受到伤害**：
```
直接扣除血量，同时显示伤害数字
击杀时掉落金币，10% 概率掉落血包
```

**荆棘反伤**：
```
玩家被碰撞时，对敌人造成 thorns 点伤害
```

**吸血**：
```
击杀敌人时，恢复敌人最大HP * lifesteal 的生命值
```

---

## 7. 依赖关系

### 7.1 运行时依赖

**无外部运行时依赖** - 游戏完全使用浏览器原生 API 实现：
- HTML5 Canvas 2D Context
- Web Audio API
- localStorage
- CSS3 动画
- requestAnimationFrame

### 7.2 开发时依赖

| 包名 | 版本 | 说明 |
|------|------|------|
| `@lark-apaas/coding-html-devserver` | ^0.1.5 | 开发服务器与构建工具（内部封装 Vite） |

### 7.3 工具链说明

开发工具链由 `@lark-apaas/coding-html-devserver` 提供：
- **dev 模式**：基于 Vite 的开发服务器，支持 HMR 热更新
- **build 模式**：纯文件拷贝（`src/` → `dist/output/`），不打包不加 hash

### 7.4 平台集成

项目配置了飞书 aPaaS 平台集成（`.spark/meta.json`）：
- 应用 ID：`app_1790jfmbrps`
- 在线地址：`https://4kfsajp4yc8bz.aiforce.cloud/app/app_1790jfmbrps`

---

## 8. 项目运行方式

### 8.1 环境要求

- Node.js（建议 16+）
- npm 或 yarn

### 8.2 安装依赖

```bash
cd app_1790jfmbrps
npm install
```

### 8.3 开发模式

```bash
npm run dev
```

- 启动开发服务器，默认端口 `8001`
- 支持 HMR 热更新，修改代码自动刷新
- 访问地址：`http://localhost:8001`

**环境变量**：
- `CLIENT_DEV_PORT` - 自定义 dev server 端口（默认 8001）

**Dev Server 端点**：
- `/` → 游戏首页
- `/dev/health` → 健康检查接口

### 8.4 构建生产版本

```bash
npm run build
```

- 产物输出到 `dist/output/` 目录
- 构建仅做文件拷贝，产物与源码完全一致
- 可直接部署到任何静态文件服务器

### 8.5 操作说明

**游戏内操作**：
| 按键 | 功能 |
|------|------|
| W / ↑ | 向上移动 |
| S / ↓ | 向下移动 |
| A / ← | 向左移动 |
| D / → | 向右移动 |
| K | 打开/关闭信息面板 |
| ESC | 暂停/继续游戏 |

**鼠标操作**：
- 点击菜单按钮、商店商品
- 悬浮查看锁定物品的解锁条件

---

## 9. 移动端适配

### 9.1 视口配置

**文件位置**：[index.html](file:///d:/游戏开发/怪奇旅伴/app_1790jfmbrps/src/index.html#L1-L8)

```html
<meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
<meta name="apple-mobile-web-app-capable" content="yes">
<meta name="mobile-web-app-capable" content="yes">
<meta name="format-detection" content="telephone=no">
```

关键配置：
- `maximum-scale=1.0, user-scalable=no` - 禁止用户缩放
- `apple-mobile-web-app-capable` - 支持iOS添加到主屏幕
- `format-detection` - 禁用电话号自动识别

### 9.2 Canvas响应式

**文件位置**：[index.html](file:///d:/游戏开发/怪奇旅伴/app_1790jfmbrps/src/index.html#L1410-L1422)

```javascript
function resizeCanvas() {
    const container = document.getElementById('gameContainer');
    const w = container.clientWidth;
    const h = container.clientHeight;
    canvas.width = w;
    canvas.height = h;
    window.W = w;
    window.H = h;
    window.canvasScale = Math.min(w / 900, h / 600);
}
resizeCanvas();
window.addEventListener('resize', resizeCanvas);
```

### 9.3 虚拟方向键

**文件位置**：[index.html](file:///d:/游戏开发/怪奇旅伴/app_1790jfmbrps/src/index.html#L1032-L1043)

移动端显示虚拟D-Pad，支持：
- 方向键（上下左右）
- 信息面板按钮
- 暂停按钮

### 9.4 触摸事件处理

**文件位置**：[index.html](file:///d:/游戏开发/怪奇旅伴/app_1790jfmbrps/src/index.html#L2287-L2327)

```javascript
// 移动端触摸控制
const mobileKeys = {};
function updateMobileKeys(dir, pressed) {
    if (pressed) {
        mobileKeys[dir] = true;
        if (dir === 'up') keys['w'] = true;
        // ... 其他方向映射
    } else {
        // ... 释放处理
    }
}
```

### 9.5 响应式断点

| 断点 | 适配内容 |
|------|---------|
| ≤768px 或 ≤500px高 | 显示移动端控件，隐藏键盘提示 |
| ≤480px | 缩小虚拟按键尺寸 |

### 9.6 移动端操作

| 操作 | 功能 |
|------|------|
| 点击方向键 | 移动角色 |
| 点击 📋 按钮 | 打开/关闭信息面板 |
| 点击 ⏸ 按钮 | 暂停/继续游戏 |
| 双指捏合 | 禁止缩放 |

---

## 10. 成就系统

### 10.1 成就数据结构

**文件位置**：[index.html](file:///d:/游戏开发/怪奇旅伴/app_1790jfmbrps/src/index.html#L1577-L1598)

```javascript
const achievements = [
    { id: 'ninja_clear_normal', char: 0, name: '梦境初探', desc: '忍者喵通关普通难度', icon: '🎯' },
    // ... 40个成就
];
```

- `char: -1` - 通用成就（任意角色可解锁）
- `char: 0-3` - 角色专属成就

### 10.2 成就列表

**角色专属成就**（每个角色6个，共24个）：

| 成就类型 | 忍者喵 | 法师兔 | 铁甲龟 | 金币鼠 |
|---------|--------|--------|--------|--------|
| 通关普通 | 梦境初探 🎯 | 魔幻初成 🎯 | 铁壁初成 🎯 | 掘金初试 🎯 |
| 通关困难 | 梦中潜行 🌙 | 奥法精通 🌙 | 铜墙铁壁 🌙 | 淘金达人 🌙 |
| 通关噩梦 | 暗影行者 💫 | 虚空法师 💫 | 不灭龟魂 💫 | 黄金之魂 💫 |
| 击杀100 | 斩妖除魔 ⚔️ | 魔法轰击 🔮 | 铁甲破敌 🛡️ | 机智猎手 🧀 |
| 击杀300 | 忍术绝技 🌸 | 元素掌控 ✨ | 坚不可摧 🏰 | 狡黠之王 👑 |
| 伤害5000 | 一击必杀 🗡️ | 魔力爆发 ⚡ | 反击之力 🔨 | 理财高手 💎 |

**通用成就**（20个）：

| ID | 名称 | 图标 | 达成条件 |
|----|------|------|---------|
| global_wave12 | 破晓之征 | 🏆 | 累计通关12波 |
| global_atkspd70 | 疾风骤雨 | ⚡ | 攻击速度达到70% |
| global_armor15 | 铜墙铁壁 | 🛡️ | 护甲达到15点 |
| global_gold200 | 小康之家 | 💰 | 单波获得200金币 |
| global_gold500 | 财运亨通 | 💎 | 单波获得500金币 |
| global_gold800 | 一夜暴富 | 👑 | 单波获得800金币 |
| global_score1500 | 崭露头角 | 📊 | 单局得分达到1500 |
| global_score2500 | 声名鹊起 | 🌟 | 单局得分达到2500 |
| global_hp200 | 生命之力 | ❤️ | 生命值达到200 |
| global_scoremult120 | 幸运加持 | 🍀 | 得分倍率提升至120% |
| global_kill50 | 小试牛刀 | ⚔️ | 单局击杀50只怪物 |
| global_kill100 | 战场主宰 | 🗡️ | 单局击杀100只怪物 |
| global_gold1000 | 敛财专家 | 🧀 | 单局获得1000金币 |
| global_kill500 | 屠魔勇士 | 🔥 | 累计击杀500只怪物 |
| global_kill1000 | 噩梦克星 | 💀 | 累计击杀1000只怪物 |
| global_clear_hell | 地狱征服者 | 👹 | 通关地狱难度 |
| global_clear_endless | 无尽深渊 | 🌌 | 无尽模式存活30波 |
| global_companion3 | 三人成众 | 👥 | 同时拥有3个旅伴 |
| global_relic5 | 收藏家 | 📦 | 单局获得5个道具 |
| global_thorns10 | 以牙还牙 | 🌵 | 荆棘伤害达到10点 |

### 10.3 成就存储

**文件位置**：[index.html](file:///d:/游戏开发/怪奇旅伴/app_1790jfmbrps/src/index.html#L1600-L1604)

```javascript
let unlockedAchievements = {};
try {
    const savedAch = localStorage.getItem('bizarreBrigade_achievements');
    if (savedAch) unlockedAchievements = JSON.parse(savedAch);
} catch(e) {}
```

成就数据持久化存储在 `localStorage`，刷新不丢失。

### 10.4 成就解锁检查

**文件位置**：[index.html](file:///d:/游戏开发/怪奇旅伴/app_1790jfmbrps/src/index.html#L1725-L1780)

游戏结束时调用 `checkAchievements()` 检查成就，包括：
- 角色专属成就（通关、击杀、伤害）
- 通用成就（波次、属性、得分、金币等）

---

## 附录：代码行数统计

| 部分 | 估计行数 | 占比 |
|------|---------|-----|
| HTML 结构 | ~250 行 | ~8% |
| CSS 样式 | ~950 行 | ~30% |
| JavaScript 逻辑 | ~1950 行 | ~55% |
| 空行/注释 | ~200 行 | ~7% |
| **总计** | **~2800 行** | **100%** |

---

*Wiki 版本：v1.2*  
*最后更新：2026-06-29*
