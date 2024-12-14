---
title: Stm32F407_PWM_Master_Slave
date: 2022-01-18 21:32:02
tags: STM32
---

## 1.概述

使用STM32主定时器+从定时器实现产生可控数量的PWM脉冲

## 2.创建工程

1. new progect 

2. CubeMX Setting

  * RCC->Crystal

  * SYS->Serial Wire

  * Clock Configuration->168

  * TIM4

    ![image-20220118201958020](image-20220118201958020.png)

    ![image-20220118202025867](image-20220118202025867.png)

    ![image-20220118202048425](image-20220118202048425.png)

  * TIM2

    ![image-20220118202135289](image-20220118202135289.png)

    ![image-20220118202142804](image-20220118202142804.png)

    ![image-20220118202148330](image-20220118202148330.png)

    ![image-20220118203457549](image-20220118203457549.png)

  * Save

## 3.Code

---

1. main.c

   ```c
   /* USER CODE BEGIN 0 */
   void generate_pulse(uint32_t value)
   {
   	if(!value)
   		return;
   	__HAL_TIM_SET_AUTORELOAD(&htim2,value-1); //设置要输出的PWM脉冲
   	HAL_TIM_Base_Start_IT(&htim2);             //启动从定时器
   	__HAL_TIM_SET_COUNTER(&htim4, 0);
   	HAL_TIM_PWM_Start(&htim4, TIM_CHANNEL_2);  //启动主定时器PWM输出
   }
   
   void  HAL_TIM_PeriodElapsedCallback(TIM_HandleTypeDef *htim)
   {
       if(htim == &htim2)
       {
           if(__HAL_TIM_GET_FLAG(&htim2, TIM_FLAG_CC2) != RESET)  //判断是否触发中断
           {
               __HAL_TIM_CLEAR_FLAG(&htim2, TIM_FLAG_CC2);      //清除中断标志
               HAL_TIM_PWM_Stop(&htim4, TIM_CHANNEL_2);   //关闭主定时器
               HAL_TIM_Base_Stop_IT(&htim2);         //关闭从定时器
           }
       }
   }
   /* USER CODE END 0 */
   
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
     MX_TIM2_Init();
     MX_TIM4_Init();
     /* USER CODE BEGIN 2 */
   
     /* USER CODE END 2 */
   
     /* Infinite loop */
     /* USER CODE BEGIN WHILE */
     while (1)
     {
       /* USER CODE END WHILE */
   
       /* USER CODE BEGIN 3 */
   	generate_pulse(10);
   	HAL_Delay(1000);
     }
     /* USER CODE END 3 */
   }
   ```

   ## 4.Test

   ![image-20220118202840364](image-20220118202840364.png)