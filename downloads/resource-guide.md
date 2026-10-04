# 暮色世界客户端资源数据库与玩家参考

本目录来自已下载的公开客户端 `client-index.wasm` 与 `client-index.data`。资源包版本为 SFPK 2，技能展示库标记客户端 build 10149。全部 5,252 个资源均已通过 AES-256-GCM 认证、解压长度校验，索引和资源数据逐字节覆盖到文件末尾。原始来源及每个资源 SHA-256 可查 `manifest.json`。

这套资料可以查询技能显示、装备词缀、导师服务、任务地点和静态地图；**不是当前服务器完整数值配置**。库中存在某个技能ID不代表服务器启用了该技能、导师可教授该技能或角色已经学会。WoW DBC 展示数据还包含旧版、测试、怪物、物品触发等法术，不应把全部记录当作玩家职业技能。

## 文件入口

| 文件 | 内容 |
|---|---|
| `catalog.json` | 44,670 个技能/法术ID并集，统一字段、原条目和字段来源 |
| `items.json` | 2,086 个物品ID图标映射、2,012 随机词缀、95 suffix、2,646 附魔 |
| `classes.json` | 44 条职业/专业导师的姓名、副标题、服务和地点，保留原出处 |
| `quests.json` | 256 条向导任务记录、117 条服务NPC记录 |
| `map-index.json` | 4 个世界文件和120个地图分块的JSON路径及各表行数 |
| `maps/<map>/world.json` | 世界、区域、分块连接、标记、区域覆盖及触发点 |
| `maps/<map>/chunks/*.json` | 地图分块的图像、绘制、静态碰撞、覆盖、导航原始表 |
| `growth-availability.json` | 人物成长资料的缺失状态，不填猜测曲线 |
| `resources.sqlite` | 可本地查询的SQLite数据库 |
| `tables/*.csv` | UTF-8 BOM表格导出，空单元格表示缺失；数组为JSON文本 |
| `manifest.json` | 输入摘要、认证方法、资源目录、来源DBC摘要、限制 |
| `database-verification.json` | 数据库完整性检查与实际表计数 |
| `raw/` | 原始已认证的JSON、地图文件和素材说明（默认168个文件） |

## 技能怎样阅读

`catalog.json.spells` 每条都有整数 `id`、`name`，以及 `rank`、`passive`、`range`、`cost`、`cooldown`、`cast`、`equipment`、`description`。没有来源值时为 `null`，不会用0或空字符串代替未知。`raw.spellbook` 保存技能书原条目，`raw.tooltip` 保存光环/物品说明原条目，`raw.additional_name` 保存额外名称。`sources` 是原资源内部定位，例如 `assets/items/spell-tooltips.json#spellbook/133`。

详细技能书记录共有 4,396 条；光环/物品触发说明共有 7,022 条；额外名字字典有 37,650 条。各集合可能重叠，不能简单相加。`hidden_aura` 和 `hidden_book` 保存客户端隐藏列表标记，也不是服务器禁用开关。

- `rank` 是“等级 1”等**技能等级显示文字**，不是角色学习等级。
- `raw.spellbook.range_check` 保存目标类别、射程类型与最小/最大数组。数组两项的完整含义仍需客户端消费逻辑核对。
- `recovery_ms`、`category_recovery_ms`、`global_cooldown_ms` 是客户端提供的毫秒值，数据库中也单独建列。缺失不等于无冷却。
- `cost`、`cast`、`cooldown` 保留显示文字，例如“23% 基础法力值”“基础1.5 秒施法（受被动与急速影响）”。没有基础法力成长表时不能直接算实际消耗。
- `power_cost` 等附加结构完整保留在 `raw`。装备要求文字保留在 `equipment`。
- 相同名字的高低技能等级、触发法术和光环可使用不同ID；不要只凭名字合并。

示例：ID 133 的技能书条目为“火球术”“等级 1”，有敌方目标检查、35码范围、23%基础法力消耗和1500毫秒GCD；其光环tooltip描述是持续火焰伤害。统一条目优先显示技能书描述，两个原始描述都保留。伤害公式和当前服务器具体伤害值没有在这份展示导出中出现。

## 物品与职业资料的边界

2,086个 `items` 记录来自图标表，只有物品ID和图标键。名称、品质、等级和属性为 `null`。随机词缀的 `enchants` 是关联附魔ID数组；suffix还提供 `allocation` 数组。附魔只有 `text` 和 `amount`，其最终属性计算和装备适用条件未提供，不能把所有 `amount` 都解释成某项属性加成。

任务向导中确实出现“战士导师”“法师导师”“游侠导师”，副标题包括“铁卫与剑锋”“元素与灵愈”“影刃与驭兽”；也出现“秘法导师”“元素与祈愿”等地区文字。这里保留可核对的导师资料，不由这些文字推导职业ID、完整专精树或技能所属职业。专业导师副标题含“烹饪 1—75”“主专业 · 采药 1—150”等客户端文字，这些是导师说明，不是人物等级成长曲线。

目前资源包完整目录未找到以下服务器表：每级升级经验、种族/职业基础属性、每级基础生命/法力、导师技能学习等级、完整伤害/治疗成长公式。它们在 `growth_availability` 中标记为不可用。本次没有获取或推断服务器未下发的资料。

## SQLite schema

| 表 | 行数 | 主键及主要字段 |
|---|---:|---|
| `resources` | 5,252 | `path`; ordinal、offset、packed_size、original_size、compression、sha256、authenticated、extracted |
| `spells` | 44,670 | `id`; name/rank/passive/range/cost/cooldown/cast/equipment/description；隐藏标记、has_spellbook、4个冷却数值、sources_json、raw_json |
| `items` | 2,086 | `id`; name、icon、level、quality、stats_json、source、raw_json、availability |
| `random_modifiers` | 4,753 | `(kind,id)`; name/prefix/text/amount、enchants_json、allocation_json、source、raw_json |
| `maps` | 124 | `source`; json_path、magic、version、table_counts_json、footer_hex |
| `map_rows` | 76,316 | `(source,table_name,row_index)`; row_json |
| `quests` | 256 | `source`; id、map、title、mainline、raw_json |
| `services` | 117 | `source`; entry、map、name、subtitle、actor、services_json、points_json、raw_json |
| `growth_availability` | 5 | `field`; available=0、reason、source |
| `metadata` | 4 | `key`; value_json，保存客户端build、manifest、限制和schema版本 |

`source` 指原始资源路径和内部定位；`raw_json` 保存原条目，方便对照字段。缺失值使用SQL `NULL`。重复任务ID在不同地图向导中可重复，故任务表以来源为主键。SQLite采用UTF-8，可用Python内置sqlite3或SQLite桌面工具查询。

```sql
-- 详细展示资料优先；该查询不证明角色已学会这些技能。
SELECT id,name,rank,cost,cooldown,cast
FROM spells WHERE has_spellbook=1 AND name LIKE '%火球%';

SELECT name,subtitle,points_json,source
FROM services WHERE services_json LIKE '%class_trainer%';

SELECT kind,id,name,text,amount,enchants_json
FROM random_modifiers WHERE name LIKE '%智力%' OR text LIKE '%智力%';

SELECT source,row_index,row_json
FROM map_rows WHERE table_name='collision' LIMIT 20;
```

## 地图格式与未验证语义

已验证二进制语法：SFWD世界文件版本2、SFCH分块版本1；头部依次是4字节magic、u16LE版本、u16LE表数量。每表依次是u16LE名称字节数、UTF-8名称、u16LE列数、u32LE行数。单元格tag：0=null，1=i64LE，2=f64LE，3=u16LE字节数加UTF-8。全部124个文件成功消费全部表，末尾保留8字节footer，其语义未验证。

各表以 `column_count`、`row_count`、`rows` 保存，不猜测所有列名。世界表包含world/region/chunk/link/marker/area/npc_station/trigger。分块表包含chunk/image/sprite/frame/draw/collision/coverage/navigation。静态碰撞看起来为矩形 `[x,y,width,height]`；navigation例如 `[97,205,51,1]` 和 `[113,205,10,1]`、`[113,221,35,1]`，符合 `[行,起始列,长度,连通分区]` 的行程压缩结构，但需要客户端导航消费逻辑确认列语义与像素换算。JSON保留全部891,625条draw绘制行；SQLite集中存储导航、碰撞、世界关系等资料，没有再次存储图像/绘制行。

静态地图不涵盖服务器动态角色、移动怪物和最新地图变更。拿到这些文件不等于已验证角色可以自动绕开所有树木或阻挡物。

## 离线重建

在项目根目录运行（Node与Python均只使用内置库）：

```powershell
& 'C:\Program Files\nodejs\node.exe' resource-unpack.cjs
.venv\Scripts\python.exe resource-build-db.py
```

默认对全部资源解密、认证、解压和哈希，但只落盘JSON、地图、list和md；加 `--all` 会把全部素材落盘到 `game-data/raw`。解包程序只接受已验证WASM SHA-256快照，客户端升级后应重新核对函数和密钥地址。数据库重建只更新本目录的SQLite表。流程不使用浏览器profile、CDP、截图或服务器写入。

认证方式：f3568将WASM静态内存148688和148720处的两组32字节异或得到AES-256密钥；索引IV为资源头20:32、AAD为0:32、tag为32:48。资源IV为头部20:28加1起始序号u32LE，AAD为头部0:32加索引条目到压缩标志，tag来自索引条目。压缩标志1使用zlib。`manifest.json` 记录原始输入摘要和每份已解资源摘要；`database-verification.json` 确认 `PRAGMA integrity_check` 为 `ok`。
