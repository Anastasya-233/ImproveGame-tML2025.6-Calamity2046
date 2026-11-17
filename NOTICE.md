# NOTICE — 修改说明

本仓库是 **ImproveGame** 的非官方修改版,用于适配 tModLoader **2025.6.3.0** + Calamity Mod **2.0.4.6**。

## 上游项目

| 项目 | 地址 | 授权 |
|---|---|---|
| ImproveGame | https://github.com/487666123/ImproveGame | MIT |
| 上游作者 | 局长(487666123)及全体贡献者 | — |
| 基线版本 | **v1.8.1.8**(commit `5a11a2eeb22e8e73d29f8315d38497072285baae`,2025-06-02) | — |

原始版权声明保留于 `LICENSE`:

```
MIT License

Copyright (c) 2022 局长
```

## 修改清单

修改日期:2026-11-17

### 1. `UIFramework/BlurMakeSystem.cs`

- **原因**:原代码使用 C# 14 的 `field` 上下文关键字,而 .NET 8 SDK 的编译器(Roslyn 4.8 / C# 12)无法解析。
- **改动**:改写为显式后备字段 `_blurZoomMultiplierDenominator`,getter/setter 行为与原自动属性完全一致(含 `Math.Max(value, 1f)` 的下限保护)。
- 补丁:`patches/0001-blurmake-csharp14-field-keyword.patch`(+7 / −2)

### 2. `Common/ModSystems/ModIntegrationsSystem.cs`

- **原因**:上游 v1.8.1.8 的 Calamity 联动按**新版灾厄**编写。在 Calamity 2.0.4.6 上,
  `PostSetupContent()` 中 `mod.Find<ModBuff>("BlueCandleBuff")` 会抛 `KeyNotFoundException`
  (`The given key 'BlueCandleBuff' was not present in the dictionary`),
  导致 tModLoader **自动禁用整个 ImproveGame**,报「该模组加载时发生错误」。
- **改动**:新增 `TryFindItem` / `TryFindBuff` 两个辅助方法,并把以下 6 个对接函数
  由「硬查找」改为「找不到就跳过」:
  - `AddBuffIntegration`
  - `AddBuffConflicts`
  - `AddCraftStationIntegration`
  - `AddInfBuffsConsume`
  - `AddHomeTpIntegration`
  - `AddFishingAccIntegration`
- **未删减任何功能**:旧版灾厄中存在的物品/增益仍会正常接入,仅缺失项被安全跳过。
- 补丁:`patches/0002-calamity-2.0.4.6-compat.patch`(+43 / −9)

### 3. 构建配置(不入源码,仅影响编译)

- 只引用 `tModLoader.deps.json` 登记的库版本,避免 tML 目录内残留的高版本程序集
  (如 `MonoMod.Utils 25.0.10`)被编译期引用、运行期加载失败。
- 未改动 `build.txt`、本地化文件、资源文件或任何玩法逻辑。

## 本仓库不包含的第三方内容

| 内容 | 原因 |
|---|---|
| Calamity Mod / Calamity Mod Music | 其授权不允许再分发,请通过官方渠道获取 |
| Calamity 汉化包 | 第三方作者作品 |
| tModLoader / Terraria | Steam 独占分发 |
| 任何存档、配置、密钥 | 隐私 |

## 免责

- 本项目与 ImproveGame 原作者无关,请勿向上游反馈本版本的问题。
- 若原作者认为本项目不当,请联系删除。
