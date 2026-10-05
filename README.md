# Nox Tab Footer v1.1

适用 Minecraft Java 26.2。独立的数据包 + 资源包，命名空间 `nox_tab`，不依赖起床战争。

## 安装与使用

1. 将 `Nox Tab Footer - Datapack.zip` 放入存档的 `datapacks`。
2. 将 `Nox Tab Footer - Resourcepack.zip` 放入 `resourcepacks` 并启用。
3. 执行 `/reload`，客户端按 F3+T。
4. `/trigger tab_footer` 打开编辑器，填入文案、数据读取对象及格式，点击保存。

首次安装为空，不显示额外文案。设置保存在存档的 command storage，重载后保留。
重置或保存空文本会删除文案与文本缓存，并关闭底部显示。编辑器可以重复打开。

`/trigger tab_footer` 无需 OP。保存和重置调用原版管理命令，需要 OP；普通玩家不能更改全服文案。
Solo 和 4s 已内置整合后的实现，不需要额外叠加这个独立包。

## 共享文案与目标选择器

Tab 的文案由服务端全员共享，不是每个人自己的文案。目标选择器是**数据读取对象**，不是接收者。
例如选择 `@a[name=Nox_Obscura,limit=1]`，然后显示 Kills，就让所有人看到 Nox_Obscura 的击杀数。
`@s` 在保存时绑定编辑者的 UUID，重载及服务器定时刷新时仍然指向该玩家。
选择器匹配多名实体时取第一个；推荐明确指定玩家名或 `limit=1`。
无匹配对象或文本组件错误时不保存，保留原来的文案。保存后目标离线会暂停重绘，保留上次有效显示，回来后恢复。

原版 `scoreboard objectives setdisplay list` 没有接收者参数，这套方法借用 list 计分板和资源包着色器绘制底部信息。
它不提供真正的个人 Tab footer。不同玩家分别看到自己的 Kills，需要服务端模组/插件控制各客户端收到的内容。

## 普通文本格式

选“普通文本（颜色代码）”。支持实际的 `§`、输入字面量 `\u00a7`，也兼容示例中的 `|u00a7`。
支持 `§0`—`§f` 的十六种颜色，以及 `§l` 粗体、`§o` 斜体、`§n` 下划线、`§m` 删除线、`§r` 重置。
支持直接换行，以及字面量 `\n`。双引号、单引号、反斜杠都可保存。

```text
\u00a7eI'm \u00a7b\u00a7lNox_Obscura
\n
|u00a77123
```

示例中直接换行与 `\n` 都会产生换行；只想单独换一次时使用其中一种即可。

## 文本组件：Kills、玩家名与 NBT

选“文本组件（score / selector / nbt）”，填入 JSON（也接受合法 SNBT 文本组件）。
数据读取对象填 `@s` 或明确的玩家选择器。组件内的 `@s` 指向这个读取对象。

显示玩家的 Kills（需要已有 `Kills` objective）：

```json
[{"text":"Kills: ","color":"yellow"},{"score":{"name":"@s","objective":"Kills"},"color":"aqua","bold":true}]
```

显示玩家名及 Y 坐标：

```json
[{"text":"Player: ","color":"green"},{"selector":"@s","color":"aqua"},{"text":" / Y: ","color":"gray"},{"nbt":"Pos[1]","entity":"@s","color":"yellow"}]
```

读取 command storage：

```json
[{"text":"Round: ","color":"yellow"},{"nbt":"round","storage":"example:game","color":"aqua"}]
```

支持原版可解析的 `score`、`selector`、`nbt`（entity/storage/block）及嵌套 `extra`，保留颜色和上述样式。
计分板分数通过 `score` 读取，不是玩家 NBT。`translate` 显示 fallback，未提供时显示键名；不进行逐客户端语言切换。

## 显示与兼容

- 每行居中，较长文案会延伸背景；玩家名保持原版玩家区域居中。
- 玩家加入或离开约 0.25 秒内重算位置，按住 Tab 不用松开；20 人以上按原版多列布局计算。
- 动态分数与 NBT 约每秒刷新。最多六行，每行约 256 像素，超长自动换行；输入最多 2048 字符。
- 提供常用 BMP 字符、中文及拉丁字母；不支持的字符替换为 `?`。不保证补充平面 emoji。
- 占用 list 显示槽，使用独立的 `NoxTabList` 镜像分数。配置 `settings.score_objective` 可同时显示原有计分；为空时只显示文案。原 objective 的数据、数字颜色和 below_name 样式都不改动。关闭时可恢复指定的 list objective。
- 原版客户端不为旁观者绘制 Tab 分数。当所有在线玩家都为旁观者时，没有可用于绘制的条目，底部文案也暂不显示。
- 使用 `text.vsh` / `text.fsh`。与其他替换同一着色器的资源包同时使用，需要合并着色器；Solo 和 4s 已完成合并。
- 字体来源和许可证在资源包 `licenses` 目录，包含 Unifont 源文件及其原始许可。

## 数据包集成 API

```mcfunction
# 从命令/其他数据包更改共享文本：
data modify storage nox_tab:config settings.raw set value "§eHello\n§bWorld"
data modify storage nox_tab:config settings.format set value "plain"
data modify storage nox_tab:config settings.target set value "@a[limit=1]"
function nox_tab:enable

# 同时显示某个 objective 的分数（原数据与样式不修改）：
data modify storage nox_tab:config settings.score_objective set value "Kills"
function nox_tab:enable

# 不显示玩家的 list 分数，只显示文案：
data modify storage nox_tab:config settings.score_objective set value ""
function nox_tab:enable

# 设置关闭后恢复的 list objective（必须已存在）：
data modify storage nox_tab:config settings.restore set value "health"

# 清空内容并停止显示：
function nox_tab:reset
```

通用前置包的默认设置为空。Solo 和 4s 的整合版本只从 `data/bw/function/tab/config.mcfunction` 读取文案；修改后 `/reload` 生效，没有 trigger 或编辑对话框。独立库保留保存、重置及 OP 编辑功能。

v1.1 修正了 below_name 被 Tab 格式染色的问题。名字和文本组件的临时读取改用不可见、无碰撞的 item_display，同步清理早期版本残留的探针矿车。
