---
title: "Insecure Randomness"
actions: ['checkAnswer', 'hints']
material: 
  editor:
    language: sol
    startingCode: |
    pragma solidity ^0.8.0;

    contract InsecureRandomNumber {
        constructor() payable {}

        function guess(uint256 _guess) public {
            uint256 answer = uint256(
                keccak256(
                    abi.encodePacked(block.timestamp, block.difficulty, msg.sender)
                ) 
            );

            if (_guess == answer) {
                (bool sent,) = msg.sender.call{value: 1 ether}("");
                require(sent, "Failed to send Ether");
            }
        }
    }
    answer: > 
    pragma solidity ^0.8.0;

    contract InsecureRandomNumber {
        constructor() payable {}

        function guess(uint256 _guess) public {
            uint256 answer = uint256(
                keccak256(
                    abi.encodePacked(block.timestamp, block.difficulty, msg.sender)
                ) 
            );

            if (_guess == answer) {
                (bool sent,) = msg.sender.call{value: 1 ether}("");
                require(sent, "Failed to send Ether");
            }
        }
    }
---

# Insecure Randomness

Insecure Randomness เป็นช่องโหว่ที่เกิดจากการสุ่มค่าที่สามารถคาดเดาได้ง่าย เนื่องจาก Solidity ไม่สามารถสุ่มเลขจริง ๆ ได้ จึงต้องใช้ pseudorandom

ค่าที่นิยมใช้งานและส่งผลให้เกิดช่องโหว่

1. `block.timestamp` ไทม์สแตมป์ของบล็อกปัจจุบัน
2. `blockhash(uint blockNumber)` แฮชของบล็อกที่กำหนด (สำหรับ 256 บล็อกสุดท้ายเท่านั้น)
3. `block.difficulty` ความยากของบล็อกปัจจุบัน
4. `block.number` หมายเลขบล็อกปัจจุบัน
5. `block.coinbase` ที่อยู่ของ miner ของบล็อกปัจจุบัน

## ตำแหน่งในโค้ด

การใช้ `block.timestamp` และ `block.difficulty` ในการสร้างตัวเลขสุ่มอยู่ภายในฟังก์ชัน guess ไม่ใช่การสุ่มที่ปลอดภัย เพราะ miners สามารถปรับเปลี่ยนค่า `block.timestamp` และมีการคำนวณ `block.difficulty` ตามกฎของระบบ

ตัวอย่างตำแหน่งในโค้ดที่มีช่องโหว่:

``` Solidity
uint256 answer = uint256(
    keccak256(
        abi.encodePacked(block.timestamp, block.difficulty, msg.sender)
    )
);
```

## ปัญหา

ผู้โจมตีสามารถใช้วิธีการเช่น การสร้าง fallback function ที่ใช้ gas มากเกินไป หรือทำให้การโอนเงินล้มเหลว ส่งผลให้ smart contract ไม่สามารถดำเนินการสำคัญ เช่น การส่ง Ether หรือการอัปเดตข้อมูลได้ ซึ่งอาจกระทบต่อผู้ใช้งานและระบบแบบกระจายศูนย์ (dApps) อย่างรุนแรง

## แนวทางการป้องกัน

1. การใช้ Oracles (เช่น Oraclize)
   Oracles ให้ค่า randomness จากแหล่งภายนอก แต่ต้องระมัดระวังในการพึ่งพาความน่าเชื่อถือ และสามารถใช้ Oracles หลายตัวเพื่อเพิ่มความปลอดภัย
2. Commitment Schemes
   ใช้กลไกแบบ commit-reveal เพื่อสร้าง randomness ที่ปลอดภัย มีการใช้งานอย่างกว้างขวางใน coin flipping, zero-knowledge proofs และการคำนวณแบบปลอดภัย เช่น RANDAO
3. Chainlink VRF
   เครื่องมือสร้างตัวเลขสุ่ม (RNG) ที่ยืนยันความยุติธรรมและความปลอดภัยได้ ช่วยให้ smart contracts เข้าถึง randomness โดยไม่ลดทอนความปลอดภัยหรือความสะดวก
4. Signidice Algorithm
   เหมาะสำหรับ pseudo-random number generation (PRNG) ในแอปพลิเคชันที่มีผู้ใช้สองฝ่าย โดยใช้ลายเซ็นดิจิทัลเพื่อสร้าง randomness
5. Bitcoin Block Hashes
   ใช้ block hashes จาก Bitcoin Blockchain ผ่าน Oracles เช่น BTCRelay เพื่อสร้าง entropy อย่างไรก็ตาม วิธีนี้มีความเสี่ยงจากปัญหาแรงจูงใจของนักขุด (miner incentive problem) จึงควรใช้อย่างระมัดระวัง

## ทดสอบ

เนื่องจากมีหลายวิธีในการป้องกัน
สำหรับข้อนี้จึงสามารถผ่านไปได้เลย หากคุณต้องการศึกษาเพิ่มเติม แนะนำให้ศึกษาวิธีใช้งาน Chainlink VRF
