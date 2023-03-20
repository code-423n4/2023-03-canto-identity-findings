[G-01] Change public state variable visibility to private
If it is preferred to change the visibility of the state variables to private, this will save significant gas.
```
canto-pfp-protocol/src/ProfilePicture.sol
29:    uint256 public numMinted;
35:    string public subprotocolName;

canto-bio-protocol/src/Bio.sol
15:    uint256 public numMinted;
18:    mapping(uint256 => string) public bio;

canto-namespace-protocol/src/Namespace.sol
17:    Tray public immutable tray;
20:    ERC20 public note;
40:    uint256 public nextNamespaceIDToMint;
43:    mapping(string => uint256) public nameToToken;
46:    mapping(uint256 => string) public tokenToName;

canto-namespace-protocol/src/Tray.sol
37:    uint256 public immutable trayPrice;
47:    ERC20 public note;
50:    address public immutable namespaceNFT;
```

[G-02] It's better to use nested-if, and avoid multiple check combinations
```
canto-namespace-protocol/src/Namespace.sol
136:          if (tileData.fontClass != 0 && _characterList[i].skinToneModifier != 0) {
186:          if (nftOwner != msg.sender && getApproved[_id] != msg.sender && !isApprovedForAll[nftOwner][msg.sender])

canto-namespace-protocol/src/Tray.sol
175:   if (
            namespaceNFT != msg.sender &&
            trayOwner != msg.sender &&
            getApproved(_id) != msg.sender &&
            !isApprovedForAll(trayOwner, msg.sender)
        ) revert CallerNotAllowedToBurn();
184:   if (numPrelaunchMinted != type(uint256).max && _id <= numPrelaunchMinted)
```
https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Tray.sol#L175-L180
[G-03] Use uncheck{} for the increment of memory variables, which wouldn't be overflow or underflow
```
2023-03-canto-identity/canto-pfp-protocol/src/ProfilePicture.sol
80:           uint256 tokenId = ++numMinted;

canto-bio-protocol/src/Bio.sol
124:          uint256 tokenId = ++numMinted;

canto-namespace-protocol/src/Namespace.sol
115:          uint256 namespaceIDToMint = ++nextNamespaceIDToMint;

canto-namespace-protocol/src/Tray.sol
227:          prelaunchMinted = _nextTokenId() - 1;
```

[G-04] To store false as uint256(0), and true as uint256(1)

EVM stores boolean values as a uint8 type, which takes up two bytes, it is actually more expensive to access the value. This is because it EVM words are 32 bytes long, so extra logic is needed to tell the VM to parse a value that is smaller than standard.
```
canto-bio-protocol/src/Bio.sol
51:    bool prevByteWasContinuation;

canto-namespace-protocol/src/Namespace.sol
123:   bool isLastTrayEntry = true;
```

[G-05] **tileData.characterModifier** might be assigned the same value when **tileData.fontClass != 0**
```
canto-namespace-protocol/src/Namespace.sol
134:            uint8 characterModifier = tileData.characterModifier;
135:
136:            if (tileData.fontClass != 0 && _characterList[i].skinToneModifier != 0) {
137:                revert CannotFuseCharacterWithSkinTone();
138:            }
139:            
140:            if (tileData.fontClass == 0) {
141:                // Emoji
142:                characterModifier = _characterList[i].skinToneModifier;
143:            }
144:            bytes memory charAsBytes = Utils.characterToUnicodeBytes(0, tileData.characterIndex, characterModifier);
145:            tileData.characterModifier = characterModifier;
```
Recommandation:
```
            if (tileData.fontClass == 0) {
                // Emoji
                uint8 characterModifier = _characterList[i].skinToneModifier;
                tileData.characterModifier = characterModifier;
            }
            else {
                uint8 characterModifier = tileData.characterModifier;
            }
            bytes memory charAsBytes = Utils.characterToUnicodeBytes(0, tileData.characterIndex, characterModifier);
```

[G-06] Revert the transaction as soon as possible when it meet the condition

```
canto-namespace-protocol/src/Namespace.sol
134:            uint8 characterModifier = tileData.characterModifier;
135:
136:            if (tileData.fontClass != 0 && _characterList[i].skinToneModifier != 0) {
137:                revert CannotFuseCharacterWithSkinTone();
138:            }
```

Recommandation:
```
134:            if (tileData.fontClass != 0 && _characterList[i].skinToneModifier != 0) {
135:                revert CannotFuseCharacterWithSkinTone();
136:            }
137:
138:            uint8 characterModifier = tileData.characterModifier;
```
https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Namespace.sol#L134-L138



This should be put on the beginning of **characterToUnicodeBytes** function
```
canto-namespace-protocol/src/Utils.sol
126:            revert InvalidSkinToneModifierProvided(_characterModifier);
```
https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Utils.sol#L125