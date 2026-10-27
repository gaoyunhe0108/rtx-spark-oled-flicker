# RTX Spark (N1X) OLED panel flicker — ASUS ProArt P14 H7407BA, Windows 11 ARM64

**社区问题报告 / Community bug report** — 非官方，未获 NVIDIA / ASUS 确认。  
Not affiliated with NVIDIA or ASUS. This is a community report with locally verified evidence.

[中文](#中文) · [English](#english)

---

## 中文

### 一句话

内置三星 OLED（ATNA40LE04-0，2880×1800@120Hz）在深灰/深色背景下，**移动鼠标或打字时整屏亮度瞬间变亮，停止操作后回落**；反复操作即形成闪烁。**只要有一路第二显示输出在活动（外接屏复制/扩展模式、甚至 HDMI 诱骗器），内置屏就不再闪烁**。

### 环境

| 项 | 值 |
|---|---|
| 机型 | ASUS ProArt P14 H7407BA（BIOS H7407BA.303，2026-09-22） |
| 系统 | Windows 11 26H1 · 28000.3231 · ARM64 |
| SoC / GPU | NVIDIA RTX Spark N1X（20 核 CPU + 6144 核 Blackwell GPU，sm_121） |
| 驱动 | 617.42（32.0.16.1742），2026-09-30 |
| 内置面板 | Samsung ATNA40LE04-0（SDC424E），14" 触控 OLED，HDR1000 True Black，VRR 支持 30–120Hz |

### 已实测排除

- GPU 负载/DVFS：停掉本地大模型推理（显存 71GB→1.8GB）后照样闪
- 光感自适应亮度：`powercfg` ADAPTBRIGHT=0（已关）
- Windows 动态刷新率 DRR：关；下拉框仅固定 120/60
- 固定 60Hz：照样闪（刷新率整族排除）
- HDR：关；自适应颜色：关
- Windows 亮度 / Gamma：动/静采样期间 WMI 亮度恒 66%、Gamma 表不变（调节在面板层）
- 驱动崩溃：近 3 天无 TDR / 显示驱动错误

### 关键阳性证据

| 条件 | 内置屏表现 |
|---|---|
| 仅内置屏 | 闪 |
| 接外接屏（复制模式） | **不闪** |
| 接外接屏（扩展模式） | **不闪** |
| 插 HDMI 诱骗器（仅模拟 EDID，无真实屏） | **不闪** |

### 结论（机制判断，待厂商确认）

闪烁发生在 Windows 显示栈之下：**内置 OLED 在 eDP 省电态（PSR/面板 D3）与唤醒态之间亮度不一致**，而该省电态只在"内置屏是唯一活动显示输出"时才进入。驱动注册表中存在相关电源参数（`DP_SKIP_PANEL_D3_IN_PSR=1`、`RmPanelBlenDsByAuxToPwrOffMs=400`），均无用户开关。

### 可用缓解

1. **接外接显示器**（复制或扩展模式）——实测有效
2. **HDMI 诱骗器**——实测有效、即插即用；代价是 GPU 持续认为有外接输出，功耗可能略升
3. 只用笔记本单屏时目前无解，等待驱动/BIOS 修复

---

## English

### TL;DR

Internal Samsung OLED (ATNA40LE04-0, 2880x1800@120Hz) shows a full-screen brightness bump whenever there are new frames (mouse movement, typing) on dark/dark-gray content; brightness returns when input stops. **As soon as a second display output is active — external monitor (duplicate or extended mode) or even a dummy HDMI EDID plug — the flicker disappears entirely.**

### Environment

| Item | Value |
|---|---|
| Laptop | ASUS ProArt P14 H7407BA (BIOS H7407BA.303, 2026-09-22) |
| OS | Windows 11 26H1, build 28000.3231, ARM64 |
| SoC / GPU | NVIDIA RTX Spark N1X (20-core CPU + 6144-core Blackwell GPU, sm_121) |
| Driver | 617.42 (32.0.16.1742), 2026-09-30 |
| Internal panel | Samsung ATNA40LE04-0 (SDC424E), 14" touch OLED, DisplayHDR 1000 True Black, VRR 30–120Hz |

### Ruled out locally

- GPU load/DVFS: stopping local LLM inference (71GB -> 1.8GB VRAM) — flicker persists
- Ambient-light adaptive brightness: `powercfg` ADAPTBRIGHT=0
- Windows Dynamic Refresh Rate (DRR): off; refresh dropdown offers only fixed 120/60
- Fixed 60Hz: flicker persists (refresh-rate family excluded)
- HDR: off; Adaptive color: off
- Windows DCS brightness / gamma LUT: constant during idle and motion sampling (change happens below the Windows display stack)
- Driver crashes: no TDR / display driver errors in 3 days

### Key positive evidence

| Condition | Internal panel |
|---|---|
| Internal display only | flickers |
| + external monitor (duplicate) | **no flicker** |
| + external monitor (extended) | **no flicker** |
| + HDMI dummy plug (EDID emulator only) | **no flicker** |

### Conclusion (mechanism hypothesis, not vendor-confirmed)

The flicker occurs below the Windows display stack: the internal OLED panel has a brightness delta between its eDP low-power state (PSR / panel D3) and its active state, and that state is only entered when the internal panel is the sole active display output. Related driver registry values exist (`DP_SKIP_PANEL_D3_IN_PSR=1`, `RmPanelBlenDsByAuxToPwrOffMs=400`) with no user-facing switch.

### Workarounds

1. Attach an external monitor (duplicate or extended mode) — verified
2. HDMI dummy plug (EDID emulator) — verified; may slightly increase GPU power draw
3. No solution for internal-only use yet — waiting on driver/BIOS fix

---

## Related work / links

- [LexZeon/spark-refresh-lab](https://github.com/LexZeon/spark-refresh-lab) — similar input-triggered brightness artifact on an MSI Prestige N16 Flip AI+ (same N1X SoC, Samsung OLED, Windows on ARM); animation-based mitigation
- RTX Spark laptop OLED panel suppliers (6x Samsung / 1x LG): [pconline report](https://news.pconline.com.cn/2183/21831722.html)
- Same-SoC Linux bring-up: [pfeifferj/archlinuxarm-proart-p16](https://github.com/pfeifferj/archlinuxarm-proart-p16), [LeonMarzollDev/proart-p16-linux](https://github.com/LeonMarzollDev/proart-p16-linux)

## Disclaimer

All findings are from a single unit with manual observation; no photometric measurement was performed. "PSR / panel D3" is a mechanism hypothesis based on evidence, not a vendor-confirmed root cause. Use at your own risk.
