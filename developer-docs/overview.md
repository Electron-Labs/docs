# Overview

Electron ZK-Bridge follows a design where all transactions are validated using ZK Light Clients, and not by a multi-sig in the middle.

In our bridge, we verify ZK-proofs of the NEAR Light Client on Ethereum, and verify ZK-proofs of the Ethereum Light Client on NEAR, as shown below -

<figure><img src="../.gitbook/assets/image (4).png" alt=""><figcaption></figcaption></figure>

Hence, there are two Transaction Flows:

1. Ethereum to Near
2. Near to Ethereum

Let us look at each one of them.
