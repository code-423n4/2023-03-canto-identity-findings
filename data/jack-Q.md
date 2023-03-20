#### Low

Docs specify length has to be shorter than 200

code:
bio.sol - https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-bio-protocol/src/Bio.sol#L123

`if (bytes(_bio).length == 0 || bytes(_bio).length > 200) revert InvalidBioLength(bytes(_bio).length);`

should be:
`if (bytes(_bio).length == 0 || bytes(_bio).length > 199) revert InvalidBioLength(bytes(_bio).length);`

Code does not do what the readme says.
The tests shows that they intend to accept 200 characters, but the docs specify shorter than 200.

---

#### QA

non-critical:
Code styling issue. 
https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Namespace.sol#L121

code:
`uint256[] memory uniqueTrays = new uint256[](_characterList.length);`
[PASS] testFusingAsOwnerOfTray() (gas: 476771)

should be:
`uint256[] memory uniqueTrays = new uint256[](numCharacters);`
[PASS] testFusingAsOwnerOfTray() (gas: 476771)


