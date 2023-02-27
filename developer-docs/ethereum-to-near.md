# Ethereum To NEAR

The relayer must do things - relay the Ethereum Light Client Headers to NEAR, and relay the transaction from Eth to NEAR. Lets us look at each of them.

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

##
