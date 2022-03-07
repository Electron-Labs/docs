# Overview

ED25519 has emerged as a popular signature scheme in the blockchain ecosystem. Verifying blockchain consensus requires verifying a large number of ED25519 signatures (validators' signatures). Hence, there is a need for succinct verification of a batch of ED25519 signatures.

We have developed a library that enables creating a zk-snark proof of ED25519 signatures.

The code can be found [here](https://github.com/Electron-Labs/circom-binary-ops). See the following sections to see the underlying details of how it works.

