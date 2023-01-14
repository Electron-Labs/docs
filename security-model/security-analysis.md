# Security Analysis

### Quick Overview of Light Clients in PoS systems

In typical PoS, a block consists of following fields -

1. The list of all transactions contained in that block
2. A Merkle root of all these transactions
3. Finally, the list of all validators signatures that sign on the Merkle root of the transactions.
4. All informations such as blockheight, blockhash, voting power, etc are also part of the block-header and get’s signed by the validators.

```json
{
	"block_hash": "96kX4zv5XnXACdjSvVyDtzytBR1dqu4eNXLmkqsEFR8v",
	"block_height": 80418378,
	"tx_merkle_root": "32ZPM9LCVqdSpG7zxkS6JgLA4KFiBCttiphZi2SkLadb",
	"validtor_signatures": [
            "ed25519:qj1PB9eXrEehf31nZQbsqDx4waSgFdWyBsMw2T6KFvuE2n8HGip8ppNmaVKgyBfd5Pa3pV78CLw6kPstgQdTXNV",
            "ed25519:Buut6TbPop9BDQBdBfxcTw1CGsubyEcwWX9xgfsaw9isaLVDmugk8anmvhHLWGYEgM3YjxE3tYWpoPsVksVxU9U",
            "ed25519:5yxML6vgLfwCJCDiRNveu3wvv1PbZeWr8tP86tuDF1ENMiWv921UkiE9UeeQkd91Bvw6kc3LkDtdfQnbRVrUSxTs",
            "ed25519:2tWXSajBhsHJSQjPHPszgEMG4gyyG9ag5v9i3Qvty9MaEw6j7EixmHQFDVdS3FUHDEaDpSijPkxaEmy8wKkyKajo",
            "ed25519:4LuEg2cvxkdh2aQJsL86Utux1tkPtPQTGnwomneryPNZTzH2LdezP6rhb3FuJ2nrVr4vD1fegYZTGLSwA1jqxsCq",
            "ed25519:2DYNfHrrkSpfHTtHGrqXYEdfPkmM1EuKgnsecBVmfjgXZQLeuC9joWuiuPjfSjN1QehegY9BSeUUiGR74Ftn7s7C",
						...
						...
        ],
	"next_validator_set_hash": "D1M6VFrhBrFasuidZvWZQvzNc7ALLitHE4psEfJTbCis"
}
```

Note: This is all overly simplified, but in principal, this is what happens in PoS.

Now, in order to run a full node, one has to download all the transactions, verify them one by one in order to convince themselves about the validity of the chain. However, a light node avoids downloading all the transactions by simply tracking the blockheaders. If a light node wants to verify a transaction, it simply checks whether the given transaction has a valid merkle path leading to the merkle root (which has been signed by the validators).

### **How do LC bridges use this?**

In a very simple Light client bridge, we implement the light node as a smart contract on the receiving chain. Now, if the relayer submits transactions that are invalid, the on-chain light node can detect and reject them. But what if the relayer submit’s fake blockheaders? This is not possible since the relayer would have to create fake digital signatures (which is not possible as guaranteed by the cryptography behind digital signatures).

Please note that in the given blockheader, the validator’s also sign on the set of validators that will sign the next block. Hence, a light node already knows the validator’s who signatures it needs to check for.

Conclusion: Unless the relayer can convince the 67% validators to sign a fake block, the bridge cannot be hacked.

_We consider this Security Level 1: i.e. the relayer and 67% validators must collude together to successfully hack the bridge._





So how can we make sure we don’t have to trust that validator’s are not colluding together? Let’s talk about Security Level 2.

<figure><img src="../.gitbook/assets/image (3).png" alt=""><figcaption></figcaption></figure>

In order to prevent chain forking, PoS systems impose slashing on double signing. This means that if validators sign two different blocks at the same height, they will get slashed. How does slashing actually happen? The entity (could be anyone) who detects the doubly signed block can submit this block to the canonical chain, and this will trigger the slashing module. Typically there are rewards to reporting such malicious behaviour.

**So how can we take advantage of this to improve the security of our bridge?**

Consider the scenario that validators collude together to attack our bridge and successfully submit a double signed block on the destination chain. We assume that there are multiple relayer’s in the network, and that at least one of them is honest. This relayer will see that the block in the canonical chain at height H is different than the one in the destination chain light node. This honest relayer will then pick this doubly signed block and submit it to the canonical chain leading to slashing for 67% of all validators on the main chain. The relayer will do this in hope of getting rewarded for reposting malicious behaviour.

Hence, we can consider the bridge to be secure as long as the $ value of the incoming transaction is less than the $ value of the stake that would be lost due to slashing. We are also assuming that there is at least one honest relayer in the network.



\<add content explaining the need to have multiple relayers in the network>



#### **Further Reading -**

Another good article for security analysis of LC bridges is given here -[https://medium.com/composable-finance/trustless-bridging-ii-byzantine-fault-tolerance-591e8b2d196e](https://medium.com/composable-finance/trustless-bridging-ii-byzantine-fault-tolerance-591e8b2d196e)

