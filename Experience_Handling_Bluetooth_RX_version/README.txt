READ ME : Experience handling project

On start-up:
	-> USART1 is initialized with interrupts in reception
	-> Red LED is turned ON 
	-> Green LED is turned OFF
	-> User LED is turned OFF


TIM1 generates a PWM of 10kHz frequency: Global clock at 84MHz, Prescaler at 84 and period at 100 ticks; 84MHz/Prescaler = 1Mhz; 1MHz/period = 10kHz -> PWM frequency.
	Output on TIM1 CH1 Port A pin 8 (PA8); 

TIM2 interrupts every second and increments a global variable counter. 
	Global clock at 84MHz, Prescaler at 42000 and period at 2000 ticks; 84MHz/Prescaler = 2kHz; 2MHz/period = 1 Hz -> Interrupt frequency.
	TIM2 interrupt handling in function "void HAL_TIM_PeriodElapsedCallback(TIM_HandleTypeDef *htim)" main.c file

	In the interrupt, the variable counter is incremented and checked. 
	If counter > threshold:
		- User LED turns off
		- TIM2 interrupt is disabled
		- Red LED turns on
		- Green LED turns off
		- TIM1 PWM is disabled

GPIO:
	Red LED to signal experience in not started or is stopped at pin PB6, label: RedLED
	Green LED to signal experience in not started or is stopped at pin PC7, label GreenLED

	User LED to signal experience is started or ongoing at pin PA5, label UserLED

USART1 working with HC05 module in slave mode, baudrate 115200 (MAC: 98d3,32,115105)
	Interrupts at every reception, interrupt handling in function "void HAL_UART_RxCpltCallback(UART_HandleTypeDef *huarts)" main.c file. 

	When an 'S' (for Start) is received, and counter < Threshold:
		- User LED turns on
		- TIM2 interrupt is started
		- Red LED turns off
		- Green LED turns on
		- TIM1 PWM is started

	When a 'E' (for End) is received:
		- User LED turns off
		- TIM2 interrupt is disabled
		- Red LED turns on
		- Green LED turns off
		- TIM1 PWM is disabled

TO implement in Master project: 

	USART Asynchronous mode, 115200 baudrate. Connect Master bluetooth device
	Use the "HAL_StatusTypeDef HAL_UART_Transmit(UART_HandleTypeDef *huart, uint8_t *pData, uint16_t Size, uint32_t Timeout)" function to send the instructions:
		- 'S' to start the experience
		- 'E' to stop the experience. 

	Example: 

	UART_HandleTypeDef huart1;

	static void MX_USART1_UART_Init(void);

	char* bufferTX = "S";

	MX_USART1_UART_Init();

	HAL_UART_Transmit(&huart2, (uint8_t *)bufferTX, 8, 10);

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
	
	