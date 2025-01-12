---
title: "Phishing with tx.origin"
actions: ['checkAnswer', 'hints']
material: 
  editor:
    language: sol
    startingCode: |
    pragma solidity ^0.8.0;

    contract Wallet {
        address public owner;

        constructor() payable {
            owner = msg.sender;
        }

        function transfer(address payable _to, uint256 _amount) public {
            //แก้ไขที่นี่
            require(tx.origin == owner, "Not owner");

            (bool sent, ) = _to.call{value: _amount}("");
            require(sent, "Failed to send Ether");
        }
    }

    contract Attack {
        address payable public owner;
        Wallet wallet;

        constructor(Wallet _wallet) {
            wallet = Wallet(_wallet);
            owner = payable(msg.sender);
        }

        function attack() public {
            wallet.transfer(owner, address(wallet).balance);
        }
    }
    answer: > 
    pragma solidity ^0.8.0;

    contract Wallet {
        address public owner;

        constructor() payable {
            owner = msg.sender;
        }

        function transfer(address payable _to, uint256 _amount) public {
            require(msg.sender == owner, "Not owner");

            (bool sent, ) = _to.call{value: _amount}("");
            require(sent, "Failed to send Ether");
        }
    }

    contract Attack {
        address payable public owner;
        Wallet wallet;

        constructor(Wallet _wallet) {
            wallet = Wallet(_wallet);
            owner = payable(msg.sender);
        }

        function attack() public {
            wallet.transfer(owner, address(wallet).balance);
        }
    }
---

# Phishing with tx.origin

tx.origin Authentication Vulnerability เกิดขึ้นเนื่องจาก `tx.origin` เป็นตัวแปร Global ใน Solidity ที่ใช้ดึงที่อยู่ (address) ของบัญชี EOA (Externally Owned Account) ตัวต้นทางที่ทำการเริ่มต้นธุรกรรม
หากใช้ `tx.origin` สำหรับตรวจสอบสิทธิ์และการยืนยันตัวตน จะทำให้ผู้ใช้งานเสี่ยงต่อการโจมตีจากการหลอกลวง (Phishing attacks)

ลำดับเหตุการณ์ที่แสดงให้เห็นการโจมตี

1. Alice สร้างและปรับใช้สัญญา Wallet พร้อมกับ 10 Ether
   1. Alice เป็นเจ้าของสัญญา (owner = Alice)
   2. Alice ฝาก 10 Ether เข้าในสัญญา Wallet
2. Eve สร้างและปรับใช้สัญญา Attack โดยกำหนดที่อยู่ของสัญญา Wallet ของ Alice
   1. Eve สร้างสัญญา Attack และส่งที่อยู่ของ Wallet ของ Alice เป็นพารามิเตอร์ให้กับ constructor ของ Attack
   2. สัญญา Attack มีฟังก์ชัน attack ซึ่งเรียกใช้ฟังก์ชัน transfer ของ Wallet
3. Eve หลอก Alice ให้เรียกใช้ฟังก์ชัน attack ในสัญญา Attack
   1. Eve อาจใช้วิธีการทางสังคม (social engineering) เช่นการส่งลิงก์หรือการเขียนโปรแกรมให้ Alice เรียกใช้ฟังก์ชัน attack
   2. เมื่อ Alice เรียกใช้ฟังก์ชัน attack, tx.origin จะเป็นที่อยู่ของ Alice (EOA ที่เริ่มต้นธุรกรรม)
4. Eve โอน Ether จาก Wallet ของ Alice ไปยังบัญชีของเธอเองได้สำเร็จ
    1. ฟังก์ชัน transfer ใน Wallet ตรวจสอบ require(tx.origin == owner) ผ่านสำเร็จ เนื่องจาก tx.origin คือ Alice (ซึ่งเป็นเจ้าของ)
    2. Wallet ดำเนินการโอน Ether ไปยังที่อยู่ของ Eve (owner ใน Attack) ตามที่ฟังก์ชัน attack ระบุไว้
    3. Ether 10 หน่วยใน Wallet ถูกโอนไปยัง Eve

## ตำแหน่งในโค้ด

ตัวอย่างตำแหน่งในโค้ดที่มีช่องโหว่:

`require(tx.origin == owner, "Not owner");`

## ปัญหา

tx.origin ดึงที่อยู่ของบัญชี EOA ที่เริ่มต้นธุรกรรม แต่ไม่ได้ตรวจสอบว่าเป็นผู้เรียกฟังก์ชัน (msg.sender) ที่มีสิทธิ์โดยตรง

## แนวทางการป้องกัน

เปลี่ยนจากการใช้งาน `tx.origin` เป็น `msg.sender` ซึ่งระบุผู้เรียกธุรกรรมคนล่าสุดแทน เมื่อต้องการยืนยันตัวตน

## ทดสอบ

แก้ไข require ให้ตรงตามแนวทางการป้องกัน
