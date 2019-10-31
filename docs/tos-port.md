# AWTK 在腾讯 TOS 上的移植笔记

本文以 STM32f103ze 为例，介绍了 AWTK 在 RTOS 上移植的经验。与其说移植，倒不如说是集成。因为 RTOS 通常没有提供标准的 LCD 驱动接口，显示部分并不需要特别的改动。所做的事情不过是把 AWTK 放到 RTOS 的一个线程中执行而已。

## 1. 加入 TOS 相关文件。

AWTK 已经移植到 STM32f103ze 裸系统上，为了简单起见，直接在 [awtk-stm32f103ze-raw](https://github.com/zlgopen/awtk-stm32f103ze-raw/) 基础上加入 TOS 支持。

* 在 Keil 中增加下列文件：

```
TencentOS/kernel/core/tos_event.c
TencentOS/kernel/core/tos_fifo.c
TencentOS/kernel/core/tos_global.c
TencentOS/kernel/core/tos_mmblk.c
TencentOS/kernel/core/tos_mmheap.c
TencentOS/kernel/core/tos_msg.c
TencentOS/kernel/core/tos_mutex.c
TencentOS/kernel/core/tos_pend.c
TencentOS/kernel/core/tos_queue.c
TencentOS/kernel/core/tos_robin.c
TencentOS/kernel/core/tos_sched.c
TencentOS/kernel/core/tos_sem.c
TencentOS/kernel/core/tos_sys.c
TencentOS/kernel/core/tos_task.c
TencentOS/kernel/core/tos_tick.c
TencentOS/kernel/core/tos_time.c
TencentOS/kernel/core/tos_timer.c
TencentOS/kernel/pm/tos_pm.c
TencentOS/kernel/pm/tos_tickless.c
TencentOS/arch/arm/arm-v7m/common/tos_cpu.c
TencentOS/arch/arm/arm-v7m/common/tos_fault.c
TencentOS/arch/arm/arm-v7m/cortex-m3/armcc/port_c.c
TencentOS/arch/arm/arm-v7m/cortex-m3/armcc/port_s.S
```

* 增加 include 的路径

```
TencentOS/arch/arm/arm-v7m/common/include
TencentOS/arch/arm/arm-v7m/cortex-m3/armcc
TencentOS/kernel/core/include
TencentOS/kernel/hal/include
TencentOS/kernel/pm/include
TencentOS/TOS-CONFIG
```

* 修改配置文件

根据自己的需要修改配置 TencentOS/TOS-CONFIG/tos_config.h：

> 一般来说不需要修改，使用官方提供的即可。我用的是 [TencentOS-Demo](https://github.com/jiejieTop/TencentOS-Demo) 项目中的。

```
#include "stm32f10x.h"
#include <stdio.h>

#define TOS_CFG_TASK_PRIO_MAX       10u

#define TOS_CFG_ROUND_ROBIN_EN      1u

#define TOS_CFG_OBJECT_VERIFY_EN       0u

#define TOS_CFG_EVENT_EN            1u

#define TOS_CFG_MMBLK_EN            1u

#define TOS_CFG_MMHEAP_EN           1u

#define TOS_CFG_MMHEAP_POOL_SIZE    0x3000

#define TOS_CFG_MUTEX_EN            1u

#define TOS_CFG_QUEUE_EN            1u

#define TOS_CFG_TIMER_EN            1u

#define TOS_CFG_SEM_EN              1u

#if (TOS_CFG_QUEUE_EN > 0u)
#define TOS_CFG_MSG_EN     1u
#else
#define TOS_CFG_MSG_EN     0u
#endif

#define TOS_CFG_VFS_EN                  1u

#define TOS_CFG_MSG_POOL_SIZE           10u

#define TOS_CFG_IDLE_TASK_STK_SIZE      80u

#define TOS_CFG_CPU_TICK_PER_SECOND     1000u

#define TOS_CFG_CPU_CLOCK               (SystemCoreClock)
```

## 2. 加入针对 TOS 实现的线程和同步的函数。

```
src/platforms/cmsis_os/mutex.c
src/platforms/cmsis_os/semaphore.c
src/platforms/cmsis_os/sys_tick.c
src/platforms/cmsis_os/thread.c
```

主要就是 SysTick 中断的实现，从 TencentOS-Demo 中拷贝过来就行了。

> 非 arm 平台 SysTick 函数的名称可能不一样，根据自己的需要调整。

```
static volatile uint64_t g_sys_tick;

void tos_tick(void) {
  if (tos_knl_is_running()) {
    tos_knl_irq_enter();
    tos_tick_handler();
    tos_knl_irq_leave();
  }
}

void SysTick_Handler(void) {
  g_sys_tick++;
  tos_tick();
}
```

> 这些实现目前还不完善，勉强可用吧，有空再来搞。

## 3. 在线程中启动 AWTK

```

void* awtk_thread(void* args) {
  gui_app_start(320, 480);

  return NULL;
}

static ret_t awtk_start_ui_thread(void) {
  tk_thread_t* ui_thread = tk_thread_create(awtk_thread, NULL);
  return_value_if_fail(ui_thread != NULL, RET_BAD_PARAMS);

  tk_thread_set_priority(ui_thread, 3);
  tk_thread_set_name(ui_thread, "awtk");
  tk_thread_set_stack_size(ui_thread, 2048);

  return tk_thread_start(ui_thread);
}

static void hardware_prepare(void) {
  NVIC_PriorityGroupConfig(NVIC_PriorityGroup_2);
  systick_init();
  led_init();
  button_init();
  TFT_Init();
  TFT_ClearScreen(BLACK);
  FLASH_Init();
  TOUCH_Init();

  TIM3_Init(50, 7199);
  rtc_init();
}

int main() {
  hardware_prepare();
  platform_prepare();

  tos_knl_init();
  tos_robin_config(TOS_ROBIN_STATE_ENABLED, (k_timeslice_t)500u);

  awtk_start_ui_thread();

  tos_knl_start();
}
```

这里与裸系统不同的地方，主要有两个：

* 1. 在线程中启动 AWTK。
* 2. 要提前调用 platform\_prepare，platform\_prepare 负责初始化内存，放在 tk_init 中就有些晚，需要单独提出来调用。

为此 platform\_prepare 函数做了防重复调用的处理。

```
static bool_t s_inited = FALSE;
static uint32_t s_heam_mem[4096];

ret_t platform_prepare(void) {
	if(!s_inited) {
		s_inited = TRUE;
    tk_mem_init(s_heam_mem, sizeof(s_heam_mem));
	}
  return RET_OK;
}
```

AWTK 集成 RTOS 是非常简单的，以上过程大概花了 2 个小时吧。只要 RTOS 本身好移植，集成 AWTK 和 RTOS 只是分分钟的问题。
