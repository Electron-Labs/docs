---
description: >-
  Introducing large numbers in circom to deal with the large prime associated
  with ED25519
---

# Large numbers in circom

### Introducing large numbers in circom

An `F_p`-arithmetic circuit is a circuit consisting of set of wires that carry values from the field F\_p and connect them to addition and multiplication gates `modulo p`.

For circom, the default `F_p`, where `p = 21888242871839275222246405745257275088548364400416034343698204186575808495617` is defined as the set of numbers `{0,...,p-1}`. This is the same prime as the group order of elliptic curve alt\_bn128.

Now, dealing with numbers larger than the aforementioned prime is not possible using the native operators and field elements in circom. We need to introduce a new representation to deal with large numbers and define arithmetic operations on the same. We're going to dicuss two methods to doing the same

### Binary (Base 2) representation

The easiest approach is to represent all numbers in binary format, with each signal/variable (field element) in circom representing each bit. Since the only enumerable value for each signal/variable is `0 | 1`, implementing addition, subtraction (using 2's complement), multiplication is fairly straightfoward. Any well known algorithms can be used to do the same. However, using a binary representation is not efficient because we end up using only a fraction of information each signal/variable can capture, requiring a much larger set of signal/variables to define the same computation. This, in turn, leads to an exponentially larger set of constraints compared to native operations, which translates to larger proving time and larger memory requirements.

### Using a larger base

We can represent numbers in the same manner a lot of BigInteger libraries do, by fitting smaller chunks inside the largest possible native element and using an array of such chunks to form larger numbers. We can fit at max `d = p - 1` in a single signal/variable. Instead of base 2 (binary), we can let each signal represent a "digit" up to this number `d`. In other words, a number system with base `d`. Now the information captured by each signal can be maximized, reducing the the required computation for basic arithmetic operations compared to a binary representation.

> The base we used in our implementation is 2^85, which is much smaller than the maximum base that we can use. There's a speciifc reason for this, related to our implemnetation of performing modulus on our representation of large numbers. This is discussed in later sections.
