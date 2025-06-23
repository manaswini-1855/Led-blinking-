#include <LPC17xx.h>

#define TRIG_PIN (1 << 10)    	// P2.10 as trigger
#define ECHO_PIN (1 << 11)  	// P2.11 as echo

volatile uint32_t start_time = 0, end_time = 0, pulse_width = 0;
volatile uint8_t echo_received = 0;
   

void GPIO_Init(void) {
    LPC_GPIO2->FIODIR |= TRIG_PIN;  		// Set trigger pin as output
    LPC_GPIO2->FIODIR &= ~ECHO_PIN; 		// Set echo pin as input
    LPC_GPIOINT->IO2IntEnR |= ECHO_PIN; 	// Enable rising edge interrupt
    LPC_GPIOINT->IO2IntEnF |= ECHO_PIN; 	// Enable falling edge interrupt
    NVIC_EnableIRQ(EINT3_IRQn);         			// Enable GPIO interrupt
}

void Timer_Init(void) {
    LPC_SC->PCONP |= (1 << 1);        // Power on Timer 0
    LPC_SC->PCLKSEL0 |= (1 << 2);     // Set Timer 0 clock to CCLK (100 MHz)
    LPC_TIM0->TCR = 2;                		// Reset Timer 0
    LPC_TIM0->PR = 99;                // Prescale for 1 µs (100 MHz / 100 - 1)
    LPC_TIM0->TCR = 1;                // Enable Timer 0
}


void EINT3_IRQHandler(void) {
    if (LPC_GPIOINT->IO2IntStatR & ECHO_PIN) { 		// Rising edge detected
      //  start_time = Get_Time();
        LPC_GPIOINT->IO2IntClr |= ECHO_PIN; 			// Clear interrupt flag
    }

    if (LPC_GPIOINT->IO2IntStatF & ECHO_PIN) { 		// Falling edge detected
  //      end_time = Get_Time();
    //    pulse_width = end_time - start_time;
        echo_received = 1; 								// Signal that echo is received
        LPC_GPIOINT->IO2IntClr |= ECHO_PIN; 			// Clear interrupt flag
    }
}


void Trigger_Sensor(void) {
	
    LPC_GPIO2->FIOSET = TRIG_PIN;     // Set trigger pin high
    for (uint32_t i = 0; i < 1000; i++) {}; // 10 µs delay
    LPC_GPIO2->FIOCLR = TRIG_PIN;     // Set trigger pin low
}




int main(void) {

    GPIO_Init();
    Timer_Init();

    while (1) {
        Trigger_Sensor();
        while (!echo_received); // Wait for echo
        echo_received = 0;     // Reset flag
      
        
    }
}
