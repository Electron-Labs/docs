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

Base Point is defined as `(Bx, By)` where `By = 4/5`

### Defining Point Addition&#x20;

Say you have two points P and Q on the curve. If we draw a straight line through P and Q, it will intersect the curve at another point R as shown in the diagram. When this happens, we say that R is the sum of points P and Q. This is how pt addition is defined. Now let’s calculate R given P and Q.

![Point Addition on the Curve](<../.gitbook/assets/Curve pt addn.png>)

Given that `P + Q = R` . R is calculated as follows:-

$$
(x_1, y_1) + (x_2, y_2) = \left( \frac{x_1y_2 + x_2y_1}{1 + dx_1x_2y_1y_2}, \frac{y_1y_2 - ax_1x_2}{1-dx_1x_2y_1y_2} \right)
$$

This formula is derived by finding a point R that lies both on the curve and on the straight line through P and Q. To see detailed derivation, see [https://martin.kleppmann.com/papers/curve25519.pdf](https://martin.kleppmann.com/papers/curve25519.pdf) (although it's for a different curve, but the concept is the same)

The above equation can be re-arranged to a polynomial form (R1CS). Points that satisfy this polynomial will always satisfy the property `P + Q = R`

