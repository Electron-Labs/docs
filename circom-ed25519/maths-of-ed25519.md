# Maths of ED25519

ED25519 signature uses a curve and involves performing operations on this curve. Let's dive deeper.

All operations in this signature scheme are defined under modulo `p` where&#x20;

$$
p = 2^{255} - 19
$$

The curve equation is defined as follows :-

$$
ax^2 + y^2 = 1 + dx^2y^2
$$
