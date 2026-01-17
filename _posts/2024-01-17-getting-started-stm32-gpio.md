---
layout: post
title: "Getting Started with STM32 GPIO"
date: 2024-01-17 10:00:00 +0000
categories: [Embedded, STM32, Tutorial]
tags: [stm32, c, gpio]
author: "Engineer 123"
excerpt: "A comprehensive guide to configuring General Purpose Input/Output (GPIO) pins on STM32 microcontrollers using direct register access."
---

## Introduction

General Purpose Input/Output (GPIO) is the most fundamental peripheral in any microcontroller. In this tutorial, we will explore how to configure a GPIO pin as an updated output to blink an LED on the STM32F103 series.

This guide focuses on **Register Level Programming** rather than using HAL, ensuring you understand exactly what is happening under the hood.

## Understanding the Registers

To control a pin, we need to interact with several registers:
1.  **RCC_APB2ENR**: To enable the clock for the GPIO Port.
2.  **GPIOx_CRL/CRH**: To configure the pin mode (Input/Output) and speed.
3.  **GPIOx_ODR**: To write data to the Output Data Register.

### Enabline the Clock
Before using any peripheral, its clock must be enabled. For GPIOC, this is done in the Reset and Clock Control (RCC) register.

```c
// Enable Clock for Port C
RCC->APB2ENR |= (1 << 4);
```

## The Code

Here is the complete C code to blink an LED connected to Pin C13. Note that on the Blue Pill board, the LED is active low.

```c
#include "stm32f10x.h"

void delay(int rep) {
    for (; rep > 0; rep--) {
        for (int i = 0; i < 100000; i++) {} // Dump delay
    }
}

int main(void) {
    // 1. Enable Clock for GPIOC
    RCC->APB2ENR |= (1 << 4);

    // 2. Configure PC13 as Output Push-Pull (2MHz)
    // Clear bits 20-23
    GPIOC->CRH &= ~(0xF << 20);
    // Set Mode to 2MHz (10) and CNF to Push-Pull (00) -> 0010 -> 0x2
    GPIOC->CRH |= (0x2 << 20);

    while (1) {
        // 3. Toggle PC13
        GPIOC->ODR ^= (1 << 13);
        delay(5);
    }
}
```

## Conclusion

Direct register access provides minimal overhead and precise control. While HAL libraries are convenient, understanding the hardware abstraction is crucial for high-reliability engineering.

Stay tuned for the next article on **Interrupts and Timers**.
