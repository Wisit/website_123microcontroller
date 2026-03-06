---
layout: post
title: "Scheduling (Round-Robin & STCF): จัดคิวซีพียูอย่างไรให้ยุติธรรม ศาสตร์เบื้องหลัง Multitasking ของสาย Embedded"
lang: th
ref: c-scheduling-round-robin-stcf
permalink: /th/c-scheduling-round-robin-stcf/
description: "เจาะลึกการทำงานของ OS Scheduler ทบทวนอัลกอริทึมสุดคลาสสิกอย่าง Round-Robin และ STCF (Shortest Time-to-Completion First) ในระบบ Multitasking"
image: assets/images/scheduling-round-robin-stcf.jpg
tags: [C Programming, Embedded C, RTOS, Scheduling, Round-Robin, STCF, OS Architecture]
---

<div class="adsense-slot"></div>

## บทนำ (Introduction)

สวัสดีครับน้องๆ วิศวกรและเพื่อนนักพัฒนาชาว www.123microcontroller.com ทุกคน! กลับมาพบกับวิศวกรขอบตาดำๆ กันอีกครั้งครับ 

ในบทความก่อนหน้านี้เราได้คุยกันถึงปรัชญาของ Multitasking แบบ Cooperative และ Pre-emptive ไปแล้ว แต่น้องๆ สงสัยไหมครับว่า ในระบบที่ CPU มีแค่คอร์เดียวและต้องรันหลายๆ งาน (Tasks) พร้อมกัน ตัวระบบปฏิบัติการหรือ "Scheduler (ตัวจัดคิว)" มันใช้ตรรกะอะไรในการตัดสินใจว่า "ใครจะได้ใช้ CPU เป็นคนต่อไป?" 

วันนี้พี่จะพาไปแงะคัมภีร์วิศวกรรมซอฟต์แวร์ เพื่อดูอัลกอริทึมการจัดคิวที่คลาสสิกที่สุดอย่าง **Round-Robin** และอัลกอริทึมสายซิ่งอย่าง **STCF (Shortest Time-to-Completion First)** กันครับ ว่ามันทำงานอย่างไร และมีบทบาทอย่างไรในโลกของ Embedded Systems ไปลุยกันเลย!

## เนื้อหาหลัก (Core Concept): ผู้คุมกฎแห่งเวลา และศิลปะการแบ่งปัน CPU

ในบริบทที่กว้างขึ้นของระบบ Multitasking นั้น แหล่งข้อมูลชั้นครูได้เปรียบเทียบการรันหลายโปรแกรมพร้อมกันว่า เหมือนคนหลายคนกำลังอ่านหนังสือเล่มเดียวกัน แน่นอนว่า ณ เสี้ยววินาทีหนึ่ง มีคนอ่านได้แค่คนเดียว แต่ถ้าทุกคน "ผลัดกันอ่าน" อย่างรวดเร็ว มันก็จะดูเหมือนทุกคนได้อ่านพร้อมๆ กัน หน้าที่การจัดคิวนี้คือของ Scheduler ซึ่งมีอัลกอริทึม 2 ตัวที่น่าสนใจดังนี้ครับ:

*   **1. Round-Robin Scheduling (ยุติธรรม ไม่ทิ้งใครไว้ข้างหลัง):**
    *   **หลักการ:** เป็นหนึ่งในวิธีที่ง่ายและเป็นที่นิยมที่สุด หลักการคือการแบ่งเวลาของ CPU ออกเป็นช่วงสั้นๆ เท่าๆ กัน เรียกว่า **"Time Slice"** หรือ Tick
    *   **การทำงาน:** เมื่อ Task หนึ่งถูกรันจนหมดโควตาเวลา (Time Slice) ของตัวเอง ตัว Scheduler จะสั่งหยุด (Preempt) แล้วนำ Task นั้นไปต่อท้ายแถว จากนั้นก็ดึง Task ถัดไปในคิวขึ้นมารันต่อวนไปเรื่อยๆ เป็นวงกลม
    *   **จุดเด่น:** ยุติธรรมและรับประกันได้เลยว่าไม่มี Task ไหนถูกปล่อยทิ้งให้รอจนแห้งตาย (Starvation-free) และมีความเป็น Deterministic (คาดเดาเวลาได้)
    *   **จุดอ่อน:** ระบบนี้ไม่สนใจ "ความสำคัญ (Priority)" หรือ "ขนาด (Size)" ของ Task เลย ในงานสมองกลฝังตัว บาง Task อาจใช้เวลาแค่ 1 มิลลิวินาที (ms) ในขณะที่อีก Task ใช้เวลาเป็น 100 ms การให้เวลาเท่ากันหมดอาจทำให้ Task ที่ใช้เวลาน้อยต้องรอนานเกินความจำเป็น และมีค่า Turnaround time (เวลาเฉลี่ยตั้งแต่เริ่มจนงานเสร็จ) ที่ค่อนข้างแย่

*   **2. STCF - Shortest Time-to-Completion First (ใครเสร็จก่อน ได้รันก่อน):**
    *   **หลักการ:** เกิดมาเพื่อแก้ปัญหา Turnaround time ของ Round-Robin ครับ อัลกอริทึมนี้จะประเมินว่า Task ไหนใช้เวลาในการทำงาน "น้อยที่สุด" จะได้สิทธิ์รันก่อนเสมอ
    *   **การทำงานแบบไดนามิก:** หากมี Task ใหม่แทรกเข้ามาในคิว ตัว Scheduler จะเปรียบเทียบเวลาที่เหลือของ Task ที่กำลังรันอยู่ กับเวลาของ Task ใหม่ หาก Task ใหม่ใช้เวลาน้อยกว่า มันจะ "แย่ง (Preempt)" CPU ไปใช้ทันที!
    *   **จุดเด่น:** ลดการทำ Context Switch (การสลับบริบทของ CPU) ได้ดีมาก และทำให้งานโดยรวมเสร็จไวขึ้น
    *   **จุดอ่อน (ทำไมถึงไม่เหมาะกับ Embedded):** มันเป็น Dynamic Scheduler ที่กินพลังประมวลผลของ CPU ในการคำนวณคิวสูงมาก และที่ร้ายแรงที่สุดคือ หากมี Task สั้นๆ โผล่มาเรื่อยๆ Task ที่ต้องใช้เวลาทำงานนานๆ จะถูกแย่ง CPU ตลอดกาล (Starvation) 
    *   **มรดกตกทอด:** แม้ STCF จะไม่ค่อยถูกใช้ตรงๆ ใน RTOS แต่ปรัชญาของมันที่ว่า "งานสั้นควรได้สิทธิ์ก่อน" ได้กลายมาเป็นรากฐานของการออกแบบ **Rate-Monotonic Scheduling (RMS)** ซึ่งเป็นระบบกำหนด Priority แบบตายตัวที่ฮิตที่สุดในงาน Real-Time โดยให้ Task ที่มีคาบเวลาสั้นสุด ได้ Priority สูงสุดนั่นเองครับ!

![Scheduling Algorithms: Round-Robin vs STCF]({{ site.baseurl }}/assets/images/scheduling-round-robin-stcf-diagram.jpg)

## ตัวอย่างโค้ด (Code Example):

เพื่อให้เห็นภาพ พี่ขอยกตัวอย่างการจำลอง **Round-Robin Scheduler** แบบพื้นฐานที่สุด โดยใช้ Array ของพอยน์เตอร์ฟังก์ชัน (Function Pointers) ในภาษา C ครับ (โค้ดนี้จำลองแนวคิด Cooperative Round-Robin ที่แต่ละ Task ทำงานสั้นๆ แล้ววนลูปคิวถัดไป)

```c
#include <stdio.h>
#include <stdint.h>

/* ประกาศ Function Prototypes สำหรับ Tasks */
void task_read_sensor(void);
void task_update_display(void);
void task_send_serial(void);

/* สร้าง Array ของ Function Pointers เก็บรายชื่อ Task ในคิว */
#define NUM_TASKS 3
void (*task_queue[NUM_TASKS])(void) = {
    task_read_sensor,
    task_update_display,
    task_send_serial
};

/* ========================================================
 * ตัวจำลอง Round-Robin Scheduler แบบง่าย
 * ======================================================== */
int main(void) {
    uint8_t current_task_index = 0;
    
    printf("Starting Round-Robin Scheduler...\n");
    
    /* ลูปการทำงานหลัก (Main Loop) */
    while (1) {
        /* รัน Task ตามคิวปัจจุบัน */
        task_queue[current_task_index]();
        
        /* สลับคิวไปยัง Task ถัดไป แบบ Round-Robin (วงกลม) */
        current_task_index++;
        if (current_task_index >= NUM_TASKS) {
            current_task_index = 0; /* วนกลับไป Task แรกเมื่อจบวง */
        }
        
        /* ในระบบจริง ตรงนี้อาจจะมีการหน่วงเวลา หรือรอ Timer Tick 
         * ก่อนที่จะสลับไปรัน Task ถัดไป */
    }
    
    return 0;
}

/* ========================================================
 * การทำงานของแต่ละ Task (ต้องทำงานให้เสร็จเร็วๆ แล้วคืน CPU)
 * ======================================================== */
void task_read_sensor(void)  { /* อ่านค่าเซ็นเซอร์... */ }
void task_update_display(void){ /* อัปเดตหน้าจอ LCD... */ }
void task_send_serial(void)  { /* ส่งข้อมูลผ่าน UART... */ }
```

## ข้อควรระวัง / Best Practices:

การเลือกใช้อัลกอริทึมจัดคิวในงานสมองกลฝังตัว มีหลุมพรางที่ต้องระวังดังนี้ครับ:

1.  **ความเหนื่อยล้าจากการสลับ Context (Context Switch Overhead):** ในระบบ Pre-emptive Round-Robin หากคุณตั้งค่า Time Slice (Tick) ไว้ "สั้นเกินไป" CPU จะสูญเสียเวลาไปกับการเซฟและกู้คืนค่า Register (Context Switch) มากกว่าเวลาที่ได้รันโค้ดของ Task เสียอีก! ซึ่งส่งผลเสียต่อประสิทธิภาพ (Efficiency) อย่างรุนแรง ต้องบาลานซ์ค่า Tick ให้พอดีครับ
2.  **ระวังปัญหา Starvation:** หากคุณพยายามประยุกต์ใช้แนวคิดแบบ STCF หรือรันระบบที่อนุญาตให้ Task ความสำคัญสูงแทรกได้ตลอดเวลา (Dynamic preemption) ต้องมั่นใจว่า Task ที่สำคัญน้อย (เช่น Task บันทึก Log) จะยังมีโอกาสรันบ้าง ไม่เช่นนั้นระบบอาจจะทำงานผิดพลาดเพราะคิวหลังๆ ไม่เคยได้รับการประมวลผลเลย
3.  **Round-Robin ไม่การันตี Real-Time:** หากระบบของคุณมี Task ฉุกเฉินที่ต้องตอบสนองทันที (เช่น ตรวจจับไฟไหม้ หรือหยุดมอเตอร์) การใช้ Round-Robin ล้วนๆ จะทำให้ระบบพังได้ เพราะ Task ฉุกเฉินต้องไปต่อคิวชาวบ้าน! แนะนำให้ใช้อัลกอริทึมแบบ **Priority-based Preemptive** ควบคู่กัน โดยให้ Task ฉุกเฉินมี Priority สูงสุด แล้วใช้ Round-Robin จัดการเฉพาะ Task ที่มี Priority เท่ากันเท่านั้นครับ

<div class="adsense-slot"></div>

## สรุป (Conclusion)

**Round-Robin** คือความยุติธรรมที่เรียบง่าย ในขณะที่ **STCF** คือความพยายามรีดเค้นเวลาให้เสร็จไวที่สุด อัลกอริทึมทั้งสองนี้สะท้อนให้เห็นถึงศิลปะของ **Scheduler** ในการบริหารทรัพยากร CPU ที่มีอยู่อย่างจำกัดให้เกิดประโยชน์สูงสุดในโลกของ Multitasking ครับ

น้องๆ คนไหนกำลังเขียน RTOS ใช้เอง หรือกำลังจูนค่า Timer Tick ใน FreeRTOS/RTX51 แล้วเจอปัญหาระบบหน่วง หรืออยากแชร์ประสบการณ์การปรับแต่ง Scheduler ลองแวะเข้ามาตั้งกระทู้พูดคุยและโชว์โค้ดกันต่อได้ที่เว็บบอร์ด **www.123microcontroller.com** ของพวกเราได้เลยนะครับ! แล้วพบกันใหม่ในบทความหน้า Happy Coding ครับทุกคน!
