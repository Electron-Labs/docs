# Developer Docs

In overview zkBridge currently works on Light Client Model with the small addition of signature verifications of the respectiv light client headers using zero knowledge proofs.

Transaction Flows:

1. Ethereum to Near
2. Near to Ethereum

## **Ethereum To Near :**

### Ethereum To NEAR Light Client Flow :

1. We store last 8192 (1 period = 256 epochs (1 epoch = 32 slots)) headers on `Ethereum Light Client on NEAR` contract.
2. Latest Beacon Header is sent after every \~32 blocks(1 epoch) which is used to verify every execution header before it which are constantly streamed to the NEAR smart contract.
3. The light client works on Altair Light Client sync\_committee model.
4. The sync committee updation and signature verification of headers is done using BLS signature zkproofs.

### Ethereum To NEAR Transaction Flow :

### Ethereum:

Ethereum side consists of a contract called `ERC20Locker.sol` . This contract is used to lock tokens by the user to the bridge by calling :

```solidity
function lockToken(
    address ethToken,
    uint256 amount,
    string memory accountId
) public pausable(PAUSED_LOCK)
```

The user who wants to lock the tokens beforehand has to give approval to the contract to transfer the tokens he wants to lock and then call this function.

`ERC20Locker.sol` doesn’t accept all kinds of ERC-20 tokens only the pre-approved tokens which are stored in the mapping:

```solidity
mapping(address => bool) public supportedTokens;
```

This mapping is controlled by an admin controlled function:

```solidity
function modifySupportedTokens (
    address tokenAddress,
    bool allow
) public onlyAdmin {
    supportedTokens[tokenAddress] = allow;
}
```

Once the token is transferred from user’s account to ERC20Locker.sol address it is considered to be locked and emits a Lock event and increases the `LockNonce` by 1:

```solidity
emit Locked(address(ethToken), msg.sender, amount, accountId, ++lockNonce);
```

This event is basically self explanatory other than `accountId` and `lockNonce` :

1. `accountId` here is a user sent argument which tells us what address to mint the wrapped tokens to on NEAR side.
2.  `lockNonce` is a variable declared as :

    ```solidity
    uint128 public lockNonce;
    ```

    and initialises from 0. While `lockNonce` is a simple uint128 variable which increments by 1 every time some user locks and ERC20 token. But has been deeply thought on and helps us maintain the order of transactions from Ethereum to NEAR and makes sure we dont miss any transaction. As `lockNonce` of a transaction to be accepted on NEAR side has to be one more than the `lockNonce` on NEAR side which too initialises from 0.

### Relayer:

Relayer uses webSockets to subscribe to the Locked event log transactions, the query looks like:

```
let query = {
        "jsonrpc": "2.0",
        "id": 1 ,
        "method": "eth_subscribe" ,
        "params": ["logs",{"address":ETH_CONTRACT_ADDRESS, "topics":[LOCK_EVENT]}]
    }
```

This webSocket subscription pushes transactions to RabbitMq queue as it receives them. The RabbitMq consumer (which picks up transaction and processes them further) waits till the the last updated block (`last_block_number`) on `Ethereum Light Client on NEAR` contract is ahead of the block of the transaction.

Once the last updated beacon block on `Ethereum Light Client on NEAR` contract is greater than the transaction’s block, a [merkle patricia tree](https://rockwaterweb.com/ethereum-merkle-patricia-trees-javascript-tutorial/) proof for the transaction is created and sent to the Bridge contract on NEAR where the merkle hash is verified against the `finalised_headers` on the Ethereum light client contract on NEAR.

### NEAR:

A lock event transaction along with its proof is sent to mint function on NEAR smart contract:

```rust
let Proof {
    log_index,
    log_entry_data,
    receipt_index,
    receipt_data,
    header_data,
    proof,
};
```

Smart contract verifies the below stated things:

1. Verifies that the above log was emitted from the correct smart contract address on ethereum.
2. `lockNonce` of the logs is 1 greater than the `lockNonce` stored in the state of NEAR smart contract.
3. Trie proof of the log event is verified and header of the transaction should be finalised for transaction to be valid.
4. If the near recipient address is correct then the amount emitted in the lock event is minted to the recipient NEAR account.

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
