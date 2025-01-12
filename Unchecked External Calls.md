---
title: "Unchecked External Calls"
actions: ['checkAnswer', 'hints']
material: 
  editor:
    language: sol
    startingCode: |
    pragma solidity ^0.8.0;

    contract CallExample {
        function sendEther(address payable recipient) public payable {
            //แก้ไขและเพิ่มการตรวจสอบที่นี่
            recipient.call{value: msg.value}("");
        }
    }
    answer: > 
    pragma solidity ^0.8.0;

    contract CallExample {
        function sendEther(address payable recipient) public payable {
            (bool success,) = recipient.call{value: msg.value}("");
            require(success, "Call failed");
        }
    }
---

# Unchecked External Calls

Unchecked External Calls เป็นช่องโหว่ที่เกิดจากการเรียกฟังก์ชันของ External Calls โดยไม่ตรวจสอบผลลัพธ์ของการเรียกใช้งาน เช่น การไม่ตรวจสอบว่า Call สำเร็จหรือไม่ ส่งผลให้สัญญาอาจดำเนินการต่อโดยสันนิษฐานว่าการโอนสำเร็จ

## ตำแหน่งในโค้ด

ไม่มีการตรวจสอบว่า recipient.call สำเร็จหรือไม่

ตัวอย่างตำแหน่งในโค้ดที่มีช่องโหว่:

``` Solidity
function sendEther(address payable recipient) public payable {
    recipient.call{value: msg.value}("");
}
```

## ปัญหา

หากไม่มีการตรวจสอบผลลัพธ์ของการเรียกใช้งานสัญญาภายนอก (เช่น call หรือ send) ธุรกรรมอาจล้มเหลวโดยไม่สามารถแจ้งเตือนหรือจัดการข้อผิดพลาดได้อย่างเหมาะสม

## แนวทางการป้องกัน

1. ตรวจสอบค่า Return Value ทุกครั้ง โดยใช้ฟังก์ชัน require เพื่อตรวจสอบผลลัพธ์ของการเรียกฟังก์ชัน
2. หากเป็นไปได้ ให้ใช้ transfer() แทน send() เนื่องจาก transfer() จะย้อนกลับธุรกรรมหากการเรียกภายนอกล้มเหลว `recipient.transfer(msg.value);`

## ทดสอบ

1. ให้สร้าง bool ที่ชื่อว่า success เพื่อรับผลลัพธ์ `(bool X,) = Y.call{value: msg.value}("");`
2. ใช้ require ตรวจสอบตัวแปร success หากไม่สำเร็จให้แจ้งว่า `Call failed`
