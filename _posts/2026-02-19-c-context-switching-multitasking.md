---
layout: post
title: "Context Switching: เวทมนตร์สลับร่างและกลไกที่ซ่อนอยู่เบื้องหลัง Multitasking ของสาย Embedded"
lang: th
ref: c-context-switching-multitasking
permalink: /th/c-context-switching-multitasking/
description: "เจาะลึกกลไกการสลับการทำงานของ CPU (Context Switch) ในระบบ Multitasking และ RTOS รวมถึงการใช้แบตเตอรี่และหน่วยความจำ"
image: assets/images/context-switching-multitasking.jpg
tags: [C Programming, Embedded C, RTOS, Context Switching, Multitasking, OS Architecture, Assembly]
---

<div class="adsense-slot"></div>

## บทนำ (Introduction)

สวัสดีครับน้องๆ วิศวกรและเพื่อนนักพัฒนาชาว www.123microcontroller.com ทุกคน! กลับมาพบกับวิศวกรขอบตาดำๆ กันอีกครั้งครับ วันนี้เราจะมาขุดลึกลงไปถึงแก่นแท้ของระบบปฏิบัติการ (OS) และเรื่องของ **Multitasking** กันครับ

ในบทความก่อนหน้านี้เราพูดถึงการทำงานแบบหลายงานพร้อมกัน (Multitasking) บนไมโครคอนโทรลเลอร์ที่มี CPU เพียงคอร์เดียว ว่ามันคือการสร้าง "ภาพลวงตา" ให้ดูเหมือนทำงานพร้อมกัน แต่เบื้องหลังภาพลวงตาอันแนบเนียนนี้ มีพระเอกปิดทองหลังพระที่ชื่อว่า **"Context Switch" (การสลับบริบท)** ซ่อนอยู่ครับ กระบวนการนี้เปรียบเสมือนหัวใจที่ทำให้ Multitasking เกิดขึ้นได้จริง วันนี้เราจะมาเจาะลึกจากแหล่งข้อมูลระดับโลกกันว่า Context Switching คืออะไร ทำงานอย่างไร และมันแอบ "กินแรง" CPU ของเราไปมากน้อยแค่ไหน ลุยกันเลยครับ!

## เนื้อหาหลัก (Core Concept): Context Switching คืออะไรในโลกของ Multitasking?

ในบริบทที่กว้างขึ้นของระบบ Multitasking นั้น หน้าที่หลักของตัวจัดคิว (Scheduler) คือการแบ่งปันเวลา CPU ให้กับหลายๆ โปรแกรม (Tasks หรือ Threads) แต่การที่จะเปลี่ยนให้ CPU เลิกทำงาน Task A แล้วหันไปทำ Task B ได้นั้น CPU จะต้องทำสิ่งที่เรียกว่า **Context Switch**

แหล่งข้อมูลชั้นครูได้อธิบายกระบวนการนี้ด้วยการเปรียบเทียบและการทำงานระดับฮาร์ดแวร์ไว้ดังนี้ครับ:

*   **1. นิยามของคำว่า "Context" (บริบท):**
    ลองจินตนาการว่ามีคนหลายคนกำลังสลับกันอ่านหนังสือเล่มเดียวกัน (เปรียบเหมือน CPU คอร์เดียว) การจะสลับคนอ่านได้ แต่ละคนต้องมี "ที่คั่นหนังสือ (Bookmark)" ส่วนตัว เพื่อจดจำว่าตัวเองอ่านค้างไว้ที่หน้าไหน บรรทัดไหน
    ในโลกของ CPU นั้น "ที่คั่นหนังสือ" ก็คือ **Context** ซึ่งหมายถึงสถานะทั้งหมดของรีจิสเตอร์ (CPU Registers) ณ เสี้ยววินาทีนั้นๆ ซึ่งประกอบไปด้วย:
    *   **Program Counter (PC) หรือ Instruction Pointer:** ชี้ว่าคำสั่งต่อไปที่จะต้องรันอยู่บรรทัดไหนใน Memory
    *   **Stack Pointer:** ชี้ว่าตำแหน่งข้อมูล Local variables ล่าสุดของ Task นี้อยู่ที่ไหน
    *   **General Purpose Registers:** ค่าตัวแปรต่างๆ ที่ CPU กำลังคำนวณค้างไว้
*   **2. กลไกการสลับร่าง (The Switching Process):**
    เมื่อเกิดเหตุการณ์ที่ต้องสลับ Task (เช่น หมดเวลา Time slice หรือมี Interrupt เข้ามา) ระบบจะทำการ "หยุด" Task ที่รันอยู่ จากนั้นก๊อปปี้ค่า Context ทั้งหมดของมันไป "เซฟเก็บไว้ในหน่วยความจำ (Memory/Stack)" หลังจากนั้นก็จะไป "ดึงค่า Context" ของ Task ตัวใหม่ที่กำลังจะรันขึ้นมาโหลดใส่รีจิสเตอร์ของ CPU แล้วสั่งรันต่อจากจุดเดิม
*   **3. เป็นเรื่องของฮาร์ดแวร์ (Hardware-Specific):**
    เพราะ Context Switching ต้องยุ่งกับการดึงและยัดค่าลงรีจิสเตอร์ของ CPU โดยตรง โค้ดส่วนนี้จึงต้องเขียนด้วย **ภาษาแอสเซมบลี (Assembly Language)** เสมอ และหน้าตาโค้ดจะเปลี่ยนไปตามสถาปัตยกรรมของชิปแต่ละเบอร์ (เช่น ARM Cortex-M จะเขียนไม่เหมือน PIC หรือ 8051)

![Context Switching Process]({{ site.baseurl }}/assets/images/context-switching-multitasking-diagram.jpg)

## ตัวอย่างโค้ด (Code Example):

เนื่องจากกระบวนการ Context Switch จริงๆ ต้องเขียนด้วย Assembly บทความนี้พี่จึงขอยกตัวอย่างโครงสร้างภาษา C ที่ใช้จำลอง "ที่คั่นหนังสือ" หรือ **Task Control Block (TCB)** ซึ่งเป็นโครงสร้างที่ RTOS ทั่วไปใช้เก็บ Context ของแต่ละ Task ครับ

```c
#include <stdint.h>

/* 
 * โครงสร้างสำหรับเก็บ "ที่คั่นหนังสือ" ของแต่ละ Task 
 * มักเรียกว่า TCB (Task Control Block)
 */
typedef struct {
    uint32_t* stack_pointer;    // เก็บตำแหน่ง Stack ของ Task นี้
    uint8_t   priority;         // ลำดับความสำคัญ
    uint8_t   task_state;       // สถานะ (เช่น READY, RUNNING, BLOCKED)
} TaskControlBlock_t;

TaskControlBlock_t* current_task;
TaskControlBlock_t* next_task;

/* 
 * ฟังก์ชันนี้จะถูกเรียกโดย Timer Interrupt (เช่น SysTick)
 * เพื่อสลับการทำงานจาก current_task ไปยัง next_task
 */
void RTOS_Scheduler_ISR(void) {
    /* 1. ค้นหา Task ถัดไปที่พร้อมจะรันและมี Priority สูงสุด (ลอจิกของ Scheduler) */
    next_task = find_highest_priority_ready_task();
    
    if (next_task != current_task) {
        /* 
         * 2. ทำการ Context Switch!
         * 🛡️ โค้ดด้านล่างนี้เป็นเพียงคอมเมนต์จำลองการทำงาน 
         * ของจริงต้องเป็น Assembly (เช่น _asm volatile (...)) 
         */
         
        // [Assembly] PUSH รีจิสเตอร์ทั้งหมดของ CPU ลงใน Stack ปัจจุบัน
        // [Assembly] current_task->stack_pointer = CPU_STACK_POINTER;
        
        // [Assembly] CPU_STACK_POINTER = next_task->stack_pointer;
        // [Assembly] POP รีจิสเตอร์ทั้งหมดของ CPU ขึ้นมาจาก Stack ใหม่
        
        current_task = next_task;
    }
    /* 3. เมื่อจบ ISR ตัว CPU จะกระโดดไปรันคำสั่งของ next_task ทันที! */
}
```

## ข้อควรระวัง / Best Practices:

การมีระบบที่สามารถทำ Context Switch ได้ ถือเป็นดาบสองคมในงาน Embedded ครับ ผู้เชี่ยวชาญเตือนถึงหลุมพรางสำคัญที่คุณต้องระวังดังนี้:

1.  **ปัญหาคอขวดจาก Overhead (Context Switch Overhead):**
    การบันทึกและกู้คืนรีจิสเตอร์ไม่ใช่ของฟรี! มันใช้เวลาและ CPU Cycles (Overhead) ลองนึกภาพว่าคุณตั้งเวลาสลับ Task (Time slice) ไว้ที่ 100 ms แต่ตัว CPU ต้องใช้เวลาทำ Context Switch ถึง 100 ms นั่นแปลว่า CPU ของคุณจะสูญเสียเวลาทำงานไปถึงครึ่งหนึ่งเต็มๆ! การออกแบบระบบที่ดีต้องปรับจูนเวลา Time Slice ให้สมดุลกับความเร็วของ CPU เสมอ
2.  **อันตรายต่อตัวแปรแชร์ (Race Conditions):**
    ในระบบ Preemptive การเกิด Context Switch สามารถ "แทรกขึ้นมาเมื่อไหร่ก็ได้" สมมติว่า Task A กำลังเขียนข้อมูลลง Buffer ชิ้นหนึ่ง แล้วถูกสลับให้ Task B (ที่มี Priority สูงกว่า) มารันต่อ หาก Task B ดันเข้ามาแก้ข้อมูลใน Buffer เดียวกัน ข้อมูลจะพังพินาศทันที! วิศวกรจึงต้องปกป้องตัวแปรที่ใช้ร่วมกันเสมอด้วยกลไกทาง Synchronization อย่าง **Mutex** หรือการปิด Interrupt ชั่วคราว (Critical Sections)
3.  **กิน Memory อย่างมหาศาล (Stack per Task):**
    เนื่องจากแต่ละ Task ต้องมีที่เซฟ Context เป็นของตัวเอง ระบบปฏิบัติการแบบ Multithreading (เช่น RTOS) จะต้องจองพื้นที่หน่วยความจำ **Stack แยกเฉพาะให้กับทุกๆ Task** หากคุณสร้าง Task มากเกินไป RAM ของไมโครคอนโทรลเลอร์จะหมดอย่างรวดเร็ว และเสี่ยงต่อการเกิด Stack Overflow ได้ง่ายมากๆ ครับ

<div class="adsense-slot"></div>

## สรุป (Conclusion)

**Context Switch** คือกลไกที่ทำหน้าที่ "หยุด ถอด บันทึก โหลด และเดินหน้าต่อ" ให้กับ CPU ครับ มันเป็นส่วนเติมเต็มที่ทำให้สถาปัตยกรรม **Multitasking** สามารถสร้างระบบที่ซับซ้อนและตอบสนองได้แบบ Real-time แม้ว่าการสลับงานจะสร้าง Overhead และอาจก่อให้เกิดบั๊กที่ซับซ้อนหากจัดการตัวแปรไม่ดี แต่มันก็เป็นเทคนิคพื้นฐานที่โปรแกรมเมอร์สาย Embedded ทุกคนต้องทำความเข้าใจเพื่อให้ใช้ RTOS ได้อย่างเต็มประสิทธิภาพครับ

น้องๆ คนไหนกำลังสงสัยว่า MCU ที่ตัวเองใช้อยู่ (เช่น STM32 หรือ ESP32) เซฟรีจิสเตอร์กี่ตัวตอนทำ Context Switch หรือเคยปวดหัวกับการตั้งค่า Stack Size ใน FreeRTOS มาก่อน อย่าลืมแวะเข้ามาตั้งกระทู้แชร์ประสบการณ์ และพูดคุยกันต่อที่เว็บบอร์ด **www.123microcontroller.com** ของพวกเราได้เลยนะครับ! แล้วพบกันใหม่ในบทความหน้า Happy Coding ครับทุกคน!
