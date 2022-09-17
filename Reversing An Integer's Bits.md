
16-09-2022 18:03
Tags: [[Binary]], [[C (Programming Language)]], #infinityrd, #software-development 

---

# Reversing An Integer's Bits
Our algorithm intends to mirror, or reverse, the bits in an integer:
$$01101101 \implies 10110110$$
## The Idea Behind The Algorithm
#### An Initial Presentation
1. **Reversing 2 Bits -** Lets start with a simple example. Consider the following: $$01$$If we were to swap the position of each bit, we would get the bits in reverse order:
$$10$$
2. **Reversing a [[Nibble]]-** Now, lets look at a less trivial example:
$$1011$$
If we first swap each couple, and then swap adjacent bits we would get the bits in reverse order:
*  Swap couples:
$$1110$$
* Swap every two adjacent bits:
$$1101$$
3. **Reversing a Byte-** This is also true for a byte, $10101011$, for example:
*  We swap foursomes:
$$10111010$$
* Then we swap couples:
$$11101010$$
* Lastly we swap every two adjacent bits:
$$11010101$$
![[Mirror.png]]

#### Generalising Our Observation Into An Algorithm 
Turns out this is not works, but there's a pattern that allows us to extend this for $2^n$ bits. 
You might have noticed a pattern with how many steps are needed to reverse in our examples. For 2 bit it's 1 step, for 4 it's 2 steps and for 8 it's 3 steps. That is, if we have $2^n$ bits we need $n$ steps to reverse it.
If we try to generalise, we would say that to reverse $2^n$ bits, we would first swap adjacent groups of $\frac{2^n}{2}= 2^{n-1}$ bits, then swap adjacent groups of $2^{n-2}$ bits etc, until we reach groups of $2^0=1$ bits, and swap them. Then we have reversed our number.

## Implementing The Algorithm For An Integer via Bitwise Operations.
```C
#define CHOOSE_ADJACENT_SINGLES 0xAAAAAAAA    /* 1010101010101010 */
#define CHOOSE_ADJACENT_COUPLES 0xCCCCCCCC    /* 1100110011001100 */
#define CHOOSE_ADJACENT_FOURSOMES 0xF0F0F0F0  /* 1111000011110000 */
#define CHOOSE_ADJACENT_EIGHTSOMES 0xFF00FF00 /* 1111111100000000 */

unsigned int ReverseBitsInInteger(unsigned int number)
{
	number = (((number & CHOOSE_ADJACENT_SINGLES) >> 1) | ((number & ~(CHOOSE_ADJACENT_SINGLES)) << 1));
	number = (((number & CHOOSE_ADJACENT_COUPLES) >> 2) | ((number & ~(CHOOSE_ADJACENT_COUPLES)) << 2));
	number = (((number & CHOOSE_ADJACENT_FOURSOMES) >> 4) | ((number & ~(CHOOSE_ADJACENT_FOURSOMES)) << 4));
	number = (((number & CHOOSE_ADJACENT_EIGHTSOMES) >> 8) | ((number & ~(CHOOSE_ADJACENT_EIGHTSOMES)) << 8));
	return ((number >> 16) | (number << 16));
}
```

#### Understanding The First Line
What is going on here?
Well, lets consider the first line of code:
```C
number = (((number & CHOOSE_ADJACENT_SINGLES) >> 1) | ((number & ~(CHOOSE_ADJACENT_SINGLES)) << 1));
```
We have two symmetrical operations:
```C
((number & CHOOSE_ADJACENT_SINGLES) >> 1)
```
```C
((number & ~(CHOOSE_ADJACENT_SINGLES)) << 1)
```

If we consider the first we can see that `number & CHOOSE_ADJACENT_SINGLES` basically wipes out all of the bits except those in even places.
This is because if a bit is in an uneven place it'll be wiped out and replaced with 0, and if it is in an even place it's value will be saved - if it is a 0 then `0&1 = 0`, and if it is a 1 then `1&1 = 1`.
Afterwards `>> 1` moves those bits to uneven places.
![[number & adjacent 1.png]]
The expression `number & ~(CHOOSE_ADJACENT_SINGLES)` does the same, only it selects the bits in the uneven places and moves them to even places with `<<1`.

Thus, the whole first line swaps bits and even places and bits with uneven places.

#### Understanding The Next Lines
Now that we've seen what the first line does, it is easy to see what the next ones will do also.
```C
number = (((number & CHOOSE_ADJACENT_COUPLES) >> 2) | ((number & ~(CHOOSE_ADJACENT_COUPLES)) << 2));
```
will swap adjacent couples. The `number & CHOOSE_ADJACET_COUPLES` logic will stay the same (except now we don't select bits in even and uneven places, but couples in "even" and "uneven" places), and we need to shift twice and not once, as we are now dealing with 2 bits at a time.
In the same vain in the lines:
```C
number = (((number & CHOOSE_ADJACENT_FOURSOMES) >> 4) | ((number & ~(CHOOSE_ADJACENT_FOURSOMES)) << 4));
number = (((number & CHOOSE_ADJACENT_EIGHTSOMES) >> 8) | ((number & ~(CHOOSE_ADJACENT_EIGHTSOMES)) << 8));
```
we replace nibbles and byte, respectively.
At the end we need to swap 16 bits, which is what the last line does:
```C
return ((number >> 16) | (number << 16));
```

```ad-note
title: Why don't we need to choose bits in the last line?
We don't need to select the left 2 bytes and the right 2 bytes like we did with smaller number of bits, since we are dealing with an unsigned integer:  `number << 16` will move the values of the 2 right bytes to the 2 left bytes, and `number >> 16` will move the values of the 2 left bytes to the 2 right bytes position. The new bits that the shift will produced by the shifting will be zeros (note that if we where dealing with signed integer it is possible the bits produced by right shift would be `1`s and not `0`s (see [[Bit Shift Operations]] under **Forcing Zeros When Right Shifting In C**).
```


That's it, we reverse the bits!

---
```ad-note
title: Reference
* [5 Ways To Reverse Bits Of An Integer](https://aticleworld.com/5-way-to-reverse-bits-of-an-integer/) 
```
