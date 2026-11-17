# ImproveGame 2025.6.3.0 适配版(非官方)

> **非官方修改版**:本项目是 [ImproveGame](https://github.com/487666123/ImproveGame) 的社区适配补丁,
> 让「更好的体验」可以在 **tModLoader 2025.6.3.0 + Calamity Mod 2.0.4.6** 这套较旧的环境里正常运行。
> 与原作者无关,请勿向原作者反馈本版本的问题。

---

## 这是什么 / 为什么需要

ImproveGame 在 Steam 创意工坊只发布 **2025.9 / 2026.1 / 2026.3** 三个分支,**没有 2025.6**。
所以在 tModLoader 2025.6.3.0 上,创意工坊里**没有任何版本能装**。

而很多人还在玩 **Calamity 2.0.4.6**(或更早的 2.0.4.3)——
一旦升级 tModLoader,创意工坊就会自动把灾厄顶成新版,旧存档有风险。

本项目基于 ImproveGame **v1.8.1.8**(2025-06-02)源码,打 **2 处**小补丁后重新编译,
产出可直接用于 tModLoader 2025.6.3.0 的 `.tmod`。

---

## 适用环境

| 组件 | 版本 | 说明 |
|---|---|---|
| tModLoader | **2025.6.3.0** | 必须完全一致(见下方「环境复刻」) |
| Terraria | 1.4.4.9 | tML 内置 |
| Calamity Mod | **2.0.4.6** | 旧版,工坊已下架 |
| Calamity Mod Music | 2.0.5 | — |
| **ImproveGame** | **1.8.1.8** | 本仓库 Release 提供 |

---

## 环境复刻(重要)

这套环境的三个组成部分**都需要自己准备**,本项目只提供 ImproveGame:

| 需要什么 | 从哪里来 |
|---|---|
| Calamity Mod 2.0.4.6 / 2.0.4.3 | Steam 工坊已下架。可在社区归档站 [Terraria Internet Archive](https://hia64.github.io) 查找历史构建(非官方归档,版权归原作者) |
| tModLoader 2025.6.3.0 | Steam 没有旧版分支;可从 [tML GitHub Releases](https://github.com/tModLoader/tModLoader/releases) 或上述归档站获取。**如果你之前装过并保留过备份,那是最好来源** |
| ImproveGame | 本仓库 Releases |

> ⚠️ **请自行确认你取得的文件来源合法**。本项目不托管、不再分发 Calamity / tModLoader / 汉化包。

### 安装后务必锁版本

Steam 会按 depot manifest 校验 `steamapps\common\tModLoader`,
**一旦你点击 Steam 的「开始游戏」,被手动替换过的文件会被自动"修复"回最新版**,
创意工坊随即把灾厄顶成新版。

推荐做法:

1. 把 2025.6.3.0 的整套安装**复制成独立副本**,例如
   `F:\SteamLibrary\steamapps\common\tModLoader-2025.6.3.0`
   (放在 `common\` 同级,tML 才能正确找到 `..\..\workshop`)
2. 在副本里放一个 `steam_appid.txt`,内容 `1281930`(直接启动时 Steamworks 需要)
3. 用副本里的 `start-tModLoader.bat` 启动,**永远不要点 Steam 的「开始游戏」**
4. (可选)把 `appmanifest_1281930.acf` 的 `AutoUpdateBehavior` 改为 `1`,关闭后台自动更新

---

## 安装 ImproveGame

1. 下载 Release 里的 **`ImproveGame.tmod`**
2. 放入 `C:\Users\<用户名>\Documents\My Games\Terraria\tModLoader\Mods\`
3. 启动 tML → 「模组」→ 勾选 **ImproveGame** → 重新加载

**不需要**另外安装 SilkyUIFramework —— 该版本已把 UI 框架内嵌在源码内。

---

## 本版本改了什么

### 1. C# 14 语法兼容(等价改写,无行为变化)

`UIFramework/BlurMakeSystem.cs`:原代码使用 C# 14 的 `field` 关键字,改写为显式后备字段。

```csharp
// 原
public static float BlurZoomMultiplierDenominator { get; internal set => field = Math.Max(value, 1f); } = 4f;

// 改(行为完全一致)
private static float _blurZoomMultiplierDenominator = 4f;
public static float BlurZoomMultiplierDenominator {
    get => _blurZoomMultiplierDenominator;
    internal set => _blurZoomMultiplierDenominator = Math.Max(value, 1f);
}
```

### 2. Calamity 旧版本兼容(**关键修复**)

`Common/ModSystems/ModIntegrationsSystem.cs`:上游 v1.8.1.8 的灾厄联动按**新版灾厄**编写。
在 Calamity 2.0.4.6 上,`PostSetupContent()` 里 `mod.Find<ModBuff>("BlueCandleBuff")`
抛出 `KeyNotFoundException`,导致 tModLoader **自动禁用整个 ImproveGame**:

```
该模组加载时发生错误 ImproveGame v1.8.1.8
该模组已被自动禁用。
System.Collections.Generic.KeyNotFoundException: The given key 'BlueCandleBuff' was not present in the dictionary.
  at ImproveGame.Common.ModSystems.ModIntegrationsSystem.AddBuffIntegration(...)
```

修复:新增 `TryFindItem` / `TryFindBuff`,把 6 个对接函数改为「找不到就跳过」。
**没有删减功能**——旧版灾厄里存在的物品/增益仍会正常接入。

补丁见 `patches/`,可直接 `git apply` 到上游源码。

---

## 从源码重现编译

1. 获取 ImproveGame 源码并切到 commit `5a11a2ee`(v1.8.1.8)
2. 应用 `patches/` 下两个补丁
3. 确保本机 tModLoader 为 **2025.6.3.0**
4. **构建关键点**:tML 安装目录的 `Libraries/` 里会残留多个版本的库
   (如 `monomod.utils` 同时有 25.0.4 / 25.0.8 / 25.0.10)。
   `tMLMod.targets` 用通配符引用 `Libraries/**/*.dll`,MSBuild 会挑**最高版本**,
   而运行时只提供 `tModLoader.deps.json` 登记的那一版
   → 表现为 `Could not load file or assembly 'MonoMod.Utils, Version=25.0.10.0'`,
   12 个功能模块的钩子全部失效。
   **解决**:按 `tModLoader.deps.json` 生成精确引用列表,替换掉通配符。
5. 用 .NET 8 SDK 编译(`dotnet build`),产物即 `ImproveGame.tmod`

---

## 已知问题

- 进世界时可能出现少量 `KeyNotFoundException`(`PrefixInfo.Load` / `DataPlayer.LoadData`),
  来自读取旧存档格式,不影响游玩。
- 本版本不能替代 Calamity / 汉化包 / tModLoader。

---

## 授权

- 基于 ImproveGame 的 **MIT License**(见 `LICENSE`),版权归原作者 **局长 / 487666123** 所有。
- 本仓库**只**分发修改后的 ImproveGame 构建产物与补丁,**不包含** Calamity、汉化包、tModLoader。
- 若原作者认为本项目不当,请联系删除。

## 致谢

- [ImproveGame](https://github.com/487666123/ImproveGame) — 局长 及全体贡献者
- [tModLoader](https://github.com/tModLoader/tModLoader)
- [Solaestas.tModLoader.ModBuilder](https://github.com/Solaestas/Solaestas.tModLoader.ModBuilder)
- [Terraria Internet Archive](https://hia64.github.io) — 旧版模组归档
