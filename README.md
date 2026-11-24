# STM32-EXTI
# STM32 EXTI Button Debouncer
A button debouncer for the STM32 Nucleo F446RE, built with an external interrupt (EXTI) and a timer interrupt. One physical button press gives exactly one count, and the onboard LED toggles once per press.

When you press a mechanical button, the metal contacts bounce for a few milliseconds. A microcontroller is fast enough to see every bounce, so one press can trigger the interrupt many times and be counted as several presses.
The solution
The first button edge starts a short timer. The code ignores further edges while it waits, then checks that the button is still pressed. Only then does it count the press.
 # How it works
1.  The button on PC13 is pulled low when pressed, so the falling edge fires the EXTI interrupt.
2.  The EXTI callback sets a busy flag, resets TIM2, and starts it with its interrupt enabled. While the flag is set, extra edges are ignored.
3.  TIM2 fires an interrupt every 1 ms. The timer callback counts these ticks.
4.  After 20 ticks (about 20 ms), the timer callback stops TIM2 and reads PC13 again.
5.  If the pin is still low, it is a real press, so the press count goes up by one.
6.  The busy flag is cleared, ready for the next press.
7.  The main loop watches the press count. When it changes, it toggles the LED on PA5.
The interrupt callbacks stay short and only collect information. The main loop takes the action.
# Hardware
• Board: STM32 Nucleo-F446RE
• Button: onboard blue user button (B1) on PC13
• LED: onboard green LED (LD2) on PA5
• IDE: STM32CubeIDE with STM32CubeMX and the HAL library
 # CubeMX configuration
Peripheral
Setting
PC13
GPIO external interrupt, falling edge detection
PA5
GPIO output (onboard LED)
NVIC
EXTI line[15:10] interrupt enabled
TIM2
Clock source: Internal Clock
TIM2
Prescaler: 83
TIM2
Counter period: 999
TIM2 NVIC
TIM2 global interrupt enabled
With an 84 MHz timer clock, a prescaler of 83 gives a 1 MHz tick, and a period of 999 gives an interrupt every 1 ms. Twenty of those make the 20 ms debounce time.
 # Key code
• HAL_GPIO_EXTI_Callback() starts the debounce timer on the first edge.
• HAL_TIM_PeriodElapsedCallback() counts 20 ms, re-checks the pin, and counts valid presses.
• The main loop compares press_count to its last value and toggles the LED when it changes.
Variables shared between the interrupts and the main loop (debounce_busy, press_count) are declared volatile.
 # How to use
1.  Clone this repository or download it.
2.  Open the .ioc file in STM32CubeIDE and click Generate Code. This recreates the Drivers folder.
3.  Build and flash the project to the Nucleo-F446RE.
4.  Press the blue button. The green LED toggles once per press.
 # What I learned
• How EXTI turns a pin edge into an interrupt
• Why mechanical buttons bounce and how a timer fixes it
• Using a flag so interrupts and the main loop can pass information
• Keeping interrupt callbacks short
 # Possible improvements
• Detect long presses and double presses
• Print the press count over UART
• Make the debounce time configurable
# Watch the demo on LinkedIn"

Author
Niraj, Electrical Engineering student learning embedded systems.
