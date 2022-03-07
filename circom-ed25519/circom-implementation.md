---
description: Zk-proof of a single ED25519 signature
---

# Circom Implementation

We will now discuss how we create a zk-proof of a single ED25519 signature. This section assumes a working knowledge of zk-snarks and circom/snarkJS.

### Step1: Making ED25519 zk-ready

First, we must represent all the ED25519 operations as polynomials so that they can be represented in R1CS constraint model.

Let's start with point addition. Re-arranging the point addition equation discussed in the previous section, we get:-

$$
x_3 =  x_1y_2 + x_2y_1 + (\small 121665/121666\normalsize )x_1x_2x_3y_1y_2
\newline
y_3 + (\small 121665/121666\normalsize )x_1x_2y_1y_2y_3 = y_1y_2 + x_1x_2
$$

Hence, if three points `(x1, y1)`, `(x2, y2)` and `(x3,y3)` satisfy these two polynomials, we can say that the third point is the sum of the first two points.

In our circom implementation, rather than using cartesian coordinates, we use the radix format as defined in the reference implementation (given [here](https://datatracker.ietf.org/doc/html/rfc8032#page-20)). This changes how the polynomials look, but  the core logic is the same. Please see the code [here](https://github.com/Electron-Labs/circom-ed25519/blob/master/circuits/point-addition.circom).

### Step2: Defining ED25519 prime field inside the altbn prime field

Circom and zk-snarks define all numbers and operations under modulo of the albn128 prime number, where the prime number is `altbn_P = 21888242871839275222246405745257275088548364400416034343698204186575808495617`

This means any operation performed in circom has a `modulo altbn_P` applied to it by default. Note that the prime `p` used by ED25519 is different and greater in value than `altbn_P`

Hence, we must define an approach that allows us to perform ed25519 operations while keeping all numbers within the circom prime field.

One way to solve this problem is to represent all numbers as an array of base2 numbers (binary). This would make sure that no single element exceeds `altbn_P` but the entire array could represent larger numbers. In fact we can choose any base `B` such that `B < altbn_P`

We have chosen to use base `2^51`.

### Step3: Defining adder, multiplier, and modulus in base 2^51

\<add here>

