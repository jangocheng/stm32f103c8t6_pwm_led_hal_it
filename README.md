#stm32f103c8t6_pwm_led 
=======================
中断的方式控制pwm。 <br>
使用的timer3, ch2,ch3,ch4   <br>


##注意事项：
------------
###给定pulse的值，设置占空比，看起来会呼吸
```java
/* Private variables ---------------------------------------------------------*/
const uint16_t MAX_PULSE=255;	
const uint8_t indexWave[]={1,1,2,2,3,4,6,8,10,14,19,25,33,44,59,80,107,143,191,255,
													 255,191,143,107,80,59,44,33,25,19,14,10,8,6,4,3,2,2,1,1};

/* USER CODE END PV */
```
###初始化，开启中断和使能
```java 
/* USER CODE BEGIN 2 */
HAL_TIM_Base_Start_IT(&htim3);
HAL_TIM_PWM_Start(&htim3,TIM_CHANNEL_2);
HAL_TIM_PWM_Start(&htim3,TIM_CHANNEL_3);
HAL_TIM_PWM_Start(&htim3,TIM_CHANNEL_4);
  /* USER CODE END 2 */
```

###timer3的初始化设置，注意分频：
```java
/* TIM3 init function */
static void MX_TIM3_Init(void)
{

  TIM_MasterConfigTypeDef sMasterConfig;
  TIM_OC_InitTypeDef sConfigOC;

  htim3.Instance = TIM3;
  htim3.Init.Prescaler = 2000-1;
  htim3.Init.CounterMode = TIM_COUNTERMODE_UP;
  htim3.Init.Period = MAX_PULSE;
  htim3.Init.ClockDivision = TIM_CLOCKDIVISION_DIV1;
  htim3.Init.AutoReloadPreload = TIM_AUTORELOAD_PRELOAD_DISABLE;
  if (HAL_TIM_PWM_Init(&htim3) != HAL_OK)
  {
    _Error_Handler(__FILE__, __LINE__);
  }

  sMasterConfig.MasterOutputTrigger = TIM_TRGO_RESET;
  sMasterConfig.MasterSlaveMode = TIM_MASTERSLAVEMODE_DISABLE;
  if (HAL_TIMEx_MasterConfigSynchronization(&htim3, &sMasterConfig) != HAL_OK)
  {
    _Error_Handler(__FILE__, __LINE__);
  }

  sConfigOC.OCMode = TIM_OCMODE_PWM1;
  sConfigOC.Pulse =  0;
  sConfigOC.OCPolarity = TIM_OCPOLARITY_HIGH;
  sConfigOC.OCFastMode = TIM_OCFAST_DISABLE;
  if (HAL_TIM_PWM_ConfigChannel(&htim3, &sConfigOC, TIM_CHANNEL_2) != HAL_OK)
  {
    _Error_Handler(__FILE__, __LINE__);
  }

  sConfigOC.Pulse =  0;
  if (HAL_TIM_PWM_ConfigChannel(&htim3, &sConfigOC, TIM_CHANNEL_3) != HAL_OK)
  {
    _Error_Handler(__FILE__, __LINE__);
  }

  sConfigOC.Pulse =  0;
  if (HAL_TIM_PWM_ConfigChannel(&htim3, &sConfigOC, TIM_CHANNEL_4) != HAL_OK)
  {
    _Error_Handler(__FILE__, __LINE__);
  }

  HAL_TIM_MspPostInit(&htim3);

}
```


###在中断函数里设置pwm
```java
/* USER CODE BEGIN 4 */
 void change_pwm_pulse(uint32_t channel,uint16_t value)
{
	if(value>MAX_PULSE){
		return;
	} 
 
 
 TIM_OC_InitTypeDef sConfigOC; 
    sConfigOC.OCMode = TIM_OCMODE_PWM1;
  sConfigOC.Pulse = value; 
	    sConfigOC.OCPolarity = TIM_OCPOLARITY_HIGH;
 
    sConfigOC.OCFastMode = TIM_OCFAST_DISABLE;
 
  if (HAL_TIM_PWM_ConfigChannel(&htim3, &sConfigOC, channel) != HAL_OK)
  {
    _Error_Handler(__FILE__, __LINE__);
  } 
 
	HAL_TIM_PWM_Start(&htim3, channel);
	 
	
}


void HAL_TIM_PeriodElapsedCallback(TIM_HandleTypeDef *htim){
	static uint8_t pwmIndex=1;
	static uint8_t periodCnt=0;
	periodCnt++;
	if(periodCnt>=20){
		__HAL_TIM_SET_COMPARE(htim,TIM_CHANNEL_2,indexWave[pwmIndex]);
		
		pwmIndex++;
		if(pwmIndex>=40){
			pwmIndex=0;
		}
		
		periodCnt=0;
	}
	 
	
}

/* USER CODE END 4 */

```

