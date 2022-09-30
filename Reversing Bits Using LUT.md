# Reversing Bits Using LUT
## The Algorithm's Idea
Assume we wish to reverse the bits of an integer.
We could perform it in the following manner (which we'll accommodate with an example `i = 12456` for clarity):

1. Split the integer into byte-size sections.

![revese_lut_1](https://user-images.githubusercontent.com/93342363/193327590-6239a373-384f-417b-bc84-d6cd238a3fa7.png)

2. Reverse each byte independently.

![reverse_lut_2](https://user-images.githubusercontent.com/93342363/193327640-15e6dd2a-7dd2-4522-b9ac-1661fdab3072.png)

3. "Stitch" each reverse byte in a reversed order.

![reverse_lut_3](https://user-images.githubusercontent.com/93342363/193327661-8aa3bad4-badd-4d3d-8d7c-3af19588d4e8.png)

#### Reversing Each Byte Using LUT
While the first and third stage are straightforward, the heart of this method is in the second stage where we need to reverse each byte.
Adopting this [[Divide And Conquer]] approach simplifies our problem. What one should notice, when we treat each byte independently, it is simply a number, whose value is between $0$ and $2^8-1=255$.
Now, for each such value there's one corresponding value, also between $0$ and $255$, which is its reverse value.
E.g., for $i=0$, its reverse value is $0$. For $i=1$, its reverse value is $128$ etc.
We can formalize this as a [[Table|Hash Function]] $R(i) = r$, such that for each $0 \leq i \leq 255$, $r$ is $i$'s corresponding reverse value.

Therefore, if we can create a look-up table $BitReverseTable256$ such that $BitReverseTable256[i]$ is $i$'s reverse value. 

The table would look:
```C
BitReverseTable256[0] = 0b00000000;
BitReverseTable256[1] = 0b10000000; 
BitReverseTable256[2] = 0b01000000; 
BitReverseTable256[3] = 0b11000000; 

BitReverseTable256[4] = 0b00100000;
BitReverseTable256[5] = 0b10100000; 
BitReverseTable256[6] = 0b01100000; 
BitReverseTable256[7] = 0b11100000;

BitReverseTable256[8] = 0b00010000;
BitReverseTable256[9] = 0b10010000;
BitReverseTable256[10] = 0b01010000;
BitReverseTable256[11] = 0b11010000;

BitReverseTable256[12] = 0b00110000;
BitReverseTable256[13] = 0b10110000;
BitReverseTable256[14] = 0b01110000;
BitReverseTable256[15] = 0b11110000;

...
```

## Implementation
```C
static const unsigned char BitReverseTable256[256] = 
{
#define R2(n) n,     n + 2*64,     n + 1*64,     n + 3*64
#define R4(n) R2(n), R2(n + 2*16), R2(n + 1*16), R2(n + 3*16)
#define R6(n) R4(n), R4(n + 2*4 ), R4(n + 1*4 ), R4(n + 3*4 )
R6(0), R6(2), R6(1), R6(3)
};

unsigned int ReverseBitsUsingLUT(unsigned int value)
{
	unsigned int result;
	unsigned char * value_byte_ptr = (unsigned char *) &value;
	unsigned char * result_byte_ptr = (unsigned char *) &result;

	result_byte_ptr[3] = BitReverseTable256[value_byte_ptr[0]]; 
	result_byte_ptr[2] = BitReverseTable256[value_byte_ptr[1]]; 
	result_byte_ptr[1] = BitReverseTable256[value_byte_ptr[2]]; 
	result_byte_ptr[0] = BitReverseTable256[value_byte_ptr[3]];
}
```

# Computing The LUT
Before we dive into explaining how to compute the LUT, it should be noted that it is as efficient to write the 256 numbers of the LUT by hand.
With that said, lets dive into the implementation before us, which employs recursive mathematics to compute the table.  

To understand this computation I want to begin with a semi-naive approach of noticing the structure and the underlining patterns in it.

## The LUT's General Structure

#### Are we actually computing 256 values?
First lets convince ourselves that we even get 256 values.
First we have:
```C
R6(0), R6(2), R6(1), R6(3)
```

But each `R6(n)` is, by recursion, itself 4 values:
```C
R6(n) = R4(n), R4(n + 2*4 ), R4(n + 1*4 ), R4(n + 3*4)
```

Then each `R4(n)` is also, by recursion, 4 values:
```C
R4(n) = R2(n), R2(n + 2*16), R2(n + 1*16), R2(n + 3*16)
```

And each `R2(n)` is also, by recursion, 4 values.
```C
R2(n) = n,     n + 2*64,     n + 1*64,     n + 3*64
```

We have reached the base case.
Therefore we have 4 values for each $R2$, 4 $R2$s for each $R4$s, 4 $R4$s for each $R6$ and - lastly - 4 $R6$s. 
So we would get $4*4*4*4=2^2*2^2*2^2*2^2=2^8=256$ values. Great! 

#### How does the table's structure form?
After we are convinced that we get 256 value, lets try to understand how the table is structured.
The relationship between the recursively defined $R6$, $R4$ and $R2$ to $BitReverseTable256$ might seem mysterious.
Starting to unveil the fog surrounding their relation, we can think of the entire recursion as forming a [[tree]]. The order of the the leafs will be the order of the look-up table.

Here a fragment of that tree representation:

![reverse_lut_table_as_a_tree](https://user-images.githubusercontent.com/93342363/193327781-2106bba9-ef32-4ef9-9b00-679af6abb008.png)

The leafs, or the values $r_0, r_1, r_2...$ are the values in $BitReverseTable256[0], BitReverseTable256[1], BitReverseTable256[2]...$ respectively.

We can determine that:
* $R6(0), R6(2), R6(1), R6(3)$ each fabricate $\frac{256}{4}=64$ different values. 
$R6(0)$ fabricates the values $r_0$-$r_{63}$, $R6(1)$ fabricates the values $r_{64}$-$r_{127}$ etc.
* $R4(n)$, $R4(n + 2*4) ... R4(n + 3*4)$ each fabricate $\frac{64}{4}=16$ values.
$R4(0)$ fabricates the values $r_0-r_{15}$, $R4(0+2*4)$ fabricates the values $r_{16}-r_{31}$ etc.
* $R2(n), R2(n +2*16) ... R2(n + 2*16)$ each fabricate $\frac{16}{4} = 4$ values.

## How are the actual values computed?
We have seen that each segment of the tree is in charge of computing the values of a subset of $BitReverseTable256$.
We still haven't explained how the actual computation happens.

If we take a very simple example we can obviously see that this happens. Take $R2(0)$, which should compute (as we have seen in the previous chapter) the values $r_0 - r_3$. And indeed:
$$R2(0) = 
\begin{equation}
\begin{cases}
&0 \\
&0 + 2*64 = 128 \\
&0 + 1*64 = 64 \\
&0 + 3*64 = 192\\
\end{cases}
\end{equation}=
\begin{cases}
&00000000 \\
&10000000 \\
&01000000 \\
&11000000 \\
\end{cases}
$$

#### Covering all of the byte options, two bits at a time
Something curious that one might notice in the example above is that we covered the entire range of possibilities regarding the last 2 bits:
$$00,01,10,11$$
This curious behaviour is linked directly to the way $R2$ is defined:
```C
R2(n) = n,     n + 2*64,     n + 1*64,     n + 3*64
```
The four constants $0*64, 1*64, 2*64, 3*64$ are the four possibilities for setting the last two bits, the seventh and eighth bits.

This pattern continues with $R4$:
```C
R4(n) = R2(n), R2(n + 2*16), R2(n + 1*16), R2(n + 3*16)
```
Ignoring other complications for the moment, we might notice that here again the four constants $0*16, 1*16, 2*16, 3*16$ are the four possibilities for setting the fifth and sixth bits.

This is also true for the constants $R6$ which cover the possibilities of the third and forth bits.
Lastly, the first and second bits are determine by the statement $R6(0), R6(2), R6(1), R6(3)$ - the constants $0,1,2,3$ are nothing more than the four values that could be expressed with the first two bits.
To summarise, the value of each two bits in the byte are determined by a line in the recursion:

![reverse_lut_relation_between_bits_and_recursion_levels](https://user-images.githubusercontent.com/93342363/193327817-94634ca5-25f4-4f70-9b93-f27ce6b4bbfa.png)

#### How numbers are constructed gradually in each step of the recursion
Now we can understand what is $n$. We basically construct a number 2 bits at a time.
Each value out of the 256 values that are expressed by a byte is represented by a sum of powers of two.
For example, $78=01001110=2^1+2^2+2^3+2^6$.
Since we have determined that in the first step the first two bits are determined. This is essentially like adding to $n$ either $2^0$ or $2^1$, or both of them or none of them. After the second step we add to either $n$ $2^2$ or $2^3$, or both of them or none of them, and so on.

The way each level of the recursion determines each bit is perhaps best represented again by employing a tree-like representation:

![reverse_lut_determining_bits_in_four_levels](https://user-images.githubusercontent.com/93342363/193327838-5e773ff5-6674-4013-8f04-1c26f5b73562.png)

### How does the reversal occurs?
We have seen how the values of the look-up table are determined recursively, and we have seen the role each level of the recursion has in this. However, we still haven't seen why $r_n$ would actually be the reversal of $n$. 

To understand that last piece of the puzzle, I'll try to give an overview of an algorithm that reverses a byte, and then I'll show that the recursive formals are doing the same mathematically.

Intuitively, we can give the following algorithm for reversing a byte:
* Reverse the order each two bits
* Flip each two adjacent bits

![reverse_lut_reversing_a_byte](https://user-images.githubusercontent.com/93342363/193327864-f95accb3-3ba7-45de-968f-bbe8da320da8.png)

#### Reversing the order
We can already note a few striking similarities between the way the the recursive definition is performed and this simple algorithm.
For start, we can note the in effect the way our recursion restricts two bits at a time from the lowest ones to the higher ones follows the same logic as the reversal of the order of bit pairs in our algorithm.

If we take a deeper look, we would notice that since $R6(0)$ restricts the two least significant bits in $r_n \in R6(0)$ to be $00$, this means that the two most significant bits of $n \in R6(0)$ are $00$. This means that the largest $n \in R6(0)$ is $00111111=63$, which is evidently the same range we have determined previously!
What about the next segment, rooted at $R6(2)$? Similarly since $R6(2)$ restricts the two least significant bits in $r_n \in R6(2)$ to be $10$, this means that the two most significant bits of $n \in R6(2)$ are $01$. Therefore the largest $n \in R6(2)$ is $01111111=127$, again affirming the range we conveyed earlier.  

Therefore, the sequence of the recursion $R6 \implies R4 \implies R2 \implies r$ is identical to reversing the order of each two bits - $R6$ corresponds to bits 7 and 8 of $n$, $R4$ corresponds to bits 5 and 6 of $n$ etc.

#### Flipping adjacent bits
We now can understand what the curious sequence that appears in all of our recursion formals - $0, 2, 1, 3$
Naively we would think that the sequence should be, well, $0,1,2,3$...

If we are dividing a byte into 4 pairs of 2 bits, then the value of each pair can be represented by this sequence $0, 1, 2, 3$ or $00, 01, 10, 11$.
However, we are dealing with flipped adjacent bits, therefore $00, 10, 01, 11$, or... yes - $0, 2, 1, 3$. 
This is the source of this curious sequence. It is essential for the alignment of the value of $r_n$ with the value of $n$.

## From $n$ to $r_n$ and from $r_n$ to $n$
To cement and reinforce what this exactly means, consider as an example $n=146=10010010$.
$R6(1)$ corresponds to the most significant bits of $n$ being $10$ which flipped is $01=1$.
$R4(2)$ corresponds to the fifth and sixth bits of $n$ being $01$ which flipped is $10 = 2$.
$R2(0)$ corresponds to the third and forth bits of $n$ being 00.
Lastly, at the base of the recursion the least significant bits of $n$ are flipped to $01$.

The node at the end of this path in the tree is $r_n=01001001$.

![reverse_lut_from_n_to_r_n](https://user-images.githubusercontent.com/93342363/193327908-f9da8cae-fcc0-41bb-a26e-8b1735998d29.png)

Employing this understanding can also help us understand why the reverse of any $n$, the value $r_n$, will be in the nth place in the LUT.
What we should realize is that each step of restricting the bits is at the same time restricting the range of possible $n$s:

![reverse_lut_restricting_n_ranges](https://user-images.githubusercontent.com/93342363/193327932-4eb4900c-813a-459a-93ae-c73b5edf3f68.png)


---
```ad-note
title: Reference
* The code has been adapted from this [source](http://graphics.stanford.edu/~seander/bithacks.html#BitReverseTable)
* [An explanation for the confusing LUT](https://stackoverflow.com/questions/15107398/algorithm-behind-the-generation-of-the-reverse-bits-lookup-table8-bit)
* For an alternative way, which also has an $O(1)$ time-complexity & a better space complexity, see [[Reversing An Integer's Bits]].
```
