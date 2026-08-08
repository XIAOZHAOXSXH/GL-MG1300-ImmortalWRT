# GL-MG1300 移植记录

## 来源

设备通过只读 SSH 审计确认，原厂版本为 OpenWrt 22.03.4 r20123-38ccc47687，内核 5.10.176，设备树 compatible 为 `glinet,gl-mg1300`。运行中的 DTB 与 UBI kernel volume 内嵌 DTB 的 SHA256 一致，作为本移植的硬件依据。

## Flash 布局

| 芯片 | MTD | 偏移 | 大小 | 标签 |
| --- | --- | --- | --- | --- |
| SPI-NOR | mtd0 | `0x000000` | `0x080000` | `u-boot` |
| SPI-NOR | mtd1 | `0x080000` | `0x010000` | `u-boot-env` |
| SPI-NOR | mtd2 | `0x090000` | `0x010000` | `factory` |
| SPI-NOR | mtd3 | `0x0a0000` | `0x040000` | `log` |
| SPI-NOR | mtd4 | `0x0e0000` | `0x020000` | `CFG` |
| SPI-NAND | mtd5 | whole chip | `0x08000000` | `ubi` |

SPI-NOR 是 Eon EN25QE16A，25 MHz；SPI-NAND 是 Macronix，128 MiB、128 KiB 擦除块、2048 字节页，50 MHz。原厂 UBI 有 `kernel`、`rootfs` 和 `rootfs_data` 卷。镜像定义保留 4 MiB kernel 预算，并使用 120832 KiB 的保守可用镜像大小。

## 设备树决策

- NOR 第一分区使用真实标签 `u-boot`，不是参考文件中的 `fip`。
- 保留原厂 NOR/NAND 总线频率 25/50 MHz。
- 使用原厂 `mediatek,mtd-eeprom = <&factory 0x0>` 和 `dbdc` 描述 MT7615 双频校准；没有添加不存在的第二校准分区。
- Ethernet 使用 factory `0x4000`：WAN 为基址，LAN 为基址加 1。
- LAN/WAN 使用交换机 port@2/port@4；原厂没有第二个物理 LAN 口。
- GPIO16 保留为 EV_SW 模式开关，GPIO18 为 KEY_RESTART；GPIO13/14 为 active-low LED，run LED 默认点亮。
- SDHCI 保持默认禁用，不添加 `kmod-mmc-mtk`。设备审计没有发现 MMC 主机、mmcblk 或 eMMC。
- GPIO12 作为 USB 供电导出；原厂 `usb-control` 是专有驱动接口，未伪造为主线设备。
- 升级镜像检查阶段保留原厂 `vm.min_free_kbytes=1024` 设置，降低 128 MiB 内存设备升级时的内存压力。

## 产物

工作流会只收集 `ramips/mt7621` 下的 MG1300 factory/sysupgrade 镜像、`BUILD_INFO.txt` 和 `SHA256SUMS`。原始 factory、CFG、UBI、U-Boot 和完整设备树不进入仓库或 Release，因为其中包含设备唯一数据。
