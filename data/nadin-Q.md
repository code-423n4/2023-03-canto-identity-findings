## Non-Critical Issues List
### Issue 
## [N-01] Use of bytes.concat() instead of abi.encodePacked(,)
## [N-02] NON-LIBRARY/INTERFACE FILES SHOULD USE FIXED COMPILER VERSIONS, NOT FLOATING ONES
## [N-03] Function writing that does not comply with the Solidity Style Guide
## [N-04] Constants should be defined rather than using magic numbers
## [N-05] Use a single file for all system-wide constants
## [N-06] Include return parameters in NatSpec comments
## [N-07] Use descriptive names for Contracts and Libraries
## [N-08] SOLIDITY COMPILER OPTIMIZATIONS CAN BE PROBLEMATIC
### Total:  08 issues
## [N-01] Use of bytes.concat() instead of abi.encodePacked(,)
Rather than using abi.encodePacked for appending bytes, since version 0.8.4, bytes.concat() is enabled
Since version 0.8.4 for appending bytes, bytes.concat() can be used instead of abi.encodePacked(,).
```
File: canto-bio-protocol/src/Bio.sol
116:        return string(abi.encodePacked("data:application/json;base64,", json));
```
https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-bio-protocol/src/Bio.sol#L116

```
File: canto-namespace-protocol/src/Namespace.sol 
95:                    abi.encodePacked(
105:        return string(abi.encodePacked("data:application/json;base64,", json));
```
https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Namespace.sol#L95
https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Namespace.sol#L105

```
File: canto-namespace-protocol/src/Tray.sol
135:                    abi.encodePacked(
145:         return string(abi.encodePacked("data:application/json;base64,", json));
```
https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Tray.sol#L135
https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Tray.sol#L145

```
File: canto-namespace-protocol/src/Utils.sol 
104:            bytes memory character = abi.encodePacked(
110:                character = abi.encodePacked(character, EMOJIS[byteOffset + i]);
115:                    character = abi.encodePacked(character, hex"F09F8FBB");
117:                    character = abi.encodePacked(character, hex"F09F8FBC");
119:                    character = abi.encodePacked(character, hex"F09F8FBD");
121:                    character = abi.encodePacked(character, hex"F09F8FBE");
123:                    character = abi.encodePacked(character, hex"F09F8FBF");
135:            return abi.encodePacked(bytes1(asciiStartingIndex + uint8(_characterIndex)));
145:            bytes memory character = abi.encodePacked(bytes1(asciiStartingIndex + uint8(_characterIndex)));
149:                character = abi.encodePacked(
158:                character = abi.encodePacked(
167:                character = abi.encodePacked(
197:                    return abi.encodePacked(FONT_SQUIGGLE[0], FONT_SQUIGGLE[1]);
199:                    return abi.encodePacked(FONT_SQUIGGLE[2], FONT_SQUIGGLE[3], FONT_SQUIGGLE[4]);
202:                    return abi.encodePacked(FONT_SQUIGGLE[5 + offset], FONT_SQUIGGLE[6 + offset]);
204:                    return abi.encodePacked(FONT_SQUIGGLE[47]);
206:                    return abi.encodePacked(FONT_SQUIGGLE[48], FONT_SQUIGGLE[49], FONT_SQUIGGLE[50]);
```

## [N-02] NON-LIBRARY/INTERFACE FILES SHOULD USE FIXED COMPILER VERSIONS, NOT FLOATING ONES
### Contexts: All contract
### Description:
Pragma statements can be allowed to float when a contract is intended for consumption by other developers, as in the case with contracts in a library or EthPM package. Otherwise, the developer would need to manually update the pragma in order to compile locally.
https://swcregistry.io/docs/SWC-103
### Recommendation:
Ethereum Smart Contract Best Practices - Lock pragmas to specific compiler version.
https://consensys.github.io/smart-contract-best-practices/development-recommendations/solidity-specific/locking-pragmas/

## [N-03] Function writing that does not comply with the Solidity Style Guide
Order of Functions; ordering helps readers identify which functions they can call and to find the constructor and fallback definitions easier. But there are contracts in the project that do not comply with this.

https://docs.soliditylang.org/en/v0.8.17/style-guide.html

Functions should be grouped according to their visibility and ordered:

constructor

receive function (if exists)

fallback function (if exists)

external

public

internal

private

within a grouping, place the view and pure functions last

## [N-04] Constants should be defined rather than using magic numbers
```
File: canto-pfp-protocol/src/ProfilePicture.sol
60:        if (block.chainid == 7700) {
```
https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-pfp-protocol/src/ProfilePicture.sol#L60
```
File: canto-namespace-protocol/src/Namespace.sol
81:        if (block.chainid == 7700) {
```
https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Namespace.sol#L81

```
File: canto-namespace-protocol/src/Utils.sol
258:            iteratedState = _currState * 15485863;
259:            iteratedState = (iteratedState * iteratedState * iteratedState) % 2038074743;
```
https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Utils.sol#L258-L259

## [N-05] Use a single file for all system-wide constants
There are many addresses and constants used in the system. It is recommended to put the most used ones in one file (for example constants.sol, use inheritance to access these values).

This will help with readability and easier maintenance for future changes. This also helps with any issues, as some of these hard-coded values are admin addresses.

#### constants.sol
Use and import this file in contracts that require access to these values. This is just a suggestion, in some use cases this may result in higher gas usage in the distribution.

## [N-06] Include return parameters in NatSpec comments
#### Contexts: All Contract
### Description
It is recommended that Solidity contracts are fully annotated using NatSpec for all public interfaces (everything in the ABI). It is clearly stated in the Solidity official documentation. In complex projects such as Defi, the interpretation of all functions and their arguments and returns is important for code readability and auditability.

https://docs.soliditylang.org/en/v0.8.15/natspec-format.html
### Recommendation
Include return parameters in NatSpec comments

## [N-07] Use descriptive names for Contracts and Libraries
This codebase will be difficult to navigate, as there are no descriptive naming conventions that specify which files should contain meaningful logic.

Prefixes should be added like this by filing:

Interface I_
absctract contracts Abs_
Libraries Lib_
We recommend that you implement this or a similar agreement.

## [N-08] SOLIDITY COMPILER OPTIMIZATIONS CAN BE PROBLEMATIC
#### Description:
```
foundry.toml:
1: [profile.default]
2: optimizer = true
3: optimizer_runs = 1_000
4: via-ir=true
```
Protocol has enabled optional compiler optimizations in Solidity.
There have been several optimization bugs with security implications. Moreover, optimizations are actively being developed. Solidity compiler optimizations are disabled by default, and it is unclear how many contracts in the wild actually use them.
Therefore, it is unclear how well they are being tested and exercised.
High-severity security issues due to optimization bugs have occurred in the past. A high-severity bug in the emscripten-generated solc-js compiler used by Truffle and Remix persisted until late 2018. The fix for this bug was not reported in the Solidity CHANGELOG.
Another high-severity optimization bug resulting in incorrect bit shift results was patched in Solidity 0.5.6. More recently, another bug due to the incorrect caching of keccak256 was reported.
A compiler audit of Solidity from November 2018 concluded that the optional optimizations may not be safe.
It is likely that there are latent bugs related to optimization and that new bugs will be introduced due to future optimizations.
Exploit Scenario:
A latent or future bug in Solidity compiler optimizations—or in the Emscripten transpilation to solc-js—causes a security vulnerability in the contracts.
Recommendation:
Short term, measure the gas savings from optimizations and carefully weigh them against the possibility of an optimization-related bug. Long term, monitor the development and adoption of Solidity compiler optimizations to assess their maturity.