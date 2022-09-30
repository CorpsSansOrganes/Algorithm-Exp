
30-09-2022 21:08
Tags: [[Bitwise Operations]], [[Look-Up Tables]], [[Binary]], #infinityrd, #software-development 

---

# Counting Bits Using LUT
## The Algorithm's Idea
The idea is quite simple.
Assume we are counting the bits of an integer, `i=12456` for example.

Our algorithm would prescribe:
1. Split the integer into 4 bytes. 
2. Count the number of bits in each byte independently.
3. Sum to get the result.

The thing that is unique about this particular implementation is how the second stage is done.
Treated independently, each byte has some value $n$ from $0$ (or $00000000$) to $255$ ($11111111$). To perform the second stage we would use a LUT.
In this look up table, $LUT[n]$ will have the number of set bit of $n$.

That way, assuming we have the LUT of size 256, we could perform the second stage - where the "heavy lifting" usually occurs, in [[Computational Complexity|time complexity]] $O(1)$. 

## Implementation
Written in [[C (Programming Language)]]:
```C
static const unsigned char BitsSetTable256[256] = 
{
	#define B2(n) n,     n+1,     n+1,     n+2
	#define B4(n) B2(n), B2(n+1), B2(n+1), B2(n+2)
	#define B6(n) B4(n), B4(n+1), B4(n+1), B4(n+2)
	B6(0), B6(1), B6(1), B6(2)
};

unsigned int CountBitsUsingLUT(unsigned int value)
{
	unsigned int result;
	unsigned char * value_byte_ptr = (unsigned char *) &value;

	result = BitsSetTable256[value_byte_ptr[0]];
	result += BitsSetTable256[value_byte_ptr[1]];
	result += BitsSetTable256[value_byte_ptr[2]];
	result += BitsSetTable256[value_byte_ptr[3]];

	return result;
}
```

# Computing The LUT
While the algorithm and its implementation are straightforward, the way the look-up table is computed is not.

Let's first take a look at the LUT's general structure, and afterwards we'll try to understand why $BitSetTable256[n]$ is equal the number of set bits in $n$.

## The LUT's General Structure
#### Are we even computing 256 values?
Let's convince ourselves that 256 values are computed.
First we have:
```C
B6(0), B6(1), B6(1), B6(2)
```

Each one of these four elements `B6`, by recursion, is itself 4 values:
```C
B6(n) = B4(n), B4(n+1), B4(n+1), B4(n+2)
```

Then each one of those elements `B4` is, again by recursion, is itself 4 values:
```C
B4(n) = B2(n), B2(n+1), B2(n+1), B2(n+2)
```

And lastly each one of those elements `B2` is 4 values:
```C
B2(n) = n,     n+1,     n+1,     n+2
```

With that we reached the base case.
We therefore get $4*4*4=2^8=256$ values. Good news!

#### How does the table's structure form?
We are convinced that we get 256 values. Now let's look at how the table is structured.
We can think of the entire recursion as forming a [[tree]]. The order of the leafs will be the order of the look-up table.

Here's a fragment of this tree: 

![counting_lut_tree](https://user-images.githubusercontent.com/93342363/193350090-23c0034e-3b07-48b1-acc9-efd2791806d1.png)


where $v_0, v_1, v_2...$ are the values in $BitSetTable256[0], BitSetTable256[1], BitSetTable256[2]$ etc.

It should be noted that $B6(0), B6(1), B6(1), B6(2)$ each has $\frac{256}{4}=64$ different values. $B6(0)$ has the values $v_0$ up to $v_{63}$, the first $B6(1)$ the values $v_{64}$ up to $v_{127}$ and so on.
Similarly, $B4$ subdivides farther, each having under it $\frac{64}{4}=16$ values, and $B2$ has $\frac{16}{4}=4$ values.

## How are the actual values computed?
#### Examining a base case to see the pattern
The understand how the values are computed, let us consider the following base case:
What are the possible values of set bits in 2 bits? Since in 2 bits the biggest number that could be represented is 3, this is the same as asking what are the values $r_0$ to $r_3$, which are covered as we have seen by $B2(0)$:
$$B2(0) =\begin{equation}
\begin{cases}
&00 \\
&01 \\
&10 \\
&11 \\
\end{cases}
\end{equation} \implies
\begin{cases}
&0 =00\\
&1 =01\\
&1 =01\\
&2 =10\\
\end{cases}$$

Curiously, we see here the same pattern that we see through-out the recursive formulas - $0, 1, 1, 2$.
We now can understand that this pattern represents the number of set bits in a pair of bits.
Now, since one of the four lines in our recursion has the same pattern we can guess that each stage of the recursion corresponds to two bits of the byte.

![counting_lut_pair_of_bits_and_stage_of_recursion](https://user-images.githubusercontent.com/93342363/193350144-5f4371df-e6de-49ff-9f01-baf09146e5a2.png)

#### The algorithm for counting set bits of a byte
The code snippet that we've seen above, which computes the values for the LUT, therefore follows the following simple algorithm:
1. Count the number of set bits in each pair of bits
2. Add the results

## From $n$ to $v_n$
To cement our understanding, lets consider the an example and follow it throughout the LUT - $n=135=10000111$.
$B6$ corresponds to the most significant bits of $n$, which are $10$.
Lets refer our conversion table:
$$\begin{cases}
&00 \implies 0 \\
&01 \implies 1 \\
&10 \implies 1 \\
&11 \implies 2 \\
\end{cases}$$
We can determine that the second $B6(1)$ corresponds to our bits.
The next two bits, the sixth and fifth, correspond to $B4(0)$.
The forth and third bits which are $01$ correspond to the first $B2(1)$.
Lastly we have the second and first bits, $11$, corresponding to $n+2$.

This path across the tree could be represented like so:

![counting_lut_from_n_to_v_n](https://user-images.githubusercontent.com/93342363/193350175-08def042-8025-45c4-b25b-5f0632ee5e9e.png)


We can see that we get the correct number of set bits.

#### Why is $v_n$ in the right place?
We can employ our understanding of the way in which each stage of the recursion restricts the values of $n$ to convince our selves that $v_n$ will be in the nth place of $BitSetTable256$.
Earlier we made it a point to distinguish between the first $Bn(1)$ and the second $Bn(1)$. That is because each correspond to a different range of $n$s. 
Looking the same example as before $n=135$:

![counting_lut_from_v_n_to_n](https://user-images.githubusercontent.com/93342363/193350239-58a71e62-66c0-4e6c-a13c-c03a0b0057f2.png)

---
```ad-note
title: Reference
* The implementation is based on this [source](http://graphics.stanford.edu/~seander/bithacks.html#CountBitsSetTable)
* An alternative which is $O(1)$ time-complexity AND space-complexity see [[Count Number Of Set Bits Of An Integer]].
```
