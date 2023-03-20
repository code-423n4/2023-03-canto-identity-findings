### Rug vectors by the owner frontrun

``` solidity
Namespace.sol
195: 
196:     function changeNoteAddress(address _newNoteAddress) external onlyOwner {
197:         address currentNoteAddress = address(note);
198:         note = ERC20(_newNoteAddress);
199:         emit NoteAddressUpdate(currentNoteAddress, _newNoteAddress);
200:     }
201: 
202:     /// @notice Change the revenue address
203:     /// @param _newRevenueAddress New address to use
204:     function changeRevenueAddress(address _newRevenueAddress) external onlyOwner {
205:         address currentRevenueAddress = revenueAddress;
206:         revenueAddress = _newRevenueAddress;
207:         emit RevenueAddressUpdated(currentRevenueAddress, _newRevenueAddress);
208:     }
Tray.sol
210:     function changeNoteAddress(address _newNoteAddress) external onlyOwner {
211:         address currentNoteAddress = address(note);
212:         note = ERC20(_newNoteAddress);
213:         emit NoteAddressUpdate(currentNoteAddress, _newNoteAddress);
214:     }
215: 
216:     /// @notice Change the revenue address
217:     /// @param _newRevenueAddress New address to use
218:     function changeRevenueAddress(address _newRevenueAddress) external onlyOwner {
219:         address currentRevenueAddress = revenueAddress;
220:         revenueAddress = _newRevenueAddress;
221:         emit RevenueAddressUpdated(currentRevenueAddress, _newRevenueAddress);
222:     }
223: 
```

## Recommended Mitigation Steps
As a mitigation add a timelock and make sure the owner is a multisig and not an EOA.

### Critical Changes Should Use Two-step Procedure
It's possible to lose the ownership under specific circumstances.
Because an human error it's possible to set a new invalid owner. When you want to change the owner's address it's better to propose a new owner, and then accept this ownership with the new wallet.

``` solidity
Namespace.sol
11: contract Namespace is ERC721, Owned {
Tray.sol
13: contract Tray is ERC721A, Owned {
Tray.sol
104:     ) ERC721A("Namespace Tray", "NSTRAY") Owned(msg.sender) {
Namespace.sol
77:     ) ERC721("Namespace", "NS") Owned(msg.sender) {    
```

## Recommended Mitigation Steps
Lack of two-step procedure for critical operations leaves them error-prone. Consider adding two step procedure on the critical functions

### Lack of Input Validation
Lack of input validation is the process of checking input data to ensure that it is valid and safe to process. Therefore, it is important for contracts to consider the lack of input validation to ensure that what is produced is secure and protected from potential security vulnerabilities. This can be achieved by ensuring that every user input is thoroughly checked and validated at every level of data processing, from user interface to database. By doing so, the application can be reliable and protected from security threats that may arise due to lack of input validation.

``` solidity
Namespace.sol
73:     constructor(
74:         address _tray,
75:         address _note,
76:         address _revenueAddress
77:     ) ERC721("Namespace", "NS") Owned(msg.sender) {
78:         tray = Tray(_tray);
79:         note = ERC20(_note);
80:         revenueAddress = _revenueAddress;

Tray.sol
98:     constructor(
99:         bytes32 _initHash,
100:         uint256 _trayPrice,
101:         address _revenueAddress,
102:         address _note,
103:         address _namespaceNFT
104:     ) ERC721A("Namespace Tray", "NSTRAY") Owned(msg.sender) {
105:         lastHash = _initHash;
106:         trayPrice = _trayPrice;
107:         revenueAddress = _revenueAddress;
108:         note = ERC20(_note);
109:         namespaceNFT = _namespaceNFT;

```



