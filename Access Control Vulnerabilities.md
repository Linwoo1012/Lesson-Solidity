---
title: "Access Control Vulnerabilities"
actions: ['checkAnswer', 'hints']
material: 
  editor:
    language: sol
    startingCode: |
    pragma solidity ^0.8.0;

    // สร้าง modifier ที่นี่

    contract VulnerableContract {
        address public owner;
        mapping(address => uint256) public balances;

        constructor() {
            owner = msg.sender;
        }

        function deposit() public payable {
            balances[msg.sender] += msg.value;
        }

        function withdraw(uint256 amount) public {
            require(balances[msg.sender] >= amount, "Insufficient balance");
            balances[msg.sender] -= amount;
            payable(msg.sender).transfer(amount);
        }

        // เพิ่ม modifier ที่นี่
        function changeOwner(address newOwner) public {
            owner = newOwner;
        }
    }
    answer: > 
    pragma solidity ^0.8.0;

    modifier onlyOwner() {
        require(msg.sender == owner, "Not authorized");
        _;
    }

    contract VulnerableContract {
        address public owner;
        mapping(address => uint256) public balances;

        constructor() {
            owner = msg.sender;
        }

        function deposit() public payable {
            balances[msg.sender] += msg.value;
        }

        function withdraw(uint256 amount) public {
            require(balances[msg.sender] >= amount, "Insufficient balance");
            balances[msg.sender] -= amount;
            payable(msg.sender).transfer(amount);
        }

        function changeOwner(address newOwner) public onlyOwner{
            owner = newOwner;
        }
    }
---

# Access Control Vulnerabilities

Access Control Vulnerability คือช่องโหว่ที่เกิดจากการขาดการตรวจสอบสิทธิ์ (authentication) และการอนุญาต (authorization) ในการเข้าถึงฟังก์ชันบางอย่างในสัญญา ตัวอย่างเช่น ในกรณีที่ฟังก์ชันที่สำคัญ เช่น `changeOwner()` ไม่มีการตรวจสอบว่าใครสามารถเรียกใช้งานได้ อาจทำให้ผู้โจมตีสามารถเข้าถึงและเปลี่ยนแปลงข้อมูลสำคัญได้

## ตำแหน่งในโค้ด

ฟังก์ชัน changeOwner(address newOwner) ไม่มีการตรวจสอบสิทธิ์ใด ๆ ซึ่งทำให้ใครก็ได้สามารถเปลี่ยนเจ้าของของสัญญาได้

ตัวอย่างตำแหน่งในโค้ดที่มีช่องโหว่:

``` Solidity
function changeOwner(address newOwner) public {
    owner = newOwner;
}
```

## ปัญหา

หากไม่มีการควบคุมหรือกำหนดสิทธิ์การเข้าถึงฟังก์ชันและข้อมูลสำคัญในสัญญาอย่างเพียงพอ จะส่งผลให้ผู้ที่ไม่ได้รับอนุญาตสามารถเข้าถึงหรือดำเนินการบางอย่างที่สำคัญได้ เช่น การขโมยทรัพย์สินดิจิทัลจากสัญญาหรือการเปลี่ยนแปลงเจ้าของของสัญญา การขาดการตรวจสอบสิทธิ์อาจนำไปสู่การสูญเสียทรัพย์สินของผู้ใช้หรือความเสียหายทางการเงินจากการโจมตี

## แนวทางการป้องกัน

เพิ่ม modifier เพื่อตรวจสอบสิทธิ์ของผู้ที่เรียกใช้งานฟังก์ชันที่สำคัญ

## ทดสอบ

1. สร้าง modifier
   1. สร้าง modifier ที่ชื่อ onlyOwner
   2. กำหนดเงื่อนไขให้ `msg.sender == owner` หากไม่ตรงตามเงื่อนไขให้แจ้งว่า `Not authorized`
2. เพิ่ม `onlyOwner` modifier ที่ฟังก์ชั่น `changeOwner`
