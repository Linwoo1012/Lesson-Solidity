---
title: "Self Destruct"
actions: ['checkAnswer', 'hints']
material: 
  editor:
    language: sol
    startingCode: |
    pragma solidity ^0.8.0;

    contract EtherGame {
        uint256 public constant TARGET_AMOUNT = 7 ether;
        //เพิ่มตัวแปร balance ที่นี่ 

        address public winner;

        function deposit() public payable {
            require(msg.value == 1 ether, "You can only send 1 Ether");

            //แก้ไขการเปลี่ยนแปลง balance ที่นี่
            uint256 balance = address(this).balance;
            require(balance <= TARGET_AMOUNT, "Game is over");

            if (balance == TARGET_AMOUNT) {
                winner = msg.sender;
            }
        }

        function claimReward() public {
            require(msg.sender == winner, "Not winner");
            //เพิ่มตัวแปร amount แก้ไข balance และ value ที่นี่
            (bool sent,) = msg.sender.call{value: address(this).balance}("");
            require(sent, "Failed to send Ether");
        }
    }

    contract Attack {
        EtherGame etherGame;

        constructor(EtherGame _etherGame) {
            etherGame = EtherGame(_etherGame);
        }

        function attack() public payable {
            address payable addr = payable(address(etherGame));
            selfdestruct(addr);
        }
    }
    answer: > 
    pragma solidity ^0.8.0;

    contract EtherGame {
        uint256 public constant TARGET_AMOUNT = 7 ether;
        uint256 public balance;
        address public winner;


        function deposit() public payable {
            require(msg.value == 1 ether, "You can only send 1 Ether");

            balance += msg.value;
            require(balance <= TARGET_AMOUNT, "Game is over");


            if (balance == TARGET_AMOUNT) {
                winner = msg.sender;
            }
        }

        function claimReward() public {
            require(msg.sender == winner, "Not winner");
            uint256 amount = balance;
            balance = 0;
            (bool sent,) = msg.sender.call{value: amount}("");
            require(sent, "Failed to send Ether");
        }
    }

    contract Attack {
        EtherGame etherGame;

        constructor(EtherGame _etherGame) {
            etherGame = EtherGame(_etherGame);
        }

        function attack() public payable {
            address payable addr = payable(address(etherGame));
            selfdestruct(addr);
        }
    }
---

# Self Destruct

การใช้คำสั่ง selfdestruct ในสัญญาโจมตีเพื่อโอน Ether ไปยังสัญญาเป้าหมายโดยตรง โดยไม่สนใจเงื่อนไขที่กำหนดในสัญญาเป้าหมาย

## ตำแหน่งในโค้ด

ตัวอย่างตำแหน่งในโค้ดที่มีช่องโหว่:

`uint256 balance = address(this).balance;`

## ปัญหา

selfdestruct สามารถใช้สร้างข้อมูลผิดพลาดใน Blockchain เช่น การส่ง Ether ไปยังสัญญาเป้าหมายที่ไม่คาดคิด

## แนวทางการป้องกัน

การใช้ `address(this).balance` เพื่อตรวจสอบยอดคงเหลือของสัญญานั้นอาจทำให้เกิดข้อผิดพลาดหาก Ether ถูกโอนเข้าสัญญาโดยตรงผ่านฟังก์ชัน selfdestruct
ใช้ state variable ควบคุม balance อย่างชัดเจนและปรับเปลี่ยนเฉพาะผ่านฟังก์ชันที่ตั้งใจไว้

## ทดสอบ

1. ให้สร้าง uint256 ชื่อว่า balance และให้ตั้งเป็น public
2. ฟังก์ชั่น deposit
   1. แก้ไข `uint256 balance = address(this).balance` เป็นเพิ่มค่า balance จาก msg.value
3. ฟังก์ชั่น claimReward
   1. สร้าง uint256 amount และให้มีค่าเท่ากับ balance
   2. แก้ไข balance ให้เป็น 0
   3. เปลี่ยน value ที่ call จาก `address(this).balance` เป็น `amount`
