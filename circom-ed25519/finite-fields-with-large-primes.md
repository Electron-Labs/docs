# Finite fields with large primes

#### Modulus operator

Arithmetic operations on a finite field `F_p` for a prime `p` are defined as the operator over natural numbers, modulo `p`

For eg. addition over the finite field is defined as follows

`a ⊕ b = (a + b) % p`

To define all relevant arithmetic operations over the fintie field, we need to implement the modulus operator inside circom. We perform modulo in such a way that long division is avoided. The algorithm is as follows -

#### Implementation

Let the prime `p` for the curve be defined as p = 2^255 - 19. Given input `x`, we need to calculate `x % p`

We define `c = 2^255`.

Now,\
`r = x % 2^255`

> Note that `r` <  2^255. Also, `r` is simply the least significant 255 bits of `x`

`q = x // 2^255`

> `q` is simply `x` sans the least significant 255 bits of `x`

Hence we derive equation (1)

```python
x = q*c + r
x = q*(p+19) + r   -- (1)
```

Simplifying (1) and taking modulus `p` on both sides

```python
x % p = 19*q % p + r % p
```

Now, calculate `r % p` as:

```python
if (r>=p): r % p = r - p
if (r<p): r % p = r
```

Apply this algorithm recursively to calculate `19q % p`. In the circuit, we apply the above algorithm recursively, and hence `x` can be of arbitrary size.

> 📘 Note
>
> Calculating `r % p`, there is not way to define conditional logic inside a circuit. However, a multiplexer serves the same purpose when one out of multiple values needs to be selected. In this particular case, we provide both `r` and `r-p` as the input signals to the multiplexer, and assign the selector to be `0 | 1` based on whether `r - p` underflowed or not.



> #### &#x20;









de



d

ewq

d

$$
f(x) = x * e^{2 pi i \xi x}
$$

