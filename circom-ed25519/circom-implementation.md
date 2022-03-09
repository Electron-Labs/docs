---
description: Zk-proof of a single ED25519 signature
---

# Circom Implementation

We will now discuss how we can define the ED25519 maths in the R1CS model, by using Circom. We will then use snarkJS to generate zk-proofs of ED25519 signatures. This section assumes a working knowledge of zk-snarks and Circom/snarkJS.

### Step1: Defining ED25519 prime field inside the altbn prime field

Both Circom/zk-snarks and ED25519 are defined under a finite field. This means all math operations are performed under modulo a prime number.

Prime Number for ED25519: `p = 2^255 - 19`

Prime Number for Circom/zk-snarks: `altbn_P = 21888242871839275222246405745257275088548364400416034343698204186575808495617`

Note that the prime `p` used by ED25519 is different and greater in value than `altbn_P`. Hence, we must define an approach that allows us to perform ED25519 operations while keeping all numbers within the snark prime field.

One way to solve this problem is to represent all numbers as an array of base2 numbers (binary). This would make sure that no single element exceeds `altbn_P` but the entire array could represent larger numbers. In fact, we can choose any base `b` such that `b < altbn_P`

Max value of any number in ED25519 is `p`and it can be represented in a max of 255 bits. In base`2^51`, we need a max of 5 elements to represent all ed25519 numbers. The number of constraints is the same when multiplying `1`\*`1` or `2^51`\*`2^51`. Hence, it makes sense to use the largest base possible.

We have chosen to use base `2^51`. (although base`2^85` is even better). Note that `2^51 < altbn_P` which is necessary.

Next, we must define addition, multiplication, and modulus operator in base`2^51` inside circom, so that we can start doing ED25519 maths.

### Step2: Defining adder, multiplier, and modulus in base 2^51

Inputs and outputs to these circuits are arrays where each element of the array is less than `2^51`

#### Adder

Find the implementation of base2^51 Adder [here](https://github.com/Electron-Labs/circom-ed25519/blob/master/circuits/chunkedadd.circom#L52)

#### Multiplier

Find the implementation of base2^51 Multiplier [here](https://github.com/Electron-Labs/circom-ed25519/blob/master/circuits/binmulfast.circom#L87). The algorithm is basically standard long multiplication, where the carry is handled in the end.&#x20;

#### Modulus

Find the implementation of base2^51 Modulus [here](https://github.com/Electron-Labs/circom-ed25519/blob/master/circuits/modulus.circom#L58). This template takes a base2^51 array as input and applied modulo `p = 2^255 - 19`. In the code, `p` is also represented as base2^51.

We perform modulo in such a way that long division is avoided. The algorithm is as follows -&#x20;

```
p = 2^255 - 19
define c = 2^255

Given input x s.t max(x) < p^2, calculate x%p
x%c = r (remainder) (note that max(r) < c)
x//c = q (quotient) (note that max(q) < p^2/c)
x = q*c + r
x = q*(p+19) + r
x%p = 19q%p + r%p

Now, calculate r%p -
if (r>=p): r%p = r-p
if (r<p): r%p = r

The same logic is extended to calculate 19q
```

In the circuit, we can't use if/else conditionals but a multiplexer. In the circuit, we apply the above algorithm recursively, and hence `x` can be of arbitrary size.

### Step3: Defining ED25519 Point Addition in Circom

First, we must represent all the ED25519 operations as polynomials so that they can be represented in the R1CS constraint model.

Let's start with point addition. Re-arranging the point addition equation discussed in the previous section, we get:-

$$
x_3 =  x_1y_2 + x_2y_1 + (\small 121665/121666\normalsize )x_1x_2x_3y_1y_2
\newline
y_3 + (\small 121665/121666\normalsize )x_1x_2y_1y_2y_3 = y_1y_2 + x_1x_2
$$

Hence, if three points `(x1, y1)`, `(x2, y2)` and `(x3,y3)` satisfy these two polynomials, we can say that the third point is the sum of the first two points.

In our circom implementation, rather than using cartesian coordinates, we use the radix format as defined in the reference implementation (given [here](https://datatracker.ietf.org/doc/html/rfc8032#page-20)). This changes the polynomials a bit, but the core logic is the same. Please see the code [here](https://github.com/Electron-Labs/circom-ed25519/blob/master/circuits/point-addition.circom).

### Step4: Defining ED25519 Scalar Multiplication in Circom

\<add here>

### Step5: Defining Full Signature Verification in Circom

\<add here>
