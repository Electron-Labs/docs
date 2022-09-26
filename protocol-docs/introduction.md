---
description: Introducing Electron Protocol
---

# Introduction

Today, we present a new protocol to build privacy-preserving applications on blockchains. Blockchains of today rely on executing transactions and smart contracts on public nodes, to ensure their validity. However, this exposes “everything” to “everyone” meaning all applications and users' data become completely public.

We are introducing a new protocol that allows you to build private preserving Dapps. Using Electron, you can build everything you can also build with a smart contract but do so in a manner that preserves the users' privacy.

![](../.gitbook/assets/image.png)

In the current blockchain architecture, the state is stored on-chain, and managed via a smart contract. Users then interact with this state via transactions.

In the privacy setting, we want the on-chain state and transactions to be encrypted, and in a manner such that the validity and immutability of the blockchain are not affected. We achieve this property using a magical cryptographic construct called Zero-Knowledge Proofs.

![](<../.gitbook/assets/image (1).png>)

Let's try to develop a "blackbox" understanding of ZKPs.
