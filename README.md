# Home Assistant JK BMS BLE Integration (New Protocol Adaptive)

[![hacs_badge](https://img.shields.io/badge/HACS-Custom-41BDF5.svg)](https://github.com/hacs/integration)
[![Maintainer](https://img.shields.io/badge/maintainer-%40YourGitHubUsername-blue.svg)](https://github.com/YourGitHubUsername)

这是一个专为极控 (JiKong) BMS 开发的 Home Assistant 蓝牙集成插件。

### 🌟 核心突破：深度适配 2025/2026 新版 PB/PD 固件
目前市面上绝大多数极控集成（包括基于 Syssi 的老版实现）在面对极控新一代**逆变器/大容量保护板（型号如 JK-BD6A20S4PD, JK-PB 系列等）**时，由于厂家修改了 `0x02` 报文结构并插入了功率字段，会导致严重的偏移错误：
* **错误表现**：剩余电量显示 154% 或 0%、循环次数显示 21 亿次、总电压显示 80.8V、温度显示 0.0°C 或不可用。
* **本插件方案**：通过对报文的逐字节逆向工程，重新定义了 `JK_PB` 协议地图，实现了 **100% 对齐官方 App** 的遥测精度。

---

## 🚀 主要功能

目前只启用了sensor实体，其它的还有待适配，暂未完成。

* **全量遥测 (Sensors)**：
    * **电芯监控**：支持 8-32 串电芯独立电压监测（自动过滤未接串）。
    * **物理指标**：总电压 (mV 精度)、总电流、**实时功率 (硬件原生字段)**。
    * **电量统计**：剩余电量 (SOC %)、**电池总容量 (Ah)**、**剩余容量 (Ah)**、循环次数。
    * **温度矩阵**：MOS 管温度、电池探头温度 1 & 2 (精准剥离未接入探头的脏数据)。
    * **寿命预估**：基于实时电流的预计充满/耗尽时间。
* **实时控制 (Switches & Buttons)**：
    * 充电 MOS 开关、放电 MOS 开关、均衡开关。
    * BMS 系统一键重启。
* **诊断告警 (Binary Sensors)**：
    * 包括过压、欠压、过温、短路等 12 项保护状态监测。
* **参数配置 (Numbers)**：
    * 支持在 HA 界面直接修改电池总容量设置、OVP/UVP 保护阈值。
 


---

## 🛠️ 支持硬件

* **极控全系列 BLE BMS**：
    * **新协议版 (推荐)**：JK-PB 系列、JK-PD 系列、JK-BD / JK-B2A (V11.x 及以上固件)。
    * **经典版**：JK04, JK02 等老款 24S/32S 硬件。
* **连接方式**：支持蓝牙直连或通过 **ESP32 Bluetooth Proxy** 远程接入。

---

## 📦 安装方法

### 方法 1：HACS (推荐)
1.  打开 Home Assistant，进入 **HACS** 面板。
2.  点击右上角三个点 -> **自定义存储库 (Custom repositories)**。
3.  输入本项目 GitHub 地址，类别选择 **集成 (Integration)**。
4.  点击下载并重启 Home Assistant。

### 方法 2：手动安装
1.  下载本项目源代码。
2.  将 `custom_components/jk_bms_ble` 文件夹拷贝到你 HA 配置目录下的 `custom_components` 文件夹内。
3.  重启 Home Assistant。

---

## ⚙️ 配置与使用

1.  进入 **设置 -> 设备与服务 -> 添加集成**。
2.  搜索 `JK BMS BLE` 并输入保护板的 **蓝牙 MAC 地址**。
3.  **关键步骤**：若发现数据错位，请点击集成卡片上的 **选项 (Options)**，在协议版本下拉菜单中选择 **`JK_PB`**。

---

## 📈 逆向工程参考 (Technical Reference)
本项目通过分析极控 300 字节 `0x02` 报文，确定了新版固件的内存偏移量（Offset）：
| 数据项 | 偏移量 (Byte) | 格式 | 比例 |
| :--- | :--- | :--- | :--- |
| 总电压 (Voltage) | 150 | <I | 0.001 |
| 实时功率 (Power) | 154 | <i | 0.001 |
| 总电流 (Current) | 158 | <i | 0.001 |
| 电池温度 2 | 164 | <h | 0.1 |
| **剩余电量 (SOC)** | **173** | **<B** | **1** |
| 剩余容量 (Remain Ah) | 174 | <I | 0.001 |
| 总容量 (Total Ah) | 178 | <I | 0.001 |

---

## 📄 免责声明
本项目仅供技术交流。修改 BMS 参数存在风险，请在操作前确保了解锂电池安全知识。作者不对任何因使用本插件导致的硬件损坏或安全事故负责。

---

### 🤝 贡献
如果你发现数据依然存在偏差，请开启 Debug 日志，并提交包含 `RAW 0x02 Payload` 字段的 Issue。
