---
layout: post
title: "เจาะลึก STM32 Boot Process: เกิดอะไรขึ้นก่อนเข้า main()?"
lang: th
ref: stm32-boot-process
permalink: /stm32-boot-process/
description: "สรุปกระบวนการ Boot ของ STM32 และความลับของ Vector Table อธิบายเรื่อง Reset Sequence, Memory Map และการทำงานของ Startup Code แบบเข้าใจง่าย"
image: assets/images/stm32-boot-cover.jpg
tags: [STM32, Embedded Systems, ARM Cortex-M, Bare Metal]
---

<div class="adsense-slot"></div>

# เจาะลึก STM32 Boot Process: ความลับที่ซ่อนอยู่ก่อนบรรทัดแรก

สวัสดีครับนักเรียนวิชา Embedded ทุกท่าน! วันนี้เราจะมาเจาะลึกหัวข้อที่เป็น "จุดกำเนิด" ของทุกสรรพสิ่งในไมโครคอนโทรลเลอร์ นั่นคือ **กระบวนการ Boot (Boot Sequence)** และ **ตารางเวกเตอร์ (Vector Table)** ครับ

ถ้าเปรียบเทียบ `main()` คือฉากเปิดของภาพยนตร์ สิ่งที่เราจะคุยกันวันนี้คือ "งานเบื้องหลังก่อนถ่ายทำ" ครับ เนื้อหานี้ประยุกต์มาจากเอกสารทางเทคนิคอย่าง *Mastering STM32* เพื่อให้เห็นภาพชัดเจนในบริบทของการทำงานจริง

## 1. ปริศนา 4 ไบต์แรก: CPU เริ่มทำงานอย่างไร? (The Reset Sequence)

หลายคนเข้าใจว่าพอกด Reset ปุ๊บ CPU จะวิ่งไปที่ `main()` ปั๊บ... ผิดถนัดครับ! ฮาร์ดแวร์ของ Cortex-M มีขั้นตอนศักดิ์สิทธิ์ในการ "ตื่นนอน" ดังนี้:

1.  **Hardware Fetch:** ทันทีที่ไฟเข้า CPU จะวิ่งไปอ่านข้อมูลจาก Memory Address `0x0000 0000` เป็นที่แรก ค่าที่อ่านได้จะถูกนำไปใส่ใน **MSP (Main Stack Pointer)** เพื่อเตรียมพื้นที่ Stack
2.  **Fetch Reset Vector:** จากนั้น CPU จะขยับไปอ่านที่ Address `0x0000 0004` ค่าตรงนี้คือที่อยู่ของฟังก์ชัน `Reset_Handler`
3.  **Jump:** CPU กระโดดไปทำงานที่ `Reset_Handler` ซึ่งเป็นคำสั่งแรกที่จะรันจริง

**Note:** นี่คือเหตุผลว่าทำไม Vector Table ต้องอยู่ตรงจุดเริ่มต้นของ Memory เสมอครับ

![แผนภาพอธิบายขั้นตอนการดึงข้อมูล MSP และ Reset Vector ของ CPU]({{ site.baseurl }}/assets/images/stm32-boot-diagram.jpg)

## 2. Boot Modes: การสลับร่างของ Memory (Aliasing)

คุณอาจสงสัยว่า *"อ้าว! Flash ของ STM32 มันเริ่มที่ `0x0800 0000` ไม่ใช่เหรอ? แล้ว CPU ไปอ่าน `0x0000 0000` ได้ไง?"*

คำตอบคือเทคนิคที่เรียกว่า **Memory Aliasing** ครับ เมื่อเราเซตขา **BOOT0** และ **BOOT1** ฮาร์ดแวร์จะทำการ "วาร์ป" (Map) หน่วยความจำที่เราเลือก มาวางทับที่ `0x0000 0000` ให้ CPU เห็น:

* **Boot from Flash:** เอา Flash (`0x0800 0000`) มาแปะที่ Address 0 (โหมดปกติ)
* **Boot from System Memory:** เอา ROM ภายใน (ที่มี Bootloader จากโรงงาน) มาแปะแทน
* **Boot from SRAM:** เอา RAM มาแปะ (ใช้สำหรับ Debug เร็วๆ)

## 3. Vector Table: แผนที่นำทางยามฉุกเฉิน

Vector Table คืออาเรย์ที่เก็บที่อยู่ของฟังก์ชันสำคัญๆ (Exception Handlers & ISRs) เอาไว้ครับ โดยโครงสร้างมาตรฐานเป็นดังนี้:

| Offset | หน้าที่ (Function) | คำอธิบาย |
| :--- | :--- | :--- |
| **0x00** | Initial SP | ค่าเริ่มต้นของ Stack Pointer |
| **0x04** | Reset | จุดเริ่มต้นโปรแกรม (`Reset_Handler`) |
| **0x08** | NMI | เหตุการณ์ร้ายแรงที่ห้ามมองข้าม |
| **0x0C** | HardFault | เมื่อโค้ดพัง (เช่น เข้า Memory ผิด) จะมาที่นี่ |
| **0x40+** | IRQs | Interrupt ภายนอก (Timer, UART, etc.) |

## 4. Reset Handler: พ่อบ้านผู้จัดเตรียมบ้าน

ก่อนจะถึง `main()` ต้องมีคนมาจัดบ้านให้เรียบร้อยก่อน คนคนนั้นคือ **Reset Handler** (อยู่ในไฟล์ `startup_stm32xxxx.s`) หน้าที่คือ:

1.  **Copy .data:** ย้ายค่าตัวแปรที่มีค่าเริ่มต้น (เช่น `int a = 5;`) จาก Flash มาใส่ RAM
2.  **Zero .bss:** เคลียร์ค่าตัวแปรที่ไม่มีค่าเริ่มต้น (เช่น `int b;`) ใน RAM ให้เป็น 0 ทั้งหมด
3.  **SystemInit:** ตั้งค่า Clock เบื้องต้น
4.  **Call main():** ส่งไม้ต่อให้ฟังก์ชัน `main()` ของเรา

<div class="adsense-slot"></div>

## 5. เทคนิคขั้นสูง: Vector Table Relocation (VTOR)

สำหรับการทำ **Custom Bootloader** หรือการรัน RTOS เราอาจจำเป็นต้องย้าย Vector Table ไปไว้ที่อื่น

* **Cortex-M3/M4 (STM32F4 เป็นต้นไป):** ง่ายมาก! มีรีจิสเตอร์ชื่อ `SCB->VTOR` เราสามารถเขียนบอก CPU ได้เลยว่าตารางย้ายไปอยู่ที่ไหน
* **Cortex-M0 (STM32F0):** ไม่มี VTOR! นี่คือจุดปราบเซียน ต้องใช้วิธี Copy ตารางไปไว้ที่ RAM แล้วสั่ง Hardware Remap ให้ RAM มาบังที่ Address 0 แทน

## สรุป (Verdict)

ความเข้าใจเรื่อง Boot & Vector Table แยกมือสมัครเล่นออกจากมืออาชีพครับ:
* ถ้าบอร์ดนิ่งสนิท เช็ค **BOOT Pins**
* ถ้าตัวแปรเริ่มต้นมั่ว เช็ค **Linker Script**
* ถ้าทำ **Bootloader** ต้องแม่นเรื่อง **VTOR**

หวังว่าภาพจิ๊กซอว์ชิ้นสำคัญนี้จะช่วยให้คุณเข้าใจการทำงานของ STM32 ได้ลึกซึ้งยิ่งขึ้นนะครับ!