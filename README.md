# 动漫放置领域 (Anime Idle Realm)

基于 HTML+CSS+JavaScript 的动漫风放置养成游戏。角色/敌人/技能数据从 JSON 读取，配合 generateTool (ComfyUI) 可批量生成游戏美术资源。

## 快速开始

浏览器直接打开 `index.html` 即可游玩。首次进入自动赠送一个 R 级初始角色。

## 项目结构

```
cfe/
├── index.html              # 完整游戏（HTML+CSS+JS 单文件）
├── data/                   # 游戏数据（JSON）
│   ├── characters.json     # 可玩角色
│   ├── enemies.json        # 敌人/怪物
│   ├── skills.json         # 技能
│   ├── stages.json         # 关卡
│   └── items.json          # 物品/装备
├── assets/                 # 图片资源（ComfyUI 生成后放入）
│   ├── characters/         # 角色立绘 <id>.png
│   ├── enemies/            # 敌人立绘 <id>.png
│   ├── skills/             # 技能图标 <id>.png
│   ├── items/              # 物品图标 <id>.png
│   └── ui/                 # UI 元素（预留）
└── generateTool/           # ComfyUI 图片生成工具
    ├── generate.py         # 生成脚本
    ├── workflow.json       # 工作流定义
    ├── output/             # 生成图片输出目录
    └── README.md
```

## 游戏系统

### 角色
- 10 个初始角色，分 4 种稀有度：N / R / SR / SSR
- 5 种元素：火、水、风、光、暗
- 4 种定位：attacker / defender / healer / support
- 每级成长：`属性 = 基础值 + 成长率 × (等级 - 1)`
- 升级经验：`50 × 等级^1.5`
- 最多 4 人上阵

### 属性克制
```
火 > 风 > 水 > 火 (1.5x 伤害, 0.75x 受伤)
光 <> 暗 (2.0x 互克)
光/暗 对 火/水/风 中立(1.0x)
```

### 战斗（CTB 速度条回合制）
- 速度决定行动频率
- 伤害公式：`ATK × 技能倍率 × 元素克制 × (1 - DEF/(DEF+200)) × 暴击 × 随机`
- 技能可附带效果：灼烧/中毒/眩晕/护盾/属性增减
- 多波次关卡，Boss 战
- 支持 1x/2x 速度切换、暂停、逃跑

### 抽卡
| 稀有度 | 概率 | 保底 |
|--------|------|------|
| SSR | 3% | 90 抽 |
| SR | 12% | 10 抽 |
| R | 35% | - |
| N | 50% | - |

消耗召唤券（1抽/张）或宝石（50/抽，450/十连）。

### 离线挂机
- 最多累积 12 小时收益
- 离线效率为手动的 50%
- 体力每 6 分钟恢复 1 点
- 上线弹出收益总结

### 物品
- **装备**：武器/防具/饰品，可给角色穿戴加属性
- **消耗品**：药水/召唤券
- **材料**：元素结晶/龙鳞（用于觉醒，预留）

## AI 助手工作流（添加新内容）

### 标准三步

```bash
# 1. 生成美术资源
cd G:\WebGames\cfe\generateTool

# 角色立绘（竖版 768x1344）
python generate.py \
  --prompt "1girl, silver hair, ice magic, cold expression, snow" \
  --lora "qingg_anima_v1_alter-000002.safetensors" \
  --lora-strength 0.8 \
  --width 768 --height 1344

# 敌人立绘（方形 512x512）
python generate.py \
  --prompt "monster, fire slime, cute, chibi, game monster" \
  --width 512 --height 512

# 技能图标（方形 256x256）
python generate.py \
  --prompt "fire slash icon, simple, game skill icon style" \
  --width 256 --height 256

# 2. 移动到 assets 对应目录
cp generateTool/output/<最新png> assets/characters/<角色id>.png

# 3. 编辑 data/ 下对应 JSON 文件，添加条目
#    刷新浏览器即可看到新内容
```

### JSON 数据格式

#### characters.json 条目模板
```json
{
  "id": "ice_mage",                    // 唯一ID，与图片文件名匹配
  "name": "冰霜法师",                   // 中文名
  "rarity": "SSR",                      // N/R/SR/SSR
  "element": "water",                   // fire/water/wind/light/dark
  "role": "attacker",                   // attacker/defender/healer/support
  "base_stats": {
    "hp": 1100, "atk": 170, "def": 75, "spd": 100,
    "crit_rate": 0.1, "crit_dmg": 1.7
  },
  "growth_rates": {
    "hp": 140, "atk": 20, "def": 8, "spd": 3
  },
  "skill_ids": ["ice_lance", "blizzard"],  // 引用 skills.json 的技能ID
  "portrait": "assets/characters/ice_mage.png",
  "gacha_weight": 5,                    // 抽卡权重（越大越常见，SSR=5, SR=15, R=35, N=50）
  "lora_config": {                      // 🔑 记录生成参数，方便复现
    "lora": "ice_mage_v1.safetensors",
    "lora_strength": 0.8,
    "prompt": "1girl, silver hair, ice magic, cold expression",
    "width": 768,
    "height": 1344
  },
  "description": "操控冰霜之力的神秘法师。"
}
```

#### enemies.json 条目模板
```json
{
  "id": "slime_fire",
  "name": "火焰史莱姆",
  "element": "fire",
  "stats": { "hp": 300, "atk": 45, "def": 25, "spd": 35, "crit_rate": 0.03, "crit_dmg": 1.3 },
  "skill_ids": ["flame_burst_mini"],
  "portrait": "assets/enemies/slime_fire.png",
  "lora_config": { "lora": "", "lora_strength": 0.0, "prompt": "monster, fire slime, cute, chibi", "width": 512, "height": 512 },
  "gold_drop": 40,
  "exp_drop": 25,
  "item_drops": [{ "item_id": "fire_crystal", "drop_rate": 0.1 }]
}
```

#### skills.json 条目模板
```json
{
  "id": "ice_lance",
  "name": "冰晶之枪",
  "type": "damage",           // damage/heal/buff/debuff
  "element": "water",
  "target": "single",         // single/all_allies/all_enemies/self
  "base_multiplier": 2.0,     // 技能倍率
  "scale_stat": "atk",        // 按哪个属性计算
  "cooldown": 3,              // 冷却回合数
  "mana_cost": 35,            // 预留，暂未使用
  "effects": [                // 附加效果数组
    { "type": "spd_down", "chance": 0.5, "duration": 2, "value": 0.7 }
  ],
  "icon": "assets/skills/ice_lance.png",
  "description": "投掷冰晶长枪，有概率降低敌人速度。"
}
```

#### stages.json 条目模板
```json
{
  "id": "stage_2_5",
  "chapter": 2,
  "stage": 5,
  "name": "暗影王座",
  "unlock_condition": "clear_stage_2_4",
  "waves": [
    {
      "enemies": [
        { "enemy_id": "shadow_wolf", "level": 15 },
        { "enemy_id": "shadow_wolf", "level": 15 }
      ]
    },
    {
      "enemies": [
        { "enemy_id": "boss_dark_lord", "level": 20 }
      ]
    }
  ],
  "base_gold": 300,
  "base_exp": 200,
  "first_clear_bonus": {
    "gold": 2000,
    "items": [{ "item_id": "gacha_ticket", "quantity": 5 }]
  }
}
```

#### items.json 条目模板
```json
// 装备类
{
  "id": "ice_blade",
  "name": "冰霜之剑",
  "type": "equipment",
  "subtype": "weapon",
  "rarity": "SR",
  "icon": "assets/items/ice_blade.png",
  "description": "寒气逼人的利剑。",
  "stackable": false,
  "sell_price": 500,
  "equip_slot": "weapon",
  "equip_stats": { "atk": 40, "crit_rate": 0.05 },
  "equip_requirement": { "level": 10 }
}
// 消耗品类
{
  "id": "hp_potion_large",
  "name": "大型生命药水",
  "type": "consumable",
  "rarity": "R",
  "icon": "assets/items/hp_potion_large.png",
  "description": "恢复一名角色1000点生命值。",
  "stackable": true,
  "sell_price": 150,
  "effect": { "type": "heal", "value": 1000 }
}
```

### 效果类型参考

| 类型 | 说明 | value 含义 |
|------|------|-----------|
| `burn` / `poison` / `bleed` | 持续伤害(DOT) | 每回合造成 maxHp × value 伤害 |
| `stun` | 眩晕跳过回合 | 忽略 |
| `shield` | 吸收伤害护盾 | 护盾量 = scale_stat × value |
| `atk_up` / `atk_down` | 攻击增减 | 倍率 (1.3 = +30%, 0.7 = -30%) |
| `def_up` / `def_down` | 防御增减 | 倍率 |
| `spd_up` / `spd_down` | 速度增减 | 倍率 |
| `taunt` | 强制嘲讽 | 忽略 |

### 批量生成脚本示例

```python
# batch_generate.py - 批量生成所有缺少立绘的角色
import json, subprocess, os
from pathlib import Path

ROOT = Path(r'G:\WebGames\cfe')
with open(ROOT / 'data/characters.json', encoding='utf-8') as f:
    data = json.load(f)

for char in data['characters']:
    portrait_path = ROOT / char['portrait']
    if portrait_path.exists():
        print(f'  [跳过] {char["name"]} - 已有立绘')
        continue

    cfg = char.get('lora_config', {})
    cmd = [
        'python', str(ROOT / 'generateTool/generate.py'),
        '--prompt', cfg.get('prompt', f'1girl, {char["name"]}'),
        '--lora', cfg.get('lora', 'qingg_anima_v1_alter-000002.safetensors'),
        '--lora-strength', str(cfg.get('lora_strength', 0.8)),
        '--width', str(cfg.get('width', 768)),
        '--height', str(cfg.get('height', 1344)),
    ]
    print(f'  [生成] {char["name"]} ...')
    subprocess.run(cmd, cwd=ROOT / 'generateTool')

    # 移动最新的输出文件
    output_dir = ROOT / 'generateTool/output'
    latest = max(output_dir.glob('*.png'), key=os.path.getmtime, default=None)
    if latest:
        os.rename(latest, portrait_path)
        print(f'  [完成] {char["name"]} -> {portrait_path}')
```

## 存档

存档保存在浏览器 localStorage，键名为 `cfe_save`。包含：
- 金币/宝石/体力
- 拥有的角色及等级/经验
- 队伍配置
- 已通关卡
- 物品数量
- 抽卡保底计数
- 离线时间戳

清除浏览器数据会重置进度。开发调试可在控制台执行 `localStorage.removeItem('cfe_save')` 重置存档。
