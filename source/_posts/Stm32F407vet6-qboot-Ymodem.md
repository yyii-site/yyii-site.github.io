---
title: Stm32F407vet6_qboot_Ymodem
date: 2022-03-05 22:22:41
tags: STM32
---

stm32f407vet6 qboot Ymodem PuTTY

## 1.qboot project

---

1. enable onchip and w25qxx flash

2. add qboot package

![image-20220305212321866](image-20220305212321866.png)

3. build download

![image-20220305212353411](image-20220305212353411.png)



## 2.app project

---

1. enable onchip and w25qxx flash

2. add ota_downloader

   ![image-20220305213051752](image-20220305213051752.png)

3. edit link.lds

   0x20000 = 128k          384 = 512-128

   ![image-20220305213650986](image-20220305213650986.png)

3. build and download

   qboot jump app

   ![image-20220305214540437](image-20220305214540437.png)

4. change main.c

```c
#include <rtthread.h>
#include <board.h>

#define DBG_TAG "main"
#define DBG_LVL DBG_LOG
#include <rtdbg.h>

int main(void)
{
    int count = 1;

    while (count++)
    {
        //LOG_D("Hello RT-Thread!");
        rt_thread_mdelay(1000);
    }

    return RT_EOK;
}

#define RT_APP_PART_ADDR    0x08020000
#define NVIC_VTOR_MASK      0xFFFFFF80
static int app_vtor__reconfig(void)
{
    SCB->VTOR = RT_APP_PART_ADDR & NVIC_VTOR_MASK;

    return 0;
}
INIT_BOARD_EXPORT(app_vtor__reconfig);

```

6. build download

   qboot jump app success

   ![image-20220305215300979](image-20220305215300979.png)

7. **\packages\ota_downloader-latest\tools\ota_packager\rt_ota_packaging_tool.exe**

   ![image-20220305215722299](image-20220305215722299.png)

8. ExtraPuTTY (PuTTY Session Manager)

   ![image-20220305220535007](image-20220305220535007.png)

9. Ymodem Send

   ![image-20220305220451993](image-20220305220451993.png)

10. System restart

    Qboot erase partition app

    Release firmware from download to app

    Jump to applicaton 

    ![image-20220305220906216](image-20220305220906216.png)
