# CMCC RAX3000M EMMC 的Openwrt编译次仓库
## 👉[主仓库](https://github.com/PlanetEditorX/Actions-OpenWrt-RAX3000M-EMMC)
**闭源驱动** | [开源驱动](README_mainline.md)

# Actions-rax3000m-emmc
使用 GitHub Actions 在线编译定制 CMCC RAX3000M eMMC version 的 immortalwrt-mt798x 固件

## 固件特性
使用 [hanwckf](https://github.com/hanwckf) 大佬的 [immortalwrt-mt798x](https://github.com/hanwckf/immortalwrt-mt798x) 项目仓库，'openwrt-21.02' 分支源码编译，无线使用 mtwifi 原厂无线驱动，内核版本 5.4.x

项目详情：[immortalwrt-mt798x项目介绍](https://cmi.hanwckf.top/p/immortalwrt-mt798x)

固件默认选中软件包
`f2fsck, losetup, mkf2fs, kmod-fs-f2fs, kmod-mmc, luci-app-ksmbd, luci-i18n-ksmbd-zh-cn, ksmbd-utils` 等

添加集成软件包
`cfdisk, htop, openssl-util, kmod-fuse, kmod-usb-net-ipheth, kmod-usb-net-rndis, luci-app-argon-config, luci-app-autoreboot, luci-app-diskman, luci-app-ksmbd, luci-app-nlbwmon, luci-app-openclash, luci-app-openvpn, luci-app-ttyd, luci-app-upnp, luci-app-usb-printer, luci-app-zerotier, luci-theme-argon`
等并预置 openclash 内核

加入由 [1715173329 天灵](https://github.com/1715173329) 使用 js 重写，[237大佬](https://www.right.com.cn/forum/?364126) 适配硬件 QoS 的 [luci-app-eqos-mtk](https://github.com/padavanonly/immortalwrt-mt798x/commit/7c8019ce4bcb1a79c01c517b62e49f059ca70049)

## 使用说明
在 Actions 选择该工作流手动点击 Run workflow 执行编译，等待固件编译完成上传至 releases 发布即可下载

### 配置说明
- 默认 LAN IP 已更改为 `192.168.6.1`，可在 `scripts/diy.sh` 处修改

- 需要取消集成或添加其他软件包可在 `configs/rax3000m-emmc_mtksdk.config` 处参考注释内容自行修改或添加配置选项

- 默认构建使用 OpenWrt 原生 luci 无线控制界面，如需使用 MTK SDK 无线控制界面 (luci-app-mtk) 请在 Run workflow 时取消勾选 “Use mtwifi-cfg”，或在 workflow 配置文件中将 `USE_MTWIFI_CFG` 中 `default: true` 的 true 改为 false，重新编译刷入使用

- 默认构建 eeprom 替换为 H3C NX30 Pro 提取版本（来自 [237大佬](https://www.right.com.cn/forum/?364126) 提取）以增大无线功率，**原厂 eeprom 无线信号 2.4G: 23dBm, 5G: 22dBm；替换 nx30pro_eeprom 后 2.4G: 25dBm, 5G: 24dBm**。如需恢复使用默认 eeprom 请在 Run workflow 时取消勾选 “Use nx30pro eeprom”，或在 workflow 配置文件中将 `USE_NX30PRO_EEPROM` 中 `default: true` 的 true 改为 false，重新编译刷入使用  
  > eMMC 设备读取 eeprom 方式与 nand 闪存设备不同，所以修改方式也不同。其他设备包括 CMCC RAX3000M nand 版本若希望使用高功率 eeprom 可直接改用 237 的 https://github.com/padavanonly/immortalwrt-mt798x 仓库源码编译，或参考 https://github.com/padavanonly/immortalwrt-mt798x/commit/32ccfa4af7cd8eb6193c0394a06f5d32ac49c1f7 进行修改

- 默认使用 26 MHz 闪存频率，52 MHz 下闪存读写速率相较 26 MHz 下翻倍，可达到 40MB/s 以上，**但部分机器因闪存体质差异，使用 52 MHz 闪存频率固件可能会出现 I/O 报错 (`dmesg | grep error` 查看)，无法正常使用，甚至可能无法启动**，你可以在 [Releases](https://github.com/AngelaCooljx/Actions-rax3000m-emmc/releases) 处查找不同闪存频率固件。自行构建 52 MHz 版本需要在 Run workflow 时勾选 “Use 52MHz max-frequency”，或在 workflow 配置文件中将 `USE_52MHZ` 中 `default: false` 的 false 改为 true，重新编译刷入使用

## 如何刷入
参考 https://t.me/nanopi_r2s/637 刷入单分区版 GPT BL2 FIP, 再通过 custom U-Boot 刷写 sysupgrade.bin 固件
> 已增加 CMCC RAX3000M eMMC 版 U-Boot，GPT BL2 FIP 刷入方式如下：
> ```
> dd if=mt7981-cmcc_rax3000m-emmc-gpt.bin of=/dev/mmcblk0 bs=512 seek=0 count=34 conv=fsync
> echo 0 > /sys/block/mmcblk0boot0/force_ro
> dd if=/dev/zero of=/dev/mmcblk0boot0 bs=512 count=8192 conv=fsync
> dd if=mt7981-cmcc_rax3000m-emmc-bl2.bin of=/dev/mmcblk0boot0 bs=512 conv=fsync
> dd if=/dev/zero of=/dev/mmcblk0 bs=512 seek=13312 count=8192 conv=fsync
> dd if=mt7981-cmcc_rax3000m-emmc-fip.bin of=/dev/mmcblk0 bs=512 seek=13312 conv=fsync
> ```
> 对应 ImmortalWrt CMCC RAX3000M eMMC version (custom U-Boot layout)、Q-WRT、及其他 eMMC 单分区版固件。

~~路由器进入 uboot 需要手动设置本机 IP 192.168.1.100 网关 192.168.1.1 DNS 192.168.1.1，~~ 新版 custom U-Boot 已支持 DHCP，浏览器输入 `192.168.1.1` 进入 Web-UI 刷写固件，所有文件可在 https://firmware.download.immortalwrt.eu.org/uboot/mediatek 获取

## 注意事项
此分区布局默认不创建 eMMC 闪存最后一块 56G 大分区，你需要使用 `cfdisk /dev/mmcblk0` 为最后一块剩余空闲容量手动创建 /dev/mmcblk0p7 分区并通过 mkfs.ext4 格式化以挂载使用，此后更新刷入其他固件则无需再进行相同操作，固件可以自动挂载

## Credits
- [XiaoBinin/Actions-immortalwrt](https://github.com/XiaoBinin/Actions-immortalwrt)
- [ImmortalWrt](https://github.com/immortalwrt/immortalwrt)
- [hanwckf/immortalwrt-mt798x](https://github.com/hanwckf/immortalwrt-mt798x)
- [lgs2007m/immortalwrt-mt798x-rax3000m-emmc](https://github.com/lgs2007m/immortalwrt-mt798x-rax3000m-emmc)
- [GL-iNet](https://github.com/gl-inet)
- [padavanonly](https://github.com/padavanonly)
- [Microsoft Azure](https://azure.microsoft.com)
- [GitHub Actions](https://github.com/features/actions)
- [OpenWrt](https://github.com/openwrt/openwrt)
- [tmate](https://github.com/tmate-io/tmate)
- [mxschmitt/action-tmate](https://github.com/mxschmitt/action-tmate)
- [csexton/debugger-action](https://github.com/csexton/debugger-action)
- [Cowtransfer](https://cowtransfer.com)
- [WeTransfer](https://wetransfer.com/)
- [Mikubill/transfer](https://github.com/Mikubill/transfer)
- [softprops/action-gh-release](https://github.com/softprops/action-gh-release)
- [ActionsRML/delete-workflow-runs](https://github.com/ActionsRML/delete-workflow-runs)
- [dev-drprasad/delete-older-releases](https://github.com/dev-drprasad/delete-older-releases)
- [peter-evans/repository-dispatch](https://github.com/peter-evans/repository-dispatch)

## License

[MIT](https://github.com/P3TERX/Actions-OpenWrt/blob/main/LICENSE) © [**P3TERX**](https://p3terx.com)
