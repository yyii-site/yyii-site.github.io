---
title: Stm32F407-ADC-DMA
date: 2021-11-12 23:40:27
tags: STM32
---

stm32 ADC DMA rt-thread studio

## 1.Rt-Thrad Studio

---

![image-20211112222940162](image-20211112222940162.png)

## 2.CubeMX Settings

---

1. System Core >> RCC >> HSE >> Crystal(外部晶振)
2. System Core >> SYS >> Debug(选择调试接口)
3. Connectivity >> USART3 >> Asynchronous(用于finsh命令行)
4. Analog >> ADC1 >>

Parameter Settings

![image-20211112224235969](image-20211112224235969.png)

![image-20211112224811725](image-20211112224811725.png)

5. Analog >> ADC1 >> DMA Settings

![image-20211112233043884](image-20211112233043884.png)

6. Clock Configuration(配置时钟)  HCLK >> 168 >> Enter

![image-20211111212633234](image-20211111212633234.png)

7. GENERATE COED (default seting)

![image-20211112225130880](image-20211112225130880.png)

## Code

---

1. applications/main.c

```c
#include <rtthread.h>

#define DBG_TAG "main"
#define DBG_LVL DBG_LOG
#include <rtdbg.h>

extern void usr_board_init(void);
extern uint16_t read_adc_value(uint8_t channel);

int main(void)
{
    int count = 1;

    usr_board_init();

    while (count++)
    {
        LOG_D("Hello RT-Thread!");
        rt_kprintf("PA3_adc = %d   ", read_adc_value(0));
        rt_kprintf("PA4_adc = %d   ", read_adc_value(1));
        rt_kprintf("PA5_adc = %d   ", read_adc_value(2));
        rt_kprintf("PA6_adc = %d   ", read_adc_value(3));
        rt_kprintf("\n");
        rt_thread_mdelay(1000);
    }

    return RT_EOK;
}
```

2. cubemx/Src/main.c

```c
/* USER CODE BEGIN 0 */

#define ADC_CHANNEL_SUM         4
#define ADC_BUFF_LOOP           30
uint16_t adc_dma_value[ADC_CHANNEL_SUM * ADC_BUFF_LOOP];

void usr_board_init(void)
{
    MX_DMA_Init();
    MX_ADC1_Init();
    HAL_ADC_Start_DMA(&hadc1, (uint32_t*)&adc_dma_value, ADC_CHANNEL_SUM * ADC_BUFF_LOOP);
}

uint16_t read_adc_value(uint8_t channel)
{
    if (channel < ADC_CHANNEL_SUM) {
        return adc_dma_value[channel];
    }
    return 0;
}
/* USER CODE END 0 */

/**
  * Enable DMA controller clock
  */
static void MX_DMA_Init(void)
{

  /* DMA controller clock enable */
  __HAL_RCC_DMA2_CLK_ENABLE();

  /* DMA interrupt init */
  /* DMA2_Stream0_IRQn interrupt configuration */
  //HAL_NVIC_SetPriority(DMA2_Stream0_IRQn, 0, 0);
  //HAL_NVIC_EnableIRQ(DMA2_Stream0_IRQn);

}
```

## 3.Build && Test

PA3_adc

![image-20211112233634255](image-20211112233634255.png)

---
