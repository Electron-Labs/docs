---
description: >-
  Simple and easy to use smart contract based SDK to use zk-ed25519 in your
  project
---

# How to Integrate in your project?

Designed for rust L1's and Dapps to bring zk to their blockchain. Deploy zk-rollups, built ultralight clients, secure cross-chain bridges, enable privacy...

### Benchmarks

All benchmarks were run on a 16-core 3.0GHz, 32G RAM machine (AWS c5a.4xlarge instance).

|                                      | verify.circom |
| ------------------------------------ | ------------- |
| Constraints                          | 2564061       |
| Circuit compilation                  | 72s           |
| Witness generation                   | 6s            |
| Trusted setup phase 2 key generation | 841s          |
| Trusted setup phase 2 contribution   | 1040s         |
| Proving key size                     | 1.6G          |
| Proving time (rapidsnark)            | 6s            |
| Proof verification time              | 1s            |



