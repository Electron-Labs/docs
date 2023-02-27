# Overview

NEAR validators use the ED25519 signature scheme for signing blocks. Hence, for executing NEAR Light client in solidity, we need to verify 100's of validator's Ed25519 signatures. Hence, there is a need for succinct verification of a batch of ED25519 signatures. As such, we have developed a ZK-prover enables creating a zk-snark proof of ED25519 signatures.

The code can be found [here](https://github.com/Electron-Labs/circom-binary-ops). See the following sections to see the underlying mathematical details of how it works.

Our product is built on Circom and we use snarkJS and Rapidsnark for proof generation.

