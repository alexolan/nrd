# NootedRed RDNA2 合并说明

**合并版**: 上游 ChefKossInc/NootedRed master（v0.9.0-dev，含至 v0.8.10 的全部变更）
**fork 基线**: NootedRed-RDNA2 fork（基于 v0.8.2，forker 在 R5 7535HS / 660M 上实测）
**合并日期**: 2026-10-27

## 合并方式

以上游 master 为基体（吸收官方半年演进：文件结构重构、SMU/cgs 新架构、
AppleGFXHDA 修复、Mojave 兼容、新固件），
逐项核对 fork 改动在上游的吸收情况，仅重放上游仍然缺失、且经 fork 实测的适配点。

## fork 改动逐项去向

| fork 改动 (相对 v0.8.2) | 上游状态 | 处理 |
|---|---|---|
| IOPCIMatch 的 RDNA2 家族 ID (0x164C 等) | base v0.8.2 起即存在（非 fork 新增） | 无需处理 |
| CAIL→TTL 目录重组 | ✅ 已吸收（上游同构） | 无需处理 |
| GPU_TASK_SINGLE_CHANNEL 枚举 + GPUTaskSingleChannel 参数 | ✅ 已吸收 | 无需处理 |
| PP_WAIT_ON_REGISTER_TIMEOUT_DEFAULT=2000 | ✅ 已吸收 | 无需处理 |
| CAIL_AsicRevForRlcFw=177 | ✅ 已吸收 | 无需处理 |
| X6000FB 删 PP_* 默认参数、加 DalForceSingleDispPipeSplit | ✅ 已吸收 | 无需处理 |
| SMU busy-wait 超时重写 (NRed::waitForFunc) | 上游换新 SMU/cgs 架构 | 不移植（无对应物，硬移植会引入 bug） |
| HWLibs.cpp 459 行适配 | 上游 0.9.0 重写 HWLibs（移至根目录） | 不移植（同上） |
| **X5000.xml 3 处 `SMU_DisableMmhubPowerGating=1`** | ❌ 上游缺失 | **已移植**（Renoir/GreenSardine/Rembrandt 三个 personality 块） |
| **X6000FB 0x15DD 块 6 个 PP 电源参数** | ❌ 上游缺失 | **已移植**（PP_DisableVoltageIsland=1 / PP_DriverEnableNBDPM=1 / PP_GfxOffControl=1 / PP_SclkDeepSleepDisable=0 / PP_Sumo_DisableGfxPowerGatingInUVD=0 / PP_ThermalAutoThrottlingEnable=1） |

## CI

`.github/workflows/build.yml`：构建 Release（x86_64）→ 解包 zip → ad-hoc 重签
（`codesign -fs -`）→ 验证 → 原名重打 → 上传 Artifact / tag 时自动发 Release。
上游 v0.8.10 发布包实测 kext 无 `LC_CODE_SIGNATURE`（打包脚本先于 Xcode 签名执行），
故 CI 必须显式重签。产物：`NootedRed-0.9.0-RELEASE.zip`。

## 适配目标

AMD Rembrandt-R **R5 7535HS / Radeon 660M**（RDNA2，iGPU 家族 DID
`0x164C / 0x1636 / 0x1638`，NootedRed 按 Renoir/GreenSardine 属性路径处理）。
fork 作者（keybordguy/NootedRed-RDNA2）与目标机型同为 7535HS，仅 660M 实测过；
上游 IOPCIMatch 自 v0.8.2 起即含这些 ID，0.9.0 沿 Renoir 路径持续演进，
但无 660M 专项适配声明——fork 的 SMU/电源实测参数是本合并版对 660M 的增益。
若遇核显异常，可对照纯上游构建做 A/B 回退定位。
