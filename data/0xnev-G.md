| Count | Title | Instances |
|:--:|:-------|:--:|
| [G-01] | Initialize stack variables outside loop | 9 |
| [G-02] | Use nested `if` statements to avoid multiple check combinations using `&&` | 6 |
| [G-03] | Move validation statements to top of function when validating input parameters | 1 |
| [G-04] | State variables not changed after deployment can be immutable | 1 |
| [G-05] | Sort solidity operators based on short-circuiting| 2 |
| [G-06] | Use a more recent version of solidity | All |
| [G-07] | Functions guranteed to revert when called by normal users can be marked `payable` | 5 |
| [G-08] | Mark state variables as `private` instead of `public`  | 8 |

| Total Gas-Optimization Issues | 8 |
|:--:|:--:|

### [G-01] Initialize stack variables outside loop
```solidity
9 results - 2 files

/Bio.sol
43:    function tokenURI(uint256 _id) public view override returns (string memory)
56:        for (uint i; i < lengthInBytes; ++i) {
57:            bytes1 character = bioTextBytes[i];
61:                bytes1 nextCharacter;

/Namespace.sol
110:    function fuse(CharacterData[] calldata _characterList) external
122:        for (uint256 i; i < numCharacters; ++i) {
123:            bool isLastTrayEntry = true;
124:            uint256 trayID = _characterList[i].trayID;
125:            uint8 tileOffset = _characterList[i].tileOffset;
134:            uint8 characterModifier = tileData.characterModifier;
144:            bytes memory charAsBytes = Utils.characterToUnicodeBytes(0, tileData.characterIndex, characterModifier);
146:            uint256 numBytesChar = charAsBytes.length;
156:                address trayOwner = tray.ownerOf(trayID);
```

Description:
Consider initializing the stack variables before the loop to avoid reinitialization on every loop

Recommendation:
Consider initializing the stack variables before the loop to save gas 


### [G-02] Use nested `if` statements to avoid multiple check combinations using `&&`
```solidity
6 result - 2 files

/Namespace.sol
110:    function fuse(CharacterData[] calldata _characterList) external 
136:            if (tileData.fontClass != 0 && _characterList[i].skinToneModifier != 0)

157:                if (
158:                    trayOwner != msg.sender &&
159:                    tray.getApproved(trayID) != msg.sender &&
160:                    !tray.isApprovedForAll(trayOwner, msg.sender)

184:    function burn(uint256 _id) external 
186:        if (nftOwner != msg.sender && getApproved[_id] != msg.sender && !isApprovedForAll[nftOwner][msg.sender])


/Tray.sol
173:    function burn(uint256 _id) external
175:        if (
176:            namespaceNFT != msg.sender &&
177:            trayOwner != msg.sender &&
178:            getApproved(_id) != msg.sender &&
179:            !isApprovedForAll(trayOwner, msg.sender)

184:            if (numPrelaunchMinted != type(uint256).max && _id <= numPrelaunchMinted)

231:    function _beforeTokenTransfers
240:            if (startTokenId <= numPrelaunchMinted && to != address(0))
```

Description:
Using nested `if` statements is cheaper than using `&& ` for multiple check combinations. Additionally, it improves readability of code and better coverage reports.

Recommendation:
Replace `&&` used within `if` and `else if` statements with nested `if` statements

### [G-03] Move validation statements to top of function when validating input parameters
Context: 

```solidity
1 result - 1 file

/ProfilePicture.sol
79:    function mint(address _nftContract, uint256 _nftID) external
81:        if (ERC721(_nftContract).ownerOf(_nftID) != msg.sender)
82:            revert PFPNotOwnedByCaller(msg.sender, _nftContract, _nftID);
```

Description:
Input parameters that require validation should be done first in functions to avoid execution of the rest of the function logic and waste gas

Recommendation:
Consider shifting the input validation to the top of the function for the above instances

### [G-04] State variables not changed after deployment can be immutable
```solidity
1 result - 1 file

/ProfilePicture.sol
35:    string public subprotocolName;
```

Description:
State variables that are not changed after deployment time (set in constructor) can be set as `immutable`. This avoids a Gsset (20000 gas) and saves deployment gas.


### [G-05] Sort solidity operators based on short-circuiting
Context:
```solidity
2 result - 2 files 

/Bio.sol
 60: if ((i > 0 && (i + 1) % 40 == 0) || prevByteWasContinuation || i == lengthInBytes - 1)

123: (bytes(_bio).length == 0 || bytes(_bio).length > 200) revert InvalidBioLength(bytes(_bio).length)
```

We can sequence `OR/AND (|| , &)` oeprators in solidity as it applies the common short circuiting rule. 

This means that in the expression `f(x) || g(y)`, if `f(x)` evaluates to true, `g(y)` will not be evaluated even if it may have side-effects. 

Similarly,  in the expression `f(x) && g(y)`, if `f(x)` evaluates to false, `g(y)` will not be evaluated even if it may have side-effects.

So setting less costly function to “f(x)” and setting costly function to “g(x)” is more gas efficient.

### [G-06] Use a more recent version of solidity

Use a Solidity version of at least 0.8.2 to get simple compiler automatic inlining.

Use a Solidity version of at least 0.8.3 to get better struct packing and cheaper multiple storage reads.

Use a Solidity version of at least 0.8.4 to get custom errors, which are cheaper at deployment than revert()/require() strings.\ and et bytes.concat() instead of abi.encodePacked(<bytes>,<bytes>)

Use a Solidity version of at least 0.8.10 to have external calls skip contract existence checks if the external call has a return value.

Use a solidity version of at least 0.8.12 to get string.concat() instead of abi.encodePacked(<str>,<str>)


### [G-07] Functions guranteed to revert when called by normal users can be marked `payable`
```solidity
5 results - 2 files

/Namespace.sol
196: function changeNoteAddress(address _newNoteAddress) external onlyOwner
204: function changeRevenueAddress(address _newRevenueAddress) external onlyOwner

/Tray.sol
210: function changeNoteAddress(address _newNoteAddress) external onlyOwner
218: function changeRevenueAddress(address _newRevenueAddress) external onlyOwner
225: function endPrelaunchPhase() external onlyOwner
```

Description:
If a function modifier such as `onlyOwner` is used, the function will revert if a normal user tries to pay the function. 

Marking the function as payable will lower the gas cost for legitimate callers because the compiler will not include checks for whether a payment was provided.

Recommendation:
Consider marking the functions as payable. However, it is valid if protocol decides against it due to confusion, readability and possibility of receiving funds.

### [G-08] Mark state variables as `private` instead of `public` 

```solidity
8 results - 4 files

/ProfilePicture.sol
35: string public subprotocolName;

/Bio.sol
18: mapping(uint256 => string) public bio;

/Namespace.sol
17: Tray public immutable tray;
40: uint256 public nextNamespaceIDToMint;
43: mapping(string => uint256) public nameToToken;
46: mapping(uint256 => string) public tokenToName;

/Tray.sol
50: address public immutable namespaceNFT;
70: bytes32 public lastHash;
```

Description:
For state variables not accessed externally, we can set them as `private` instead of `public` to save significant gas
