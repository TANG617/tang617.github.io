---
title: STM32 SystemClock Modified When Debugging with OpenOCD
date: 2024-05-4 17:45:38
---
## Phenomenon
The chip is STM32F4( and it's doesn't matter from the view in the end, while the number of code line may vary)

When enabling PLL in Clock Configuration in CubeMX, no matter how low or high the SYSCLK or HCLK is, it would stuck at `SystemClock_Config();`
![image.png](https://cdn.jsdelivr.net/gh/TANG617/images/202405041717291.png)

Specifically, it stuck when configuring RCC and PLL
```c
//Line 149 in main.c
if (HAL_RCC_OscConfig(&RCC_OscInitStruct) != HAL_OK)  
{  
  Error_Handler();  
}

//Line 550 in stm32f4xx_hal_rcc.c
if (((RCC_OscInitStruct->PLL.PLLState) == RCC_PLL_OFF) ||  
    (READ_BIT(pll_config, RCC_PLLCFGR_PLLSRC) != RCC_OscInitStruct->PLL.PLLSource) ||  
    (READ_BIT(pll_config, RCC_PLLCFGR_PLLM) != (RCC_OscInitStruct->PLL.PLLM) << RCC_PLLCFGR_PLLM_Pos) ||  
    (READ_BIT(pll_config, RCC_PLLCFGR_PLLN) != (RCC_OscInitStruct->PLL.PLLN) << RCC_PLLCFGR_PLLN_Pos) ||  
    (READ_BIT(pll_config, RCC_PLLCFGR_PLLP) != (((RCC_OscInitStruct->PLL.PLLP >> 1U) - 1U)) << RCC_PLLCFGR_PLLP_Pos) ||  
    (READ_BIT(pll_config, RCC_PLLCFGR_PLLQ) != (RCC_OscInitStruct->PLL.PLLQ << RCC_PLLCFGR_PLLQ_Pos)))
```

## Suspicion
## System's clock is too high
The official highest clock speed for STM32F4 is 168 MHz and the HSI and HSE is 16 MHz and 8MHz.
Something strange happens:
- Decreasing the clock to 72MHz doesn't help at all. Which excludes the overclock speed.
- If the system clock comes directly from HSI or HSE, the problem dismisses.
	- The system works fine in this circumstance, however, 16MHz or 8MHz is too slow for application.
	- Even HSI encounters the problem, the failure of HSE or the wrong layout could be excluded.
- If the PLL is participated, no matter how low the clock speed is, or whether the clock source comes from HSI or HSE, the problem is always encountered

After that, I tried to use the examples provided by manufacturer, which is also weird:
- The source code of RCC/PLL is pretty much similar to generated before, while none of them could work
- The binary program provided could work perfectly! Which excludes the problems from hardware.

After some tries, the problem is narrowed down to the RCC/PLL configuration 

### CubeMX and ST's bug
Though I have generated code thousands times directly from CubeMX and it ran perfectly before on STM32F103, it could be possible a vital bug made by ST official. And it could be true. Some reported a bug of mis-configuring a register in HAL few years ago and ST promised to solve the problem by updating a firmware pack.
https://community.st.com/t5/stm32-mcus-embedded-software/bugs-encountered-in-f4-rcc-hal-code-are-there-any-workarounds/td-p/209967
Sadly, I am at the latest version of firmware and the bug should be fixed. I even tried to retreat to the previous version, it still can's solve the problem.

### Debugger's fault
Debugger comes to my mind because of a coincidence that flashing the firmware without debug could dismiss the problem. I have to say I was in a trap at the very beginning of solving this problem: the first time I encountered it is because I want to grab the raw data with debug mode from a serial receiving code. The blink code works fine when flashing directly.
OK, the problem is narrowed down to the debugger, ST-Link.

## Reasons
I was inspired by https://www.eevblog.com/forum/microcontrollers/stm32-clock-gets-modified-when-debugger-is-connected/

**OpenOCD modifies register when triggering `reset-init`**
There are 3[types of `reset` by OpenOCD](https://openocd.org/doc/html/General-Commands.html):
![image.png](https://cdn.jsdelivr.net/gh/TANG617/images/202405041756683.png)
When `reset init` is triggered, the code from [stm32f4x.cfg](https://github.com/openocd-org/openocd/blob/master/tcl/target/stm32f4x.cfg) below would be executed:

```bash
$_TARGETNAME configure -event reset-init {
	# Configure PLL to boost clock to HSI x 4 (64 MHz)
	mww 0x40023804 0x08012008   ;# RCC_PLLCFGR 16 Mhz /8 (M) * 128 (N) /4(P)
	mww 0x40023C00 0x00000102   ;# FLASH_ACR = PRFTBE | 2(Latency)
	mmw 0x40023800 0x01000000 0 ;# RCC_CR |= PLLON
	sleep 10                    ;# Wait for PLL to lock
	mmw 0x40023808 0x00001000 0 ;# RCC_CFGR |= RCC_CFGR_PPRE1_DIV2
	mmw 0x40023808 0x00000002 0 ;# RCC_CFGR |= RCC_CFGR_SW_PLL

	# Boost JTAG frequency
	adapter speed 8000
}
```
It's obvious that RCC and PLL is configured to 64MHz instead of 16MHz from HSI by default. It's considered that OpenOCD needs to initialize the flash controller in order to program the memory.
After that, there is no more process of reset and the data in register remains.
![image.png](https://cdn.jsdelivr.net/gh/TANG617/images/202405041803009.png)

## Solution
- Use `reset halt` instead of `reset init` for debug
  The operation is pretty simple, change the option in CLion from:
  ![image.png](https://cdn.jsdelivr.net/gh/TANG617/images/202405041802597.png)
  to:
  ![image.png](https://cdn.jsdelivr.net/gh/TANG617/images/202405041803273.png)

## Besides
It's hard to believe the problem stuck me for 2 days only requires a click.
It's my first time loading `peropherals svd` in CLion for debug. It saves a lot of effort comparing to referring data sheet.
Here's a [repo with all `svd` file for STM32](https://github.com/tinygo-org/stm32-svd/blob/main/svd/stm32f407.svd)
