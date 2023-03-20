##

### [G-1] USE A MORE RECENT VERSION OF SOLIDITY

Use a Solidity version of at least 0.8.2 to get simple compiler automatic inlining.

Use a Solidity version of at least 0.8.3 to get better struct packing and cheaper multiple storage reads.

Use a Solidity version of at least 0.8.4 to get custom errors, which are cheaper at deployment than revert()/require() strings.

Use a Solidity version of at least 0.8.10 to have external calls skip contract existence checks if the external call has a return value.

FILE : 2023-03-canto-identity/canto-pfp-protocol/src/ProfilePicture.sol

   2: pragma solidity >=0.8.0;

[ProfilePicture.sol#L2](https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-pfp-protocol/src/ProfilePicture.sol#L2)

##

### [G-2] OPTIMIZE NAMES TO SAVE GAS

public/external function names and public member variable names can be optimized to save gas.  Below are the interfaces/abstract contracts that can be optimized so that the most frequently-called functions use the least amount of gas possible during method lookup. Method IDs that have two leading zero bytes can save 128 gas each during deployment, and renaming functions to have lower method IDs will save 22 gas per call, [per sorted position shifted](https://medium.com/joyso/solidity-how-does-function-name-affect-gas-consumption-in-smart-contract-47d270d8ac92)

FILE : 2023-03-canto-identity/canto-pfp-protocol/src/ProfilePicture.sol

    ///@audit mint(),getPFP() 
    8: contract ProfilePicture is ERC721 {

FILE : 2023-03-canto-identity/canto-bio-protocol/src/Bio.sol

    ///@audit mint()
    9: contract Bio is ERC721 {

FILE : 2023-03-canto-identity/canto-namespace-protocol/src/Namespace.sol

    ///@audit fuse(),burn(),changeNoteAddress(),changeRevenueAddress()
    11: contract Namespace is ERC721, Owned {

FILE : 2023-03-canto-identity/canto-namespace-protocol/src/Tray.sol

    ///@audit buy(),burn(),getTile(),getTiles(),changeNoteAddress(),changeRevenueAddress(),endPrelaunchPhase(),
    13: contract Tray is ERC721A, Owned {

##

### [G-3]  ADD UNCHECKED {} FOR SUBTRACTIONS WHERE THE OPERANDS CANNOT UNDERFLOW BECAUSE OF A PREVIOUS REQUIRE() OR IF-STATEMENT . This saves 30-40 gas

FILE : 2023-03-canto-identity/canto-bio-protocol/src/Bio.sol

  62: if (i != lengthInBytes - 1) {

[Bio.sol#L62](https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-bio-protocol/src/Bio.sol#L62)

FILE : 2023-03-canto-identity/canto-namespace-protocol/src/Tray.sol

    if (prelaunchMinted != type(uint256).max) revert PrelaunchAlreadyEnded();
    prelaunchMinted = _nextTokenId() - 1;

[Tray.sol#L226-L227](https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Tray.sol#L226-L227)

##

### [G-4] CAN MAKE THE VARIABLE OUTSIDE THE LOOP TO SAVE GAS

In Solidity, declaring a variable inside a loop can result in higher gas costs compared to declaring it outside the loop. This is because every time the loop runs, a new instance of the variable is created, which can lead to unnecessary memory allocation and increased gas costs

On the other hand, if you declare a variable outside the loop, the variable is only initialized once, and you can reuse the same instance of the variable inside the loop. This approach can save gas and optimize the execution of your code

GAS SAMPLE TEST IN remix ide(Without gas optimizations) :

Before Mitigation :

     function testGas() public view {

        for(uint256 i = 0; i < 10; i++) {
          address collateral = msg.sender;
          address  collateral1 = msg.sender;
            
        }

The execution Cost : 2176 

After Mitigation :

     function testGas() public view {
    address collateral; address  collateral1;
        for(uint256 i = 0; i < 10; i++) {
         collateral = msg.sender;
         collateral1 = msg.sender;
            
        }

The execution Cost : 2086 . 

>  Hence clearly after mitigation the gas cost is reduced. Approximately its possible to save 9 gas for every iterations 

FILE : 2023-03-canto-identity/canto-bio-protocol/src/Bio.sol

[Bio.sol#L56-L61](https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-bio-protocol/src/Bio.sol#L56-L61)

FILE: 2023-03-canto-identity/canto-namespace-protocol/src/Namespace.sol

[Namespace.sol#L122-L138](https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Namespace.sol#L122-L138)

[Utils.sol#L155-L167](https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Utils.sol#L155-L167)

##

### [G-5] Use nested if and, avoid multiple check combinations

FILE : 2023-03-canto-identity/canto-bio-protocol/src/Bio.sol

[Bio.sol#L60](https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-bio-protocol/src/Bio.sol#L60)

[Bio.sol#L71-L83](https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-bio-protocol/src/Bio.sol#L71-L83)

FILE : 2023-03-canto-identity/canto-namespace-protocol/src/Tray.sol

[Tray.sol#L175-L179](https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Tray.sol#L175-L179)

[Tray.sol#L184](https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Tray.sol#L184)

[Tray.sol#L240](https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Tray.sol#L240)

[Namespace.sol#L186](https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Namespace.sol#L186)

[Namespace.sol#L157-L161](https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Namespace.sol#L157-L161)

[Namespace.sol#L136](https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Namespace.sol#L136)

##

### [G-6] MULTIPLE ADDRESS/ID MAPPINGS CAN BE COMBINED INTO A SINGLE MAPPING OF AN ADDRESS/ID TO A STRUCT, WHERE APPROPRIATE

Saves a storage slot for the mapping. Depending on the circumstances and sizes of types, can avoid a Gsset (20000 gas) per mapping combined. Reads and subsequent writes can also be cheaper when a function requires both values and they both fit in the same storage slot. Finally, if both fields are accessed in the same function, can save ~42 gas per access due to not having to recalculate the key’s keccak256 hash (Gkeccak256 - 30 gas) and that calculation’s associated stack operations

FILE : 2023-03-canto-identity/canto-namespace-protocol/src/Namespace.sol


   mapping(string => uint256) public nameToToken;

    /// @notice Maps NFT IDs to (ASCII) names
    mapping(uint256 => string) public tokenToName;

    /// @notice Stores the character data of an NFT
    mapping(uint256 => Tray.TileData[]) private nftCharacters;

[Namespace.sol#L43-L49](https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Namespace.sol#L43-L49)

##

### [G-7] SETTING THE CONSTRUCTOR TO PAYABLE

Saves ~13 gas per instance

FILE : 2023-03-canto-identity/canto-pfp-protocol/src/ProfilePicture.sol

  57: constructor(address _cidNFT, string memory _subprotocolName) ERC721("Profile Picture", "PFP") {

[ProfilePicture.sol#L57](https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-pfp-protocol/src/ProfilePicture.sol#L57)

FILE : 2023-03-canto-identity/canto-bio-protocol/src/Bio.sol

   32:  constructor() ERC721("Biography", "Bio") {

[Bio.sol#L32](https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-bio-protocol/src/Bio.sol#L32)

FIle: 2023-03-canto-identity/canto-namespace-protocol/src/Namespace.sol

    73:  constructor(

[Namespace.sol#L73](https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Namespace.sol#L73)

FILE : 2023-03-canto-identity/canto-namespace-protocol/src/Tray.sol

   98:  constructor(

[Tray.sol#L98](https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Tray.sol#L98)

FILE: 2023-03-canto-identity/canto-namespace-protocol/src/Utils.sol

##

### [G-8]  USAGE OF UINTS/INTS SMALLER THAN 32 BYTES (256 BITS) INCURS OVERHEAD

When using elements that are smaller than 32 bytes, your contract’s gas usage may be higher. This is because the EVM operates on 32 bytes at a time. Therefore, if the element is smaller than that, the EVM must use more operations in order to reduce the size of the element from 32 bytes to the desired size

(https://docs.soliditylang.org/en/v0.8.11/internals/layout_in_storage.html)

Each operation involving a uint8 costs an extra 22-28 gas (depending on whether the other operand is also a variable of type uint8) as compared to ones involving uint256, due to the compiler having to clear the higher bits of the memory word before operating on the uint8, as well as the associated stack operations of doing so. Use a larger size then downcast where needed

FILE : 2023-03-canto-identity/canto-namespace-protocol/src/Utils.sol

  131:  uint8 asciiStartingIndex = 97; // Starting index for (lowercase) characters
  138:  uint8 asciiStartingIndex = 97;
  272:  uint8 lastByteVal = uint8(_startingSequence[3]);
  273:  uint8 lastByteSum = lastByteVal + _characterIndex;

[Utils.sol#L131](https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Utils.sol#L131)

##

### [G-9] Use constants instead of type(uint256).max to save gas 

FILE : 2023-03-canto-identity/canto-namespace-protocol/src/Tray.sol

   73 :  uint256 private prelaunchMinted = type(uint256).max;

   122:  if (numPrelaunchMinted != type(uint256).max) {

   152:  if (prelaunchMinted == type(uint256).max) {

   184:   if (numPrelaunchMinted != type(uint256).max && _id <= numPrelaunchMinted)

   226:  if (prelaunchMinted != type(uint256).max) revert PrelaunchAlreadyEnded();

   238:  if (numPrelaunchMinted != type(uint256).max) {

[Tray.sol#L73](https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Tray.sol#L73)

##

### [G-10] INTERNAL or PRIVATE FUNCTIONS ONLY CALLED ONCE CAN BE INLINED TO SAVE GAS

Not inlining costs 20 to 40 gas because of two extra JUMP instructions and additional stack operations needed for function calls.

FILE : 2023-03-canto-identity/canto-namespace-protocol/src/Utils.sol

   function characterToUnicodeBytes(
        uint8 _fontClass,
        uint16 _characterIndex,
        uint256 _characterModifier
    ) internal pure returns (bytes memory) {

[Utils.sol#L73-L77](https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Utils.sol#L73-L77)

    function _getUtfSequence(bytes memory _startingSequence, uint8 _characterIndex)
        private
        pure
        returns (bytes memory)

[Utils.sol#L267-L270](https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Utils.sol#L267-L270)

FILE : 2023-03-canto-identity/canto-namespace-protocol/src/Tray.sol

   245: function _drawing(uint256 _seed) private pure returns (TileData memory tileData) {

[Tray.sol#L245](https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Tray.sol#L245)

   

##

### [G-11] PUBLIC FUNCTIONS NOT CALLED BY THE CONTRACT SHOULD BE DECLARED EXTERNAL INSTEAD

External functions do not require a context switch to the EVM, while public functions do. 

Its possible to save 10-15 gas using external instead public for every function call 

FILE : 2023-03-canto-identity/canto-bio-protocol/src/Bio.sol

  43:    function tokenURI(uint256 _id) public view override returns (string memory) {

[Bio.sol#L43](https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-bio-protocol/src/Bio.sol#L43)

FILE : 2023-03-canto-identity/canto-namespace-protocol/src/Namespace.sol

  90:  function tokenURI(uint256 _id) public view override returns (string memory) {

[Namespace.sol#L90](https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Namespace.sol#L90)

FILE : 2023-03-canto-identity/canto-namespace-protocol/src/Tray.sol

  119:  function tokenURI(uint256 _id) public view override returns (string memory) {

[Tray.sol#L119](https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Tray.sol#L119)



##

### [G-12] Use assembly to write address storage values

FILE : 2023-03-canto-identity/canto-namespace-protocol/src/Namespace.sol

    79: note = ERC20(_note);
    80: revenueAddress = _revenueAddress;

[Namespace.sol#L79-L80](https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Namespace.sol#L79-L80)

FILE : 2023-03-canto-identity/canto-namespace-protocol/src/Tray.sol

    107: revenueAddress = _revenueAddress;
    108: note = ERC20(_note);

[Tray.sol#L107-L108](https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Tray.sol#L107-L108)

##

### [G-13] Duplicated require()/if() checks should be refactored to a modifier or function

FILE : 2023-03-canto-identity/canto-namespace-protocol/src/Tray.sol

  196: if (!_exists(_trayId)) revert TrayNotMinted(_trayId);

  204: if (!_exists(_trayId)) revert TrayNotMinted(_trayId);

[Tray.sol#L204](https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Tray.sol#L204)

FILE : 2023-03-canto-identity/canto-bio-protocol/src/Bio.sol

  44:  if (_ownerOf[_id] == address(0)) revert TokenNotMinted(_id);

FILE : 2023-03-canto-identity/canto-namespace-protocol/src/Namespace.sol

   91:  if (_ownerOf[_id] == address(0)) revert TokenNotMinted(_id);

Recommendation
You can consider adding a modifier like below.

 modifier check() {
        require(!_exists(_trayId),"");
        _;
    }

##

### [G-14] Functions guaranteed to revert_ when callled by normal users can be marked payable

If a function modifier or require such as onlyOwner-admin is used, the function will revert if a normal user tries to pay the function. Marking the function as payable will lower the gas cost for legitimate callers because the compiler will not include checks for whether a payment was provided. The extra opcodes avoided are CALLVALUE(2), DUP1(3), ISZERO(3), PUSH2(3), JUMPI(10), PUSH1(3), DUP1(3), REVERT(0), JUMPDEST(1), POP(2) which costs an average of about 21 gas per call to the function, in addition to the extra deployment cost.

FILE : 2023-03-canto-identity/canto-namespace-protocol/src/Namespace.sol

  function changeNoteAddress(address _newNoteAddress) external onlyOwner {
  function changeRevenueAddress(address _newRevenueAddress) external onlyOwner {

[Namespace.sol#L196-L208](https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Namespace.sol#L196-L208)

FILE : 2023-03-canto-identity/canto-namespace-protocol/src/Tray.sol

   function changeNoteAddress(address _newNoteAddress) external onlyOwner {
   function changeRevenueAddress(address _newRevenueAddress) external onlyOwner {
   function endPrelaunchPhase() external onlyOwner {
    
[Tray.sol#L210-L229](https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Tray.sol#L210-L229)

Recommendation
Functions guaranteed to revert when called by normal users can be marked payable (for only onlyOwner, mixedAuth, authorized functions)

##

### [G-15] The result of function calls should be cached rather than re-calling the function

FILE : 2023-03-canto-identity/canto-bio-protocol/src/Bio.sol

bytes(_bio).length should be cached instead of re-calling the function

  123: if (bytes(_bio).length == 0 || bytes(_bio).length > 200) revert InvalidBioLength(bytes(_bio).length);

[https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-bio-protocol/src/Bio.sol#L123](https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-bio-protocol/src/Bio.sol#L123)

##

### [G-16] IF statements that check input arguments should be at the top of the function

Checks that involve constants should come before checks that involve state variables, function calls, and calculations. By doing these checks first, the function is able to revert before wasting a Gcoldsload (2100 gas*) in a function that may ultimately revert in the unhappy case


 function mint(address _nftContract, uint256 _nftID) external {
        uint256 tokenId = ++numMinted;
        if (ERC721(_nftContract).ownerOf(_nftID) != msg.sender)
            revert PFPNotOwnedByCaller(msg.sender, _nftContract, _nftID);
        ProfilePictureData storage pictureData = pfp[tokenId];
        pictureData.nftContract = _nftContract;

[ProfilePicture.sol#L79-L88](https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-pfp-protocol/src/ProfilePicture.sol#L79-L88)

##

### [G-17] Use delete to clear variables instead of zero assignment to save gas 

FILE : 2023-03-canto-identity/canto-pfp-protocol/src/ProfilePicture.sol

   103:    nftID = 0; 

[ProfilePicture.sol#L103](https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-pfp-protocol/src/ProfilePicture.sol#L103)

FILE : 2023-03-canto-identity/canto-bio-protocol/src/Bio.sol

   93:   bytesOffset = 0;

[Bio.sol#L93](https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-bio-protocol/src/Bio.sol#L93)





   
















