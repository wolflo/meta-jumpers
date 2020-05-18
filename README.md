# Meta Jumpers
Solidity contracts that jump into their own metadata (presumably for malicious purposes).
These contracts were both deployed to Rinkeby, verified on Etherscan without any selfdestruct or delegatecall opcodes in their source code, then selfdestructed.
Etherscan does not verify the metadata that the solidity compiler appends to the compiled bytecode, and this is still valid bytecode as far as the EVM is concerned.
Anything hidden in the metadata can be jumped to and executed.
See the deployed 0.4 contract at [0xf8bb3f896f8a69a49e6daad1860106d8a44c3503](https://rinkeby.etherscan.io/address/0xf8bb3f896f8a69a49e6daad1860106d8a44c3503#code) and the 0.6 contract at [0x587f0016bf800E621B7cBBB2644B4E902233Ec7f](https://rinkeby.etherscan.io/address/0x587f0016bf800e621b7cbbb2644b4e902233ec7f#code).

The point is to verify / illustrate that metadata is currently treated as valid evm bytecode, and our methods for verifying its validity are lacking.
But, this is not just about trusting or not trusting Etherscan code snippets.
Appending executable bytecode to contracts and considering it data just because we call it data is a dangerous pattern.  

See [EIP 2327](https://eips.ethereum.org/EIPS/eip-2327) for a potential mitigation for this issue.  

### Some details and limitations
#### version 0.4
- The original meta-jumper code used solidity 0.4.24. This is a fairly old compiler version, and was used because more recent versions of solidity do not not allow for jumps or direct stack manipulation in assembly.

#### version 0.6
- As it turns out, function pointers provide access to arbitrary jumps in what is likely a more deceptive way than raw assembly jumps. The metadata that version ^0.6 of the solidity compiler outputs is structured a bit differently than what is discussed below and contains an ipfs hash rather than a swarm hash, but the idea is the same.

#### Concealing from Etherscan
- Etherscan does seem to verify the structure of the metadata.
For solidity ^0.4, this structure is
```
0xa1 0x65 'b' 'z' 'z' 'r' '0' 0x58 0x20 <32 bytes swarm hash> 0x00 0x29
```
- We can only replace bytes in the swarm hash, and we also need to be wary of accidental push instructions in the hash, as the EVM will invalidate jumpdests that are within push data.
- [bin/jumper](https://github.com/wolflo/meta-jumper06/blob/master/bin/jumper06) was used to insert the selfdestruct snippet and contains more details on the manipulation of the swarm hash.
