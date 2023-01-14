# High Level Design

In the traditional bridge design, the bridge is secured by operators of a multi-sig that sign off on the transactions. However, this means that t he bridge security and asset custody is completely in the hands of the multi-sig operators.

Electron Protocol follows a security model where _no such centralized multi-sig operators are needed_.

### So, how do we achieve this?

In order to check the validity of transactions, we must a submit a proof of transaction validity. This means we must prove that the given transaction was included in the blockchain (Source Chain). This is called Proof of Consensus.

How do we construct a proof of consensus?

Blockchains provide a simple protocol called Light Client. A Light Client is a simple protocol designed to enable transaction verification in computational limited environments.

Hence, we will basically implement the Light Client of Source Chain inside the smart contract of Destination Chain. This means that if the relayer submits any spurious transactions, the light node can simply detect it and reject the transaction.

<figure><img src="https://s3.us-west-2.amazonaws.com/secure.notion-static.com/857db684-2146-43b2-9c4e-afda0072de57/Screenshot_2023-01-09_at_9.52.53_PM.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&#x26;X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&#x26;X-Amz-Credential=AKIAT73L2G45EIPT3X45%2F20230114%2Fus-west-2%2Fs3%2Faws4_request&#x26;X-Amz-Date=20230114T121332Z&#x26;X-Amz-Expires=86400&#x26;X-Amz-Signature=b0dce8c382604d06e5d785d5964519655487dc8f3d95770434cd458955279745&#x26;X-Amz-SignedHeaders=host&#x26;response-content-disposition=filename%3D%22Screenshot%25202023-01-09%2520at%25209.52.53%2520PM.png%22&#x26;x-id=GetObject" alt=""><figcaption></figcaption></figure>

However, when connecting to expensive blockchains such as Ethereum, executing a Light Node code inside a solidity smart contract is not feasible due to very high gas requirement.

For example, executing the NEAR light client in Ethereum would cost \~50 million gas. This is because we have to verify 100’s of validators signatures in solidity.

So how do we solve this?

Rather than submitting the Light Client Proofs directly, we instead construct a zk-proof of validity and submit that on-chain instead. The cost of verifying zk-proofs of Ethereum is order of magnitude cheaper than verifying the light client directly

<figure><img src="../.gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

**Does zk-proof generation introduce any new trust assumptions?**

No, unlike oracle designs etc, this does not happen with ZK-proving. This is because unlike oracles, etc , which are based on economic security, ZK-proofs are secured by Mathematics. zk-provers cannot construct fake proofs since they will simply fail to verify on-chain.
