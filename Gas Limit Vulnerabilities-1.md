---
title: "Gas Limit Vulnerabilities-1"
actions: ['checkAnswer', 'hints']
material: 
  editor:
    language: sol
    startingCode: |
    pragma solidity ^0.8.0;

    contract GasLimitExample {
        uint256[] public data;

        function populateArray(uint256 n) public {
            for (uint256 i = 0; i < n; i++) {
                //เพิ่มเงื่อนไขการตรวจสอบ Gas ที่เหลืออยู่

                data.push(i);
            }
        }

        function getDataCount() public view returns (uint256) {
            return data.length;
        }
    }
    answer: > 
    pragma solidity ^0.8.0;

    contract GasLimitExample {
        uint256[] public data;

        function populateArray(uint256 n) public {
            for (uint256 i = 0; i < n; i++) {
                if (gasleft() < 10000) {
                    revert("Gas limit too low, aborting");
                }    
                data.push(i);
            }
        }

        function getDataCount() public view returns (uint256) {
            return data.length;
        }
    }
---

# Gas Limit Vulnerabilities

Gas Limit Vulnerability เกิดจากการใช้งานปริมาณแก๊สมากเกินการปริมาณสูงสุดที่สามารถใช้ได้ในบล็อก หากการดำเนินการสัญญาเกินขีดจำกัดนี้ อาจนำไปสู่การทำธุรกรรมล้มเหลว โดยช่องโหว่ประเภทนี้มักพบได้บ่อยใน loop

## ตำแหน่งในโค้ด

ฟังก์ชันนี้มีการวนลูป ตามจำนวน n ที่ผู้เรียกฟังก์ชันกำหนด หากค่า n มีขนาดใหญ่มาก การดำเนินการอาจต้องใช้ gas จำนวนมากจนเกินกว่าที่ block จะรองรับ ส่งผลให้ธุรกรรมล้มเหลว

ตัวอย่างตำแหน่งในโค้ดที่มีช่องโหว่:

``` Solidity
for (uint256 i = 0; i < n; i++) {
    data.push(i);
}
```

## ปัญหา

ทำให้เกิด Denial of Service (DoS) ผู้โจมตีสามารถโจมตี Smart Contract โดยการเรียกฟังก์ชันที่ใช้ Gas Limit สูง ๆ เพื่อทำให้ Gas ใช้หมด

## แนวทางการป้องกัน

ตรวจสอบ Gas Consumption โดยใช้ gasleft() ที่เป็นฟังก์ชันที่ใช้ตรวจสอบ Gas ที่เหลืออยู่ เพื่อตรวจสอบว่า Gas เหลืออยู่เพียงพอหรือไม่ระหว่างการทำงาน

## ทดสอบ

ให้ใช้ if โดยกำหนดหาก `gasleft() < 10000` ให้ `revert("Gas limit too low, aborting")`
