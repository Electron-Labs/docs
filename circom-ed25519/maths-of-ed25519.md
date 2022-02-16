# Maths of ED25519

ED25519 signature uses a curve and involves performing operations on this curve. Let's dive deeper.

All operations in this signature scheme are defined under modulo `p` where&#x20;

$$
p = 2^{255} - 19
$$

The curve equation is defined as follows:-

$$
ax^2 + y^2 = 1 + dx^2y^2
$$

where `a = -1` and `d = - 121665/121666`

Base Point is defined as `(Bx, By)` where `By = 4/5`. Here the `/` operator represents the inverse modulo operation wrt `p`. Hence `By = 4*mod_inverse(5,p) => By = 46316835694926478169428394003475163141307993866256225615783033603165251855960`

Substitute `By` in the curve equation to calculate `Bx = 15112221349535400772501151409588531511454012693041857206046113283949847762202`

### Defining Point Addition&#x20;

Say you have two points P and Q on the curve. If we draw a straight line through P and Q, it will intersect the curve at another point R as shown in the diagram. When this happens, we say that R is the sum of points P and Q. This is how pt addition is defined. Now let’s calculate R given P and Q.

![Point Addition on the Curve](<../.gitbook/assets/Curve pt addn.png>)

Given that `P + Q = R` . R is calculated as follows:-

$$
(x_1, y_1) + (x_2, y_2) = \left( \frac{x_1y_2 + x_2y_1}{1 + dx_1x_2y_1y_2}, \frac{y_1y_2 - ax_1x_2}{1-dx_1x_2y_1y_2} \right)
$$

This formula is derived by finding a point R that lies both on the curve and on the straight line through P and Q. To see detailed derivation, see [https://martin.kleppmann.com/papers/curve25519.pdf](https://martin.kleppmann.com/papers/curve25519.pdf) (although it's for a different curve, but the concept is the same)

The above equation can be re-arranged to a polynomial form (R1CS). Points that satisfy this polynomial will always satisfy the property `P + Q = R`

### Defining Multiplication of a pt on the Curve

Given a point P on the curve, we define another point on the curve Q as the “scalar multiplication” of P and k such that `Q =k*P` where k is a constant under mod p. Scalar multiplication is defined as the repeated addition of pts:-&#x20;

$$
k=2, Q =  P+P
\newline
 k=4, Q = 2P + 2P
\newline
k=8, Q = 4P + 4P
\newline
...
\newline
k = 256, Q = 128P + 128P
$$

Hence, we can calculate `Q` in `log(k)` steps. Since `max(k) < 2^255-19`, we need to perform a max of `log(2^255-19) ~ 255` steps to calculate a scalar multiple.

### Verifying an ED25519 signature

Let’s see some definitions:-

The ED25519 key-pair consists of:

* Private Key (integer under mod p) : `privKey`
* Public key (curve point): `pubKey = privKey * B` where `B` is the base pt as defined above

#### Verification Algorithm

The ED22519 signature verification algorithm takes as input a text message `msg` + the signer's ED25519 public key `pubKey` + the ED25519 signature `{R, s}` and produces as output a boolean value (`valid` or `invalid` signature). Here `s` is a scalar, and `R` is a pt on the curve. ED25519 verification works as follows (with minor simplifications):

`EdDSA_signature_verify(msg, pubKey, signature { R, s } ) --> valid / invalid`

1. Calculate `h = SHA512(R + pubKey + msg) mod q`
2. Calculate `P1 = s * B`
3. Calculate `P2 = R + h * pubKey`
4. Return `P1 == P2`

Here `q` is the curve order. `q = 2^252 + 27742317777372353535851937790883648493`

In step one, you must be wondering how can we use `pubKey` as an input to SHA512 as `pubKey` is a curve point (not a scalar). `pubkey` in this step is represented as a "compressed" curve pt i.e only the y-coordinate. In step, 3, `pubkey` is used as a curve point.
