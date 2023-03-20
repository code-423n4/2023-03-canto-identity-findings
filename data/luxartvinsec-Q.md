# 1. Parameter does not match the expected format
https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-pfp-protocol/interface/ICidNFT.sol#L53
https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-pfp-protocol/src/ProfilePicture.sol#L99

# 2. Potential Out-of-bounds Read in Bio NFT Contract 
### Link : https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-bio-protocol/src/Bio.sol#L43
### Summary: 
The Bio NFT contract has a potential out-of-bounds read vulnerability in the `tokenURI` function. An attacker may be able to read unintended memory contents or execute arbitrary code by passing a specially crafted string as the bio value while minting an NFT.
### Impact: 
An attacker can exploit this vulnerability to read sensitive data or execute arbitrary code. If the contract is used in a production environment, this may result in significant financial losses, reputation damage, and legal liabilities.
### Recommendation: 
To fix this vulnerability, the contract should ensure that it doesn't try to access memory outside the bounds of the input string. A possible solution would be to replace the for loop that reads and splits the input string with a safer implementation that uses `bytes.split`.
### Example:
```
function tokenURI(uint256 _id) public view override returns (string memory) {
        if (_ownerOf[_id] == address(0)) revert TokenNotMinted(_id);
        string memory bioText = bio[_id];
        bytes memory bioTextBytes = bytes(bioText);
        uint lengthInBytes = bioTextBytes.length;
        // Insert a new line after 40 characters, taking into account unicode character
        uint lines = (lengthInBytes - 1) / 40 + 1;

        string[] memory strLines = new string[](lines);
        bytes[] memory byteLines = new bytes[](lines);
        uint i;
        for(i = 0; i < lines; i++){
            byteLines[i] = bytes(splitStr(bioText, 40, i));
            strLines[i] = string(byteLines[i]);
        }
        
        string memory svg = '<svg xmlns="http://www.w3.org/2000/svg" preserveAspectRatio="xMinYMin meet" viewBox="0 0 400 100"><style>text { font-family: sans-serif; font-size: 12px; }</style>';
        string memory text = '<text x="50%" y="50%" dominant-baseline="middle" text-anchor="middle">';
        for (i = 0; i < lines; ++i) {
            text = string.concat(text, '<tspan x="50%" dy="20">', strLines[i], "</tspan>");
        }
        string memory json = Base64.encode(
            bytes(
                string.concat(
                    '{"name": "Bio #',
                    LibString.toString(_id),
                    '", "description": "',
                    bioText,
                    '", "image": "data:image/svg+xml;base64,',
                    Base64.encode(bytes(string.concat(svg, text, "</text></svg>"))),
                    '"}'
                )
            )
        );
        return string(abi.encodePacked("data:application/json;base64,", json));
    }

function splitStr(string memory s, uint chunkSize, uint i) internal pure returns (string memory) {
        uint startIndex = i * chunkSize;
        if (startIndex >= bytes(s).length) {
            return "";
        }

        uint256 endIndex = startIndex + chunkSize;
        if (endIndex > bytes(s).length) {
            endIndex = bytes(s).length;
        }

        return s[startIndex:endIndex];
}
```
# 3. Insecure random number generation: 
### Link : https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L245
The current implementation of the `_drawing` function uses a simple modulo operation with the seed as an argument, which can be easily predicted by attackers. We recommend using a more secure random number generation algorithm, such as Keccak256 or SHA-3, to generate random numbers.
```
function _drawing(uint256 _seed) private pure returns (TileData memory tileData) {
    uint256 res = uint256(keccak256(abi.encodePacked(_seed))) % SUM_ODDS;
    uint256 charRandValue = Utils.iteratePRNG(_seed); // Iterate PRNG to not have any biasedness / correlation between random numbers
    if (res < 32) {
        // Class is 0 in that case
        tileData.characterIndex = uint16(charRandValue % NUM_CHARS_EMOJIS);
    } else {
        tileData.characterIndex = uint16(charRandValue % NUM_CHARS_LETTERS);
        if (res < 64) {
            tileData.fontClass = 1;
            tileData.characterIndex = uint16(charRandValue % NUM_CHARS_LETTERS_NUMBERS);
        } else if (res < 80) {
            tileData.fontClass = 2;
        } else if (res < 96) {
            tileData.fontClass = 3 + uint8((res - 80) / 8);
        } else if (res < 104) {
            tileData.fontClass = 5 + uint8((res - 96) / 4);
        } else if (res < 108) {
            tileData.fontClass = 7 + uint8((res - 104) / 2);
            if (tileData.fontClass == 7) {
                // Set seed for Zalgo to ensure same characters will be always generated for this tile
                uint256 zalgoSeed = Utils.iteratePRNG(_seed);
                tileData.characterModifier = uint8(zalgoSeed % 256);
            }
        } else {
            tileData.fontClass = 9;
        }
    }
}
```
# 4. Zero `_amount` Argument in `buy` Function Allows for Supply Chain Attacks in `Tray` Contract
### Link : https://github.com/code-423n4/2023-03-canto-identity/blob/main/canto-namespace-protocol/src/Tray.sol#L150
The vulnerability in the contract lies in the `buy` function, which does not check if the `_amount` argument passed to it is zero or not. An attacker can exploit this and call the `buy` function with an `_amount` argument value of zero to cause supply chain attacks against the `Tray` contract.
### Recommendation : 
To address this vulnerability, a simple fix would be to add a `require` statement at the beginning of the `buy` function that checks if the `_amount` argument is greater than zero or not. If `_amount` is zero, the function should `revert` with an appropriate error message.
