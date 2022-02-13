# ZK for Beginners

Let's start from absolute scratch. Consider the following polynomials:-

$$
ax^3 + bx^2 + cx + d = y
\newline
ex^2 = fy^2 + gy^3 + h
$$

Here, `a`, `b`, `c`, `d`, `e`, `f`, `g`, `h` are constants. Say we are provided with a pair `(x,y)` and we want to verify whether `(x,y)` satisfies these equations. To verify, we can just substitute the provided value `(x,y)` in the equations and check that LHS = RHS for both equations.

Now, imagine that we have millions of equations and millions of variables to verify. How would we do that? We could just get some good hardware on the cloud and verify.

However, now imagine we want a smart contract to verify these millions of equations & variables. Since the resources on a blockchain are limited, you might want to avoid performing these million checks on-chain. This is where zero-knowledge proofs come in.

ZK allows us to generate a "proof" that proves that a set of variables satisfy a set of equations. The awesome thing about this proof is that it is small and fixed in size (64 bytes), independent of the number of equations and variables it is proving. Anyone could take this proof and "verify" it. The computational resources to verify this proof are cheap and constant (\~300K gas on Ethereum)

So, instead of performing millions of computations in a smart contract, one could just create a zk-proof for them, and provide it to the smart contract. The smart contract shall implement a proof verification algorithm, and if the proof checks, the smart contract can use the variables for further computation. It's like computational compression.

Another amazing property of ZK is that you can provide the variables in encrypted form. This means I can prove that a certain set of variables satisfy the given equations, without revealing the value of the variables.

Finally, I want to mention that it is possible to represent all computations in the form of polynomials. Hence, ZK can be used to prove general computations.

As next steps, we recommend reading this introduction by circom - [https://docs.circom.io/background/background/](https://docs.circom.io/background/background/)
