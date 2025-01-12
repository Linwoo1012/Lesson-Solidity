---
title: "Re-Entrancy-1"
actions: ['checkAnswer', 'hints']
material: 
  editor:
    language: sol
    startingCode: |
    pragma solidity ^0.8.0;

    contract EtherStore {
        mapping(address => uint256) public balances;

        function deposit() public payable {
            balances[msg.sender] += msg.value;
        }

        function withdraw() public {
            uint256 bal = balances[msg.sender];
            require(bal > 0);

            // แก้ไขที่นี่
            (bool sent,) = msg.sender.call{value: bal}("");
            require(sent, "Failed to send Ether");
            balances[msg.sender] = 0;
        }

        function getBalance() public view returns (uint256) {
            return address(this).balance;
        }
    }
    answer: > 
    pragma solidity ^0.8.0;

    contract EtherStore {
        mapping(address => uint256) public balances;

        function deposit() public payable {
            balances[msg.sender] += msg.value;
        }

        function withdraw() public {
            uint256 bal = balances[msg.sender];
            require(bal > 0);

            balances[msg.sender] = 0;
            (bool sent,) = msg.sender.call{value: bal}("");
            require(sent, "Failed to send Ether"); 
        }

        function getBalance() public view returns (uint256) {
            return address(this).balance;
        }
    }
---
# Re-Entrancy

Re-Entrancy เป็นช่องโหว่ที่รู้จักกันดีใน Solidity ซึ่งได้รับการโจมตีจากการใช้ช่องโหว่นี้ในเหตุการณ์สำคัญ ๆ เช่น การแฮ็ก DAO ช่องโหว่นี้เกิดขึ้นเมื่อสัญญา (contract) ทำการเรียกไปยังสัญญาอื่น และสัญญานั้นสามารถเรียกกลับมาที่สัญญาเดิมก่อนที่การทำงานของการเรียกแรกจะเสร็จสิ้น ซึ่งทำให้นักโจมตีสามารถดึงเงินจากสัญญาเดิมได้หลายครั้งและทำให้สัญญาไม่ทำงานตามที่คาดไว้

## ตำแหน่งในโค้ด

ช่องโหว่ Re-Entrancy มักจะเกิดในตำแหน่งที่ Smart Contract เรียกไปยัง External Contract หรือ address และให้ External Contract นั้นสามารถเรียกกลับไปยังฟังก์ชันเดิมก่อนที่จะอัปเดตสถานะของ Contract ซึ่งอาจทำให้เกิดการโจมตีได้

ตัวอย่างตำแหน่งในโค้ดที่มีช่องโหว่:

``` Solidity
(bool sent,) = msg.sender.call{value: bal}("");
require(sent, "Failed to send Ether");
balances[msg.sender] = 0;
```

## ปัญหา

ช่องโหว่ในโค้ดข้างต้นเกิดขึ้นเมื่อฟังก์ชัน `withdraw` ถูกเรียก ฟังก์ชันนี้จะส่ง Ether ไปยัง `msg.sender` และหลังจากนั้นจะอัปเดตยอดเงินของผู้ใช้เป็นศูนย์ แต่หาก `msg.sender` เป็นสัญญา (contract) มันสามารถเรียกฟังก์ชัน `withdraw` กลับมาอีกครั้งในระหว่างที่การโอน Ether เกิดขึ้น ทำให้ช่องโหว่นี้สามารถใช้ในการดึงเงินออกจากสัญญาหลายครั้งก่อนที่ยอดเงินจะถูกรีเซ็ต
สถานการณ์การโจมตี

1. Alice ฝาก 10 ETH เข้าสัญญา
2. Alice เรียกฟังก์ชัน withdraw เพื่อถอน 10 ETH
3. ก่อนที่ยอดเงินจะถูกอัปเดต สัญญาของ Alice (ที่เป็นผู้โจมตี) สามารถเรียกฟังก์ชัน withdraw ซ้ำ ๆ จนกระทั่งดึงเงินจากสัญญาไปหมด

## แนวทางการป้องกัน

เปลี่ยนลำดับการอัปเดตข้อมูล โดยให้เปลี่ยนแปลงสถานะของตัวแปรภายในสัญญา แล้วจึงค่อยการโต้ตอบกับสัญญา

## ทดสอบ

ให้อัปเดต `balances[msg.sender]` ก่อนการส่ง Ether
