# Performance Metrics

### Benchmarks

All metrics were measured on a 16-core 3.0GHz, 32G RAM machine (AWS c5a.4xlarge instance).

|                                      | Single ED25519 Signature |
| ------------------------------------ | ------------------------ |
| Constraints                          | 2,564,061                |
| Circuit compilation                  | 72s                      |
| Witness generation                   | 6s                       |
| Trusted setup phase 2 key generation | 841s                     |
| Trusted setup phase 2 contribution   | 1040s                    |
| Proving key size                     | 1.6G                     |
| Proving time (rapidsnark)            | 6s                       |
| Geth Proof verification time         | 10 ms                    |
| Proof verification Gas Cost on EVM   | \~227,000                |











