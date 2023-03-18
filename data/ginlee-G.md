[G1]Setting the constructor to payable
 cut out 10 opcodes in the creation-time EVM bytecode if you declare a constructor payable. Making the constructor payable eliminates the need for an initial check of msg.value == 0 and saves 13 gas on deployment with no security risks
https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-bio-protocol/src/Bio.sol
constructor() ERC721("Biography", "Bio") {
        if (block.chainid == 7700) {
            // Register CSR on Canto mainnnet
            Turnstile turnstile = Turnstile(0xEcf044C5B4b867CFda001101c617eCd347095B44);
            turnstile.register(tx.origin);
        }
    }

https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Namespace.sol
 constructor(
        address _tray,
        address _note,
        address _revenueAddress
    ) ERC721("Namespace", "NS") Owned(msg.sender) {
        tray = Tray(_tray);
        note = ERC20(_note);
        revenueAddress = _revenueAddress;
        if (block.chainid == 7700) {
            // Register CSR on Canto mainnnet
            Turnstile turnstile = Turnstile(0xEcf044C5B4b867CFda001101c617eCd347095B44);
            turnstile.register(tx.origin);
        }
    }

https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol
https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-pfp-protocol/src/ProfilePicture.sol

[G2]Use nested if and, avoid multiple check combinations
https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-bio-protocol/src/Bio.sol
  if ((i > 0 && (i + 1) % 40 == 0) || prevByteWasContinuation || i == lengthInBytes - 1) 
  if (nextCharacter & 0xC0 == 0x80)
  if (
                        // Note that we do not need to check i < lengthInBytes - 4, because we assume that it's a valid UTF8 string and these prefixes imply that another byte follows
                        (nextCharacter == 0xE2 && bioTextBytes[i + 2] == 0x80 && bioTextBytes[i + 3] == 0x8D) ||
                        (nextCharacter == 0xF0 &&
                            bioTextBytes[i + 2] == 0x9F &&
                            bioTextBytes[i + 3] == 0x8F &&
                            uint8(bioTextBytes[i + 4]) >= 187 &&
                            uint8(bioTextBytes[i + 4]) <= 191) ||
                        (i >= 2 &&
                            bioTextBytes[i - 2] == 0xE2 &&
                            bioTextBytes[i - 1] == 0x80 &&
                            bioTextBytes[i] == 0x8D)
                    ) 

[G3]Using storage instead of memory for structs/arrays saves gas
https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Namespace.sol#L133
            Tray.TileData memory tileData = tray.getTile(trayID, tileOffset)

use storage instead of memory

[G4]Use constants instead of type(uintx).max
https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Tray.sol#L73
https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Tray.sol#L122
https://github.com/code-423n4/2023-03-canto-identity/blob/077372297fc419ea7688ab62cc3fd4e8f4e24e66/canto-namespace-protocol/src/Tray.sol#L152

type(uintx).max uses more gas in the distribution process and also for each transaction than constant usage, uint256 constant MAX_VALUE = 2**256 -1, use this instead of type(uint256).max

