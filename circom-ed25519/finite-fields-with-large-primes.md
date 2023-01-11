# Finite fields with large primes

#### Modulus operator

Arithmetic operations on a finite field `F_p` for a prime `p` are defined as the operator over natural numbers, modulo `p`

For eg. addition over the finite field is defined as follows

`a ⊕ b = (a + b) % p`

To define all relevant arithmetic operations over the fintie field, we need to implement the modulus operator inside circom. We perform modulo in such a way that long division is avoided. The algorithm is as follows -

#### Implementation

Let the prime `p` for the curve be defined as p = 2^255^ - 19. Given input `x`, we need to calculate `x % p`

We define `c = 2^255`.

Now,\
`r = x % 2^255`

> Note that `r` <  2^255. Also, `r` is simply the least significant 255 bits of `x`

`q = x // 2^255`

> #### &#x20;

$$
f(x) = x * e^{2 pi i \xi x}
$$

