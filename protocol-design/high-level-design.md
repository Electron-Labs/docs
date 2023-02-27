# High Level Design

In the traditional bridge design, the bridge is secured by operators of a multi-sig that sign off on the transactions. However, this puts the bridge security and asset custody _completely_ in the hands of the bridge operators.

Electron Protocol follows a security model where _no such centralized multi-sig operators are needed_.

### So, how do we achieve this?

In order to check the validity of transactions, we must a submit a proof of transaction validity. This means we must prove that the given transaction was included in the blockchain (Source Chain). This is called Proof of Consensus.

How do we construct a proof of consensus?

Blockchains provide a simple protocol called Light Node. A Light Node is "lighter" version of a full-node. As opposed to a fullnode, it does not download and verify all transactions and only download the transaction that is is interested in. As a result, it requires much lesser computational, storage and network resources. is a simple protocol designed to enable transaction verification in computational limited environments.

Hence, we basically implement the Light Node of Source Chain as a smart contract on the Destination Chain. This means that if the relayer submits any spurious transactions, the light node can simply detect it and reject the transaction.

<figure><img src="../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

However, when connecting to expensive blockchains such as Ethereum, executing a Light Node code inside a solidity smart contract is not feasible due to very high gas requirement.

For example, executing the NEAR light client in Ethereum would cost \~50 million gas. This is because we have to verify 100’s of validators signatures in solidity.

So how do we solve this?

Rather than submitting the Light Client Proofs directly, we instead construct a zk-proof of validity and submit that on-chain instead. The cost of verifying zk-proofs of Ethereum is order of magnitude cheaper than verifying the light client directly.

<figure><img src="../.gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

**Does zk-proof generation introduce any new trust assumptions?**

No, unlike oracle designs etc, this does not happen with ZK-proving. This is because unlike oracles, etc , which are based on economic security, ZK-proofs are secured by Mathematics. zk-provers cannot construct fake proofs since they will simply fail to verify on-chain.
