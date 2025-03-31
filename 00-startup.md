# LicheePi 4A Startup

> 感谢甲辰计划RISC-V开发板随缘漂流计划，我申请到了LicheePi 4A 16GB RAM + 128GB eMMC，这里是[Github仓库](https://github.com/rv2036/riscv-board-wandering)。
>
> 本文主要记录如何进入黑窗口、烧录新系统等。

## 0-进入黑窗口

1. 查阅sipeed的[文档](https://wiki.sipeed.com/hardware/zh/lichee/th1520/lpi4a/2_unbox.html)，按照说明安装散热硅脂和散热风扇；
2. 将开发板上的 `VIN:12V`连接电源，此时开发板上的红灯亮起，风扇旋转；
3. 通过RV Debugger PLUS将模块的 `TX`连接开发板的 `U0-RX`，模块的 `RX`连接开发板的 `U0-TX`，模块的两个 `GND`连接开发板的两个 `GND`，然后将模块和笔记本连接，此时模块上的红灯亮起；
4. 在Windows中打开设备管理器，找到端口，查看是COM几；
5. 在MobaXterm中新建一个session是serial，Serial Port选择刚才查看到的COM，Speed选择115200，然后连接即可；
6. 此时就进入了LicheePi 4A的终端，可以通过debian用户名，debian密码进行登录。

![1743336380892](image/00-startup/1743336380892.png)

## 1-烧录新系统

这里以烧录OERV为例，新系统下载[来源](https://images.oerv.ac.cn/board?uri=products/sipeed/licheepi_4a.json&name=LicheePi+4A)，Windows驱动安装[参考](https://wiki.sipeed.com/hardware/zh/lichee/th1520/lpi4a/4_burn_image.html#Windows-%E4%B8%8B%E9%A9%B1%E5%8A%A8%E5%AE%89%E8%A3%85%28%E7%A6%81%E7%94%A8%E9%A9%B1%E5%8A%A8%E7%AD%BE%E5%90%8D%29)，也可以直接下载[这个](https://github.com/6eanut/temp/releases/tag/licheepi4a_oerv)。以上执行好之后，便可以开始烧录新系统了。

```shell
LPI4A_RAM_VARIANT='-16g'
OERV_VERSION='24.03-LTS'
zstd -d openEuler-${OERV_VERSION}-riscv64-lpi4a-base-boot.ext4.zst
zstd -d openEuler-${OERV_VERSION}-riscv64-lpi4a-base-root.ext4.zst
fastboot flash ram u-boot-with-spl-lpi4a${LPI4A_RAM_VARIANT}.bin
fastboot reboot
sleep 1
fastboot flash uboot u-boot-with-spl-lpi4a${LPI4A_RAM_VARIANT}.bin
fastboot flash boot openEuler-${OERV_VERSION}-riscv64-lpi4a-base-boot.ext4
fastboot flash root openEuler-${OERV_VERSION}-riscv64-lpi4a-base-root.ext4
```

然后重启开发板，以root用户登录，密码是openEuler12#$。

> 目前该镜像好像默认不提供wifi支持，所以在连接有线网之前，没法使用无线网。该问题已反馈到社区，等待回复。
