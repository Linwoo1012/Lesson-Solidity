---
title: "Denial of Service (DoS) Attacks"
actions: ['checkAnswer', 'hints']
material: 
  editor:
    language: sol
    startingCode: |
    pragma solidity ^0.8.0;

    contract KingOfEther {
        address public king;
        uint256 public balance;
        //ใช้ mapping ที่นี่

        function claimThrone() external payable {
            require(msg.value > balance, "Need to pay more to become the king");

            //เปลี่ยนเป็นการอัปเดตยอดเงินที่นี่
            (bool sent,) = king.call{value: balance}("");
            require(sent, "Failed to send Ether");

            balance = msg.value;
            king = msg.sender;
        }

        //เพิ่มฟังก์ชั่น withdraw ที่นี่
    }
    answer: > 
    pragma solidity ^0.8.0;

    contract KingOfEther {
        address public king;
        uint256 public balance;
        mapping(address => uint256) public balances;

        function claimThrone() external payable {
            require(msg.value > balance, "Need to pay more to become the king");

            balances[king] += balance;

            balance = msg.value;
            king = msg.sender;
        }

        function withdraw() public {
            require(msg.sender != king, "Current king cannot withdraw");

            uint256 amount = balances[msg.sender];
            balances[msg.sender] = 0;

            (bool sent,) = msg.sender.call{value: amount}("");
            require(sent, "Failed to send Ether");
        }
    }
---

# Denial of Service (DoS) Attacks

Denial of Service เกิดขึ้นเมื่อผู้โจมตีจงใจขัดขวางการทำงานปกติของระบบ ทำให้ระบบไม่สามารถใช้งานได้สำหรับผู้ใช้ตามเป้าหมาย ในบริบทของสัญญาอัจฉริยะ Solidity การโจมตีแบบ DoS เกี่ยวข้องกับการใช้ประโยชน์จากช่องโหว่เพื่อใช้ทรัพยากรต่างๆ เช่น แก๊ส รอบการทำงานของ CPU หรือพื้นที่จัดเก็บจนหมด ทำให้สัญญาไม่สามารถใช้งานได้

ตัวอย่างของ DoS ที่พบบ่อยใน Solidity

1. DoS ด้วยการรันฟังก์ชันลูปที่ยาวเกิน
2. DoS ผ่านการบล็อกการทำงานของฟังก์ชันสำคัญ
3. DoS ด้วยการโจมตี Reentrancy

## ตำแหน่งในโค้ด

Contract พยายามโอน Ether ให้กับ king (เจ้าของเดิมของบัลลังก์) แต่หาก contract ปัจจุบันของ king ไม่อนุญาตให้รับ Ether (เช่น โดยการแทรกโค้ดที่ล้มเหลวใน fallback function) จะทำให้ธุรกรรมล้มเหลวและการดำเนินการถูกบล็อก

ตัวอย่างตำแหน่งในโค้ดที่มีช่องโหว่:

``` Solidity
(bool sent,) = king.call{value: balance}("");
require(sent, "Failed to send Ether");
```

## ปัญหา

ผู้โจมตีสามารถใช้วิธีการเช่น การสร้าง fallback function ที่ใช้ gas มากเกินไป หรือทำให้การโอนเงินล้มเหลว ส่งผลให้ smart contract ไม่สามารถดำเนินการสำคัญ เช่น การส่ง Ether หรือการอัปเดตข้อมูลได้ ซึ่งอาจกระทบต่อผู้ใช้งานและระบบแบบกระจายศูนย์ (dApps) อย่างรุนแรง

## แนวทางการป้องกัน

เปลี่ยนจากการโอน Ether โดยตรง เป็นการถอนเงิน

## ทดสอบ

1. สร้าง mapping
   1. ให้ map จาก `address => uint256` โดยใช้ชื่อว่า balances และตั้งเป็น public
2. เปลี่ยนการอัปเดตยอดเงินเป็น `balances[king] += balance;`
3. สร้างฟังก์ชัน withdraw
   1. ให้ตั้งชื่อฟังก์ชั่นว่า withdraw และตั้งเป็น public
   2. ตรวจสอบว่า msg.sender ไม่ใช่ราชาคนปัจจุบัน `(msg.sender != king)`หากพบว่าไม่ตรงตามเงื่อนไขให้แจ้งว่า `Current king cannot withdraw`
   3. ให้ uint256 amount มีค่าเท่ากับ `balances[msg.sender];` จากนั้นให้ `balances[msg.sender] = 0;`
   4. ใช้ .call ในการส่ง Ether และตรวจสอบผลลัพธ์ว่าการส่งสำเร็จหรือไม่ หาไม่สำเร็จให้แจ้งว่า `Failed to send Ether`
