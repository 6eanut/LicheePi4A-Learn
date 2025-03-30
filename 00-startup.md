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
