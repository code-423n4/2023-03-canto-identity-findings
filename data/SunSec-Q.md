### [L1] Constructors and set functions without contract integrity check

## Impact
It is recommended to add a "checkContract" function to verify the integrity of the contract code during the deployment process. This can help prevent attacks that aim to exploit vulnerabilities in the contract's code or its dependencies. By including an integrity check, the contract can ensure that it is running as intended and that it has not been tampered with or modified in any way.

## Proof of Concept

https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-pfp-protocol/src/ProfilePicture.sol#L58
```
    constructor(address _cidNFT, string memory _subprotocolName) ERC721("Profile Picture", "PFP") {
        cidNFT = ICidNFT(_cidNFT);
```

https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Namespace.sol#L78-L79
```
    constructor(
        address _tray,
        address _note,
        address _revenueAddress
    ) ERC721("Namespace", "NS") Owned(msg.sender) {
        tray = Tray(_tray);
        note = ERC20(_note);
```

https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Namespace.sol#L198

```
    function changeNoteAddress(address _newNoteAddress) external onlyOwner {
        address currentNoteAddress = address(note);
        note = ERC20(_newNoteAddress);
```

## Tools Used
Manual

## Recommended Mitigation Steps
It is recommended to add a "checkContract" function.
Example:

```
function checkContract(address _account) internal view {
   require(_account != address(0), "Account cannot be zero address");

   uint256 size;
   // solhint-disable-next-line no-inline-assembly
   assembly { size := extcodesize(_account) }
   require(size > 0, "Account code size cannot be zero");
}

```

### [L2] Unchecked numMinted & nextNamespaceIDToMint Limit

## Impact
In the mint/fuse function, numMinted/nextNamespaceIDToMint increments. However, the contract does not check if numMinted/nextNamespaceIDToMint exceeds the maximum value of uint256. In an extreme case, if numMinted/nextNamespaceIDToMint surpasses the limit, it will roll back to zero, potentially causing new NFTs to overwrite existing ones. To avoid this issue, it is advisable to check whether numMinted/nextNamespaceIDToMint exceeds the maximum value.

## Proof of Concept
https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-bio-protocol/src/Bio.sol#L121-L126
```
    function mint(string calldata _bio) external {
        // We check the length in bytes, so will be higher for UTF-8 characters. But sufficient for this check
        if (bytes(_bio).length == 0 || bytes(_bio).length > 200) revert InvalidBioLength(bytes(_bio).length);
        uint256 tokenId = ++numMinted; //@audit no max value check
        bio[tokenId] = _bio;
        _mint(msg.sender, tokenId);
```
https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Namespace.sol#L115
```
  function fuse(CharacterData[] calldata _characterList) external {
        uint256 numCharacters = _characterList.length;
        if (numCharacters > 13 || numCharacters == 0) revert InvalidNumberOfCharacters(numCharacters);
        uint256 fusingCosts = 2**(13 - numCharacters) * 1e18;
        SafeTransferLib.safeTransferFrom(note, msg.sender, revenueAddress, fusingCosts);
        uint256 namespaceIDToMint = ++nextNamespaceIDToMint;  //@audit no max value check
```

## Tools Used
Manual
## Recommended Mitigation Steps
Add maximum value check.
```
// check that not exceed maximum value check.
 require(numMinted == type(uint256).max ,"exceed maximum value");
```