# RTX Spark OLED 闪屏 · 厂商反馈报告（中英双语）

> 可直接复制到 NVIDIA App 反馈 / NVIDIA 论坛 / ASUS 售后工单
> 机器：ASUS ProArt P14 H7407BA · Windows 11 26H1 (28000.3231) ARM64 · RTX Spark N1X · 驱动 617.42

---

## 中文版（华硕售后 / 华硕反馈）

**问题**：内置三星 OLED 屏（ATNA40LE04-0，2880×1800 120Hz）在深灰/深色背景下，移动鼠标或打字时整屏亮度瞬间升高，停止操作后回落到暗态；连续操作时持续闪烁。**外接显示器时完全不出现。**

**关键观察**：接入第二块显示器后（**复制模式和扩展模式都已实测**），内置屏本身也不再闪烁——同样的合成帧、只是多了一个活动输出即可消失，非常指向"eDP PSR/面板省电态只在单屏时进入"这一条件。**此外，插入 HDMI 诱骗器（仅模拟 EDID/链路、无真实屏幕）同样能消除闪烁**，说明只要 GPU 显示侧存在第二路活动输出即可抑制。

**已排除**（本机实测）：
- 关闭光线自适应亮度（powercfg ADAPTBRIGHT=0）、HDR（关）、自适应颜色（关）、动态刷新率（关，下拉框仅 120/60 固定档）
- 切换固定 60Hz 后仍然闪烁（刷新率族排除）
- 停掉本地 AI 推理负载（显存 71GB→1.8GB）后仍然闪烁（GPU 负载排除）
- 闪烁期间 Windows 亮度值（66%）与 Gamma 表完全不变（说明调节发生在面板/驱动层，不在 Windows 显示栈）
- 近 3 天无任何显示驱动崩溃/TDR 事件

**补充证据**：NVIDIA 显示类注册表存在 `DP_SKIP_PANEL_D3_IN_PSR=1`、`RmPanelBlenDsByAuxToPwrOffMs=400`，表明驱动存在 PSR/面板 D3 电源控制路径（是否进入该状态由运行时条件决定）。

**诉求**：
1. 确认 P14 H7407BA 面板（ATNA40LE04-0）是否存在已知的 PSR/面板省电态亮度差异问题
2. 检查是否有新版 BIOS（当前 H7407BA.303，2026-09-22）或面板固件可修复该问题
3. 同平台 6/7 款 OLED 机型均用三星面板，请评估是否为平台级通病

---

## English version (NVIDIA driver feedback)

**Title**: Internal OLED panel full-screen brightness flicker tied to screen activity (mouse/typing), RTX Spark N1X, Windows 11 ARM64

**Environment**
- Laptop: ASUS ProArt P14 H7407BA (SMBIOS 1.0; BIOS H7407BA.303)
- OS: Windows 11 26H1, build 28000.3231, ARM64
- GPU: NVIDIA RTX Spark N1X (6144-core Blackwell, sm_121 / compute capability 12.1)
- Driver: 617.42 (32.0.16.1742), 2026-09-30
- Internal panel: Samsung ATNA40LE04-0 (SDC424E), 2880x1800 @ 120 Hz, VESA DisplayHDR 1000 True Black, 14" touch OLED, VRR capable 30-120 Hz

**Symptom**: On dark/dark-gray content, moving the mouse or typing causes a brief full-screen brightness increase; brightness falls back when input stops. Continuous animation causes continuous flicker. No flicker on an external DisplayPort/USB-C monitor.

**Key observation**: With a second display attached (tested in both duplicate and extended mode), the flicker also disappears on the internal panel — same composited frames, one more active output. The artifact only occurs when the internal panel is the sole active display output, strongly indicating the eDP PSR / panel power-state entry condition. An HDMI dummy plug (EDID emulator, no real screen) also suppresses the flicker.

**Ruled out (locally verified)**
- Ambient-light adaptive brightness: off (`powercfg` ADAPTBRIGHT=0)
- HDR: off. Adaptive color: off. Dynamic Refresh Rate (DRR): off (refresh dropdown offers only fixed 120 Hz / 60 Hz)
- Fixed 60 Hz: flicker persists (excludes refresh-rate/VRR family)
- Stopping local GPU inference workload (71 GB -> 1.8 GB VRAM, 2% util): flicker persists (excludes GPU load/DVFS)
- Windows DCS brightness (WMI, constant 66%) and gamma LUT (constant sum) do not change during flicker -> change occurs below the Windows display stack, at the panel/eDP level
- No display-driver TDR/crash events in the last 3 days

**Evidence pointing to panel power-state transitions**: NVIDIA display class key contains `DP_SKIP_PANEL_D3_IN_PSR=1` and `RmPanelBlenDsByAuxToPwrOffMs=400`, indicating the driver exposes a PSR / panel D3 power-management path for this panel (presence of the keys does not by itself prove the state is entered).

**Request**
1. Confirm whether a panel brightness delta between eDP PSR/D3 low-power state and active state is a known issue for ATNA40LE04-0 on this platform
2. Expose a documented switch (or driver fix) to keep panel power state constant, or correct the panel power sequencing
3. Note a similar report from an MSI Prestige N16 Flip AI+ user (same N1X SoC, Samsung OLED, Windows on ARM): https://github.com/LexZeon/spark-refresh-lab

---

### 附件

- 手机拍摄视频录像
- 本目录 `RTX_Spark_闪屏排查总结_2026-10-26.md`（v3，含完整排除表）
