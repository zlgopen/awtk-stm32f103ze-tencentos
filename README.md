# AWTK针对腾讯TOS(STM32f103ze)的移植。

[AWTK](https://github.com/zlgopen/awtk)是为嵌入式系统开发的GUI引擎库。

[awtk-stm32f103ze-tencentos](https://github.com/zlgopen/awtk-stm32f103ze-tencentos)是AWTK在STM32f103ze上的移植。

本项目以[普中科技STM32F103ZET6开发实验板](https://item.taobao.com/item.htm?spm=a230r.1.14.1.50a130e8TMKYMC&id=558855281660&ns=1&abbucket=5#detail) 为载体移植，其它开发板可能要做些修改，有问题请请创建issue。

## 编译

1. 获取源码

```
git clone https://github.com/zlgopen/awtk-stm32f103ze-tencentos.git
cd awtk-stm32f103ze-tencentos
git clone https://github.com/zlgopen/awtk.git
```

2. 用keil打开user/awtk.uvproj

> 本项目目前可以运行，但还处于实验阶段。
