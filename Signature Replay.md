---
title: "Signature Replay"
actions: ['checkAnswer', 'hints']
material: 
  editor:
    language: sol
    startingCode: |
    pragma solidity ^0.8.0;

    import "github.com/OpenZeppelin/openzeppelin-contracts/blob/release-v4.5/contracts/utils/cryptography/ECDSA.sol";

    contract MultiSigWallet {
        using ECDSA for bytes32;

        address[2] public owners;

        constructor(address[2] memory _owners) payable {
            owners = _owners;
        }

        function deposit() external payable {}

        //เพิ่ม mapping ทีนี่

        //เพิ่ม Parameter ที่นี่
        function transfer(address _to, uint _amount, bytes[2] memory _sigs) external {
            //เพิ่ม _nonce ที่นี่
            bytes32 txHash = getTxHash(_to, _amount);
            //เพิ่มเงื่อนไขที่นี่

            require(_checkSigs(_sigs, txHash), "invalid sig");
            //แก้ไขค่าของ executed ที่นี่

            (bool sent, ) = _to.call{value: _amount}("");
            require(sent, "Failed to send Ether");
        }

        //เพิ่ม Parameter ที่นี่
        function getTxHash(address _to, uint _amount) public pure returns (bytes32) {
            //แก้ไขค่า hash ที่นี่
            return keccak256(abi.encodePacked(_to, _amount));
        }

        function _checkSigs(bytes[2] memory _sigs, bytes32 _txHash) private view returns (bool) {
            bytes32 ethSignedHash = _txHash.toEthSignedMessageHash();
            for (uint i = 0; i < _sigs.length; i++) {
                address signer = ethSignedHash.recover(_sigs[i]);
                bool valid = signer == owners[i];
                if (!valid) {
                    return false;
                }
            }
            return true;
        }
    }
    answer: > 
    pragma solidity ^0.8.0;

    import "github.com/OpenZeppelin/openzeppelin-contracts/blob/release-v4.5/contracts/utils/cryptography/ECDSA.sol";

    contract MultiSigWallet {
        using ECDSA for bytes32;

        address[2] public owners;

        constructor(address[2] memory _owners) payable {
            owners = _owners;
        }

        function deposit() external payable {}

        mapping(bytes32 => bool) public executed;

        function transfer(address _to, uint _amount, uint _nonce, bytes[2] memory _sigs) external {
            bytes32 txHash = getTxHash(_to, _amount, _nonce);
            require(!executed[txHash], "tx executed");
            require(_checkSigs(_sigs, txHash), "invalid sig");
            executed[txHash] = true;
            (bool sent, ) = _to.call{value: _amount}("");
            require(sent, "Failed to send Ether");
        }

        function getTxHash(address _to, uint _amount, uint _nonce) public pure returns (bytes32) {
            return keccak256(abi.encodePacked(_to, _amount, address(this), _nonce));
        }

        function _checkSigs(bytes[2] memory _sigs, bytes32 _txHash) private view returns (bool) {
            bytes32 ethSignedHash = _txHash.toEthSignedMessageHash();
            for (uint i = 0; i < _sigs.length; i++) {
                address signer = ethSignedHash.recover(_sigs[i]);
                bool valid = signer == owners[i];
                if (!valid) {
                    return false;
                }
            }
            return true;
        }
    }
---

# Signature Replay

Signature Replay เป็นช่องโหว่ที่เกิดขึ้นเมื่อสัญญายอมรับข้อความที่ลงนามแล้ว (signed messages) เพื่ออนุญาตการทำธุรกรรม แต่ไม่ได้คำนึงถึงความเป็นเอกลักษณ์ของลายเซ็น ทำให้ผู้โจมตีสามารถใช้ลายเซ็นที่ถูกต้องซ้ำหลายครั้งเพื่อทำธุรกรรมซ้ำโดยไม่ได้รับอนุญาตจากเจ้าของ

## ตำแหน่งในโค้ด

1. ฟังก์ชันไม่ได้บันทึกหรือตรวจสอบว่าธุรกรรมที่มี txHash เดียวกันนั้นเคยดำเนินการไปแล้วหรือไม่
2. Signature เดียวกันสามารถนำมาใช้ซ้ำเพื่อทำธุรกรรมซ้ำได้ (Replay Attack)

ตัวอย่างตำแหน่งในโค้ดที่มีช่องโหว่:

``` Solidity
bytes32 txHash = getTxHash(_to, _amount);
require(_checkSigs(_sigs, txHash), "invalid sig");
```

## ปัญหา

หาก signature ถูกใช้ซ้ำ ผู้โจมตีสามารถสร้างธุรกรรมที่เหมือนเดิมซ้ำ ๆ เพื่อโอน Ether ออกไปหลายครั้ง

## แนวทางการป้องกัน

ใช้ Nonce เพื่อทำให้ txHash มีเอกลักษณ์เฉพาะตัว และNonce ควรถูกบันทึกใน Contract เพื่อป้องกันการใช้ซ้ำ

## ทดสอบ

1. ให้ map จาก `bytes32 => bool` โดยใช้ชื่อว่า executed และตั้งเป็น public
2. ฟังก์ชั่น getTxHash
   1. เพิ่ม Parameter `uint _nonce`
   2. แก้ไข keccak256 โดยให้เพิ่ม `address(this)` และ `_nonce` ตามลำดับต่อจาก `_amount`
3. ฟังก์ชั่น transfer
   1. เพิ่ม Parameter `uint _nonce` ต่อจาก `uint _amount`
   2. ให้เพิ่ม `_nonce` ลงใน `getTxHash(_to, _amount)`
   3. ใช้ require เพื่อตรวจสอบ `!executed[txHash]` หากพบว่ามีการใช้ไปแล้วให้แจ้งว่า `tx executed`
   4. แก้ไขค่าของ `executed[txHash] = true`
