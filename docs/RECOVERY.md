# 测试与恢复边界

当前移植属于外测阶段。刷写前必须保留原厂 SPI-NOR 和 SPI-NAND 备份，并准备串口、U-Boot 或原厂恢复路径。不要根据文件名推断 bootloader 的刷写格式，也不要在没有实际验证的情况下把 factory 镜像写入 NOR。

`sysupgrade.bin` 只用于已经运行兼容 ImmortalWRT 设备树和 NAND UBI 布局的系统。原厂系统到本移植的第一次切换应优先使用可恢复方式，并记录串口输出、分区识别、UBI 卷和网络 MAC。任何 USB 供电或模式开关异常都应先回到原厂备份验证。
