---
layout: post
title: "Linker Scripts: แผนผังจัดสรรหน่วยความจำ (Memory Regions) จิ๊กซอว์ชิ้นกริชในด่านสุดท้ายของ Translation Process"
lang: th
ref: c-linker-scripts-memory-allocation
permalink: /th/c-linker-scripts-memory-allocation/
description: "เรียนรู้แนวคิดของ Linker Scripts และกลไก Memory Allocation เครื่องมือนำทางที่กำหนดที่อยู่ให้โค้ดและตัวแปรก่อนโหลดเข้าสู่ไมโครคอนโทรลเลอร์"
image: assets/images/c-linker-scripts-memory-allocation.jpg
tags: [C Programming, Embedded C, Linker Scripts, Memory Allocation, Translation Process]
---

<div class="adsense-slot"></div>

## บทนำ (Introduction)

สวัสดีครับน้องๆ วิศวกรและเพื่อนนักพัฒนาชาว www.123microcontroller.com ทุกคน! กลับมาพบกับวิศวกรขอบตาดำๆ กันอีกครั้งครับ วันนี้เราจะมาคุยกันถึงเรื่องราวบทสรุปของซีรีส์ **Translation Process** กันครับ

ในบทความก่อนหน้านี้ เราได้เห็นแล้วว่าคอมไพเลอร์ (Compiler) สร้าง Object files ขึ้นมา และ Linker ก็นำชิ้นส่วนเหล่านั้นมาเชื่อมต่อกัน (Linkage) แต่คำถามคือ... "แล้วโปรแกรมมันจะรู้ได้อย่างไรว่า โค้ดคำสั่งต้องไปอยู่ใน Flash Memory แอดเดรสไหน? แล้วตัวแปรต้องไปอยู่ใน RAM แอดเดรสอะไร?" คำตอบก็คือ Linker ทั่วไปจะไม่รู้ครับ! มันจึงต้องการ "เครื่องมือนำทาง" หรือแผนผังโฉนดที่ดินที่เรียกว่า **Linker Script** เข้ามาช่วยในการจัดสรรพื้นที่ (Memory Allocation) นั่นเองครับ 

เรื่องนี้ถือเป็น "ความลับสวรรค์" ที่โปรแกรมเมอร์สาย PC อาจจะไม่เคยต้องยุ่ง แต่สำหรับวิศวกรสมองกลฝังตัว (Embedded Systems) อย่างพวกเรา การเขียน Linker Script เป็นสิ่งที่แยกแยะระหว่างมือใหม่กับมือโปรเลยล่ะครับ! วันนี้เราจะมาเจาะลึกกันว่าคัมภีร์ระดับโลกอธิบายเรื่องนี้ไว้อย่างไร ไปลุยกันเลยครับ!

## เนื้อหาหลัก (Core Concept): การทำงานของ Locator และ Linker Script

ในบริบทของ Translation Process เมื่อ Linker ทำการประกอบ Object files เข้าด้วยกันสำเร็จ เราจะได้ไฟล์ที่เรียกว่า Relocatable file ซึ่งสมบูรณ์ทุกอย่าง ยกเว้นแค่ "ยังไม่มีการระบุตำแหน่งหน่วยความจำ (Unresolved addresses)" ในขั้นตอนนี้ เครื่องมือที่เรียกว่า **Locator** (ซึ่งใน GNU Toolchain อย่าง `ld` จะควบรวมอยู่ในตัว Linker เลย) จะเข้ามารับช่วงต่อ โดยมันจะอ่านข้อมูลจาก **Linker Script** เพื่อนำ Code และ Data ไปวางลงใน Memory Map ของไมโครคอนโทรลเลอร์เป้าหมายให้ถูกต้อง

ในการเขียน Linker Script เราจะต้องยุ่งกับโครงสร้างหลัก 2 ส่วน และประเภทของข้อมูล 3 กลุ่มใหญ่ๆ (Segments/Sections) ดังนี้ครับ:

**1. กลุ่มของข้อมูล (Sections/Segments):**
Linker จะมองข้อมูลในโปรแกรมของเราออกเป็นหมวดหมู่ (เหมือนการแยกประเภทห้องในบ้าน) ได้แก่:
*   **`.text` Segment:** พื้นที่เก็บ "โค้ดคำสั่ง (Executable instructions)" และข้อมูลคงที่ (Constant data) ส่วนนี้เป็นแบบ Read-only และมักจะถูกจับไปวางไว้ใน ROM หรือ Flash Memory
*   **`.data` Segment:** พื้นที่เก็บ "ตัวแปร Global หรือ Static ที่มีค่าเริ่มต้น (Initialized variables)" เช่น `int count = 10;` ส่วนนี้ต้องการพื้นที่ใน RAM เพื่อให้โปรแกรมแก้ไขค่าได้
*   **`.bss` Segment:** พื้นที่เก็บ "ตัวแปรที่ไม่มีค่าเริ่มต้น (Uninitialized variables)" ชื่อนี้ย่อมาจาก *Block Started by Symbol* ส่วนนี้จะอยู่ใน RAM แต่ความเจ๋งคือมันไม่กินพื้นที่ในไฟล์ Executable (.bin/.hex) เลย Linker แค่จด "ขนาด" เอาไว้ เพื่อให้ Startup code ไปเคลียร์พื้นที่ใน RAM ให้เป็น 0 ตอนบูตเครื่อง

**2. โครงสร้างคำสั่งใน Linker Script:**
*   **`MEMORY` Directive:** เป็นการประกาศให้ Linker ทราบว่า "ชิปตัวนี้มีหน่วยความจำฮาร์ดแวร์อะไรบ้าง เริ่มที่แอดเดรสไหน และขนาดเท่าไหร่"
*   **`SECTIONS` Directive:** เป็นตัวกำหนดว่า จะนำ Section ต่างๆ (เช่น `.text`, `.data`) ไปวางลงในพื้นที่ไหนของ `MEMORY` ที่ประกาศไว้

**3. ความลับของ LMA และ VMA:**
เรื่องนี้สำคัญมากในการทำ Embedded! เนื่องจากแอดเดรสมี 2 ประเภท:
*   **LMA (Load Memory Address):** ตำแหน่งที่ข้อมูลถูก "บันทึก" เก็บไว้จริงๆ ในตอนที่ปิดไฟ (มักจะเป็น Flash)
*   **VMA (Virtual/Runtime Memory Address):** ตำแหน่งที่ข้อมูลจะถูก "ดึงไปใช้งาน" จริงๆ ตอนโปรแกรมรัน (มักจะเป็น SRAM)
สำหรับโค้ด (`.text`) LMA และ VMA มักจะอยู่ที่ Flash เหมือนกัน แต่สำหรับ `.data` นั้น LMA จะอยู่ใน Flash (เพื่อเก็บค่าเริ่มต้นตอนปิดเครื่อง) แต่ VMA จะอยู่ใน SRAM ซึ่งซอร์สโค้ดใน C Startup ไฟล์ จะทำหน้าที่ก็อปปี้ข้อมูลจาก LMA ไปยัง VMA ก่อนที่จะเรียกฟังก์ชัน `main()` ครับ

![C Linker Script and Memory Allocation Architecture]({{ site.baseurl }}/assets/images/c-linker-scripts-memory-allocation-diagram.jpg)

## ตัวอย่างโค้ด (Code Example):

มาดูหน้าตาของ Linker Script แบบคลีนๆ สำหรับไมโครคอนโทรลเลอร์ที่มี Flash และ SRAM กันครับ สังเกตการใช้คำสั่ง `AT>` เพื่อแยก LMA และ VMA ของส่วน `.data` นะครับ

```ld
/* ==========================================
 * ตัวอย่าง Linker Script พื้นฐานสำหรับ Embedded C
 * ========================================== */

/* 1. กำหนดจุดเริ่มต้นของโปรแกรม (Entry Point) มักจะเป็นฟังก์ชันใน C Startup */
ENTRY(Reset_Handler)

/* 2. ประกาศสเปคของฮาร์ดแวร์ (MEMORY Map) */
MEMORY
{
    /* Flash เริ่มที่ 0x08000000 ขนาด 256K (อ่านและรันโค้ดได้ rx) */
    FLASH (rx) : ORIGIN = 0x08000000, LENGTH = 256K
    
    /* SRAM เริ่มที่ 0x20000000 ขนาด 64K (อ่าน เขียน และรันได้ rwx) */
    SRAM (rwx) : ORIGIN = 0x20000000, LENGTH = 64K
}

/* 3. จัดสรร Section ต่างๆ ลงใน Memory */
SECTIONS
{
    /* นำส่วนโค้ด (.text) ไปไว้ใน FLASH */
    .text :
    {
        *(.isr_vector)  /* ตาราง Interrupt Vector ต้องอยู่ต้นสุดเสมอ */
        *(.text)        /* โค้ดภาษา C ทั้งหมดจากทุกๆ Object file */
        *(.rodata)      /* ตัวแปร Read-only เช่น const */
        . = ALIGN(4);   /* 🛡️ จัดเรียง Address ให้อยู่ในขอบเขต 4 ไบต์ (Word Alignment) */
        _etext = .;     /* สร้างตัวแปร (Symbol) เก็บ Address สุดท้ายของ .text */
    } >FLASH

    /* นำส่วนตัวแปรมีค่าเริ่มต้น (.data) มี VMA อยู่ใน SRAM แต่ถูกโหลดเก็บ LMA ไว้ใน FLASH */
    .data :
    {
        . = ALIGN(4);
        _sdata = .;     /* สร้าง Symbol ชี้จุดเริ่มต้นของ .data ใน RAM */
        *(.data)
        . = ALIGN(4);
        _edata = .;     /* สร้าง Symbol ชี้จุดสิ้นสุดของ .data ใน RAM */
    } >SRAM AT> FLASH   /* >SRAM คือ VMA, AT> FLASH คือ LMA */

    /* นำส่วนตัวแปรไม่มีค่าเริ่มต้น (.bss) ไปไว้ใน SRAM อย่างเดียว */
    .bss :
    {
        . = ALIGN(4);
        _sbss = .;
        *(.bss)
        . = ALIGN(4);
        _ebss = .;
    } >SRAM
}
```
*(ในไฟล์ C Startup (เช่น `startup.c` หรือ `startup.s`) จะมีการใช้สัญลักษณ์อย่าง `_etext`, `_sdata`, `_edata` และ `_sbss` ที่สร้างโดย Linker นี้ เพื่อเป็นพอยน์เตอร์ในการทำ Memory Initialization ก่อนเข้า `main()`)*

## ข้อควรระวัง / Best Practices:

การปรับแต่ง Linker Script เป็นเหมือนการทุบโครงสร้างบ้าน หากทำพลาดโปรแกรมอาจจะพังตั้งแต่ยังไม่ทันเข้า `main()` ด้วยซ้ำ! มาตรฐานการทำงานแนะนำ Best Practices ไว้ดังนี้ครับ:

1.  **อย่าเขียนใหม่จากศูนย์ (Don't write from scratch):** กฎข้อแรกเลยครับ! ผู้ผลิตชิปหรือ Compiler (เช่น GCC, Keil, IAR) มักจะเตรียมไฟล์ Linker Script มาตรฐานนามสกุล `.ld` หรือ `.icf` มาให้พร้อมกับโปรเจกต์อยู่แล้ว ให้เรานำไฟล์นั้นมาแก้ไข (Modify) เฉพาะส่วนที่จำเป็นจะปลอดภัยที่สุด
2.  **ระวังเรื่อง Alignment:** ในสถาปัตยกรรมระดับ 32-bit อย่าง ARM Cortex-M การพยายามอ่าน/เขียนข้อมูลที่ไม่ได้จัดเรียงแอดเดรสให้หาร 4 ลงตัว (Unaligned memory access) อาจทำให้เกิด Hardware Fault จนระบบค้างได้ ดังนั้นต้องใส่คำสั่ง `. = ALIGN(4);` เพื่อจัดระเบียบขอบเขตของ Section เสมอ
3.  **การอ่าน Map File คือคัมภีร์แก้บั๊กชั้นยอด:** เมื่อคุณเจอ Error "Memory Overflow" (เช่น โค้ดใหญ่เกิน Flash) หรืออยากรู้ว่าฟังก์ชันไหนกินที่เยอะ ให้ไปเปิดไฟล์ `.map` ที่ Linker สร้างขึ้นมา ไฟล์นี้จะบอกละเอียดเลยว่าตัวแปรไหน หรือโค้ดบรรทัดไหน ไปตกอยู่ที่แอดเดรสอะไร และกินพื้นที่ไปกี่ไบต์ (Size)
4.  **ป้องกันถูก Optimize ทิ้งด้วย `KEEP`:** หากคุณมีโค้ดสำคัญที่ไม่เคยถูกเรียกใช้ตรงๆ ใน C code (เช่น ตาราง Interrupt Vector หรือ `__attribute__((section("...")))`) ลิงก์เกอร์สุดฉลาดอาจจะมองว่า "ไม่มีคนใช้ งั้นลบทิ้งละกัน!" เพื่อเป็นการรีดประสิทธิภาพ วิธีป้องกันคือให้ใส่คำสั่ง `KEEP(*(.isr_vector))` ครอบเอาไว้ใน Linker Script เสมอ เพื่อบังคับให้ Linker เก็บมันไว้นั่นเองครับ

<div class="adsense-slot"></div>

## สรุป (Conclusion)

**Linker Scripts** คือหัวใจสำคัญในขั้นสุดท้ายของการเนรมิตซอร์สโค้ดภาษา C ให้กลายเป็นเฟิร์มแวร์ที่ทำงานได้จริงบนตัวฮาร์ดแวร์ครับ มันทำหน้าที่ผสานโลกของซอฟต์แวร์ (Object files) เข้ากับสเปกทางกายภาพ (Memory regions) เช่น Flash และ SRAM ทั้งยังเป็นจุดกำเนิดของสัญลักษณ์ต่างๆ ที่ Startup Code นำไปใช้เตรียมสภาพแวดล้อมให้ไมโครคอนโทรลเลอร์พร้อมทำงานครับ 

มาถึงตรงนี้ น้องๆ ก็ได้เห็นภาพรวมของ Translation Process ตั้งแต่ Preprocessing, Compilation ยัน Linkage ครบถ้วนแล้ว! หวังว่าต่อไปนี้เวลาสั่ง Build โปรเจกต์ เราจะเข้าใจทุกสิ่งที่วิ่งอยู่หลังฉากกันมากขึ้นนะครับ ใครที่มีโปรเจกต์สนุกๆ อยากจัดการ Memory พิลึกๆ (เช่น เอาข้อมูลลง External SDRAM) หรือมีคำถามเกี่ยวกับ Map File อย่าลืมแวะเข้ามาแชร์โค้ดตั้งกระทู้คุยกันต่อได้ที่บอร์ด **www.123microcontroller.com** ของพวกเราได้เลยนะครับ! แล้วพบกันใหม่บทความหน้า Happy Coding ครับทุกคน!
