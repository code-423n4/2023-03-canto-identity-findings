### [G] SETTING THE CONSTRUCTOR TO PAYABLE

You can cut out 10 opcodes in the creation-time EVM bytecode if you declare a constructor payable. Making the constructor payable eliminates the need for an initial check of msg.value == 0 and saves 13 gas on deployment with no security risks.

### [G] USE ASSEMBLY TO WRITE ADDRESS STORAGE VALUES

**src\Tray.sol**

    constructor(
        bytes32 _initHash,
        uint256 _trayPrice,
        address _revenueAddress,
        address _note,
        address _namespaceNFT
    ) ERC721A("Namespace Tray", "NSTRAY") Owned(msg.sender) {
        lastHash = _initHash;
        trayPrice = _trayPrice;
    -	revenueAddress = _revenueAddress;
        note = ERC20(_note);
    -	namespaceNFT = _namespaceNFT;
        if (block.chainid == 7700) {
            // Register CSR on Canto mainnnet
            Turnstile turnstile = Turnstile(0xEcf044C5B4b867CFda001101c617eCd347095B44);
            turnstile.register(tx.origin);
        }
    }
    + assembly {
            sstore(revenueAddress.slot, _revenueAddress)
        }
    + assembly {
            sstore(namespaceNFT.slot, = _namespaceNFT)
        }
Similar operations can be done in the following parts of the code:

    220: revenueAddress = _newRevenueAddress;

### [G] USE CONSTANTS INSTEAD OF TYPE(UINTX).MAX

type(uint120).max or type(uint112).max, etc. it uses more gas in the distribution process and also for each transaction than constant usage.

**src\Tray.sol**

    226: if (prelaunchMinted != type(uint256).max) revert PrelaunchAlreadyEnded();
    238: if (numPrelaunchMinted != type(uint256).max) {

### [G] REQUIRE(), REVERT() STATEMENTS THAT CHECK INPUT ARGUMENTS SHOULD BE AT THE TOP OF THE FUNCTION

**src\ProfilePicture.sol**

    79: function mint(address _nftContract, uint256 _nftID) external {
        uint256 tokenId = ++numMinted;
        if (ERC721(_nftContract).ownerOf(_nftID) != msg.sender)  //should be at the first line of the function
            revert PFPNotOwnedByCaller(msg.sender, _nftContract, _nftID);
