## GAS-1: <X> <= <Y> costs more gas than <X> < <Y> + 1

### Description

In Solididy, the opcode 'less or equal' doesn't exist. So the EVM will translate this by two distinct operation, first the inferior, and then the equal which cost more gas then a strict less.

### Affected file

* Bio.sol (Line: 77, 78, 79)
* Tray.sol (Line: 124, 184, 240)

## GAS-2: Functions guaranteed to revert when called by normal users can be marked payable

### Description

If a function modifier such as onlyOwner is used, the function will revert if a normal user tries to pay the function. Marking the function as payable will lower the gas cost for legitimate callers because the compiler will not include checks for whether a payment was provided.

### Affected file

* Namespace.sol (Line: 196, 204)
* Tray.sol (Line: 210, 218, 225)

## GAS-3: Public functions not called by the contract should be declared external instead

### Description

Contracts are allowed to override their parents’ functions and change the visibility from external to public and can save gas by doing so. 

### Affected file

* Bio.sol (Line: 43)
* Namespace.sol (Line: 90)
* Tray.sol (Line: 119)

## GAS-4: Usage of uint/int smaller than 32 bytes (256 bits) incurs overhead

### Description

When using elements that are smaller than 32 bytes, your contract’s gas usage may be higher. This is because the EVM operates on 32 bytes at a time. Therefore, if the element is smaller than that, the EVM must use more operations in order to reduce the size of the element from 32 bytes to the desired size.
Each operation involving a uint8 costs an extra 22-28 gas (depending on whether the other operand is also a variable of type uint8) as compared to ones involving uint256, due to the compiler having to clear the higher bits of the memory word before operating on the uint8, as well as the associated stack operations of doing so. Use a larger size then downcast where needed.

### Affected file

* Tray.sol (Line: 195, 250, 252, 255, 259, 261, 263, 267)

## GAS-5: Using Solidity version 0.8.17 will provide an overall gas optimization

### Description

Using at least 0.8.10 will save gas due to skipped extcodesize check if there is a return value.

### Affected file

* Bio.sol (Line: 2)
* Namespace.sol (Line: 2)
* ProfilePicture.sol (Line: 2)
* Tray.sol (Line: 2)

## GAS-6: abi.encode() is less efficient than abi.encodePacked()

### Description

Use abi.encodePacked() where possible to save gas.

### Affected file

* Tray.sol (Line: 162)