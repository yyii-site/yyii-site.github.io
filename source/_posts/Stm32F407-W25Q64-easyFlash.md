---
title: Stm32F407_W25Q64_easyFlash
date: 2021-11-11 23:52:18
tags: STM32
---

stm32 flash w25q64 easyFlash rt-thread studio

## 1.RT-Thread Studio

1. new project

![image-20211111213804740](image-20211111213804740.png)

## 2.CubeMX Settings

1. System Core >> RCC >> HSE >> Crystal(外部晶振)  
2. System Core >> SYS >> Debug(选择调试接口)   
3. Connectivity >> USART3 >> Asynchronous(用于finsh命令行)  
4. Connectivity >> SPI2 >> Full-Dupex Master(Flash W25Q64)

![image-20211111212434054](image-20211111212434054.png)

5. Clock Configuration(配置时钟)

   HCLK >> 168  >> Enter

![image-20211111212633234](image-20211111212633234.png)

6. GENERATE COED (default seting)

![image-20211111212753066](image-20211111212753066.png)

![image-20210630224307585](image-20210630224307585.png)

## 3.RT-Thread Setting

1. Packages >> add >> EasyFlash

![image-20211111234323900](image-20211111234323900.png)

2. Drivers >> SPI

3. Drivers >> SPI >> SFUD

   ![image-20211111234429187](image-20211111234429187.png)

## 4.Code

1. *drivers/board.h*

```c
#define BSP_USING_UART3
#define BSP_UART3_TX_PIN       "PD8"
#define BSP_UART3_RX_PIN       "PD9"

#define BSP_USING_SPI2
```

2. *new source file*

   applications/usr_flash.c   (change cs pin and bus name)

```c
#include <rtthread.h>
#include <drv_spi.h>
#include "spi_flash.h"
#include "spi_flash_sfud.h"

rt_spi_flash_device_t w25q64;

static int rt_hw_spi_flash_with_sfud_init(void)
{
    rt_hw_spi_device_attach("spi2", "spi20", GPIOE, GPIO_PIN_3);

    /* init w25q64 */
    if (w25q64 == rt_sfud_flash_probe("w25q64", "spi20"))
    {
        return -RT_ERROR;
    }

    return RT_EOK;
}
INIT_COMPONENT_EXPORT(rt_hw_spi_flash_with_sfud_init);
```

3. copy file 

   packages\EasyFlash-v4.1.0\ports\ef_sfud_port.c  >>  applications/usr_flash.c

## 5.Build && Test

   ![image-20211112221002533](image-20211112221002533.png)

