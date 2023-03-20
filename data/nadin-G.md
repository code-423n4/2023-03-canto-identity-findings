## Gas Optimizations
### Issue
## [G-01] <X> += <Y> COSTS MORE GAS THAN <X> = <X> + <Y> FOR STATE VARIABLES
## [G-02] ++I/I++ SHOULD BE UNCHECKED{++I}/UNCHECKED{I++} WHEN IT IS NOT POSSIBLE FOR THEM TO OVERFLOW, AS IS THE CASE WHEN USED IN FOR- AND WHILE-LOOPS
## [G-03] USING FIXED BYTES IS CHEAPER THAN USING STRING
## [G-04] SETTING THE CONSTRUCTOR TO PAYABLE
## [G-05] NOT USING THE NAMED RETURN VARIABLES WHEN A FUNCTION RETURNS, WASTES DEPLOYMENT GAS
## [G-06]  Struct packing
## [G-07] Use nested if and, avoid multiple check combinations
## [G-08] Use assembly to write address storage values
## [G-09] Using delete instead of setting stakedCitizen struct to 0 saves gas
## [G-10] Use constants instead of type(uintx).max
## [G-11] Avoid using state variable in emit
### Total: 11 issues
## [G-01] <X> += <Y> COSTS MORE GAS THAN <X> = <X> + <Y> FOR STATE VARIABLES
```
File: canto-namespace-protocol/src/Namespace.sol
150:            numBytes += numBytesChar;
```
https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Namespace.sol#L150

## [G-02] ++I/I++ SHOULD BE UNCHECKED{++I}/UNCHECKED{I++} WHEN IT IS NOT POSSIBLE FOR THEM TO OVERFLOW, AS IS THE CASE WHEN USED IN FOR- AND WHILE-LOOPS
```
File: canto-pfp-protocol/src/ProfilePicture.sol
80:        uint256 tokenId = ++numMinted;
```
https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-pfp-protocol/src/ProfilePicture.sol#L80
```
File: canto-bio-protocol/src/Bio.sol
56:        for (uint i; i < lengthInBytes; ++i) {
59:            bytesOffset++;
90:                    strLines[insertedLines++] = string(bytesLines);
100:        for (uint i; i < lines; ++i) {
124:        uint256 tokenId = ++numMinted;
```
https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-bio-protocol/src/Bio.sol#L56
https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-bio-protocol/src/Bio.sol#L59
https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-bio-protocol/src/Bio.sol#L90
https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-bio-protocol/src/Bio.sol#L100
https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-bio-protocol/src/Bio.sol#L124
```
File: canto-namespace-protocol/src/Namespace.sol
115:        uint256 namespaceIDToMint = ++nextNamespaceIDToMint;
122:        for (uint256 i; i < numCharacters; ++i) {
127:            for (uint256 j = i + 1; j < numCharacters; ++j) {
147:            for (uint256 j; j < numBytesChar; ++j) {
154:                uniqueTrays[numUniqueTrays++] = trayID;
174:        for (uint256 i; i < numUniqueTrays; ++i) {
```
https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Namespace.sol#L115
https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Namespace.sol#L122
https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Namespace.sol#L127
https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Namespace.sol#L147
https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Namespace.sol#L154
https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Namespace.sol#L174
```
File: canto-namespace-protocol/src/Tray.sol 
129:        for (uint256 i; i < TILES_PER_TRAY; ++i) {
159:        for (uint256 i; i < _amount; ++i) {
161:            for (uint256 j; j < TILES_PER_TRAY; ++j) {
```
https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Tray.sol#L129
https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Tray.sol#L159
https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Tray.sol#L161
```
File: canto-namespace-protocol/src/Utils.sol 
109:            for (uint256 i = 3; i < numBytes; ++i) {
146:            for (uint256 i; i < numAbove; ++i) {
155:            for (uint256 i; i < numMiddle; ++i) {
164:            for (uint256 i; i < numBelow; ++i) {
225:        for (uint256 i; i < _tiles.length; ++i) {
```
https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Utils.sol#L109
https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Utils.sol#L146
https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Utils.sol#L155
https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Utils.sol#L164
https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Utils.sol#L225

## [G-03] USING FIXED BYTES IS CHEAPER THAN USING STRING
As a rule of thumb, use bytes for arbitrary-length raw byte data and string for arbitrary-length string (UTF-8) data. If you can limit the length to a certain number of bytes, always use one of bytes1 to bytes32 because they are much cheaper.
```
File: canto-pfp-protocol/src/ProfilePicture.sol
35:    string public subprotocolName;
57:    constructor(address _cidNFT, string memory _subprotocolName) ERC721("Profile Picture", "PFP") {
```
```
File: canto-bio-protocol/src/Bio.sol
18:    mapping(uint256 => string) public bio;
45:        string memory bioText = bio[_id];
50:        string[] memory strLines = new string[](lines);
90:                    strLines[insertedLines++] = string(bytesLines);
97:        string
99:        string memory text = '<text x="50%" y="50%" dominant-baseline="middle" text-anchor="middle">';
101:            text = string.concat(text, '<tspan x="50%" dy="20">', strLines[i], "</tspan>");
103:        string memory json = Base64.encode(
105:                string.concat(
111:                    Base64.encode(bytes(string.concat(svg, text, "</text></svg>"))),
116:        return string(abi.encodePacked("data:application/json;base64,", json));
121:    function mint(string calldata _bio) external {
```
```
File: canto-namespace-protocol/src/Namespace.sol
43:    mapping(string => uint256) public nameToToken;
46:    mapping(uint256 => string) public tokenToName;
92:        string memory json = Base64.encode(
94:                string(
105:        return string(abi.encodePacked("data:application/json;base64,", json));
168:        string memory nameToRegister = string(bName);
188:        string memory associatedName = tokenToName[_id];
```
```
File: canto-namespace-protocol/src/Tray.sol 
132:        string memory json = Base64.encode(
134:                string(
145:        return string(abi.encodePacked("data:application/json;base64,", json));
```
```
File: canto-namespace-protocol/src/Utils.sol 
223:        string memory innerData;
226:            innerData = string.concat(
231:                string(
237:                innerData = string.concat(
246:            string.concat(
```

## [G-04] SETTING THE CONSTRUCTOR TO PAYABLE
Saves ~13 gas per instance
```
File: canto-pfp-protocol/src/ProfilePicture.sol
57:    constructor(address _cidNFT, string memory _subprotocolName) ERC721("Profile Picture", "PFP") {
```
https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-pfp-protocol/src/ProfilePicture.sol#L57

```
File: canto-bio-protocol/src/Bio.sol
32:    constructor() ERC721("Biography", "Bio") {
```
https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-bio-protocol/src/Bio.sol#L32

```
File: canto-namespace-protocol/src/Namespace.sol
73:    constructor(
```
https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Namespace.sol#L73

```
File: canto-namespace-protocol/src/Tray.sol
98:    constructor(
```
https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Tray.sol#L98

## [G-05] NOT USING THE NAMED RETURN VARIABLES WHEN A FUNCTION RETURNS, WASTES DEPLOYMENT GAS
```
File: canto-pfp-protocol/src/ProfilePicture.sol
94:    function getPFP(uint256 _pfpID) public view returns (address nftContract, uint256 nftID) {
```
https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-pfp-protocol/src/ProfilePicture.sol#L94
```
File: canto-namespace-protocol/src/Tray.sol
195:    function getTile(uint256 _trayId, uint8 _tileOffset) external view returns (TileData memory tileData) {
203:    function getTiles(uint256 _trayId) external view returns (TileData[TILES_PER_TRAY] memory tileData) {
245:    function _drawing(uint256 _seed) private pure returns (TileData memory tileData) {
```
https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Tray.sol#L195
https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Tray.sol#L203
https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Tray.sol#L245

```
File: canto-namespace-protocol/src/Utils.sol
256:    function iteratePRNG(uint256 _currState) internal pure returns (uint256 iteratedState) {
```
https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Utils.sol#L256

## [G-06]  Struct packing
We can use the uint types uint8, uint16, uint32, etc, though Solidity reserves 256 bits of storage regardless of the uint type meaning, in normal instances, there's no cost saving. However, if you have multiple uints inside a struct, using a smaller-sized uint when possible will allow Solidity to pack these variables together to take up less storage. 
```
File: canto-namespace-protocol/src/Namespace.sol 
34:        uint8 tileOffset;
36:        uint8 skinToneModifier;
```
https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Namespace.sol#L30-L37
```
File: canto-namespace-protocol/src/Tray.sol
63:        uint8 characterModifier;
```
https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Tray.sol#L57-L64

## [G-07] Use nested if and, avoid multiple check combinations
### Description:
Using nested is cheaper than using && multiple check combinations. There are more advantages, such as easier to read code and better coverage reports.
```
File: canto-bio-protocol/src/Bio.sol
60:            if ((i > 0 && (i + 1) % 40 == 0) || prevByteWasContinuation || i == lengthInBytes - 1) {
71:                    if (
```
https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-bio-protocol/src/Bio.sol#L60
https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-bio-protocol/src/Bio.sol#L71-L83

```
File: canto-namespace-protocol/src/Namespace.sol
136:            if (tileData.fontClass != 0 && _characterList[i].skinToneModifier != 0) {
157:                if (
186:        if (nftOwner != msg.sender && getApproved[_id] != msg.sender && !isApprovedForAll[nftOwner][msg.sender])
```
https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Namespace.sol#L136
https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Namespace.sol#L157-L161
https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Namespace.sol#L186

```
File: canto-namespace-protocol/src/Tray.sol
175:        if (
184:            if (numPrelaunchMinted != type(uint256).max && _id <= numPrelaunchMinted)
240:            if (startTokenId <= numPrelaunchMinted && to != address(0))
```
https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Tray.sol#L175-L180
https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Tray.sol#L184
https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Tray.sol#L240

## [G-08] Use assembly to write address storage values
```
File: canto-namespace-protocol/src/Namespace.sol 
78:        tray = Tray(_tray);
79:        note = ERC20(_note);
80:        revenueAddress = _revenueAddress;
```
https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Namespace.sol#L73-L80

```
File: canto-namespace-protocol/src/Tray.sol
107:        revenueAddress = _revenueAddress;
108:        note = ERC20(_note);
109:        namespaceNFT = _namespaceNFT;
```
https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Tray.sol#L98-L109

## [G-09] Using delete instead of setting stakedCitizen struct to 0 saves gas
```
File: canto-pfp-protocol/src/ProfilePicture.sol
103:            nftID = 0; // Strictly not needed because nftContract has to be always checked, but reset nevertheless to 0
```
https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-pfp-protocol/src/ProfilePicture.sol#L103

## [G-10] Use constants instead of type(uintx).max
#### Description:
type(uint128).max or type(uint256).max, etc. it uses more gas in the distribution process and also for each transaction than constant usage.
```
File: canto-namespace-protocol/src/Tray.sol
122:        if (numPrelaunchMinted != type(uint256).max) {
152:        if (prelaunchMinted == type(uint256).max) {
184:            if (numPrelaunchMinted != type(uint256).max && _id <= numPrelaunchMinted)
226:        if (prelaunchMinted != type(uint256).max) revert PrelaunchAlreadyEnded();
238:        if (numPrelaunchMinted != type(uint256).max) {
```
https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Tray.sol#L122
https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Tray.sol#L152
https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Tray.sol#L184
https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Tray.sol#L226
https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Tray.sol#L238

## [G-11] Avoid using state variable in emit
```
File: canto-pfp-protocol/src/ProfilePicture.sol
87:        emit PfpAdded(msg.sender, tokenId, _nftContract, _nftID);
```
https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-pfp-protocol/src/ProfilePicture.sol#L87

```
File: canto-bio-protocol/src/Bio.sol
127:        emit BioAdded(msg.sender, tokenId, _bio);
```
https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-bio-protocol/src/Bio.sol#L127

```
File: canto-namespace-protocol/src/Namespace.sol
179:        emit NamespaceFused(msg.sender, namespaceIDToMint, nameToRegister);
199:        emit NoteAddressUpdate(currentNoteAddress, _newNoteAddress);
207:        emit RevenueAddressUpdated(currentRevenueAddress, _newRevenueAddress);
```
https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Namespace.sol#L179
https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Namespace.sol#L199
https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Namespace.sol#L207

```
File: canto-namespace-protocol/src/Tray.sol 
213:        emit NoteAddressUpdate(currentNoteAddress, _newNoteAddress);
221:        emit RevenueAddressUpdated(currentRevenueAddress, _newRevenueAddress);
```
https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Tray.sol#L213
https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Tray.sol#L221
