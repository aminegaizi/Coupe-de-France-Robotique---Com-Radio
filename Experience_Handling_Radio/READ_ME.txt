READ ME : Experience handling project

On start-up:
	-> USART1 is initialized with interrupts in reception
	-> USART2 gives information to serial monitor regargind Radio Module
	-> SPI1 is initialized with NRF24
	-> Red LED is turned ON 
	-> Green LED is turned OFF

TIM1 generates a PWM of 10kHz frequency: Global clock at 84MHz, Prescaler at 84 and period at 100 ticks; 84MHz/Prescaler = 1Mhz; 1MHz/period = 10kHz -> PWM frequency.
	Output on TIM1 CH1 Port A pin 8 (PA8); Change the duty cycle by changing the value of "Pulse", by default it is set at 50%

TIM2 interrupts every second and increments a global variable counter. 
	Global clock at 84MHz, Prescaler at 42000 and period at 2000 ticks; 84MHz/Prescaler = 2kHz; 2MHz/period = 1 Hz -> Interrupt frequency.
	TIM2 interrupt handling in function "void HAL_TIM_PeriodElapsedCallback(TIM_HandleTypeDef *htim)" main.c file

	In the interrupt: 
		Check for Bluetooth_EXP_Enable AND Radio_EXP_Enable flags and the value of a counter.
			If flags are set to 1 and the counter's (which counts how much time the Experience has been enabled) value is less than the threshold
			-Red LED turns off
			-Green LED turns on
			-PWM is started
		If flags are not set or counter's value is superior to the threshold
			-Red LED turns on
			-Green LED turns off
			-PWM is stopped
GPIO:
	Red LED to signal: experience in not started or is stopped at pin PB6, label: RedLED
	Green LED to signal: experience is started or ongoing pin PB5, label GreenLED


USART1 working with HC05 module in slave mode, baudrate 115200 (MAC: 98d3,32,115105)
	Interrupts at every reception, interrupt handling in function "void HAL_UART_RxCpltCallback(UART_HandleTypeDef *huarts)" main.c file. 

	When an 'S' (for Start) is received, and counter < Threshold:
		-Sets the Bluetooth EXP enable flag

	When a 'E' (for End) is received:
		-Resets the Bluetooth EXP enable flag

TO implement in Master project: 

	USART Asynchronous mode, 115200 baudrate. Connect Master bluetooth device
	Use the "HAL_StatusTypeDef HAL_UART_Transmit(UART_HandleTypeDef *huart, uint8_t *pData, uint16_t Size, uint32_t Timeout)" function to send the instructions:
		- 'S' to start the experience
		- 'E' to stop the experience. 

	SPI1 Full-Duplex Master with Baudrate Prescaler to 32 (So SPI clockage doesn't exceed 2MHz)


	Example: 

	UART_HandleTypeDef huart1;
	UART_HandleTypeDef huart2;
	SPI_HandleTypeDef hspi1;


	char* StartTX = "S";
	char* EndTX = "E";
	uint64_t TxpipeAddrs = 0x11223344AA;
	char myTxData[32] = "ENABLE EXPERIENCE";
	char RadioDisable[32] = "DISABLE EXPERIENCE";

	static void MX_GPIO_Init(void);
	static void MX_USART1_UART_Init(void);
	static void MX_SPI1_Init(void);
	static void MX_USART2_UART_Init(void);

	MX_GPIO_Init();
 	MX_SPI1_Init();
  	MX_USART2_UART_Init();
  	MX_USART1_UART_Init();


  	NRF24_stopListening();
  	NRF24_openWritingPipe(TxpipeAddrs);

  	NRF24_setAutoAck(false);
  	NRF24_setChannel(52);
  	NRF24_setPayloadSize(32);


	NRF24_write(myTxData, 32);

  	if(NRF24_write(myTxData, 32))
  	{
	  HAL_UART_Transmit(&huart2, (uint8_t *)"Transmitted Successfully\r\n", strlen("Transmitted Successfully\r\n"), 10);
  	}

	HAL_UART_Transmit(&huart2, (uint8_t *)bufferTX, 8, 10);


	/* SPI1 init function */
	static void MX_SPI1_Init(void)
	{

	  /* SPI1 parameter configuration*/
	  hspi1.Instance = SPI1;
	  hspi1.Init.Mode = SPI_MODE_MASTER;
	  hspi1.Init.Direction = SPI_DIRECTION_2LINES;
	  hspi1.Init.DataSize = SPI_DATASIZE_8BIT;
	  hspi1.Init.CLKPolarity = SPI_POLARITY_LOW;
	  hspi1.Init.CLKPhase = SPI_PHASE_1EDGE;
	  hspi1.Init.NSS = SPI_NSS_SOFT;
	  hspi1.Init.BaudRatePrescaler = SPI_BAUDRATEPRESCALER_32;
	  hspi1.Init.FirstBit = SPI_FIRSTBIT_MSB;
	  hspi1.Init.TIMode = SPI_TIMODE_DISABLE;
	  hspi1.Init.CRCCalculation = SPI_CRCCALCULATION_DISABLE;
	  hspi1.Init.CRCPolynomial = 10;
	  if (HAL_SPI_Init(&hspi1) != HAL_OK)
	  {
	    _Error_Handler(__FILE__, __LINE__);
	  }

	}	

	/* USART1 init function */
	static void MX_USART1_UART_Init(void)
	{

	  huart1.Instance = USART1;
	  huart1.Init.BaudRate = 115200;
	  huart1.Init.WordLength = UART_WORDLENGTH_8B;
	  huart1.Init.StopBits = UART_STOPBITS_1;
	  huart1.Init.Parity = UART_PARITY_NONE;
	  huart1.Init.Mode = UART_MODE_TX_RX;
	  huart1.Init.HwFlowCtl = UART_HWCONTROL_NONE;
	  huart1.Init.OverSampling = UART_OVERSAMPLING_16;
	  if (HAL_UART_Init(&huart1) != HAL_OK)
	  {
	    _Error_Handler(__FILE__, __LINE__);
	  }

	}

	/* USART2 init function */
	static void MX_USART2_UART_Init(void)
	{

	  huart2.Instance = USART2;
	  huart2.Init.BaudRate = 115200;
	  huart2.Init.WordLength = UART_WORDLENGTH_8B;
	  huart2.Init.StopBits = UART_STOPBITS_1;
	  huart2.Init.Parity = UART_PARITY_NONE;
	  huart2.Init.Mode = UART_MODE_TX_RX;
	  huart2.Init.HwFlowCtl = UART_HWCONTROL_NONE;
	  huart2.Init.OverSampling = UART_OVERSAMPLING_16;
	  if (HAL_UART_Init(&huart2) != HAL_OK)
	  {
	    _Error_Handler(__FILE__, __LINE__);
	  }

	}

	/** Configure pins as 
	        * Analog 
	        * Input 
	        * Output
	        * EVENT_OUT
	        * EXTI
	*/
	static void MX_GPIO_Init(void)
	{

	  GPIO_InitTypeDef GPIO_InitStruct;

	  /* GPIO Ports Clock Enable */
	  __HAL_RCC_GPIOC_CLK_ENABLE();
	  __HAL_RCC_GPIOH_CLK_ENABLE();
	  __HAL_RCC_GPIOA_CLK_ENABLE();
	  __HAL_RCC_GPIOB_CLK_ENABLE();

	  /*Configure GPIO pin Output Level */
	  HAL_GPIO_WritePin(GPIOB, CSNpin_Pin|CEpin_Pin, GPIO_PIN_RESET);

	  /*Configure GPIO pin : PC13 */
	  GPIO_InitStruct.Pin = GPIO_PIN_13;
	  GPIO_InitStruct.Mode = GPIO_MODE_INPUT;
	  GPIO_InitStruct.Pull = GPIO_NOPULL;
	  HAL_GPIO_Init(GPIOC, &GPIO_InitStruct);

	  /*Configure GPIO pins : CSNpin_Pin CEpin_Pin */
	  GPIO_InitStruct.Pin = CSNpin_Pin|CEpin_Pin;
	  GPIO_InitStruct.Mode = GPIO_MODE_OUTPUT_PP;
	  GPIO_InitStruct.Pull = GPIO_NOPULL;
	  GPIO_InitStruct.Speed = GPIO_SPEED_FREQ_LOW;
	  HAL_GPIO_Init(GPIOB, &GPIO_InitStruct);

	}
	