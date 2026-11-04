# Meowo

面向 **Paper / Minecraft 26.2** 的一体化生存服插件。经济钱包、家与传送、可配置菜单、番茄钟、矿石商店、聊天系统，以及一整套顺手的小功能，都在同一个 jar 里；每个模块都能单独关掉。

[功能](#功能) · [运行环境](#运行环境) · [安装](#安装) · [命令与权限](#命令与权限) · [配置](#配置) · [常见问题](#常见问题) · [开发](#开发)

## 功能

- **经济** — 固定精度的钱包（SQLite，重启不丢）、玩家转账、每日登录补助、付费 `/tpa` 与 `/home`、`/back` 回死亡点，喵达 ModaCat 商店回收矿石。
- **家与传送** — 多个家、严格落点校验与 `force` 强制传送、请求式 TPA（消息里可直接点接受 / 拒绝）、免费 `/spawn` 回城（预热、冷却、安全落点扫描）。
- **菜单** — `/menu` 心形总菜单，家、番茄钟、商店、钱包、幻翼与体型都能点；各界面共用一套水晶柱外观。槽位、图标、文案、动作全写在 `ui.yml`。
- **日常** — `/sit` 坐下、`/ride` 骑玩家、`/hat` 把手里东西戴头上、潜影盒随身开（背包里 Shift + 右键，或手持潜行右键）、`/phantom` 让夜里的幻翼不再找你、`/size` 调体型、`/mine` 稀有矿物通报。
- **专注** — 番茄钟：工作 / 短休 / 长休循环、可自建预设、BossBar + 屏幕标题 + 头顶文字三处读数。不冻结玩家，也不把人拉回原位。
- **聊天** — 私聊、离线邮件、表情、会话回看；十二位各有分工与性格的猫猫发言者。可接 OpenAI 兼容模型（默认关闭，会按次产生费用），不接模型时由内置应答台回答。
- **状态** — 静止 3 分钟自动挂机，期间免伤、不可碰撞；名牌上的状态圆点随之变化（番茄钟进行中为红色）。
- **管理** — `/invsee` 查看与搬运他人背包、`/eco` 管理余额、只读 HTTP 快照接口（默认关闭）、按玩家客户端语言自动切换中英文案。

几个贯穿全插件的取舍：

- **模块化** — 18 个功能模块在 `config.yml` 里逐个开关；关掉的模块不注册命令、监听器，也不占用数据连接。
- **数据安全** — 改余额与写流水在同一笔事务里完成；配置写错时拒绝整次重载，保留正在运行的模块。
- **配置不打架** — 升级时只补新增的键，你改过的值一概不动；语言包缺的键由内置文案补齐。
- **不抢原版命令** — `/help` 与 `/reload` 只留在 `/meowo` 下面，避免覆盖原版 `/help` 或服务器管理插件的 `/reload`。

## 运行环境

| 项目 | 要求 |
| --- | --- |
| 服务端 | **Paper 26.2**（官方 Paper；其它服务端核心未测试） |
| Minecraft / Java | 26.2 / **Java 25**（Paper 26.2 的要求） |
| 必需插件 | 无。TabooLib 运行时模块与 SQLite 驱动在首次启动时自动下载 |
| 可选插件 | **PacketEvents 2.14.0** — 客户端假实体渲染、家的原版定位栏、番茄钟头顶文字。不装时这些自动降级，其余功能照常 |
| 首次启动 | 服务端需能访问 Maven Central 与 TabooLib 仓库 |

> 上述运行时依赖下载到**服务端根目录的 `libraries/`**（与 `plugins/` 同级，不在插件文件夹里）。SQLite 驱动没有打进 jar，是为了不让单台服务器永远用不到的其它平台原生库占掉体积。离线部署请预先把这个目录搬到目标服；升级驱动版本时清掉 `libraries/org/xerial/`，否则 TabooLib 会复用旧缓存。

## 安装

1. 把 Meowo 的 jar 放进服务端的 `plugins/`。
2. 需要客户端假实体或头顶文字时，一并放入 `packetevents-2.14.0.jar`。
3. 启动一次服务器，插件会生成 `plugins/Meowo/config.yml` 与 `lang/zh_CN.yml`、`lang/en_US.yml`。
4. 按需改配置，执行 `/meowo reload`（或重启）。
5. 输入 `/meowo help` 看自己能用的命令，或 `/menu` 直接打开总菜单。

菜单布局的默认文件在 jar 内的 `ui.yml`，**插件不会主动在插件目录生成它**：要改布局，把这份文件复制到 `plugins/Meowo/ui.yml` 再改；自己放了文件之后，那一屏的按钮就完全以你的文件为准。规范见 [docs/ui-style.md](docs/ui-style.md)。

数据都在 `plugins/Meowo/` 下（`economy/wallets.db`、`chat/mail.db`、`pomodoro/pomodoro.db`、`agents/profile.db` 等 SQLite 文件）。停服后备份整个目录最稳妥；关闭模块不会删除已有数据。

## 命令与权限

主命令是 `/meowo`，可简写为 `/meo`，`/mw` 也是它的别名。**除 `help` 与 `reload` 外，每个子命令都有根级快捷方式**——`/eco`、`/home`、`/pay`、`/sell` 可以直接打，不需要 `meowo` 前缀。`/meowo help` 按权限分页，只列出你当前能用的命令。

下表的「默认」：`所有人` = 默认拥有，`OP` = 默认仅管理员拥有。所有节点都能用权限插件覆盖，完整权限树见 [plugin.yml](core/src/main/resources/plugin.yml)。

### 基础与管理

| 命令 | 说明 | 默认 |
| --- | --- | --- |
| `/meowo help [页码]` | 按权限分页列出命令 | 所有人 |
| `/meowo reload` | 校验并重载配置、语言与模块开关（菜单布局改动要重启生效） | OP |
| `/info` | 版本、实际启用的模块、订阅数 | 所有人 |
| `/render spawn <数量>`、`/render clear` | 客户端假实体，只对执行者可见，需 PacketEvents | OP |

### 经济与商店

| 命令 | 说明 | 默认 |
| --- | --- | --- |
| `/eco balance [玩家]` | 查余额，默认查自己；查他人需 `meowo.economy.balance.others` | 所有人 |
| `/eco give <玩家> <金额>`、`/eco take`、`/eco set` | 增减 / 设定余额，禁止透支 | OP |
| `/eco daily` | 补领当日登录补助（池子不足时只发剩余，不会凭空印钱） | 所有人 |
| `/eco treasury` | 查看税收池 | OP |
| `/pay <玩家> <金额>` | 转账，付款方额外付 1% 支付税 | 所有人 |
| `/sell`、`/sell gui` | 确认后卖光背包里的矿石 / 打开价目表 | 所有人 |
| `/sell hand`、`/sell all <矿物>` | 卖主手物品 / 指定矿物 | 所有人 |

### 家与传送

| 命令 | 说明 | 默认 |
| --- | --- | --- |
| `/home [编号或名称] [force]` | 回家；`force` 忽略落点检查（需 `meowo.home.force`） | 所有人 |
| `/homes`、`/sethome`、`/delhome` | 家的菜单、设置 / 迁址、删除 | 所有人 |
| `/tpa <玩家>`、`/tpaccept`、`/tpdeny`、`/tpacancel` | 请求传送、接受、拒绝、取消；成功传送才最终扣费 | 所有人 |
| `/back` | 回到上次死亡或传送之前的位置（每次扣费） | 所有人 |
| `/spawn` | 免费回城，带预热、冷却与落点扫描 | 所有人 |
| `/setspawn` | 把世界出生点设为当前位置 | OP |

### 日常与形象

| 命令 | 说明 | 默认 |
| --- | --- | --- |
| `/sit`、`/ride [玩家]` | 坐下 / 骑上附近玩家；潜行或再打一次可脱离 | 所有人 |
| `/hat` | 把手上的东西戴到头上，空手则摘下 | 所有人 |
| `/shadowbox`、`/shadowbox give <玩家>` | 打开主手上的潜影盒 / 发放一个（后者默认 OP） | 所有人 / OP |
| `/phantom [on\|off]` | 开关「不睡觉就刷幻翼」的原版机制，只影响自己 | 所有人 |
| `/size [up\|down\|reset\|倍数]` | 调整自己的体型（0.5～3 倍） | 所有人 |
| `/mine` | 矿物勘探与图鉴 | 所有人 |
| `/afk` | 手动标记离开；普通玩家靠静止超时自动进入 | OP |
| `/invsee <玩家>` | 查看他人背包并取走 / 交换物品 | OP |

### 菜单与专注

| 命令 | 说明 | 默认 |
| --- | --- | --- |
| `/menu` | 打开总菜单 | 所有人 |
| `/pomodoro`、`/pomo` | 番茄钟：`gui`（默认）、`start <预设>`、`pause`、`resume`、`stop`、`next`、`stats`、`visuals <bossbar\|title> <on\|off>` | 所有人 |

### 聊天

`/msg` 与 `/mail` 有根级快捷方式，其余挂在 `/meo` 下。

| 命令 | 说明 | 默认 |
| --- | --- | --- |
| `/meo msg <玩家或发言者> <内容>` | 私聊玩家或某只猫 | 所有人 |
| `/meo mail` | 离线私信邮箱 | 所有人 |
| `/meo who`、`/meo replay`、`/meo em` | 列出发言者、回看一段对话、表情菜单 | 所有人 |
| `/meo known`、`/meo forget` | 查看猫对你的印象 / 让猫忘掉你（需开启 `agents`） | 所有人 |

## 配置

改完执行 `/meowo reload`；配置解析失败会拒绝整次重载并保留运行中的模块，日志会点名出错的位置。启动与重载时插件只**补上新增的键**（保留原注释），不会覆盖你写的值；想恢复默认值就删掉那一行再重启。

| 文件 | 管什么 |
| --- | --- |
| `plugins/Meowo/config.yml` | 模块开关、语言策略、税率与补助、传送与出生点、聊天、番茄钟、商店价目表等 |
| `plugins/Meowo/ui.yml` | 菜单布局：槽位、图标、文案、点击动作（`action` 可以是一条以点击者身份执行的命令，也可以是 `@back` 这类内置动作） |
| `plugins/Meowo/lang/zh_CN.yml`、`en_US.yml` | 玩家可见文案；`language-mode: player` 按客户端语言切换，`server` 则全服统一用 `locale` |

常用的一段：

```yaml
language-mode: player   # player = 按玩家客户端语言；server = 全服统一
locale: zh_CN           # 控制台与兜底语言
contexts:               # 每个模块独立开关，改完 /meowo reload
  economy: true
  menu: true
  render: true
  teleport: true
  spawn: true
  agents: false         # 多猫猫 AI，唯一会按次花钱的模块
  webapi: false         # 只读 HTTP 接口，会开端口
economy:
  currency: { name: Meo, symbol: "ฅ" }
  teleport-tax: { rate: "0.001", minimum: "1.00", maximum: "233.00" }
  pay-tax: { rate: "0.01" }
  daily-subsidy: { enabled: true, amount: "1.00", timezone: Asia/Shanghai }
```

完整带注释的默认配置见 [core/src/main/resources/config.yml](core/src/main/resources/config.yml)。几处容易踩到的：

- **模块之间有依赖**：关掉 `economy` 后，`teleport`、`home`、`shop` 会一起停用；用 `/info` 看实际启用状态，不要只看配置文件。
- **菜单布局改动要重启**：`/meowo reload` 会校验 `ui.yml` 并补新增的键，但已经打开的菜单仍按旧布局构建，新布局要重启服务器或插件才生效。
- **AI 猫默认关闭**：`contexts.agents: true` 之外还要填 `agents.base-url` 与 `agents.api-key`（建议用环境变量 `MEOWO_AGENTS_API_KEY`，环境变量优先）。它是唯一按调用计费的模块，不配端点时猫只会简短回应，不会假装能回答。
- **HTTP 接口默认关闭**：打开后只监听 `127.0.0.1:8080`，提供 `/health` 与 `/api/v1/snapshot` 等只读端点，除 `/health` 外都要带 `X-Meow-Secret`。`webapi.secret` 留空时该模块**拒绝启动**；密钥可用 `MEOWO_WEBAPI_SECRET` 提供。对外请走同机的 Cloudflare Tunnel，**不要把端口直接暴露到公网**。端点清单见 [contexts/meowo-webapi/README.md](contexts/meowo-webapi/README.md)。
- **界面与文案**：菜单按钮写错（槽位越界、撞槽位、未知动作）会拒绝整次重载；语言文件支持 `§` 颜色码与 `\n` 换行，两条语言的键名与占位符必须对齐。

## 常见问题

**没装 PacketEvents 会怎样？**
`render` 报告依赖缺失并停用，家的原版定位栏与番茄钟头顶文字自动隐藏；BossBar、屏幕标题和其余模块照常工作。

**首次启动连不上外网 / 离线服怎么装？**
从一台已成功启动的服务器复制整个 `libraries/` 目录（与 `plugins/` 同级）过去，或预先放好 TabooLib 模块与 `sqlite-jdbc` 的 jar。

**`/home` 一直说「落点不安全」？**
家的坐标不会跟着世界变化：挖掉脚下方块、砌进墙里、动了世界边界都会导致回不去。用 `/home <编号或名称> force` 或 GUI 的强制传送按钮直接回到保存时的原坐标，费用相同；开关是 `home.force`，权限 `meowo.home.force`（默认所有人）。

**猫的回答很短 / 不接话？**
`agents` 默认关闭，此时由内置应答台回答。开启并配置好模型端点后才会由 AI 接管；注意它是全插件唯一会按次产生费用的模块。

**改了语言文件没生效？**
执行 `/meowo reload`。`language-mode: player` 按玩家客户端语言选文件，排查时可临时改成 `server` 并指定 `locale`。

**挂机后走动为什么没解除？**
插件从不移动挂机中的玩家，走动也不解除；下蹲、挥手、吃东西、切换物品栏或发言可以退出，被生物或船拖走超过 5 格会自动解除。

**遇到问题去哪反馈？**
先看启动日志与 `/info` 里的实际模块状态，再带着日志到 [GitHub Issues](https://github.com/YaeSaraki/Meowo/issues) 反馈。

## 开发

> 改动代码前先读 [docs/minimax/README.md](docs/minimax/README.md)：架构红线、生命周期与线程模型、数据不变量、精确到文件的改造步骤，以及本机环境的坑都在那里。本文件的上半部分是给服主看的，这里才是给改代码的人看的。

### 构建

需要 JDK **21 + 25**：Gradle 8 的 daemon 用 21，编译、JUnit 与 Paper 服务端用 25；Gradle 会自动发现已安装的 JDK，不需要系统安装 Gradle。

```bash
./gradlew check build       # 测试 + 架构门禁 + 覆盖率门禁 + 配置门禁，等于全部
./gradlew test              # 只跑测试
```

产物是含全部 context 的单 jar，位于 `core/build/libs/`。本地测试服固定 Paper 26.2 build 129、离线模式、仅监听 `127.0.0.1:25565`：

```bash
ACCEPT_MINECRAFT_EULA=true ./gradlew :core:runServer
# 或
ACCEPT_MINECRAFT_EULA=true ./scripts/start-server.sh
```

启动脚本另需 Bash、curl、Python 3；用 `MEOWO_JAVA` 指定 Java 25，用 `MEOWO_RUN_DIR` 换服务端目录。客户端联调见 `scripts/start-client.sh` 与 `scripts/dev.sh`。

### 结构

`api/` 跨模块契约 · `shared/` 无状态工具 · `platform/` Bukkit 门面 · `ui/` **唯一**允许出现 TabooLib 菜单代码的模块（屏幕是数据，不是代码）· `core/` 组合根 `MeowoRuntime` · `contexts/meowo-*/` 18 个业务模块。

依赖方向是 `interfaces → application → domain`，`domain` 不得引用 Bukkit、TabooLib、Adventure 或平台层；context 之间不互相 import，跨模块只走 `api/` 的契约（注册到 Bukkit `ServicesManager`）。模块清单以 [settings.gradle.kts](settings.gradle.kts) 为准，模块关系见 [CONTEXT-MAP.md](CONTEXT-MAP.md)。

### 测试

JUnit 5 + MockBukkit：域层测试不碰 Bukkit，存储测试跑真实 SQLite。`check` 挂着三道门禁——`verifyArchitecture`（纯领域与跨 context 导入）、`verifyCoverage`（总体行覆盖 ≥ 70%、economy domain ≥ 90%）、`ConfigSchemaTest`（配置键、模块清单、运行时装配与代码实际读取的键四者对齐）。新增 context 会进入覆盖率分母，补测试不是可选项。报告在 `build/reports/jacoco/coverageReport/html/index.html` 与各模块的 `build/reports/tests/test/index.html`。

### 对接 API

其它插件在 `plugin.yml` 声明 `depend: [Meowo]`，编译时以 `compileOnly` 依赖本项目的 `api` jar（不要再打包一份），通过服务注册表取接口。模块重载后要重新取得服务，旧引用会拒绝新请求。公开接口只用 JDK 类型，方便 Java 插件消费。

```kotlin
val economy = server.servicesManager.load(com.meowo.api.EconomyAPI::class.java)
    ?: error("Meowo economy context is disabled")
economy.deposit(player.uniqueId, BigDecimal("10.00")).whenComplete { balance, error ->
    // 回调可能在数据库线程；访问 Bukkit API 要先切回主线程
}
```

### 文档与约定

- [docs/minimax/](docs/minimax/README.md) — 开发准本，改代码前必读
- [CONTEXT-MAP.md](CONTEXT-MAP.md) 与各 `contexts/meowo-*/README.md` — 模块职责与业务规则
- [docs/ui-style.md](docs/ui-style.md) — 菜单几何与配色规范；[docs/versions.md](docs/versions.md) — 依赖版本锁定
- [docs/acceptance.md](docs/acceptance.md) — 历史验收快照；[plan.md](plan.md) — 原始需求（设计参考，不是可编译的 API 示例）
- 分支从 `main` 切出，提交信息用 conventional commits 带 scope、正文中文；加一条玩家可见文案要同时改 `zh_CN.yml` 与 `en_US.yml`，键名与占位符完全对齐。

### 许可

[MIT](LICENSE)。
