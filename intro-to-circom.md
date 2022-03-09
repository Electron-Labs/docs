---
description: 'Circom: Usage/purpose of language syntax and programming practices'
---

# Intro to Circom

This section assumes that you have completed the Circom tutorial for the multiplier circuit given [here](https://docs.circom.io/getting-started/installation/). Please make sure you have completed the section that explains the power of tau ceremony and proof generation and verification.

Let us now discuss how to make proper use of Circom.

#### Dealing with Constraints

Let's start with something simple/trivial. Say we want to prove the polynomial `x^2 + x + 2 = y` for a given `(x,y)`. In circom, we write this as:-

```
signal input x;
signal input y;

x*x + x + 2 === y;

//=== is the constraint operator
//both x and y are provided and we simply apply the polynomial equation
//This is similar to python's assert(x*x + x + 2 = y)
```

Here, we must first pre-calculate `x` and `y` and then provide them as inputs to the circuit. However, Circom allows us to calculate the value of `y` within the circuit itself. This is done using `output` signals.

```
signal input x;
signal output y; //notice y is now an output signal

x*x + x + 2 ==> y;
//notice the use of ==> instead of === here. The ==> operator first calculates
//the value of y and then applies the polynomial constraint

//note that the following statements are the same
x*x + x + 2 ==> y;
y <== x*x + x + 2;
```

Technically, we only need the `===` operator. The `<==` operator is basically Circom's attempt at making the developer's life easier. Also, note that signals are all the same in the underlying cryptography. The separation into `input` or `output`  signals is virtual and just to make things developer-friendly.

Okay, now let us say we want to prove the polynomial `x^3 + x^2 + 1 = y`

```
signal input x;
signal output y;

x*x*x + x*x + 1 ==> y;
```

This will give an error `Non-Quadratic Constraints are not allowed`. The reason for this error is that all constraints must be of the form `a*b + c` i.e. _only one multiplication is allowed_ per constraint. You can have an arbitrary number of additions. No other operators (`-` or `/)` are allowed. The reason for this comes from the underlying cryptography (something called "cryptographic pairings").

To solve this issue, we must reduce our polynomial into the form `a*b+c`. We can do this within the circuit itself by use of `intermediary` signals, as below:-

```
signal input x;
signal temp1;
signal temp2;
signal output y;

temp1 <== x*x;
temp2 <== temp1*x;
temp2 + temp1 + 1 ==> y;

//this way, all the expressions are reduced to quadratic/linear
```

This changes the polynomial we are proving. The set of polynomials we are proving in the above circuit is:-&#x20;

$$
t1 = x*x
\newline
t2 = t1*x
\newline
t1 + t2 + 1 = y
$$

#### How use the modulo operator (%) in Circom?

The underlying cryptography restricts us to just `*` and `+` operators. But we obviously want to use all operators. So how can we prove the equation `y = x%10`?

```
signal input x;
signal output y;

y <== x%10; //this will give an error of Non-Quadratic Constraints
```

Alternatively, we could write this as:-

```
signal input x:
signal input q;
signal input y;

x === q*10 + y;
//here q is the quotient and y is the remainder when x is divided by 10
```

This solves our problem. However, this way, we have to provide everything as an input signal, which means we have to write a ton of python scripts outside the circuit. Circom allows us to get around this issue by use of the `<--` operator.

```
signal input x:
signal q;
signal output y;

y <-- x%10;
q <-- y\10; //where '\' is the integer division operator
x === q*10 + y; //this works!
```

What just happened here? The `<--` is basically Circom's assignment operator. Please note that the `<--` does not set any constraints, hence we are not limited to `*` or `+` when using `<--`. The proper way to use this operator is to perform computations using this operator, and then in the end apply the `===` operator to set the polynomial constraint.

The `<--` operator is basically Circom's attempt at making the developer's life easier.

If you are maths savvy, you might have noted that the constraint `x === q*10 + y` is not mathematically complete. We must also apply the condition that `y<10`. We will now discuss how to set this condition through the use of the "less than" circuit.

```
template LessThan(n) {
    assert(n <= 252); //this is required since the altbn prime is upto 252 bits
    
    signal input in[2]; //in[0] & in[1] are integers of upto n bits  
    signal output out;

    component n2b = Num2Bits(n+1);
    //Num2Bits takes an integer input and returns it's bitfied array
    //the argument n+1 is the max size of the integer input

    n2b.in <== in[0]+ (1<<n) - in[1];

    out <== 1-n2b.out[n];
}
```

You might need pen and paper and need to do a bit of maths to understand how this circuit works. But the basic idea is that you can use the constraints model to describe general computations. One needs to realize that the logic required to describe something in circom is written in a very different way than a typical programming language.

As next steps, it might be worthwhile to check out the `Num2Bits` template mentioned above, given [here](https://github.com/iden3/circomlib/blob/master/circuits/bitify.circom#L25). There is a lot of neat stuff in [circomlib](https://github.com/iden3/circomlib/tree/master/circuits), circuits that help you achieve general computations. We recommend checking out multiplexers and comparators.

You can also check out this [base2^51 multiplier](https://github.com/Electron-Labs/circom-ed25519/blob/master/circuits/binmulfast.circom#L87). It takes two numbers, represented as arrays of base2^51 numbers, and multiplies them together, and returns an array of base2^51 numbers.

Circom does NOT allow you to apply if/else conditionals on signals. Nor are you allowed to use signals as the termination condition for loops. You can use only compilation time constants (variables in circom) for these, such as `n` in the `lessthan` template. This is because otherwise, it would make the constraints dependent on the signals, effectively meaning that the polynomials being proven are dependent on the variables themselves. This is not possible with the current construction of the underlying cryptography. So far in our work, this does not seem to matter.

One more thing, when working with Circom, sometimes it's very useful to be able to see what's inside your witness file. Use the command `snarkjs wej witness.wtns witness.json`. This command runs on your `witness.wtns` file and gives you JSON.

One last thought => in theory, the entire Circom language could be replaced by a signals.json file and another constraints.json file that specifies constraints. If you can make constraints.json compatible with Circom's R1CS file, then one could use snarkJS too with this.
