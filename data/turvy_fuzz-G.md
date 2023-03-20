 ### > Use assembly to write storage values

unoptimized
`function updateOwner(address newOwner) public {
    owner = newOwner;
}
}`
optimized
`function assemblyUpdateOwner(address newOwner) public {
    assembly {
        sstore(owner.slot, newOwner)
    }
}`

 **https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L107**

 `revenueAddress = _revenueAddress;`
 `namespaceNFT = _namespaceNFT;`

**https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Namespace.sol#L206**
**https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Namespace.sol#L80**

### > <x> += <y> costs more gas than <x> = <x> + <y> for state variables
Using the addition operator instead of plus-equals saves 113 gas.

**https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Namespace.sol#L150**

### > Reorder struct

**https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L57**

Unoptimized
**struct TileData {
 uint8 fontClass;
 uint16 characterIndex;
 uint8 characterModifier;
}**

Optimized
**struct TileData {
 uint16 characterIndex;
 uint8 fontClass;
 uint8 characterModifier;
}**

### > Use assembly to hash instead of Solidity

**contract Contract0 {
  function solidityHash(uint256 a, uint256 b) public view {
    //unoptimized
    `keccak256(abi.encodePacked(a, b));`
}**

**contract Contract1 {
function assemblyHash(uint256 a, uint256 b) public view {
//optimized
`assembly {
mstore(0x00, a)
mstore(0x20, b)
let hashedVal := keccak256(0x00, 0x40)}`
}**

**https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L162**