---
title: "Front-running Attacks"
actions: ['checkAnswer', 'hints']
material: 
  editor:
    language: sol
    startingCode: |
    pragma solidity ^0.8.0;

    contract FindThisHash {
    
        bytes32 public constant hash = 0x564ccaf7594d66b1eaaea24fe01f0585bf52ee70852af4eac0cc4b04711cd0e2;

        //เพิ่ม mapping และ ตัวแปร bool ที่นี่

        constructor() payable {}

        //เพิ่ม function commitSolution ที่นี่

        //เพิ่ม function revealSolution ทีนี่

        function solve(string memory solution) public {
            require(
                hash == keccak256(abi.encodePacked(solution)), "Incorrect answer"
            );

            (bool sent,) = msg.sender.call{value: 10 ether}("");
            require(sent, "Failed to send Ether");
        }
    }
    answer: > 
    pragma solidity ^0.8.0;

    contract FindThisHash {
    
        bytes32 public constant hash = 0x564ccaf7594d66b1eaaea24fe01f0585bf52ee70852af4eac0cc4b04711cd0e2;

        mapping(address => bytes32) public commitments;
        bool public winnerFound = false;

        constructor() payable {}

        function commitSolution(bytes32 commitment) public {
            require(!winnerFound, "Winner already found! Contract is closed.");
            commitments[msg.sender] = commitment;
        }

        function revealSolution(string memory solution, string memory salt) public {
            require(!winnerFound, "Winner already found! Contract is closed.");
            require(commitments[msg.sender] != 0, "No commitment found");

            bytes32 saltHash = stringToBytes32(salt);

            bytes32 senderHash = keccak256(abi.encodePacked(msg.sender, solution, saltHash));
            require(commitments[msg.sender] == senderHash, "Incorrect commitment");
            require(hash == keccak256(abi.encodePacked(solution)), "Incorrect solution");

            commitments[msg.sender] = 0;

            (bool sent, ) = msg.sender.call{value: 10 ether}("");
            require(sent, "Failed to send Ether");

            winnerFound = true;
        }
    }
---

# Front-running Attacks

Front-running เป็นการโจมตีที่เกิดขึ้นเมื่อผู้ไม่หวังดีดักจับธุรกรรมที่ยังไม่ได้รับการยืนยันใน `mempool` (พื้นที่เก็บธุรกรรมที่รอดำเนินการ) และส่งธุรกรรมของตัวเองที่มีค่าธรรมเนียมสูงกว่าเพื่อให้ถูกยืนยันก่อน โดยในกรณีนี้โจมตีเพื่อรับรางวัล 10 ETH

## ตำแหน่งในโค้ด

ฟังก์ชัน changeOwner(address newOwner) ไม่มีการตรวจสอบสิทธิ์ใด ๆ ซึ่งทำให้ใครก็ได้สามารถเปลี่ยนเจ้าของของสัญญาได้

ตัวอย่างตำแหน่งในโค้ดที่มีช่องโหว่:

``` Solidity
require(hash == keccak256(abi.encodePacked(solution)), "Incorrect answer");
(bool sent, ) = msg.sender.call{value: 10 ether}("");
ตำแหน่งที่มีปัญหา:
hash == keccak256(abi.encodePacked(solution)):
```

## ปัญหา

ในกรณีนี้ ผู้โจมตีสามารถขโมย `10 ETH` จากสัญญาได้โดยใช้กลยุทธ์แซงหน้าผู้ที่แก้โจทย์จริงใน `mempool` โดยไม่ต้องพิจารณาว่าใครเป็นผู้แก้โจทย์ก่อน ส่งผลให้สัญญาไม่สามารถแยกแยะระหว่างผู้ที่แก้โจทย์จริงกับผู้โจมตีที่ใช้วิธีดังกล่าว ความน่าเชื่อถือของระบบลดลงเนื่องจากความยุติธรรมถูกบั่นทอน ผู้ใช้ที่ตั้งใจแก้โจทย์อย่างถูกต้องอาจไม่ได้รับรางวัลที่ควรเป็นของตนเอง

## แนวทางการป้องกัน

ใช้วิธีการ Commit-Reveal Scheme เพื่อป้องกันการเปิดเผยคำตอบล่วงหน้า

## ทดสอบ

1. ลบฟังก์ชั่น `solve`
2. สร้าง mapping ให้ map จาก `address => bytes32` โดยใช้ชื่อว่า commitments และตั้งเป็น public
3. สร้างฟังก์ชั่น `commitSolution`
   1. รับ `bytes32 commitment` และตั้งเป็น public
   2. สร้าง require เช็ค `!winnerFound` หากไม่ตรงตามเงื่อนไขให้แจ้งว่า "Winner already found! Contract is closed."
   3. เก็บค่า `commitments[msg.sender] = commitment;`
4. สร้างฟังก์ชั่น `revealSolution`
   1. รับ `string memory solution, string memory salt` และตั้งเป็น public
   2. สร้าง require เช็ค `!winnerFound` หากไม่ตรงตามเงื่อนไขให้แจ้งว่า "Winner already found! Contract is closed."
   3. สร้าง require เช็ค `commitments[msg.sender] != 0` เพื่อตรวจสอบว่า commot มาหรือไม่ หากไม่ตรงตามเงื่อนไขให้แจ้งว่า "No commitment found"
   4. สร้างตัวแปร `bytes32 saltHash` เพื่อแปลง `salt` ให้เป็น byte32 โดยใช้ `stringToBytes32(salt)` เพื่อนำไปใช้ hash
   5. สร้างตัวแปร `bytes32 senderHash` รับค่า hash `msg.sender, solution, saltHash` โดยใช้ `abi.encodePacked` และ `keccak256`
   6. สร้าง require เช็ค `commitments[msg.sender] == senderHash` เพื่อตรวจสอบว่าคำตอบที่ส่งไป มีค่าเท่ากับค่าที่ส่งมายืนยันหรือไม่หากไม่ตรงตามเงื่อนไขให้แจ้งว่า "Incorrect commitment"
   7. สร้าง require เช็ค `hash == keccak256(abi.encodePacked(solution)` หากไม่ตรงตามเงื่อนไขให้แจ้งว่า "Incorrect solution"
   8. ปรับแก้ `commitments[msg.sender]` ให้กลับเป็น 0
   9. ส่ง Ether ไปยังผู้ชนะ `(bool sent, ) = msg.sender.call{value: 10 ether}(""); require(sent, "Failed to send Ether");`
   10. เปลี่ยน `bool winnerFound` เป็น true

## เพิ่มเติม

หากต้องการนำไปทดสอบจริง ๆ เช่น Remix ให้เพิ่มฟังก์ชั่นดังนี้

แปลงเป็น byte32 เพื่อป้อนลงฟังก์ชั่น commitSolution

``` Solidity
function generateCommitment(string memory solution, string memory salt) public view returns (bytes32) {
    bytes32 saltHash = stringToBytes32(salt);
    return keccak256(abi.encodePacked(msg.sender, solution, saltHash));
}
```

แปลงเป็น string เป็น byte32

``` Solidity
function stringToBytes32(string memory str) public pure returns (bytes32) {
    bytes32 result;
    assembly {
        result := mload(add(str, 32))
    }
    return result;
}
```
