# 暮色世界 Wiki

独立静态玩家资料站。默认读取 4,396 条详细技能；完整 44,670 个技能 ID 索引仅在勾选“包含完整技能索引”或打开非详细技能链接时加载。点击非详细索引记录才请求完整兼容资料，不把大型 catalog 作为首次打开的依赖。

## 部署

把本目录全部内容发布到静态站点根目录或项目子目录，保留 `data/`、`assets/`、`downloads/` 和 `.nojekyll`。没有本地服务地址、外部字体或 CDN 依赖。资源与下载链接都相对于当前页面目录，适用于 GitHub Pages 项目站。

GitHub Pages 可从 main 分支根目录发布。也可以将本目录打包成 ZIP，在离线环境解压后放到静态 HTTP 服务器。浏览器直接打开 `file://index.html` 通常不能 fetch JSON；离线部署需要普通静态 HTTP 服务，而不是直接双击文件。

## 数据接口

- `data/spells-detailed.json`：`{spells:[...]}`，保留消耗、射程、说明与 `original_values`。
- `data/spell-index.json`：`{spells:[{id,name,icon,details,aura}]}`，按需读取。
- `data/catalog.json`：完整兼容技能记录；只在请求非详细技能详情时读取。
- `data/items.json`、`classes.json`、`quests.json`、`map-index.json`：客户端物品图标、导师、任务与地图摘要。
- `data/icons-ui.json`：初始读取的轻量图集目录，`icons[key]` 包含 sheet 索引与 x/y/width/height。技能图标键直接使用详细记录与索引导出的 `icon`。
- `data/icons.json`：完整图标映射，仅作为来源下载；`spells[id]` 保留 generic/fallback/mapping_source/原图标。
- `assets/icons-*.png`：真实客户端图标图集。未读取的图片显示一致的缺失占位。
- `downloads/resources.sqlite`、`downloads/resource-workbook.xlsx`：配套数据库与参考表格。

技能详情地址为 `#spell/133`。导师、任务的键包含地图，例如 `#trainer/451%3A600301`；浏览器后退、前进与复制地址可恢复对应详情。

## 内容边界

技能记录存在不等于服务器已开放、角色已学会或导师可教授。等级文字不是人物学习等级。导师专精保留副标题原文，不推断完整技能树。缺失信息显示“未提供”，与 0、否及源数据空字符串区分。

图标优先采用资源图标目录中的显式技能 ID 映射，再回退到技能书图标。来源折叠区保留映射冲突；这是一项 Wiki 展示规则，未据此宣称客户端游戏运行时采用相同图标。通用图标复用会明确标注。

数据使用 textContent 呈现，不执行资料内容，不使用 eval 或动态 HTML 注入。HTTP 请求有 15 秒超时和重试；外部脚本加载失败有独立启动提示。静态资料不含账号、密码或登录依赖。
