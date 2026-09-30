# ESP32-P4-Function-EV-Board MIPI Camera Sub V1.1 — KiCad

[中文](#中文) | [English](#english)

---

## 中文

这是 Espressif 官方开发板 **ESP32-P4-Function-EV-Board** 的 MIPI 相机转接板（Camera Adapter Board, Camera Sub V1.1, 2024-05-29）的 KiCad 工程。设计版权归 Espressif 所有；本仓库只是把官方 OrCAD/Allegro 设计数据转成 KiCad，并修复了导入过程中出现的问题。

### 官方资料

- 开发板用户指南 (v1.5.2)：<https://docs.espressif.com/projects/esp-dev-kits/en/latest/esp32p4/esp32-p4-function-ev-board/user_guide.html>
- 相机转接板原理图 (PDF)：<https://dl.espressif.com/dl/schematics/esp32-p4-function-ev-board-camera-subboard-schematics.pdf>
- 相机转接板 PCB 布局 (PDF)：<https://dl.espressif.com/dl/schematics/esp32-p4-function-ev-board-camera-subboard-pcb-layout.pdf>
- 官方参考设计包（OrCAD DSN、Allegro BRD、Gerber、BOM、贴片文件）：<https://dl.espressif.com/schematics/CameraAdapterBoardReferenceDesign.zip>

### 为什么需要这个仓库

KiCad 的 OrCAD `.DSN` 导入功能目前只在 Nightly (10.99) 里有，正式版要等到 KiCad 11。即使能导入，解析结果和 Board Setup 的映射也有不少问题，导入后直接跑会有大量 ERC/DRC 报错。本工程逐项修正了这些导入问题，电路连接与官方设计保持一致。

### 打开方式

- **KiCad 10.0 及以上**即可打开（已在 KiCad 10.0.5 正式版和 Nightly 10.99 上验证）。转换和修复是在 Nightly 10.99 里完成的，之后用 KiCad 10.0.5 原生重存为 10.0 格式，以便正式版直接使用，KiCad 11 也能打开。降级前后的网表、焊盘/过孔坐标、坐标文件、Gerber 与钻孔几何、ERC/DRC 结果逐项比对一致。
- 用 KiCad 打开 `SCH_ESP32-P4_FUNCTION_EV_BOARD_MIPI_Camera_Sub_V1.1_20240529.kicad_pro`。
- 符号库 `orcad_import` 和封装库 `Camera_Sub` 都在工程目录内，已通过项目库表 (`sym-lib-table` / `fp-lib-table`) 注册，不依赖任何全局库。

### 检查结果（KiCad 10.0.5 与 Nightly 10.99 `kicad-cli`，结果相同）

| 检查 | 结果 |
| --- | --- |
| ERC | 0 |
| DRC 错误 | 0 |
| 未连接项 | 0 |
| 原理图 / PCB 一致性 | 0 |
| DRC 警告 | 57 条丝印警告，与官方 Gerber 一致（见下文） |

剩余 57 条丝印警告来自原设计本身，已对照官方 Gerber 确认，没有修改：位号字高 0.635 mm 低于 KiCad 默认下限 0.8 mm（33 条）；J1/SW1 的丝印跨过焊盘，生产时由阻焊裁掉（23 条）；"Same-Side FFC" 文字与 R20 丝印相碰（1 条）。

### 修复内容

原理图（OrCAD 导入）：
- 导入器把 OrCAD 的 "Power" 引脚全部映射为 `power_in`，导致 LDO（U1/U2）的 OUT 也成了电源输入，1V8/2V8 无驱动源。已改为 `power_out`。
- 符号库 `orcad_import` 未注册：导出为项目库 `orcad_import.kicad_sym`。
- `Footprint` 字段为空（封装名在 `OrCAD Footprint` 字段中）：已填写为 `Camera_Sub:<封装名>`。
- 3V3/GND 由 J1 从主板引入，添加 PWR_FLAG。
- 4 根 OrCAD 风格的带标签短线一端悬空：截短到标签位置，拓扑不变。

PCB（Allegro 导入）：
- 封装没有库名、Value 和原理图关联：从板上实物封装生成项目封装库 `Camera_Sub.pretty`，补全库名、Value、符号关联和原理图字段。
- 网络名与原理图不一致：标签网络加 `/` 前缀；`Net_xx` 单焊盘网络改为原理图的 `unconnected-(…)` 名称。已逐焊盘与原理图网表核对。
- 两个无位号的 Allegro 机械图形：命名为 MECH1/MECH2 并设为 board-only。
- 约束为 KiCad 默认值：恢复 Allegro 约束（最小线宽 0.127 mm、最小钻孔 0.25 mm）。
- 网络类按实际铜皮分配：`DP_CSI_A_*`（CSI 差分对，0.1778 mm）、`CS_0`（普通信号，0.127 mm）、`POWER`（3V3/AVDD_2V8/DOVDD_1V8/GND，0.635 mm，过孔 0.762/0.5）；预设线宽、过孔和差分对尺寸表填入板上实际使用的尺寸。
- GND 铺铜：间距 0.1778 mm、焊盘实心连接，重新填充。

### 版权与许可

设计版权归 Espressif Systems (Shanghai) Co., Ltd. 所有。本仓库**不授予任何许可**，详见 [LICENSE](LICENSE)。本仓库为非营利性的学习与参考分享，与 Espressif 无关联、未获其背书；如 Espressif 提出异议，将立即删除。

---

## English

KiCad project for the MIPI Camera Adapter Board (Camera Sub V1.1, 2024-05-29) of Espressif's **ESP32-P4-Function-EV-Board**. The design is copyright Espressif. This repository only converts Espressif's OrCAD/Allegro design data to KiCad and fixes the problems introduced by the import.

### Official resources

- Board user guide (v1.5.2): <https://docs.espressif.com/projects/esp-dev-kits/en/latest/esp32p4/esp32-p4-function-ev-board/user_guide.html>
- Camera adapter board schematic (PDF): <https://dl.espressif.com/dl/schematics/esp32-p4-function-ev-board-camera-subboard-schematics.pdf>
- Camera adapter board PCB layout (PDF): <https://dl.espressif.com/dl/schematics/esp32-p4-function-ev-board-camera-subboard-pcb-layout.pdf>
- Official reference design package (OrCAD DSN, Allegro BRD, Gerbers, BOM, placement): <https://dl.espressif.com/schematics/CameraAdapterBoardReferenceDesign.zip>

### Why

KiCad's OrCAD `.DSN` importer is currently only in the Nightly builds (10.99) and will ship in KiCad 11. Even then, the parser and the Board Setup mapping get a lot wrong, and a fresh import reports many ERC/DRC errors. This project fixes those import problems one by one. The connectivity matches the official design.

### Opening the project

- **Opens in KiCad 10.0 or later** (verified on KiCad 10.0.5 stable and Nightly 10.99). The conversion and fixes were done in Nightly 10.99, then the files were re-saved natively by KiCad 10.0.5 in the 10.0 format so the stable release can use them; KiCad 11 will open them too. Netlist, pad/via coordinates, placement file, Gerber and drill geometry and ERC/DRC results were compared before and after the downgrade and match.
- Open `SCH_ESP32-P4_FUNCTION_EV_BOARD_MIPI_Camera_Sub_V1.1_20240529.kicad_pro`.
- The `orcad_import` symbol library and the `Camera_Sub` footprint library live in the project folder and are registered in the project library tables. No global libraries are needed.

### Check results (KiCad 10.0.5 and Nightly 10.99 `kicad-cli`, identical)

| Check | Result |
| --- | --- |
| ERC | 0 |
| DRC errors | 0 |
| Unconnected items | 0 |
| Schematic / PCB parity | 0 |
| DRC warnings | 57 silkscreen warnings, same as Espressif's Gerbers (see below) |

The 57 remaining silkscreen warnings come from the original design and were checked against Espressif's Gerbers: 0.635 mm reference text, below KiCad's default 0.8 mm minimum (33); J1/SW1 silkscreen crossing pads, clipped by the solder mask at fabrication (23); the "Same-Side FFC" text touching R20's silkscreen (1).

### What was fixed

Schematic (OrCAD import):
- The importer maps every OrCAD "Power" pin to `power_in`, so the LDO (U1/U2) OUT pins became inputs and 1V8/2V8 had no driver. Changed to `power_out`.
- The `orcad_import` symbol library was not registered: exported to the project library `orcad_import.kicad_sym`.
- Empty `Footprint` fields (the package name was in `OrCAD Footprint`): set to `Camera_Sub:<package>`.
- PWR_FLAGs added on 3V3/GND, which enter from the main board through J1.
- Four OrCAD-style labelled stub wires had a dangling end: trimmed to the label, topology unchanged.

PCB (Allegro import):
- Footprints had no library ID, value or symbol link: generated the project footprint library `Camera_Sub.pretty` from the placed footprints and filled in the library IDs, values, symbol links and schematic fields.
- Net names did not match the schematic: label nets now carry the `/` prefix and single-pad `Net_xx` nets use the schematic's `unconnected-(…)` names. Checked pad by pad against the schematic netlist.
- Two unreferenced Allegro drawing symbols are named MECH1/MECH2 and set to board-only.
- Constraints were KiCad defaults: restored the Allegro values (0.127 mm minimum track, 0.25 mm minimum drill).
- Netclasses assigned from the routed copper: `DP_CSI_A_*` (CSI differential pairs, 0.1778 mm), `CS_0` (ordinary signals, 0.127 mm), `POWER` (3V3/AVDD_2V8/DOVDD_1V8/GND, 0.635 mm, 0.762/0.5 vias). The pre-defined track, via and differential-pair size tables hold the sizes used on the board.
- GND pour: 0.1778 mm clearance with solid pad connection, refilled.

### Copyright and license

The design is copyright Espressif Systems (Shanghai) Co., Ltd. This repository **grants no license**; see [LICENSE](LICENSE). It is a non-commercial share for study and reference, is not affiliated with or endorsed by Espressif, and will be taken down if Espressif objects.
