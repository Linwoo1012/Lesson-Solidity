---
title: "Logic Errors"
actions: ['checkAnswer', 'hints']
material: 
  editor:
    language: sol
    startingCode: |
    pragma solidity ^0.8.0;

    contract LendingPlatform {
        mapping(address => uint256) public userBalances;
        uint256 public totalLendingPool;

        function deposit() public payable {
            userBalances[msg.sender] += msg.value;
            totalLendingPool += msg.value;
        }

        //เพิ่ม event ที่นี่

        function withdraw(uint256 amount) public {
            require(userBalances[msg.sender] >= amount, "Insufficient balance");
            userBalances[msg.sender] -= amount;
            //เพิ่มการอัปเดต totalLendingPool ที่นี่

            //Emit event ที่นี่

            payable(msg.sender).transfer(amount);
        }
    }
    answer: > 
    pragma solidity ^0.8.0;

    contract LendingPlatform {
        mapping(address => uint256) public userBalances;
        uint256 public totalLendingPool;

        function deposit() public payable {
            userBalances[msg.sender] += msg.value;
            totalLendingPool += msg.value;
        }

        event Withdraw(address indexed user, uint256 amount, uint256 remainingPool);

        function withdraw(uint256 amount) public {
            require(userBalances[msg.sender] >= amount, "Insufficient balance");
            userBalances[msg.sender] -= amount;
            totalLendingPool -= amount;

            payable(msg.sender).transfer(amount);
        }
    }
---

# Logic Errors

Logic Errors เป็นข้อผิดพลาดที่เกิดขึ้นเมื่อโค้ดของสัญญาไม่ตรงกับพฤติกรรมที่ตั้งใจไว้ ช่องโหว่ประเภทนี้มีความละเอียดอ่อน มักค้นพบเมื่อมีการใช้งาน

## ตำแหน่งในโค้ด

เมื่อผู้ใช้งานถอนเงิน สัญญาอัพเดตยอดคงเหลือของผู้ใช้ `userBalances[msg.sender]` แต่ ไม่ได้ลดยอดรวมของเงินในกองทุน `totalLendingPool` ตามจำนวนที่ถอนออกไป

ตัวอย่างตำแหน่งในโค้ดที่มีช่องโหว่:

``` Solidity
userBalances[msg.sender] -= amount;
payable(msg.sender).transfer(amount);
```

## ปัญหา

ผู้โจมตีสามารถใช้วิธีการเช่น การสร้าง fallback function ที่ใช้ gas มากเกินไป หรือทำให้การโอนเงินล้มเหลว ส่งผลให้ smart contract ไม่สามารถดำเนินการสำคัญ เช่น การส่ง Ether หรือการอัปเดตข้อมูลได้ ซึ่งอาจกระทบต่อผู้ใช้งานและระบบแบบกระจายศูนย์ (dApps) อย่างรุนแรง

## แนวทางการป้องกัน

1. สร้างกรณีทดสอบที่หลากหลายเพื่อครอบคลุมทุกความเป็นไปได้ของตรรกะทางธุรกิจ เพื่อให้มั่นใจว่าโค้ดทำงานได้ตามที่ตั้งใจ
2. ดำเนินการตรวจสอบโค้ดอย่างถี่ถ้วนและจัดการการตรวจสอบความปลอดภัย เพื่อระบุและแก้ไขข้อผิดพลาดที่อาจเกิดขึ้น

## ทดสอบ

1. แก้ไขโดยเพิ่มการอัปเดต totalLendingPool
2. เพิ่ม Event สำหรับการติดตาม
   1. สร้าง event โดยใช้ชื่อว่า Withdraw โดยมี `address indexed user, uint256 amount, uint256 remainingPool`
   2. Emit event หลังจากอัปเดตข้อมูล `emit Withdraw(msg.sender, amount, totalLendingPool)`
