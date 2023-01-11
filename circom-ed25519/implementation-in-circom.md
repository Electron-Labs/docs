# Implementation in Circom

We will now discuss how we can define the Ed25519 maths in the R1CS model, by using Circom. We will then use SnarkJS to generate zk-proofs of Ed25519 signatures. This section assumes a working knowledge of zk-snarks and Circom/SnarkJS.

### Step 1: Define representation in large numbers

see previous section

### Step2: Defining adder, multiplier, and modulus in base 2^51

Inputs and outputs to these circuits are arrays where each element of the array is less than `2^51`

**Adder**

Find the implementation of base2^85 Adder [here](https://github.com/Electron-Labs/ed25519-circom/blob/master/circuits/chunkedadd.circom)

**Subtractor**

Find the implementation of base2^85 Subtractor [here](https://github.com/Electron-Labs/ed25519-circom/blob/master/circuits/chunkedsub.circom)

**Multiplier**

Find the implementation of base2^85 Multiplier [here](https://github.com/Electron-Labs/ed25519-circom/blob/master/circuits/chunkedmul.circom). The algorithm is basically standard long multiplication, where the carry is handled in the end.

**Modulus**

Find the implementation of base2^85 Modulus [here](https://github.com/Electron-Labs/ed25519-circom/blob/master/circuits/modulus.circom). This template takes a base2^85 array as input and applies modulo `p = 2^255 - 19`. In the code, `p` is also represented as base2^85.

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

### Step 4: Defining Ed25519 Scalar Multiplication in Circom

Find the implementation of scalar multiplication [here](https://github.com/Electron-Labs/ed25519-circom/blob/master/circuits/scalarmul.circom)

### Step 5: Defining Full Signature Verification in Circom

Find the implementation of a single signature verification [here](https://github.com/Electron-Labs/ed25519-circom/blob/master/circuits/verify.circom)

### Step 5: Defining Batched (Multi) Signature Verification in Circom

Find the implementation of a batch signature verification [here](https://github.com/Electron-Labs/ed25519-circom/blob/master/circuits/batchverify.circom)

###
