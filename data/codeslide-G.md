### Gas Optimizations List

| Number | Optimization Details                      | Context |
| :----: | :---------------------------------------- | :-----: |
| [G-01] | Unnecessary computation                   |    7    |
| [G-02] | Avoid using bool                          |    3    |
| [G-03] | Use a constant instead of type(uintX).max |    5    |
| [G-04] | Mark functions as payable                 |    5    |

Total 4 issues

#### [G-01] Unnecessary computation

The subtraction `lengthInBytes - 1` is done once outside and twice inside the `for` loop in Bio.sol. The value of `lengthInBytes` does not change within the loop. It would be better to perform this calculation once, store the result in a `memory` variable, and use the variable where the subtractions are used today.

1. [Bio.sol Line 49](https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-bio-protocol/src/Bio.sol#L49)
   ```solidity
   49    uint lines = (lengthInBytes - 1) / 40 + 1;
   ```
1. [Bio.sol Line 60](https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-bio-protocol/src/Bio.sol#L60)
   ```solidity
   60    if ((i > 0 && (i + 1) % 40 == 0) || prevByteWasContinuation || i == lengthInBytes - 1) {
   ```
1. [Bio.sol Line 62](https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-bio-protocol/src/Bio.sol#L62)
   ```solidity
   62    if (i != lengthInBytes - 1) {
   ```

Addition operations based on the `for` loop index `i` are done more than once in the `for` loop. It would be better to perform each calculation once, store the result in a memory variable, and use the variable where the additions are used today.

1. [Bio.sol Line 60, i + 1](https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-bio-protocol/src/Bio.sol#L60)
   ```solidity
   60    if ((i > 0 && (i + 1) % 40 == 0) || prevByteWasContinuation || i == lengthInBytes - 1) {
   ```
1. [Bio.sol Line 63, i + 1](https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-bio-protocol/src/Bio.sol#L63)
   ```solidity
   63    nextCharacter = bioTextBytes[i + 1];
   ```
1. [Bio.sol Line 73, i + 2, i + 3](https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-bio-protocol/src/Bio.sol#L73)
   ```solidity
   73    (nextCharacter == 0xE2 && bioTextBytes[i + 2] == 0x80 && bioTextBytes[i + 3] == 0x8D) ||
   ```
1. [Bio.sol Lines 75-78, i + 2, i + 3, i + 4](https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-bio-protocol/src/Bio.sol#L75-78)
   ```solidity
   75    bioTextBytes[i + 2] == 0x9F &&
   76    bioTextBytes[i + 3] == 0x8F &&
   77    uint8(bioTextBytes[i + 4]) >= 187 &&
   78    uint8(bioTextBytes[i + 4]) <= 191) ||
   ```

#### [G-02] Avoid using bool

Use `uint256(1)` and `uint256(2)` for `true`/`false` rather than `bool` to avoid a `Gwarmaccess` (100 gas), and to avoid `Gsset` (20000 gas) when changing from `false` to `true`, after having been `true` in the past. See [OpenZeppelin comment](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/58f635312aa21f947cae5f8578638a85aa2519f5/contracts/security/ReentrancyGuard.sol#L23-L27) for more details.

1. [Bio.sol Line 51](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-bio-protocol/src/Bio.sol#L51)
1. [Namespace.sol Line 123](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Namespace.sol#L123)
1. [Utils.sol Line 82](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Utils.sol#L82)

#### [G-03] Use a constant instead of type(uintX).max

`type(uint256).max` (for any uint size) uses more gas in the distribution process and also for each transaction than usage of a constant.

1. [Tray.sol Line 73](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L73)
1. [Tray.sol Line 152](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L152)
1. [Tray.sol Line 184](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L184)
1. [Tray.sol Line 226](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L226)
1. [Tray.sol Line 238](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L238)

#### [G-04] Mark functions as payable

Functions guaranteed to revert when called by normal users can be marked `payable`. If a function modifier such as `onlyOwner` is used, the function will revert if a normal user tries to pay the function. Marking the function as payable will lower the gas cost for legitimate callers because the compiler will not include checks for whether a payment was provided. The extra opcodes avoided are CALLVALUE(2), DUP1(3), ISZERO(3), PUSH2(3), JUMPI(10), PUSH1(3), DUP1(3), REVERT(0), JUMPDEST(1), POP(2) which costs an average of about 21 gas per call to the function, in addition to the extra deployment cost.

1. [Namespace.sol Line 196](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Namespace.sol#L196) `changeNoteAddress()`
1. [Namespace.sol Line 204](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Namespace.sol#L204) `changeRevenueAddress()`
1. [Tray.sol Line 210](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Namespace.sol#L210) `changeNoteAddress()`
1. [Tray.sol Line 218](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Namespace.sol#L218) `changeRevenueAddress()`
1. [Tray.sol Line 225](https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Namespace.sol#L225) `endPrelaunchPhase()`
