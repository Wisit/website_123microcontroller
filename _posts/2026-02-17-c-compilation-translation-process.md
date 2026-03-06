---
layout: post
title: "Compilation Phase: แกะกล่องดำของ Translation Process เปลี่ยนโค้ดภาษา C เป็นภาษาเครื่องสไตล์ Embedded"
lang: th
ref: c-compilation-translation-process
permalink: /th/c-compilation-translation-process/
description: "เจาะลึก Compilation Phase ขั้นตอนที่ 7 ในกระบวนการแปลภาษา C เปลี่ยนโครงสร้างการเขียนโปรแกรมให้กลายเป็นภาษาเครื่องที่ไมโครคอนโทรลเลอร์เข้าใจ"
image: assets/images/c-compilation-translation-process.jpg
tags: [C Programming, Embedded C, Compiler, Translation Process]
---

<div class="adsense-slot"></div>

## บทนำ (Introduction)

สวัสดีครับน้องๆ วิศวกรและเพื่อนนักพัฒนาชาว www.123microcontroller.com ทุกคน! กลับมาพบกับวิศวกรขอบตาดำๆ กันอีกครั้งครับ วันนี้เราจะมาเจาะลึกกระบวนการที่ถือเป็น "หัวใจหลัก" ของการสร้างซอฟต์แวร์ นั่นก็คือ **Compilation (การคอมไพล์)** ครับ

ในโลกของการเขียนโปรแกรมบนไมโครคอนโทรลเลอร์ ชิป CPU ของเราไม่รู้จักหรอกครับว่า `if`, `while`, หรือ `printf` คืออะไร มันรู้จักแค่สัญญาณไฟฟ้า 0 และ 1 เท่านั้น (Machine Language) บ่อยครั้งที่เรามักเรียกกระบวนการแปลงโค้ดทั้งหมดตั้งแต่ต้นจนจบว่า "การคอมไพล์ (Compiling หรือ Building)" แต่ในความเป็นจริง มาตรฐานภาษา C ได้ระบุว่ากระบวนการแปลงภาษาหรือ **Translation Process** นั้นมีถึง 8 ขั้นตอน (Phases) โดยในบทความก่อนเราได้พูดถึง Preprocessing (Phase 1-4) ไปแล้ว วันนี้เราจะมาดูกันครับว่า แหล่งข้อมูลชั้นครูอธิบายสิ่งที่เรียกว่า **Compilation** ในบริบทที่กว้างขึ้นของ Translation Process ไว้อย่างไร และมันมีกลไกการทำงานอย่างไรบ้าง!

## เนื้อหาหลัก (Core Concept): Compilation ในบริบทของ Translation Process

ในบริบทของ Translation Process เมื่อโค้ดของเราผ่านด่าน Preprocessor มาแล้ว (ได้เป็นไฟล์ที่เรียกว่า Translation Unit เช่นไฟล์นามสกุล `.i` ที่ไม่มีคอมเมนต์และไม่มีคำสั่ง `#` เหลืออยู่) โค้ดที่บริสุทธิ์นี้จะถูกส่งต่อไปยังขั้นตอนที่ 7 ของกระบวนการแปลภาษา ซึ่งก็คือการทำ **Compilation** อย่างแท้จริงครับ 

หน้าที่ของ Compiler คือการรับ Translation Unit เข้ามา และแปลงโครงสร้างภาษาระดับสูง (High-level constructs) ให้กลายเป็นชุดคำสั่งระดับล่าง (Assembly instructions) ที่สถาปัตยกรรมของโปรเซสเซอร์เป้าหมายเข้าใจได้ โดยกระบวนการภายในของ Compiler มักจะถูกแบ่งออกเป็น 2 ส่วนหลักๆ (Front-end และ Back-end) พร้อมด้วยขั้นตอนย่อยๆ ดังนี้ครับ:

*   **1. Lexical Analysis (การตัดคำ):** Compiler จะรับโค้ดเข้ามาทีละตัวอักษร แล้วจัดกลุ่มให้เป็น "คำ (Tokens)" เช่น คีย์เวิร์ด (`while`), ชื่อตัวแปร, ตัวเลข, หรือเครื่องหมายวรรคตอน
*   **2. Syntax Analysis (ตรวจสอบไวยากรณ์):** นำ Tokens มาเรียงต่อกันและตรวจสอบว่าถูกตามหลักไวยากรณ์ของภาษา C หรือไม่ จากนั้นจะสร้างโครงสร้างต้นไม้ที่เรียกว่า Parse Tree หรือ Abstract Syntax Tree (AST) เพื่อแสดงความสัมพันธ์ของโค้ด 
*   **3. Semantic Analysis (ตรวจสอบความหมาย):** ตรวจสอบความสมเหตุสมผล เช่น มีการประกาศตัวแปรก่อนใช้งานหรือไม่, ชนิดข้อมูล (Data Type) ตรงกันไหม, ฟังก์ชันคืนค่าถูกต้องหรือเปล่า หากพบข้อผิดพลาดในด่าน 1-3 นี้ Compiler จะพ่น **Syntax Error** หรือ **Semantic Error** ออกมาและหยุดการทำงานทันที
*   **4. Code Optimization (การเพิ่มประสิทธิภาพ):** เมื่อแน่ใจว่าโค้ดถูกต้อง Compiler จะแปลง AST ให้เป็นภาษากลาง (Intermediate Representation - IR หรือ RTL) และทำการปรับแต่งโค้ดให้ทำงานได้เร็วขึ้น (Execution speed) หรือกินพื้นที่หน่วยความจำน้อยลง
*   **5. Code Generation (การสร้างโค้ดเป้าหมาย):** นี่คือจุดที่ IR ถูกแปลงให้เป็นภาษา Assembly ที่เฉพาะเจาะจงกับฮาร์ดแวร์เป้าหมาย (เช่น ARM Cortex-M หรือ PIC) จากนั้นจะถูกส่งให้ **Assembler** แปลงเป็นภาษาเครื่อง (Machine Code) ในรูปแบบของไฟล์ **Object Code (`.o` หรือ `.obj`)** ต่อไป

**Cross-Compilation (การคอมไพล์ข้ามแพลตฟอร์ม):**
ในงาน Embedded Systems การแบ่งแยก Front-end (วิเคราะห์ภาษา C) และ Back-end (สร้างภาษาเครื่อง) ออกจากกันอย่างชัดเจน ทำให้เราสามารถทำสิ่งที่เรียกว่า **Cross-compiler** ได้ครับ นั่นคือเราใช้ PC (Host) รันโปรแกรม Compiler แต่มันสร้าง Object code สำหรับไมโครคอนโทรลเลอร์ (Target) แทนที่จะสร้างรหัสสำหรับ PC เครื่องนั้นนั่นเอง

![C Translation Process: Compilation Phase]({{ site.baseurl }}/assets/images/c-compilation-translation-process-diagram.jpg)

## ตัวอย่างโค้ด (Code Example):

ลองมาดูโค้ดภาษา C สไตล์ฮาร์ดแวร์ ที่แสดงให้เห็นว่าช่วง Compilation phase (โดยเฉพาะ Code Optimization) ช่วยวิศวกรอย่างเราได้อย่างไร

```c
#include <stdint.h>

/* กำหนด Base Address และ Offset สำหรับเข้าถึง Register */
#define GPIOA_BASE  0x40020000
#define GPIOA_MODER 0x00

/* ฟังก์ชันจำลองการตั้งค่าพอร์ตฮาร์ดแวร์ */
void init_gpio(void) {
    /* 
     * 1. 🛡️ สังเกตการใช้ 'volatile' เพื่อบอก Compiler ในช่วง Semantic Analysis
     * ว่าอย่า Optimize การเข้าถึงหน่วยความจำตำแหน่งนี้ทิ้งไปเด็ดขาด!
     * 
     * 2. Constant Folding: ในช่วง Code Optimization ตัว Compiler จะฉลาดพอที่จะ
     * นำ (0x40020000 + 0x00) มาบวกกันให้เสร็จตั้งแต่ตอนคอมไพล์ (Compile-time)
     * ทำให้ไม่ต้องเสียเวลา CPU ไปบวกเลขในตอนรันจริง (Run-time)
     */
    volatile uint32_t * const p_moder = (uint32_t *)(GPIOA_BASE + GPIOA_MODER);
    
    /* ตั้งค่าพินเป็นโหมด Output โดยการเขียนค่าลงไป */
    *p_moder = 0x00000001;
}

int main(void) {
    init_gpio();
    while(1) {
        /* ลูปการทำงานหลัก */
    }
    return 0; // ในฝั่ง Embedded โค้ดไม่ควรทำงานมาถึงจุดนี้
}
```

## ข้อควรระวัง / Best Practices:

ในตำรามาตรฐานการเขียนโค้ดและประสบการณ์ของผู้เชี่ยวชาญ ได้เตือนถึงหลุมพรางในขั้นตอน Compilation ไว้ดังนี้ครับ:

1.  **อย่าเพิกเฉยต่อคำเตือนของ Compiler (Warnings are Errors):** ในช่วง Semantic Analysis ตัว Compiler อาจจะเจอบางอย่างที่ "ผิดสังเกตแต่พอทำงานได้" เช่น แปลงชนิดข้อมูลที่ขนาดไม่เท่ากัน (Implicit conversion) มันจะพ่น Warning ออกมา วิศวกรสายแข็งควรเปิดออปชันอย่าง `-Wall` และ `-Werror` (สำหรับ GCC/Clang) เพื่อบังคับให้ทุก Warning กลายเป็น Error เพื่อป้องกันบั๊กแอบแฝง
2.  **ระวังสงครามระหว่าง Optimizer และ Hardware:** ในขั้นตอน Code Optimization Compiler จะพยายามกำจัดโค้ดที่มันคิดว่า "ไม่มีประโยชน์ (Dead code)" ทิ้งไป หากคุณอ่านค่าเซ็นเซอร์จาก Register โดยไม่ใส่คีย์เวิร์ด `volatile` Compiler อาจจะมองว่า "ค่าไม่น่าจะเปลี่ยน จะอ่านซ้ำทำไม" แล้วดึงค่าไปเก็บใน Cache/Register ของ CPU ทำให้คุณอ่านค่าเซ็นเซอร์ใหม่ไม่ได้เลย
3.  **Compiler จับได้แค่ไวยากรณ์ (Syntax) ไม่ใช่วัตถุประสงค์ (Logic):** จงจำไว้ว่าถึงแม้การแปลโปรแกรม (Translation) จะสำเร็จจนได้ไฟล์ `.o` ออกมา มันหมายความว่าคุณเขียนภาษา C ได้ถูกหลักไวยากรณ์ (Syntax valid) เท่านั้น แต่มันไม่ได้แปลว่า "โปรแกรมจะทำงานถูกต้องตามที่คุณคิด (Logical correctness)" หากคุณสั่ง `10 / 0` (หารด้วยศูนย์) Compiler บางตัวอาจจะยอมให้ผ่าน แต่ระบบจะล่มตอนรันไทม์ (Execution-time error)
4.  **ไฟล์ Object (`.o`) ยังไม่ใช่ไฟล์ Executable:** เมื่อจบ Compilation phase (Phase 7) คุณจะได้ไฟล์ Object มา แต่มันจะรันไม่ได้! เพราะที่อยู่ (Addresses) ของฟังก์ชันที่เรียกข้ามไฟล์ (เช่น `printf` หรือฟังก์ชันในไลบรารี) ยังไม่ถูกระบุตำแหน่งที่ชัดเจน (Unresolved references) ต้องรอให้ถึงมือโปรแกรม Linker (ใน Phase 8) มาประกอบร่างมันอีกครั้งครับ

<div class="adsense-slot"></div>

## สรุป (Conclusion)

สรุปแล้ว **Compilation** คือเฟสที่หนักหน่วงที่สุดใน **Translation Process** ครับ มันคือขั้นตอนการประมวลผลกล่องดำที่แปลงกระบวนทัศน์จาก "โค้ดมนุษย์อ่าน (Source code)" ไปสู่ "โค้ดเครื่องอ่าน (Object code)" ผ่านการวิเคราะห์โครงสร้างความหมายอย่างลึกซึ้ง และการรีดเร้นประสิทธิภาพให้ได้ระดับสูงสุด การเข้าใจกลไกภายในนี้จะทำให้เราสามารถประยุกต์ใช้ออปชันของคอมไพเลอร์ และเขียนโค้ด C ที่ปลอดภัยและเร็วที่สุดได้ครับ

ในบทความหน้า เราจะพากันไปดู "ด่านสุดท้าย" ของ Translation Process นั่นก็คือ **Linker** ที่จะจับ Object Code ทั้งหมดมามัดรวมกันเป็น Firmware ลงบอร์ดกันครับ! หากเพื่อนๆ คนไหนมีประสบการณ์แก้ Error แปลกๆ จาก Compiler ฝั่ง Embedded แวะเข้ามาแชร์โค้ดและพูดคุยกันต่อได้ที่บอร์ด **www.123microcontroller.com** ของพวกเราได้เลยนะครับ แล้วพบกันใหม่บทความหน้า Happy Coding ครับทุกคน!
