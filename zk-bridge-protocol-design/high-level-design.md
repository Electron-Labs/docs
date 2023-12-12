# High Level Design

In the traditional bridge design, the bridge is secured by operators of a multi-sig that sign off on the transactions. However, this puts the bridge security and asset custody _completely_ in the hands of the bridge operators.

Electron Protocol follows a security model where _no such centralized multi-sig operators are needed_.

Electron ZK-Bridge follows a design where all transactions are validated using ZK Light Clients, and not by a multi-sig in the middle.

In our bridge, we verify ZK-proofs of the NEAR Light Client on Ethereum, and verify ZK-proofs of the Ethereum Light Client on NEAR, as shown below -



<figure><img src="../.gitbook/assets/Screenshot 2023-02-28 at 12.07.30 AM.png" alt=""><figcaption></figcaption></figure>

