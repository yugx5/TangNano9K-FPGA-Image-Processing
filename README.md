# Tang Nano 9K FPGA 数字图像处理系统

本项目面向 FPGA 创新设计竞赛，基于 Sipeed Tang Nano 9K（Gowin GW1NR-9C）实现完整的数字图像处理与 HDMI 显示系统。输入图像经片上 ROM 解码后写入板载 PSRAM，并由 RTL 流水线完成灰度化、高斯滤波、Sobel 边缘检测等处理，最终以四分屏方式通过 HDMI 展示原图及处理结果。

![板上运行效果](docs/board_demo.jpg)

## 主要功能

- 纯 RTL 图像处理流水线
- RGB 转灰度、高斯滤波、Sobel 边缘检测和阈值化
- RLE 图像数据解码与片上 ROM 预存
- 板载 PSRAM 多图像平面缓存
- 640×480 HDMI 四分屏显示
- UART 状态输出和 LED 完成/错误指示
- ModelSim 仿真 Testbench 与参考测试数据
- Gowin EDA 工程、约束、实现报告及可直接下载的位流文件

## 硬件与工具

| 项目 | 说明 |
| --- | --- |
| 开发板 | Sipeed Tang Nano 9K |
| FPGA | Gowin GW1NR-LV9QN88PC6/I5 |
| 外设 | 板载 PSRAM、HDMI、USB/JTAG、USB-UART |
| 开发工具 | Gowin EDA |
| 仿真 | ModelSim |
| 视频输出 | 640×480 HDMI |
| 串口 | 115200 baud，8N1 |

## 系统数据流

```text
RLE ROM 图像数据
        │
        ▼
   RGB565 解码 ──► PSRAM 原图缓存
        │
        ▼
 RGB888/灰度化 ──► 高斯滤波 ──► Sobel ──► 阈值化
        │                         │
        └──────── PSRAM 多平面结果缓存 ────────┐
                                               ▼
                                    HDMI 四分屏显示
```

## 仓库结构

```text
.
├── FPGA_Image_Process.gprj   # Gowin EDA 工程入口
├── constraints/              # CST 引脚约束与 SDC 时序约束
├── firmware/                 # 可直接下载的 .fs 文件
├── image_data/               # 图像及 RLE/HEX 测试数据
├── src/
│   ├── algorithm/            # 灰度、高斯、Sobel、窗口与行缓存
│   ├── board_hdmi/           # HDMI/TMDS 与 UART
│   ├── board_psram/          # PSRAM 控制器
│   ├── gowin_prom_*/         # RLE ROM IP
│   ├── integration/          # 系统集成与顶层模块
│   └── storage/              # 帧缓存及结果写入逻辑
├── verification/
│   ├── tb/                   # Verilog Testbench
│   └── modelsim/             # ModelSim 运行脚本
├── reports/implementation/   # 综合、布局布线、功耗报告
└── docs/                     # 作品报告、验收报告、海报和实物照片
```

## 构建工程

1. 安装 Gowin EDA，并确认器件库中包含 `GW1NR-9C`。
2. 克隆本仓库。
3. 使用 Gowin EDA 打开 `FPGA_Image_Process.gprj`。
4. 检查顶层模块为 `nano9k_image_process_quad_v2_top`。
5. 执行综合、布局布线并生成 Bitstream。
6. 使用 Gowin Programmer 将生成的 `.fs` 文件下载至开发板。

如仅需复现实物效果，可直接使用 `firmware/FPGA_Image_Process.fs`：

1. 连接 Tang Nano 9K 与 HDMI 显示器。
2. 在 Gowin Programmer 中选择 SRAM 下载模式。
3. 下载固件并观察 HDMI 四分屏输出。
4. 可通过 USB-UART（115200，8N1）查看状态信息。

## 仿真

`verification/tb/` 提供三个主要 Testbench，`verification/modelsim/` 提供对应的 `.do` 脚本：

- `tb_rle_rom_full_frame.v`：完整 RLE ROM 帧验证
- `tb_quad_v2.v`：四分屏集成逻辑验证
- `tb_final_rle_psram_quad.v`：RLE、PSRAM 与四分屏完整链路验证

运行前请根据本机 ModelSim 安装路径调整脚本环境。仿真通过记录、算法验证过程和板上测试结果见 `docs/` 与 `reports/`。

## 文档与结果

- [竞赛作品报告](docs/competition_report.pdf)
- [上板测试验收报告](docs/hardware_validation_report.pdf)
- [项目海报](docs/project_poster.pdf)
- [Gowin Programmer 下载成功截图](docs/programming_success.png)
- [综合与实现报告](reports/implementation/)

## 第三方代码与工具使用说明

HDMI 输出部分包含 Simple Video Out（SVO）相关模块；Gowin PLL、PROM 等模块由 Gowin EDA/IP 工具生成。第三方模块的著作权和许可归原作者所有。本仓库仅用于作品审阅、教学研究与复现；如需再分发或用于其他用途，请核对相应上游许可。

项目开发、调试和文档整理过程中使用了自动化脚本及 AI 辅助工具。核心 RTL 设计、工程集成、仿真验证和板上测试均以仓库中的源码、报告与实测记录为准。

## 注意事项

- 不同 Gowin EDA 版本重新实现时，资源利用率和时序结果可能略有差异。
- HDMI 显示器需支持 640×480 时序。
- 本仓库未声明统一开源许可证；除已注明的第三方代码外，使用或再分发前请联系项目作者取得许可。

