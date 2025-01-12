---
title: "Hiding Malicious Code with External Contract"
actions: ['checkAnswer', 'hints']
material: 
  editor:
    language: sol
    startingCode: |
    pragma solidity ^0.8.0;

    contract Foo {
        Bar bar;

        constructor(address _bar) {
            bar = Bar(_bar);
        }

        function callBar() public {
            bar.log();
        }
    }

    contract Bar {
        event Log(string message);

        function log() public {
            emit Log("Bar was called");
        }
    }

    // This code is hidden in a separate file
    contract Mal {
        event Log(string message);

        // function () external {
        //     emit Log("Mal was called");
        // }

        // Actually we can execute the same exploit even if this function does
        // not exist by using the fallback
        function log() public {
            emit Log("Mal was called");
        }
    }
    answer: > 
    pragma solidity ^0.8.0;

    contract Foo {
        Bar bar;

        constructor(address _bar) {
            bar = Bar(_bar);
        }

        function callBar() public {
            bar.log();
        }
    }

    contract Bar {
        event Log(string message);

        function log() public {
            emit Log("Bar was called");
        }
    }

    // This code is hidden in a separate file
    contract Mal {
        event Log(string message);

        // function () external {
        //     emit Log("Mal was called");
        // }

        // Actually we can execute the same exploit even if this function does
        // not exist by using the fallback
        function log() public {
            emit Log("Mal was called");
        }
    }
---

# Hiding Malicious Code with External Contract

ใน Solidity สามารถทำการแปลง address ให้เป็นสัญญาได้เสมอไม่ว่า address นั้นจะเป็นสัญญาจริงหรือไม่ก็ตาม ซึ่งทำให้นักโจมตีสามารถหลอกลวงสัญญาให้ทำงานกับสัญญาที่ไม่ปลอดภัยหรือทำให้เกิดพฤติกรรมที่ไม่พึงประสงค์ได้

## ตำแหน่งในโค้ด

ช่องโหว่อยู่ที่การอ้างอิงไปยังสัญญาภายนอก (Bar) ในตัวสร้าง (constructor) ของ Foo

``` Solidity
constructor(address _bar) {
    bar = Bar(_bar);
}
```

ผู้โจมตีสามารถตั้งค่าที่อยู่ _bar เป็นสัญญาที่เป็นอันตราย (Mal) แทนที่จะเป็น Bar ซึ่งจะทำให้โค้ดที่เป็นอันตรายถูกรันเมื่อเรียกใช้ Foo.callBar()

``` Solidity
function callBar() public {
    bar.log();
}
```

## ปัญหา

ผู้ใช้อาจถูกทำให้เชื่อว่ากำลังใช้โค้ดที่ถูกต้อง แต่ในความเป็นจริงกำลังเรียกใช้โค้ดที่เป็นอันตราย

## แนวทางการป้องกัน

1. การเปิดเผยที่อยู่ของสัญญาภายนอก เพื่อให้ผู้ใช้งานสามารถตรวจสอบโค้ดของสัญญานั้นได้
2. แทนที่จะรับที่อยู่ของสัญญาภายนอก ให้สร้างสัญญาใหม่ภายในตัวสร้างเพื่อให้มั่นใจว่าไม่มีการแอบแฝงโค้ดที่ไม่พึงประสงค์

## ทดสอบ

แก้ไข require ที่ฟังก์ชั่น protected จาก `!isContract(msg.sender)` เป็น `msg.sender == tx.origin`
