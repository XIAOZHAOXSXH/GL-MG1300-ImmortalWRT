# GL.iNet GL-MG1300 ImmortalWRT

这是 GL.iNet GL-MG1300 的 ImmortalWRT 外测移植仓库。仓库只保存设备补丁、构建配置和 GitHub Actions；Actions 会在构建时以 `v25.12.1` 克隆 ImmortalWRT，因此不会把上游历史或其他作者带入本仓库。

## 已核验硬件

- MediaTek MT7621 ver:1 eco:3，128 MiB DDR3L，目标 `ramips/mt7621`
- 2 MiB SPI-NOR + 128 MiB SPI-NAND，原厂系统 OpenWrt 22.03.4，内核 5.10.176
- NOR 25 MHz，NAND 50 MHz；NAND 擦除块 128 KiB，整片 UBI
- 一个千兆 LAN 和一个千兆 WAN，WAN MAC 位于 `factory` 偏移 `0x4000`，LAN 为加 1
- MT7615 PCIe 双频 DBDC，校准数据来自 `factory` 分区起始位置
- GPIO12 为 USB 供电，GPIO13/14 为白色 system 和蓝色 run，GPIO16 为模式拨动开关，GPIO18 为 reset
- 设备没有 eMMC 或 SD：SDHCI 在原厂设备树中禁用，运行系统也没有 MMC 主机或块设备

## GitHub Actions

推送到 `main` 或手动运行 `Build ImmortalWRT GL-MG1300` 后，工作流会：

1. 克隆 ImmortalWRT `v25.12.1`
2. 应用 `patches/0001-ramips-add-glinet-gl-mg1300.patch`
3. 合并 `files/` 覆盖文件和 `configs/gl-mg1300.config`
4. 编译 `factory.bin` 与 `sysupgrade.bin`
5. 上传 Actions Artifact，并创建带 SHA256SUMS 的 GitHub Release

GitHub-hosted runner 无法访问你电脑的 `127.0.0.1:7890`。如使用自托管 runner 或可从 runner 访问的代理，可设置仓库变量 `BUILD_HTTP_PROXY`、`BUILD_HTTPS_PROXY`；不要把本机地址硬编码到工作流中。

## 刷写注意

这是外测移植，不能把“编译成功”当作“已经验证可以刷写”。第一次测试前应保留 SPI-NOR 和 SPI-NAND 的本地备份，并准备串口或原厂恢复路径。Release 中的 factory 镜像尚未在所有恢复路径上验证；sysupgrade 镜像只应从兼容的 ImmortalWRT 系统使用。仓库不提供未经核验的 `mtkupgrade` 命令或 eMMC 操作。

首次启动沿用原厂管理地址 `192.168.8.1`，并将主机名设为 `GL-MG1300`。USB 供电 GPIO 已按原厂导出，但原厂专有 `usb-control` 驱动没有被伪造移植，USB 过流控制等行为需要实机测试。

设备审计中的原始 MTD、factory、UBI、DTB 和 SSH 文件保存在本地 `device-audit-agent/`，该目录已加入 `.gitignore`，不会进入提交或 Release。
