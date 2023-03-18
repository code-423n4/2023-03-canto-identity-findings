**canto-pfp-protocol/src/ProfilePicture.sol**
- L5 - An interface with the name Turnstile is imported, but it is somewhat confusing, since a name does not contain the I first. It should be called ITurnstile.


**canto-bio-protocol/src/Bio.sol**
- L43 - The tokenURI() function has more than 74 lines of code and has multiple for and ifs loops, therefore it becomes difficult to understand. It is recommended to use auxiliary functions to improve the understanding of the code.


**canto-namespace-protocol/src/Namespace.sol**
- L110 - The fuse() function has 70 lines of code and has multiple for and ifs loops, therefore it becomes difficult to understand. It is recommended to use auxiliary functions to improve the understanding of the code.

- L78/79 - Missing checks for address(0) when assigning values ​​to address state variables.
This can be a serious problem, since if they were set with incorrect values, there would be no mechanism to update the values ​​of these two parameters.


**canto-namespace-protocol/src/Tray.sol**
- L122/124 - An if is done within an if, this should be unified to make it easier to understand and occupy fewer lines of code, like this:
if (numPrelaunchMinted != type(uint256).max && _id <= numPrelaunchMinted) revert PrelaunchTrayCannotBeUsedAfterPrelaunch(_id);

- L203 - The getTiles() function is created but it is never used throughout the project, if getTile() is used, but never the other, therefore it should be removed.

- L108/109 - Missing checks for address(0) when assigning values ​​to address state variables
This can be a serious problem, since if they were set with incorrect values, there would be no mechanism to update the values ​​of these two parameters.

- L31/34 - If a constant is only used in one place, what is the point of creating a constant in storage? This in my opinion should be created as a variable in memory in the function that is used.
Cases: NUM_CHARS_LETTERS_NUMBERS and PRE_LAUNCH_MINT_CAP.
 

**canto-namespace-protocol/src/Utils.sol**
- L24/31/37/58/59/63/64/65/66/67 - If a constant is only used in one place, what's the point of creating a constant in storage? This in my opinion should be created as a variable in memory in the function that is used.
Cases: ZALGO_NUM_ABOVE, ZALGO_NUM_BELOW, ZALGO_NUM_OVER, EMOJIS_MOD_SKIN_TONE_THREE_BYTES, EMOJIS_MOD_SKIN_TONE_FOUR_BYTES,
EMOJIS_BYTE_OFFSET_FOUR_BYTES, EMOJIS_BYTE_OFFSET_SIX_BYTES, EMOJIS_BYTE_OFFSET_SEVEN_BYTES,
EMOJIS_BYTE_OFFSET_EIGHT_BYTES and EMOJIS_BYTE_OFFSET_FOURTEEN_BYTES.

- L55 - Code is commented that if it is not needed it should be eliminated.

- L73 - The characterToUnicodeBytes() function has more than 140 lines of code and has multiple for and ifs loops, therefore it becomes difficult to understand. It is recommended to use auxiliary functions to improve the understanding of the code.

- L259 - When defining the value of iteratedState, the value is used in an operation: "2038074743" This value is not explained anywhere why it is that number. This should be explained, either with the name of a variable or with a comment.
