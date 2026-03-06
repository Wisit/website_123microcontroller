---
layout: post
title: "Synchronization (Mutexes & Semaphores): กุญแจจัดระเบียบ Multitasking หยุดปัญหารถไฟชนกันในโลก Embedded"
lang: th
ref: c-synchronization-mutexes-semaphores
permalink: /th/c-synchronization-mutexes-semaphores/
description: "ทำความรู้จักเครื่องมือ Synchronization ในระบบ RTOS เจาะลึกความแตกต่างระหว่าง Mutex และ Semaphore เพื่อป้องกันปัญหา Race Condition และการแย่งทรัพยากร"
image: assets/images/synchronization-mutexes-semaphores.jpg
tags: [C Programming, Embedded C, RTOS, Synchronization, Mutex, Semaphore, Race Condition, Multitasking]
---

<div class="adsense-slot"></div>

## บทนำ (Introduction)

สวัสดีครับน้องๆ วิศวกรและเพื่อนนักพัฒนาชาว www.123microcontroller.com ทุกคน! กลับมาพบกับวิศวกรขอบตาดำๆ กันอีกครั้งนะครับ 

เมื่อโปรเจกต์ของเราซับซ้อนขึ้นจนต้องก้าวเข้าสู่โลกของ Multitasking หรือการใช้ RTOS (Real-Time Operating System) เราจะพบว่าโค้ดที่เคยทำงานได้ดีในลูปเดี่ยวๆ อาจพังพินาศได้ง่ายๆ เมื่อมีหลาย Task ทำงานพร้อมกันและพยายามแย่งกันใช้ทรัพยากร (Shared Resources) แหล่งข้อมูลระดับเซียนได้เตือนว่า การปล่อยให้หลายๆ Thread เข้าถึงตัวแปรหรือฮาร์ดแวร์เดียวกันโดยไม่จัดระเบียบ จะนำไปสู่หายนะที่เรียกว่า **"Race Condition"**

เพื่อป้องกันปัญหานี้ เราจึงต้องใช้ศาสตร์ที่เรียกว่า **Synchronization** ซึ่งฮีโร่สองตัวหลักที่เราจะมาทำความรู้จักกันในวันนี้ก็คือ **Mutex** และ **Semaphore** ครับ สองตัวนี้ทำงานต่างกันอย่างไร และจะช่วยปกป้องโปรแกรมของเราได้อย่างไร ไปลุยกันเลยครับ!

## เนื้อหาหลัก (Core Concept): ผู้คุมกฎแห่ง Multitasking

ในบริบทของระบบปฏิบัติการหรือ Multitasking เมื่อเรามีพื้นที่โค้ดหรือทรัพยากรที่ต้องแชร์กัน (เช่น Global variables, Memory buffers, หรือ Register ของฮาร์ดแวร์) เราจะเรียกพื้นที่เสี่ยงภัยนี้ว่า **"Critical Section"** หากปล่อยให้ Thread 2 ตัวเข้าไปเขียนข้อมูลทับกันใน Critical Section ก็จะเกิด Race Condition ทำให้ข้อมูลผิดเพี้ยนไป กลไกที่ระบบปฏิบัติการเตรียมไว้ให้เราใช้จัดระเบียบมีดังนี้ครับ:

*   **1. Mutex (Mutual Exclusion) - กุญแจห้องน้ำแห่งความปลอดภัย**
    *   **หลักการ:** แหล่งข้อมูลเปรียบเทียบ Mutex ว่าเหมือน "กุญแจห้องน้ำสาธารณะ" ที่มีอยู่เพียงดอกเดียว เมื่อมี Task หนึ่งต้องการใช้ทรัพยากร (เข้าห้องน้ำ) มันจะต้องทำการ "Lock" กุญแจนี้ไว้ หากมี Task อื่นมาขอใช้ มันจะต้องถูกบล็อก (Block) และยืนรอจนกว่า Task แรกจะทำธุระเสร็จและ "Unlock" กุญแจกลับคืนมา
    *   **ความเป็นเจ้าของ (Ownership):** กฎเหล็กของ Mutex คือ "ใครล็อก คนนั้นต้องเป็นคนปลดล็อก" Thread ที่ทำการล็อก Mutex จะกลายเป็นเจ้าของ (Owner) จนกว่าตัวมันเองจะสั่งปลดล็อกครับ 
    *   **จุดประสงค์หลัก:** ใช้เพื่อปกป้อง Critical Section ไม่ให้มีคนอื่นเข้ามาแทรกแซงแบบเด็ดขาด

*   **2. Semaphore - ไม้ผลัดและเครื่องนับจำนวน**
    *   **หลักการ:** Semaphore เป็นตัวแปรจำนวนเต็ม (Integer variable) ที่ใช้เพื่อนับจำนวนทรัพยากรหรือส่งสัญญาณประสานงาน (Signaling) 
    *   **การเปรียบเทียบ:** ถ้าเป็น **Counting Semaphore** จะเหมือนเรามีห้องน้ำหลายห้องและกุญแจหลายดอก ตัวแปรจะนับว่ามีกุญแจเหลือกี่ดอก เมื่อ Task เข้าใช้งานจะทำการลดค่า (Wait/Decrement) และเมื่อใช้งานเสร็จจะเพิ่มค่า (Signal/Post/Increment) กลับคืน หากค่าเป็น 0 Task ที่มาทีหลังจะต้องรอ 
    *   **Binary Semaphore:** มีค่าแค่ 0 หรือ 1 คล้ายกับ Mutex มาก แต่ข้อแตกต่างที่สำคัญคือ Semaphore ไม่มีเรื่องความเป็นเจ้าของ (Ownership) Thread หนึ่งสามารถทำ Wait แล้วให้อีก Thread หนึ่งเป็นคนทำ Signal ก็ได้
    *   **จุดประสงค์หลัก:** นิยมใช้สำหรับการ "ส่งสัญญาณ (Signaling)" เช่น เหมือนการส่งไม้ผลัดในวิ่งผลัด Task หนึ่งอาจจะรอรับสัญญาณจากอีก Task หรือจาก Interrupt (ISR) ว่ามีข้อมูลใหม่เข้ามาแล้ว จึงค่อยตื่นขึ้นมาทำงาน

![Synchronization: Mutex vs Semaphore]({{ site.baseurl }}/assets/images/synchronization-mutexes-semaphores-diagram.jpg)

## ตัวอย่างโค้ด (Code Example):

มาดูตัวอย่างคลาสสิกของการใช้ **Mutex** ด้วยไลบรารี Pthreads ในภาษา C เพื่อป้องกัน Race Condition ตอนบวกเลขกันครับ (Clean Code สไตล์สายระบบ):

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <pthread.h>

/* สร้างตัวแปร Mutex เพื่อใช้ปกป้อง Critical Section */
pthread_mutex_t my_mutex;

/* Shared Resources (ตัวแปรที่ใช้งานร่วมกัน) */
int shared_a = 5;
int shared_b = 7;

/* ฟังก์ชันการทำงานของ Thread (Worker Thread) */
void* process_shared_data(void* arg) {
    /* 🛡️ ก้าวเข้าสู่ Critical Section: ทำการขอ Lock Mutex */
    /* ถ้ามี Thread อื่นล็อกอยู่ Thread นี้จะหยุดรอ (Block) ที่บรรทัดนี้ */
    pthread_mutex_lock(&my_mutex);
    
    /* === จุดนี้คือ Critical Section (ปลอดภัยแล้ว) === */
    shared_a = shared_a + 3;
    shared_b = shared_b - 1;
    printf("Safe access: a = %d, b = %d\n", shared_a, shared_b);
    
    /* หน่วงเวลาจำลองการประมวลผลฮาร์ดแวร์ */
    sleep(1);
    
    /* 🛡️ ทำงานเสร็จแล้ว: ทำการ Unlock คืนกุญแจให้คนอื่นใช้ต่อ */
    pthread_mutex_unlock(&my_mutex);
    /* ======================================= */
    
    return NULL;
}

int main(void) {
    pthread_t t1, t2;
    
    /* 1. เริ่มต้นการทำงานของ Mutex */
    pthread_mutex_init(&my_mutex, NULL);
    
    /* 2. สร้าง Threads หลายๆ ตัวให้ทำงานพร้อมกัน */
    pthread_create(&t1, NULL, process_shared_data, NULL);
    pthread_create(&t2, NULL, process_shared_data, NULL);
    
    /* 3. รอให้ Threads ทั้งหมดทำงานเสร็จ */
    pthread_join(t1, NULL);
    pthread_join(t2, NULL);
    
    /* 4. ทำลาย Mutex เพื่อคืนทรัพยากรให้ระบบ */
    pthread_mutex_destroy(&my_mutex);
    
    return 0;
}
```

## ข้อควรระวัง / Best Practices:

การใช้เครื่องมือ Synchronization แม้จะช่วยรักษาข้อมูล แต่ก็เปรียบเสมือนดาบสองคมที่วิศวกรซอฟต์แวร์ต้องระวังอย่างหนักครับ:

1.  **ระวังผีหลอก Deadlock (ล็อกตาย):** ปัญหานี้เกิดขึ้นเมื่อ Task A ล็อก Mutex-1 และกำลังรอ Mutex-2 ในขณะเดียวกัน Task B ก็ล็อก Mutex-2 แล้วหันมารอ Mutex-1 ต่างฝ่ายต่างถือไพ่ของอีกคนและไม่มีใครยอมปล่อย ทำให้ระบบค้างถาวร **วิธีแก้:** ต้องออกแบบระบบให้ทุก Task ทยอยล็อก Mutex "ตามลำดับเดียวกัน (Predefined order)" เสมอ เพื่อป้องกันการรอเป็นวงกลม (Circular wait)
2.  **ระวังภัยเงียบ Priority Inversion (การสลับความสำคัญ):** นี่คือบั๊กบันลือโลกของวงการอวกาศ (เช่น กรณี Mars Pathfinder) อาการคือ Task ความสำคัญต่ำสุดดันไปจับ Mutex ไว้ ทำให้ Task ความสำคัญสูงสุดที่ต้องการ Mutex ตัวนี้ต้องหยุดรอ แย่ไปกว่านั้น หากมี Task ความสำคัญปานกลางโผล่มา มันจะแย่ง CPU ไปทำงานได้หน้าตาเฉย ทำให้ Task สำคัญสุดต้องรอแบบไร้จุดหมาย การแก้ปัญหานี้ต้องใช้ฟีเจอร์ของ OS เช่น Priority Inheritance
3.  **จงทำ Critical Section ให้สั้นที่สุด (The Bottleneck):** กูรูหลายท่านเปรียบ Mutex ว่าคือ "คอขวด (Bottleneck)" ของระบบ การถือ Mutex ไว้นานเกินความจำเป็น (เช่น ไปใส่ `delay()` นานๆ ในนั้น) จะทำลายประสิทธิภาพการทำงานแบบขนานของ Multithreading ย่อยยับ จงล็อก ดึง/แก้ข้อมูล แล้วรีบปลดล็อกทันที 
4.  **ห้ามใช้ Mutex Lock ภายใน Interrupt (ISR):** ถ้าเกิด Interrupt แทรกขึ้นมา แล้วโค้ด ISR พยายามไปเรียก `mutex_lock()` ที่บังเอิญถูกถือไว้โดย Thread ที่เพิ่งโดนขัดจังหวะพอดี ระบบของคุณจะ Deadlock ทันที! ใน ISR อนุญาตให้ทำได้แค่ฟังก์ชันแบบ Non-blocking เช่นการโยน `semaphore_signal()` เพื่อปลุกให้ Task อื่นตื่นขึ้นมารับช่วงต่อเท่านั้น

<div class="adsense-slot"></div>

## สรุป (Conclusion)

ในโลกของระบบสมองกลฝังตัวที่มีการทำ Multitasking กลไก **Synchronization** เป็นหัวใจหลักที่ขาดไม่ได้ครับ **Mutex** ถูกสร้างมาเพื่อปกป้องทรัพยากรโดยให้สิทธิ์เป็น "เจ้าของชั่วคราว" ในขณะที่ **Semaphore** เปรียบเสมือน "เครื่องมือนับและตัวส่งสัญญาณ" เพื่อประสานงานข้าม Task หรือรับไม้ต่อจาก Interrupt การเลือกใช้ให้ถูกประเภทคือสิ่งที่แบ่งแยกวิศวกรมือใหม่กับมือโปรเลยครับ!

ใครเคยเขียน RTOS แล้วเจอปัญหาระบบค้างเพราะ Deadlock หรือมีเทคนิคการส่งข้อมูลแปลกๆ ระหว่าง ISR กับ Task อย่าเก็บไว้คนเดียวนะครับ! แวะเข้ามาตั้งกระทู้แชร์ประสบการณ์ และพูดคุยกันต่อที่บอร์ด **www.123microcontroller.com** ของพวกเราได้เลย แล้วพบกันใหม่ในบทความหน้า Happy Coding ครับทุกคน!
