---
title: "Timestamp Dependence"
actions: ['checkAnswer', 'hints']
material: 
  editor:
    language: sol
    startingCode: |
    pragma solidity ^0.8.0;

    contract Roulette {
        uint256 public pastBlockTime;

        constructor() payable {}

        function spin() external payable {
            require(msg.value == 10 ether); // must send 10 ether to play
            require(block.timestamp != pastBlockTime); // only 1 transaction per block

            pastBlockTime = block.timestamp;

            if (block.timestamp % 15 == 0) {
                (bool sent,) = msg.sender.call{value: address(this).balance}("");
                require(sent, "Failed to send Ether");
            }
        }
    }
    answer: > 
    pragma solidity ^0.8.0;

    contract Roulette {
        uint256 public pastBlockTime;

        constructor() payable {}

        function spin() external payable {
            require(msg.value == 10 ether); // must send 10 ether to play
            require(block.timestamp != pastBlockTime); // only 1 transaction per block

            pastBlockTime = block.timestamp;

            if (block.timestamp % 15 == 0) {
                (bool sent,) = msg.sender.call{value: address(this).balance}("");
                require(sent, "Failed to send Ether");
            }
        }
    }
---

# Timestamp Dependence

Timestamp Dependence เกิดจากการใช้ block.timestamp ที่เป็นค่าที่มักถูกใช้ใน Smart Contract เพื่อบันทึกเวลาหรือกำหนดเงื่อนไขที่ขึ้นอยู่กับเวลา แม้ว่าค่า block.timestamp จะดูเหมือนค่าที่เชื่อถือได้ แต่ค่าดังกล่าวสามารถปรับเปลี่ยนได้เล็กน้อยโดย miner

## ตำแหน่งในโค้ด

ตัวอย่างตำแหน่งในโค้ดที่มีช่องโหว่:

1. การตรวจสอบ Timestamp ซ้ำ
`require(block.timestamp != pastBlockTime); // only 1 transaction per block`
แม้จะช่วยป้องกันการทำธุรกรรมซ้ำในบล็อกเดียว แต่ยังคงใช้ block.timestamp
2. เงื่อนไขการชนะเกม
   `if (block.timestamp % 15 == 0) {`
   ผู้เล่นสามารถใช้ประโยชน์ของ block.timestamp เพื่อเพิ่มโอกาสให้เงื่อนไขนี้เป็นจริง

## ปัญหา

1. Miner สามารถควบคุมค่าของ block.timestamp ได้ภายในขอบเขตที่อนุญาต เช่น ± 15 วินาที ซึ่งอาจทำให้ผู้โจมตีสามารถควบคุมเวลาของการทำธุรกรรมในสัญญาได้
2. หาก block.timestamp ถูกใช้ในการเลือกผู้ชนะ (หรือการตัดสินใจอื่น ๆ )ผู้โจมตีอาจคำนวณเวลาและทำให้ผลลัพธ์เป็นไปตามที่ต้องการ

## แนวทางการป้องกัน

1. หลีกเลี่ยงการใช้ block.timestamp สำหรับการตัดสินใจในธุรกรรมสำคัญ เช่น การกำหนดผลลัพธ์ทางการเงิน หรือการเลือกผู้ชนะ
2. ใช้ Oracles หรือ Randomness Service เช่น Chainlink VRF เพื่อช่วยลดโอกาสที่ผู้โจมตีจะปรับเปลี่ยนผลลัพธ์ได้

## ทดสอบ

เนื่องจากการป้องกัน Timestamp Dependence นั้นไม่ตายตัว สำหรับข้อนี้จึงสามารถผ่านไปได้เลย หากคุณต้องการศึกษาเพิ่มเติม แนะนำให้ศึกษาวิธีใช้งาน Chainlink VRF
