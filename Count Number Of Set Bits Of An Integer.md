
17-09-2022 12:44
Tags: #software-development, #infinityrd, [[Binary]], [[Bitwise Operations]]

---

# Count Number Of Set Bits Of An Integer
A set bit is a bit that's equal to 1.
In [[Two Complement]], assuming a data-type of an integer, $-1=11111111111111111111111111111111$. Therefore an integer $-1$ has 32 set bits.
$2=0010$ has 1 set bits, $3=0011$ has 2 set bits etc.

Our aim is to create an algorithm that'll be able to count the number of set bytes in an integer.

## The Idea Behind The Algorithm
The Idea is quite simple.
```PSEUDOCODE
1. Count the number of set bits in 2 adjacent single bits
2. Count the number of set bits in 2 adjacent couple bits
3. Count the number of set bits in 2 adjacent nibbles
4. Count the number of set bits in 2 adjacent bytes
5. Count the number of set bits in 2 adjacent 2 Bytes
```

How would we do that, though?
#### An Example: Counting Set Bits In A Byte
Assume we wish to count how many set bit we have in two bits. We could have $0$, $1$ or $2$ set bits. To represent those options in binary we would need 2 bits: $00$, $01$ or $10$. 
It turns out we could represent in 2 bits the number of set bits that they hold.

Lets consider the following example:
$$number = 00110101$$
If we wish to have in each 2 bits the number of set bits in $number$ we could do the following:
![counting_num_of_set_bits_singles](https://user-images.githubusercontent.com/93342363/190861123-4c3dfad2-0dae-471d-85ff-1e95f2a7afec.png)

Now each 2 bits hold the number of set bits in the original bits.
We can now continue to do so for a [[Nibble]]:
![counting_num_of_set_bits_couples](https://user-images.githubusercontent.com/93342363/190861131-ff0896ad-9003-42ac-a1e4-20187598f540.png)

Lastly we would add-up the nibbles to get the total count of set bits:
![counting_num_of_set_bits_nibbles](https://user-images.githubusercontent.com/93342363/190861139-82593f85-ec28-4383-bbc8-2f358874a766.png)

#### Generalising
We can see from our example that could easily up-scale the operation for an integer.
Much like the [[Reversing An Integer's Bits]] algorithm, the number of step needed is tied to the number of bits: 3 operations for a byte with $8=2^{3}$ bits, and 5 operations for an integer with $32=2^{5}$ bits.

## An Simple Implementation
```C
#define SELECT_SINGLE_BIT 0x55555555 /* 01010101010101010101010101010101 */
#define SELECT_TWO_BITS 0x33333333   /* 00110011001100110011001100110011 */
#define SELECT_NIBBLE 0x0f0f0f0f     /* 00001111000011110000111100001111 */
#define SELECT_BYTE 0x00ff00ff       /* 00000000111111110000000011111111 */
#define SELECT_TWO_BYTES 0x0000ffff  /* 00000000000000001111111111111111 */

unsigned int CountSetBits(unsigned int n)
{
	n = (n & SELECT_SINGLE_BIT) + ((n >> 1) & SELECT_SINGLE_BIT);
	n = (n & SELECT_TWO_BITS) + ((n >> 2) & SELECT_TWO_BITS);
	n = (n & SELECT_NIBBLE) + ((n >> 4) & SELECT_NIBBLE);
	n = (n & SELECT_BYTE) + ((n >> 8) & SELECT_BYTE);
	return ((n & SELECT_TWO_BYTES) + ((n >> 16) & SELECT_TWO_BYTES));
}
```

We can see that each line represents a step out of the five that we've outlined when we generalised our observations above.
After the first line each 2 bits hold the number of set bits in x in those bits, after the second line each nibble holds the number of set bits in x in those bits etc.
Lets look at the first line as an example:
```C
n = (n & SELECT_SINGLE_BIT) + ((n >> 1) & SELECT_SINGLE_BIT);
```

We can break this line of code into two parts
1. The first operation selects only the odd bits: 
```C
(n & SELECT_SINGLE_BIT)
``` 
2. The second operation selects only the even bits:
```C
((n >> 1) & SELECT_SINGLE_BIT)
``` 

We then add the those two parts together, thus summing the number of bits in every 2 bit as we have described.
Here's an illustration of what happens in the first line:
![line_1_count_set_bits](https://user-images.githubusercontent.com/93342363/190861152-2744a8fd-a102-4d4f-a35f-830498b61432.png)

## A More Complex, More Efficient Implementation

```C
#define SELECT_SINGLE_BIT 0x55555555              /* 01010101010101010101010101010101 */
#define SELECT_TWO_BITS 0x33333333                /* 00110011001100110011001100110011 */
#define SELECT_NIBBLE 0x0f0f0f0f                  /* 00001111000011110000111100001111 */
#define SELECT_FIRST_BIT_IN_EVERY_BYTE 0x01010101 /* 00000001000000010000000100000001 */

unsigned int CountSetBits(unsigned int n)
{
	n = n - ((n >> 1) & SELECT_SINGLE_BIT);
	n = (n & SELECT_TWO_BITS) + ((n >> 2) & SELECT_TWO_BITS);
	n = ((n + (n >> 4)) & SELECT_NIBBLE)
	return (n * SELECT_FIRST_BIT_IN_EVERY_BYTE) >> 24;
}
```

What is going on here?
Well, If we consider the first three lines they should look quite familiar to us (an exception being the first line which we shall discuss shortly).
Up to a nibble, our strategy is the same as we have seen previously.
The main difference in terms of computing is the last line.
Lets discuss the differences.

#### The First Line
Let us compare the first line of this implementation the and last one:
```C
n = (n & SELECT_SINGLE_BIT) + ((n >> 1) & SELECT_SINGLE_BIT); /* Previous implementation */
n = n - ((n >> 1) & SELECT_SINGLE_BIT); /* Current implementation */
```

We can start with the similarities:
In both implementations we select the even bits by using the expression `((n >> 1) & SELECT_SINGLE_BIT`.
What about `n & SELECT_SINGLE_BIT`?
It the current, more efficient, implementation we don't do masking to the odd bits. This way we reserve one operation.
This is done by taking advantage of the form of every two bits of n. These could take any one of the following four forms: `11`, `10`, `01` or `00`.
This is of course quite trivial. 
What we should realise, though, is that we have accounted for the even bits in `(n >> 1) & SELECT_SINGLE_BIT`.
Thus, given one of the four forms that two bits could hold in n, we know what the corresponding two bits in `(n >> 1) & SELECT_SINGLE_BIT` would look like.
If 2 bits in `n` are:
$$b_1b_2$$
Then the corresponding two bits in ``(n >> 1) & SELECT_SINGLE_BIT`` would be:
$$0b_1$$
Thus:
2 bits in `n`|2 bits in `(n >> 1) & SELECT_SINGLE_BIT`|Their difference
---|---|---
00|00|00
01|00|01
10|01|01
11|01|10

Amazingly, the number of set bits in every 2 bits in `n` is precisely `n - ((n >> 1) & SELECT_SINGLE_BIT)`. This is why the first line works.

#### The Last Line
After the first 3 line, which are as we have mentioned basically operate the same as in the previous implementation, we hold in the beginning of every byte the number of set bits in that byte in `n`.

Now we come to the last line:
```C
return (n * SELECT_FIRST_BIT_IN_EVERY_BYTE) >> 24;
```

Let's break it down into two parts.

##### (n * SELECT_FIRST_BIT_IN_EVERY_BYTE)
To understand that expression, it must be noticed that since
```C
SELECT_FIRST_BIT_IN_EVERY_BYTE = (1 << 24) + (1 << 16) + (1 << 8) + 1
```
Then
```C
n * SELECT_FIRST_BIT_IN_EVERY_BYTE = (n << 24) + (n << 16) + (n << 8) + n
```

Therefore, `(n * SELECT_FIRST_BIT_IN_EVERY_BYTE)` essentially sums all of the bytes in the last byte:
![n SELECT_FIRST_BIT](https://user-images.githubusercontent.com/93342363/190861169-94da2d3d-92bd-4523-936a-be3b76dfef73.png)

```ad-note
We are not concerned about overflowing as we know that each byte could have up to 8 set bits, or $1000$.
```
Now our last byte holds the number of all set bits in `n`.
Lastly, we shift need to shift that result to the first byte, which is why we have `>> 24`.

We are done.

---
```ad-note
title: Reference
* [SWAR Algorithm](https://stackoverflow.com/questions/22081738/how-does-this-algorithm-to-count-the-number-of-set-bits-in-a-32-bit-integer-work)
```
