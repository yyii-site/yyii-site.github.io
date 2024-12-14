---
title: Stm32F407_PWM_External_Sync
date: 2022-01-18 21:42:09
tags: STM32
---

## 1.PWM与外部触发信号同步

---

1. new progect

2. CubeMX Setting

   * RCC->Cystal

   * SYS->Serial Wire

   * Clock Configuration->168

   * TIM1

     ![image-20220118205730219](image-20220118205730219.png)

     ![image-20220118205737972](image-20220118205737972.png)

     ![image-20220118205747601](image-20220118205747601.png)

     ![image-20220118205752456](image-20220118205752456.png)

     ![image-20220118205757553](image-20220118205757553.png)

   * Save

## 2.Code

---

```c
int main(void)
{
  /* USER CODE BEGIN 1 */

  /* USER CODE END 1 */

  /* MCU Configuration--------------------------------------------------------*/

  /* Reset of all peripherals, Initializes the Flash interface and the Systick. */
  HAL_Init();

  /* USER CODE BEGIN Init */

  /* USER CODE END Init */

  /* Configure the system clock */
  SystemClock_Config();

  /* USER CODE BEGIN SysInit */

  /* USER CODE END SysInit */

  /* Initialize all configured peripherals */
  MX_GPIO_Init();
  MX_TIM1_Init();
  /* USER CODE BEGIN 2 */
  HAL_TIM_PWM_Start(&htim1, TIM_CHANNEL_1);
  HAL_TIM_PWM_Start(&htim1, TIM_CHANNEL_2);
  HAL_TIM_PWM_Start(&htim1, TIM_CHANNEL_3);
  HAL_TIM_PWM_Start(&htim1, TIM_CHANNEL_4);
  /* USER CODE END 2 */

  /* Infinite loop */
  /* USER CODE BEGIN WHILE */
  while (1)
  {
    /* USER CODE END WHILE */

    /* USER CODE BEGIN 3 */
  }
  /* USER CODE END 3 */
}
```


## 3.Test

---

![image-20220118212029035](image-20220118212029035.png)

![image-20220118212038552](image-20220118212038552.png)

