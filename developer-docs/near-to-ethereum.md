# NEAR to Ethereum

## NEAR TO ETHEREUM TRANSACTION/LC FLOW:

### NEAR:

User calls the `burn` function on `NEP-141 burner contract` which has the given signature:

```rust
pub fn burn(&mut self, amount: U128, recipient: String, token_contract_eth: String)
```

The wrapped NEP-141 tokens are burned from caller’s account and `unlock_nonce` is increased by 1.

### RELAYER:

The relayer constantly reads the block for relevant transactions and once it finds a transaction, it fetches the next light client block to update using NEAR RPC endpoint : `next_light_client_block` .

It returns the `LightClientBlock` for the block as far into the future from the last known hash as possible for the light client to still accept it. Specifically, it either returns the last final block of the next epoch, or the last final known block. If there's no newer final block than the one the light client knows about, the RPC returns an empty result and we wait for the newer final block to be produced.

Relayer then creates a zkproof for validator signatures on this light client block and calls `addLightClientBlock` which receives function on `NearBridge.sol` .

```solidity
function addLightClientBlock(
    bytes memory data,
    uint256[2] memory a,
    uint256[2][2] memory b,
    uint256[2] memory c
)
```

`addLightClientBlock` verifies the block header as per [NEAR Light Client specs](https://nomicon.io/ChainSpec/LightClient) and verifies the Ed25519 signatures using the [zk-proof](https://github.com/Electron-Labs/ed25519-circom) submitted by the relayer.

Once the latest light client block is updated. We pick up the burn transaction receipts one by one and retrieve light client proof for those transactions

```jsx
near.connection.provider.sendJsonRpc(
            'light_client_proof', {
                type: 'receipt',
                receipt_id: txReceiptId,
                receiver_id: receiverId, //address of contract which received the txn
                light_client_head: latestLightClientBlockHash,//hash of the latest updated lc block
            }
        );
```

This proof data is borshified and sent over to the `ERC20Locker.sol` in the `unlockToken` function call.

### ETHEREUM:

The transaction’s light\_client\_proof consists of two proofs : transaction’s merkle root proof and block’s merkle root proof. Both the proofs are verified against the light client block header. Once these proofs are verified then the following information is extracted from the proof itself :

```solidity

function _decodeBurnResult(bytes memory data)
    internal
    returns (BurnResult memory result)
{
    Borsh.Data memory borshData = Borsh.from(data);
    uint8 flag = borshData.decodeU8();
    result.amount = borshData.decodeU128();
    bytes20 token = borshData.decodeBytes20();
    result.token = address(uint160(token));
    bytes20 recipient = borshData.decodeBytes20();
    result.recipient = address(uint160(recipient));
    uint128 nonce = borshData.decodeU128();
    result.unlockNonce = nonce;
    borshData.done();
}
```

If the `result.nonce == unlockNonce + 1` then this data is used to transfer transfer unlocked tokens to the recipient user and unlockNonce is increased by 1 :

```solidity
IERC20(result.token).safeTransfer(result.recipient, result.amount);
```
