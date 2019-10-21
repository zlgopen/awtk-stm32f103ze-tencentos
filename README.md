# AWTK 针对腾讯 OS(STM32f103ze) 的移植。

* [AWTK](https://github.com/zlgopen/awtk) 全称 Toolkit AnyWhere，是 [ZLG](http://www.zlg.cn/) 开发的开源 GUI 引擎，旨在为嵌入式系统、WEB、各种小程序、手机和 PC 打造的通用 GUI 引擎，为用户提供一个功能强大、高效可靠、简单易用、可轻松做出炫酷效果的 GUI 引擎。

* [腾讯 TOS](https://github.com/Tencent/TencentOS-tiny) 是腾讯面向物联网领域开发的实时操作系统，具有低功耗，低资源占用，模块化，安全可靠等特点，可有效提升物联网终端产品开发效率。TencentOS tiny 提供精简的 RTOS 内核，内核组件可裁剪可配置，可快速移植到多种主流 MCU （如 STM32 全系列）及模组芯片上。而且，基于 RTOS 内核提供了丰富的物联网组件，内部集成主流物联网协议栈（如 CoAP/MQTT/TLS/DTLS/LoRaWAN/NB-IoT 等），可助力物联网终端设备及业务快速接入腾讯云物联网平台。

* [awtk-stm32f103ze-tencentos](https://github.com/zlgopen/awtk-stm32f103ze-tencentos) 是 AWTK 在 [腾讯 TOS](https://github.com/Tencent/TencentOS-tiny) 上的移植。

> 本项目以 [普中科技 STM32F103ZET6 开发实验板](https://item.taobao.com/item.htm?spm=a230r.1.14.1.50a130e8TMKYMC&id=558855281660&ns=1&abbucket=5#detail) 为载体移植，其它开发板可能要做些修改，有问题请请创建 issue。

## 编译

1. 获取源码

```
git clone https://github.com/zlgopen/awtk-stm32f103ze-tencentos.git
cd awtk-stm32f103ze-tencentos
git clone https://github.com/zlgopen/awtk.git
```

2. 用 keil 打开 user/awtk.uvproj

> 本项目目前可以运行，但还处于实验阶段。

## 文档

[AWTK 在腾讯 TOS 上的移植笔记]( docs/tos-port.md)