
<h1>STM32 LED Toggle Using Register Programming</h1>

<h2>Project Overview</h2>

This project demonstrates **LED toggling using direct register-level programming**  
on the **STM32F103C8 (Bluepill)** microcontroller without using HAL GPIO functions.

The system clock is configured using:
- External crystal (HSE)
- PLL enabled to achieve **72 MHz system clock**

<h2>Key Concepts Used</h2>

- RCC register configuration
- HSE oscillator enable and stabilization
- PLL configuration and clock switching
- Flash latency and prefetch buffer
- GPIO configuration using CRH register
- Busy-wait delay using a `for/while` loop

<h2>Hardware Used</h2>

- **Microcontroller:** STM32F103C8 (Bluepill)
- **LED Pin:** PC13
- **External Crystal:** 8 MHz
- **Programmer:** ST-Link V2

<h2>Clock Configuration</h2>

- **HSE (High-Speed External)**: 8 MHz
- **PLL (Phase-Locked Loop)**: HSE × 9 → 72 MHz
- **SYSCLK (System Clock)**: 72 MHz
- **APB1 Clock**: 36 MHz (max 36 MHz)
- **APB2 Clock**: 72 MHz


Flash configuration:
- 2 wait states
- Prefetch buffer enabled


<h2 > Source Code</h2>

```c
#include "stm32f103xb.h"

void delay(uint32_t count);

int main(void)
{
    RCC->CR |= RCC_CR_HSEON;
    while (!(RCC->CR & RCC_CR_HSERDY));

    FLASH->ACR |= FLASH_ACR_LATENCY_2;
    FLASH->ACR |= FLASH_ACR_PRFTBE;

    RCC->CFGR |= RCC_CFGR_PLLSRC;
    RCC->CFGR |= RCC_CFGR_PLLMULL9;
    RCC->CFGR |= RCC_CFGR_PPRE1_DIV2;

    RCC->CR |= RCC_CR_PLLON;
    while (!(RCC->CR & RCC_CR_PLLRDY));

    RCC->CFGR |= RCC_CFGR_SW_PLL;
    while ((RCC->CFGR & RCC_CFGR_SWS) != RCC_CFGR_SWS_PLL);

    RCC->APB2ENR |= RCC_APB2ENR_IOPCEN;

    GPIOC->CRH &= ~(GPIO_CRH_MODE13 | GPIO_CRH_CNF13);
    GPIOC->CRH |= GPIO_CRH_MODE13_1;

    while (1)
    {
        GPIOC->ODR ^= GPIO_ODR_ODR13;
        delay(10000000);
    }
}

void delay(uint32_t count)
{
    while (count--);
}
