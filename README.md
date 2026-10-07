# TokyoMetroAddon — Forge 1.20.1 / MTR 3 移植版

**1.20.1 Ported Version by Luminiflux**

原项目（Fabric 1.19.2）：[https://github.com/magikal-hakase/TokyoMetroAddon-1.19.2](https://github.com/magikal-hakase/TokyoMetroAddon-1.19.2)

本目录是原 mod **完整移植**（以原版 CurseForge 发布版 `tokyometroaddon-1.19.2-0.0.1hotfix-beta.jar` 为基准，含全部 49 个方块 + 座位实体系统）到 **Minecraft 1.20.1 + Forge + MTR 3.x** 的工程。

## 版本信息

| 项目                              | 版本                           |
| ------------------------------- | ---------------------------- |
| Minecraft                       | 1.20.1                       |
| Forge                           | 47.4.26                      |
| ForgeGradle                     | 6.0.54                       |
| Gradle                          | 8.1.1                        |
| JDK                             | 17                           |
| MTR (Minecraft Transit Railway) | 1.20.1-3.2.2-hotfix-2（Forge） |
| Mappings                        | Mojang official 1.20.1       |
| 基准                           | tokyometroaddon-1.19.2-0.0.1hotfix-beta（原版发布版） |

## 构建

```
set JAVA_HOME=<JDK17 路径>
gradlew build
```

产物：`build/libs/tokyometroaddon-1.0.1.jar`（1.20.1 Ported Version by Luminiflux）

## 安装

1. 安装 Forge 1.20.1（47.x）。

2. 将 `tokyometroaddon-1.0.1.jar` 放入 `mods/`。

3. **MTR 3.2.2-hotfix-2 (Forge)** 与 **Architectury 9.1.12+forge (Forge 1.20.1)** 都必须安装 —— MTR 3.2.2 运行时依赖 Architectury `[1.26.37,)`。

* MTR 下载：[https://modrinth.com/mod/minecraft-transit-railway](https://modrinth.com/mod/minecraft-transit-railway)

* Architectury 下载：[https://modrinth.com/mod/architectury-api](https://modrinth.com/mod/architectury-api) （选 **9.1.12+forge**；**勿用 9.2.14**，其 mixin 在 1.20.1 会崩溃）

## 与原版发布版（Fabric 1.19.2 0.0.1hotfix-beta）的差异

* **构建系统**：fabric-loom → ForgeGradle，Yarn mappings → Mojang official mappings。
* **入口**：`fabric.mod.json` + `ModInitializer` → `mods.toml` + `@Mod`。
* **注册方式**：`Registry.register` / `FabricBlockSettings` → `DeferredRegister` + `BlockBehaviour.Properties`。
* **创造模式页**：`FabricItemGroupBuilder` → `CreativeModeTab` DeferredRegister + `BuildCreativeModeTabContentsEvent`（50 条目：toolbox + 49 方块，顺序与原版一致）。
* **客户端渲染层**：`BlockRenderLayerMap` → `ItemBlockRenderTypes`（`FMLClientSetupEvent` 中设置，13 个 cutout 方块 + `NoopRenderer` 实体渲染器——1.20.1 已无 `EmptyEntityRenderer`）。
* **两个空壳 mixin**（`ExampleMixin` / `ExampleClientMixin`，模板遗留）已移除。
* **`data/.../tags/painting_variant/placeable.json`**（原版残留死文件，1.19.3 已移除 painting 系统，其引用的 `notice/request/keep_left_a/keep_left_b` 从未存在）已删除，避免启动时报 tag 加载错误。
* **MTR 依赖**：`libs/` 目录下的本地 jar（官方 Modrinth 构建，`MTR-forge-1.20.1-3.2.2-hotfix-2.jar`），`build.gradle` 用 `implementation files('libs/...')` 直接引用。该 jar 为 mojmap 混淆产物（方法名为 SRG 编号 `m_XXXX_`，类名/参数类型为官方名），**运行时由 Forge modlauncher 统一 remap 到官方名**。
* **闸机/售票机覆写（移植关键点 1）**：`TokyoTicketBarrierBlock` / `TokyoTicketMachineBlock` 继承 MTR 的 `BlockTicketBarrier` / `BlockTicketMachine`，覆写 `getShape`/`getCollisionShape` 时**故意不写 `@Override`、不调用 `super` 同名方法**（编译期父类方法名为 SRG 编号，运行期双方都 remap 为官方名后覆写生效）。已用 `javap` 验证：reobf 后本 mod 方法名 `m_5939_`/`m_5940_` 与 MTR jar 完全一致。
* **注册时机差异（移植关键点 2）**：Forge 的 `DeferredRegister` 到注册事件才写入注册表。所有注册对象字段保留 `RegistryObject<T>` 类型，`.get()` 全部移到事件回调 / 运行时使用点（静态初始化调用会抛 `NullPointerException: Registry Object not present`）。
* **实体系统**：`SeatEntity`（无物理、尺寸 0×0、`getPassengersRidingOffset() = -0.1`）+ `TokyoMetroAddonEntities`（注册名 `seat`，`MobCategory.MISC`）。地铁凳右击会生成座位实体并让玩家骑乘。
* **BE 注册名**：`guide_bell_device`（与发布版一致；GitHub master 为 `guide_bell_device_sine`）。
* **API 名差异（1.20.1 vs 1.19.2 Yarn）**：`StairsBlock`→`StairBlock`；`MaterialColor`→`MapColor`；`VoxelShapes`→`Shapes`；`onStateReplaced`→`onRemove`；`getStateForNeighborUpdate`→`updateShape`；`DoorBlock(Properties)`→`DoorBlock(Properties, BlockSetType)`；`SoundType` 移入 `net.minecraft.world.level.block` 包；`EmptyEntityRenderer`→`NoopRenderer`；`ResourceLocation.fromNamespaceAndPath`；`SoundEvents.UI_BUTTON_CLICK` 返回 `Holder.Reference<SoundEvent>` 需 `.value()`。

## 方块清单（49 个，与原版发布版一致）

| 类别 | 方块 |
| --- | --- |
| 迷你瓷砖 | white / red / yellow / blue / cyan / green / brown / black `_mini_tile_block` |
| 瓷砖 | white\_tile\_block、white / light\_gray / black `_small_tile_block` |
| 站台板 | white\_platform\_plate、light\_gray\_platform\_plate、white\_slatted\_wall\_panel_block |
| 楼梯 | metro\_stairs（带端部状态）、light\_gray\_metro\_stairs |
| 铝杆 | aluminum\_spindle、aluminum\_spindle\_block、aluminum\_spindle\_slope |
| 灯具 | fluorescent\_light（single/start/end 拼接） |
| 引导/警示 | guidance\_block、caution\_block、caution\_block\_with\_guidance\_block、caution\_block\_row、caution\_block\_enclosedline、route\_marker\_blue、route\_marker\_white |
| 设施 | guide\_bell\_device（toolbox 切换音色）、emergency\_exit\_sign、aisle\_guide\_sign、aisle\_guide\_sign\_both、indoor\_fire\_hydrant、stainless\_steel\_fence、metro\_bench（可坐）、metal\_door、metal\_door\_locking |
| 海报 | poster\_notice、poster\_request、poster\_keep\_left\_white、poster\_keep\_left\_blue、poster\_no\_smoking、framed\_poster（6 种随机变体） |
| 闸机/售票（MTR 联动） | tokyo\_ticket\_barrier\_entrance、tokyo\_ticket\_barrier\_exit、tokyo\_ticket\_machine、tokyo\_fare\_adjustment\_machine、tokyo\_platform（复用 MTR `BlockPlatform`） |

物品：toolbox。实体：seat。

> 注：`tokyo_apg_door` 在原版发布版中虽有 blockstate/模型资源，但**发布版自身未注册**（代码里没有注册项）——本移植与原版行为一致，也不注册该方块。

## 验证（已完成）

* `gradlew build` 编译通过（仅剩 `[removal]` 弃用警告：`FMLJavaModLoadingContext.get()` 与 `ItemBlockRenderTypes.setRenderLayer`，均为 1.20.1 Forge 标准写法，不影响功能）。
* 生产式独立服务器冒烟测试通过（两轮）：Forge 47.4.26 + MTR 3.2.2-hotfix-2 + Architectury 9.1.12+forge + 本 mod 一同启动，mod 构造成功（`Hello Forge world!`）、声音注册成功、49 方块 + seat 实体注册无异常、无本 mod 相关 ERROR/FATAL，服务器 `Done (6.3s)`。
* 闸机/售票机覆写机制经字节级验证（`javap` reobf jar：`m_5939_`/`m_5940_` 与 MTR jar 一致）。
* 客户端（游戏内实际渲染 / 创造页）未做图形化验证 —— 本机为无显示环境；逻辑上所有方块 / 物品 / BE / 实体 / 音效 / 创造页已注册并通过服务端验证。

冒烟测试服务器位于 `../.smoketest/`（Forge installer 安装的生产式服务器 + mods 三件套），可随时 `java @user_jvm_args.txt @libraries/net/minecraftforge/forge/1.20.1-47.4.26/win_args.txt --nogui` 复现（已接受 eula）。

## 工具链（本机构建用）

便携版 JDK 17 与 Gradle 8.1.1 放在项目目录外的 `.toolchain/`（同 chats 目录下），不污染系统。
