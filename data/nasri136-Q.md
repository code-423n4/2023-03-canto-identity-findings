## Summary
### Low Risk Issues List
| Number |Issues Details|Context|
|:--:|:-------|:--:|
|[L-01]| Lock Pragmas to specific compiler version | 4 |
|[L-02]| Missing zero address validation | 2 |
|[L-03]| Missing NonReentrant modifier | 1 |
|[L-04]| Critical Address changes should use two-step procedure | 2 |
|[L-05]| Dupplicated revert() / require() should be defined as a modifier or function  | 1 |
|[L-06]| Loop Misusing  | 1 |

Total 6 issues


### Non-Critical Issues List
| Number |Issues Details|Context|
|:--:|:-------|:--:|
| [N-01]| Hardcoding chain IDs can be dangerous | 3 |
| [N-02] | Function Complexity should be cut in multiple function |1|

Total 2 issues


## [L-01]  Lock Pragmas to specific compiler version

Solidity pragma versioning should be exactly same in all contracts. Currently contracts use >=0.8.0 but should use 0.8.0 or better version could be 0.8.6

## [L-02]  Missing zero address validation
#### Tray.sol 
    
```ChangeNoteAddress(address _newNoteAddress)``` : https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L210
```ChangeRevenueAddress(address _newRevenueAddress)``` : https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L218

#### NameSpace.sol
```ChangeNoteAddress(address _newNoteAddress)``` : https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Namespace.sol#L196
```ChangeRevenueAddress(address _newRevenueAddress)``` : https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Namespace.sol#L204

#### Recommendation
Add a one line require statement checking that the new address is not 0 to prevent bad change ```require(address(<VARIABLE>) != address(0));```  

## [L-03]  Missing NonReentrant modifier

Missing NonReentrant modifier on function using external entities

#### Tray.sol
```buy(uint256 _amount)``` : https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L150

#### NameSpace.sol
```fuse(CharacterData[] calldata _characterList)``` : https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Namespace.sol#L110

#### Recommendation
Even if this is not critical it is a better usage to add a nonReentrant modifier to a function having external calls

## [L-04]  Critical Address changes should use two-step procedure

#### Tray.sol : 
```ChangeNoteAddress(address _newNoteAddress)``` : https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L210
```ChangeRevenueAddress(address _newRevenueAddress)``` : https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L218

#### NameSpace.sol
```ChangeNoteAddress(address _newNoteAddress)``` : https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Namespace.sol#L196
```ChangeRevenueAddress(address _newRevenueAddress)``` : https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Namespace.sol#L204

#### Recommendation
Use a timer or a multisig to validate critical changes like these

## [L-05]  Dupplicated revert() / require() should be defined as a modifier or function
#### Tray.sol
```Error TrayNotMinted(uint256 tokenId)``` https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L86
```Error PrelaunchTrayCannotBeUsedAfterPrelaunch(uint256 startTokenId)``` : https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L89

#### Recommendation
Use a modifier or a function to revert if conditions are not meet and make readibility easier

## [L-06]  Loop Misusing

#### Bio.sol#TokenUri.sol 

In the for loop ```i``` is initialized with ```uint i``` but after that, there is an if check that ```i > 0``` so the first character (i=0) of the ```text``` variable is not taken into account
https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-bio-protocol/src/Bio.sol#L43

#### Recommendation
Change the if of ```TokenUri``` to ```i <= 0```

## [N-01]  Hardcoding chain IDs can be dangerous

In the constructor, the contract checks if the chain ID is 7700, which appears to be a hardcoded value for the Canto mainnet. Hardcoding chain IDs can be dangerous as it makes the contract less portable and more susceptible to attacks in the event of a chain fork or unexpected chain ID change.  It would be better to use block.chainid() to avoid potential issues with future changes in Solidity's syntax.

#### Constructor in Tray.sol, Bio.sol, NameSpace.sol

## [N-02]  Function Complexity should be cut in multiple function

Function tokenUri(uint256 _id) is longer (74 Lines) than recommended (60 Lines) , it could be divided/cut to multiple function to make readility easier and avoid mistakes.