# ImproveGame 1.8.1.8 for tModLoader 2025.6.3.0(个人自用适配)

> **非官方,个人自用**。
> 起因很简单:我想玩**旧版本的灾厄**(Calamity 2.0.4.6)。
> 整套环境只能压在 tML 2025.6.3.0 上。
> 代价是:工坊里没有任何 ImproveGame 版本能装 —— 只好从源码编一个,
> 顺手修了两处问题,放出来省得别人再踩一遍。
>
> 不保证适用于所有人的环境,也不保证持续更新。用得上的话直接拿走。

---

## 背景

ImproveGame 在 Steam 创意工坊只发布 **2025.9 / 2026.1 / 2026.3** 三个分支,
没有 2025.6,所以在 tModLoader 2025.6.3.0 上创意工坊里**没有可安装的版本**。

如果你的情况是「**必须留在旧 tML**」——比如想玩老版本灾厄、保住旧存档、
或者联机时版本要一致——那本项目正好对应这个场景。

**如果只是单纯想让 tML 停在旧版、灾厄用新版也无所谓,那没必要用这个,
创意工坊里的最新 ImproveGame 更省事。**

---

## 适用环境

| 组件 | 版本 |
|---|---|
| tModLoader | **2025.6.3.0**(必须一致) |
| Terraria | 1.4.4.9(tML 内置) |
| Calamity Mod | 2.0.4.6(**仅此版本实测**) |
| **ImproveGame** | **1.8.1.8**(本仓库 Release) |

> 更早的 **2.0.4.3** 我**没有测试过**。补丁对「旧版灾厄缺少内容」做了容错,
> 理论上同样能避免崩溃,但 2.0.4.3 本体是针对 tML **2024.4.3** 编译的,
> 它能否在 2025.6.3.0 上正常加载,需要你自己验证。

---

## 安装

1. 下载 Release 里的 `ImproveGame.tmod`
2. 放进 `C:\Users\<用户名>\Documents\My Games\Terraria\tModLoader\Mods\`
3. 启动 tML → 「模组」→ 勾选 ImproveGame → 重新加载

不需要单独装 SilkyUIFramework(该版本已内嵌)。

---

## 改了什么

对应 ImproveGame v1.8.1.8 的 commit `5a11a2ee`。补丁在 `patches/`,可直接 `git apply`。

### 1. C# 14 语法兼容(等价改写)

`UIFramework/BlurMakeSystem.cs` 用了 C# 14 的 `field` 关键字,.NET 8 SDK 编不过,
改成显式后备字段,行为一致:

```csharp
// 原
public static float BlurZoomMultiplierDenominator { get; internal set => field = Math.Max(value, 1f); } = 4f;

// 改
private static float _blurZoomMultiplierDenominator = 4f;
public static float BlurZoomMultiplierDenominator {
    get => _blurZoomMultiplierDenominator;
    internal set => _blurZoomMultiplierDenominator = Math.Max(value, 1f);
}
```

### 2. Calamity 旧版本兼容(这个是真问题)

`Common/ModSystems/ModIntegrationsSystem.cs` 的灾厄联动是按新版灾厄写的。
在 Calamity 2.0.4.6 上,查找 `BlueCandleBuff` 会抛 `KeyNotFoundException`,
**tML 会直接禁用整个 ImproveGame**:

```
该模组加载时发生错误 ImproveGame v1.8.1.8
该模组已被自动禁用。
System.Collections.Generic.KeyNotFoundException: The given key 'BlueCandleBuff' was not present in the dictionary.
```

改法:新增 `TryFindItem` / `TryFindBuff`,把 6 个对接函数改成「找不到就跳过」。
功能没删,旧版灾厄里有的东西照样接入。

---

## 如果想自己锁环境(踩过的坑)

Steam 会按 manifest 校验 `steamapps\common\tModLoader`,
**手动替换过文件后,只要点一次 Steam 的「开始游戏」,就会被"修复"回最新版**,
创意工坊随即把灾厄顶成新版。我的做法:

1. 把 2025.6.3.0 整套安装复制成独立副本,例如
   `...\steamapps\common\tModLoader-2025.6.3.0`(要放在 `common\` 同级,
   tML 才能正确找到 `..\..\workshop`)
2. 副本里放 `steam_appid.txt`,内容 `1281930`(直接启动时 Steamworks 需要)
3. 用副本里的 `start-tModLoader.bat` 启动,不点 Steam 的「开始游戏」
4. 可选:把 `appmanifest_1281930.acf` 的 `AutoUpdateBehavior` 改成 `1`

旧版 Calamity / 旧版 tML 我是在社区归档站找的(非官方,版权归原作者):
[Terraria Internet Archive](https://hia64.github.io)。**本项目不提供这些文件。**

---

## 自己编译

1. 取 ImproveGame 源码,切到 `5a11a2ee`(v1.8.1.8)
2. 应用 `patches/` 下两个补丁
3. 本机 tML 必须是 2025.6.3.0
4. **关键坑**:tML 的 `Libraries/` 里会残留多个版本的库
   (例如 `monomod.utils` 同时有 25.0.4 / 25.0.8 / 25.0.10)。
   `tMLMod.targets` 用 `Libraries/**/*.dll` 通配符引用,MSBuild 会挑**最高版本**,
   而运行时只有 `tModLoader.deps.json` 里那一版
   → 报 `Could not load file or assembly 'MonoMod.Utils, Version=25.0.10.0'`,
   表现为 12 个功能模块的钩子静默失效。
   解决:按 `deps.json` 生成精确引用列表,替换通配符。
5. `dotnet build` 用 .NET 8 SDK 即可

---

## 已知问题

- 进世界时可能冒出几条 `KeyNotFoundException`(`PrefixInfo.Load` / `DataPlayer.LoadData`),
  是读旧存档格式时的容错问题,不影响游玩。
- 不含 Calamity / 汉化包 / tModLoader,需要自备。
- 我这边只验证过自己这一套环境,别的组合没测。

---

## 授权

- 基于 ImproveGame 的 **MIT License**(见 `LICENSE`),版权归原作者 **局长 / 487666123**。
- 本仓库只分发修改后的 ImproveGame 构建产物和补丁。
- 不包含 Calamity、汉化包、tModLoader(版权原因,也不打算包含)。
- 原作者若认为不妥,联系删除。

## 致谢

- [ImproveGame](https://github.com/487666123/ImproveGame) — 局长 及贡献者
- [tModLoader](https://github.com/tModLoader/tModLoader)
- [Solaestas.tModLoader.ModBuilder](https://github.com/Solaestas/Solaestas.tModLoader.ModBuilder)
