---
title: "Integer Overflow and Underflow-1"
actions: ['checkAnswer', 'hints']
material: 
  editor:
    language: sol
    startingCode: |
    pragma solidity ^0.7.0;

    contract OverflowExample {

        function testOverflow() public pure returns (uint256) {
            uint256 max = type(uint256).max;
            return max + 1; // Causes overflow
        }

        function testUnderflow() public pure returns (uint256) {
            uint256 zero = 0;
            return zero - 1; // Causes underflow
        }
    }
    answer: > 
    pragma solidity ^0.8.0;

    contract OverflowExample {

        function testOverflow() public pure returns (uint256) {
            uint256 max = type(uint256).max;
            return max + 1; // Causes overflow
        }

        function testUnderflow() public pure returns (uint256) {
            uint256 zero = 0;
            return zero - 1; // Causes underflow
        }
    }
---

# Integer Overflow and Underflow

Integer Overflow และ Integer Underflow เป็นช่องโหว่ที่เกิดขึ้นใน Solidity เมื่อค่าของตัวแปรเกินขีดจำกัดที่สามารถเก็บได้ ซึ่งอาจทำให้เกิดผลลัพธ์ที่ไม่คาดคิดในสัญญาและนำไปสู่การโจมตี

1. Integer Overflow เกิดขึ้นเมื่อค่าของตัวแปรเพิ่มขึ้นจนเกินขีดจำกัดสูงสุดที่สามารถเก็บได้ (เช่น การเพิ่มค่าตัวแปร uint256 ที่มีค่าเป็น type(uint256).max จะทำให้เกิดการ Overflow กลับไปที่ค่าต่ำสุด)
2. Integer Underflow เกิดขึ้นเมื่อค่าของตัวแปรลดลงต่ำกว่าค่าต่ำสุดที่สามารถเก็บได้ (เช่น การลบ 1 จากตัวแปร uint256 ที่มีค่าเป็น 0 จะทำให้เกิดการ Underflow)

## ตำแหน่งในโค้ด

ช่องโหว่ Integer Overflow และ Integer Underflow มักเกิดขึ้นเมื่อมีการคำนวณที่เกี่ยวข้องกับการเพิ่มหรือลบค่าในตัวแปรที่มีขีดจำกัดของชนิดข้อมูล (เช่น uint256) โดยไม่ได้ตรวจสอบขีดจำกัดก่อนทำการคำนวณ

ตัวอย่างตำแหน่งในโค้ดที่มีช่องโหว่:

1. Overflow
   `return max + 1;`
2. Underflow
   `return zero - 1;`

## ปัญหา

1. เมื่อเกิด Overflow ค่าผลลัพธ์จะถูกรีเซ็ตกลับไปที่ค่าต่ำสุดของประเภทข้อมูล ซึ่งอาจทำให้เกิดการคำนวณที่ผิดพลาดและผลกระทบที่ไม่คาดคิดในสัญญา
2. เมื่อเกิด Underflow ค่าผลลัพธ์อาจจะเป็นค่ามหาศาลที่ไม่คาดคิด ซึ่งทำให้เกิดการคำนวณที่ผิดพลาดและอาจทำให้สัญญาทำงานไม่เป็นไปตามที่คาดไว้

## แนวทางการป้องกัน

ใช้ Solidity 0.8.x หรือสูงกว่า เนื่องจากตัว Compiler จะมีการตรวจจับ Overflow และ Underflow โดยอัตโนมัติ และทำให้ธุรกรรมล้มเหลว

## ทดสอบ

ให้แก้ไขเวอร์ชั่นของ Solidity เป็น `^0.8.0`
