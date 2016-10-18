The following article is just a simplification. I am not writing this in accordance with any particular hardware architecture. I will only discuss `How floating point no are stored in memory`, but if you will want to find more authoritative sources then go for

1. [What Every Computer Scientist Should Know About Floating-Point Arithmetic](http://docs.oracle.com/cd/E19957-01/806-3568/ncg_goldberg.html)
2. https://en.wikipedia.org/wiki/IEEE_754-1985
3. https://en.wikipedia.org/wiki/Floating_point.

**Floating-point numbers are encoded by storing the significand & the exponent (along with a sign bit)**

- You will unable to understand above line until you read further

## Floating Point Number Memory Layout

A typical 32-bit floating point number store memory layout have fields like the following: [Image]

1. sign
2. exponent
3. significand(AKA mantissa)

```
|-------------32-bit---------------|

+-+--------+-----------------------+
|1| 8-bit  |        23-bit         |
+-+--------+-----------------------+
 ^    ^                ^
 |    |                |
 |    |                +-- significand 
 |    |
 |    +------------------- exponent 
 |
 +------------------------ sign bit
```

#### Sign
- The high-order bit indicates sign. 
- `0` indicates a positive value, `1` indicates negative.

#### Exponent
- The next 8 bits are used for the exponent which can be positive or negative, but instead of reserving another sign bit, they're encoded such that `1000 0000` represents `0`, so `0000 0000` represents `-128` and `1111 1111` represents `127`. 
- How this encoding work ?

	To get binary of exponent, add 128(max value can stored by 7 bit) to exponent

	- So exponent `0` represents, 0 + 128 = 128 = `1000 0000` in binary
	- Same as exponent `127` represents, 127 + 128 = 255 = `1111 1111` in binary
	- So does exponent `-128` represents, -128 + 128 = 0 = `0000 0000` in binary

#### Significand
- The remaining 23-bits are used for the significand(AKA mantissa). Each bit represents a negative power of 2 counting from the left, so:

```
01101 = 0 * 2^-1 + 1 * 2^-2 + 1 * 2^-3 + 0 * 2^-4 + 1 * 2^-5 
      = 0.25 + 0.125 + 0.03125 
      = 0.40625
```

OK ! We are done with basics.

## Let's Understand By Example

- So we consider very famous float value `3.14159`(PI) example. What if we use base 2 instead of base 10, then value of PI will be

`0.7853975 * 2^2`

- `0.7853975` is multiplied by the base 2 raised to the power of 2 to get `3.14159`. 

So our value of `3.14159` would be represented as something like

    0 10000100 11001001000011111100111
    ^     ^               ^
    |     |               |
    |     |               +--- significand = 0.7853975
    |     |
    |     +------------------- exponent = 4
    |
    +------------------------- sign = 0 (positive)

#### Sign
- You know why zero here !

#### Exponent
- In our case, Exponent is `4` which represents, 4 + 128 = 132 = `1000 0100`  in binary
- The number of bits in the exponent determines the range (the minimum and maximum values you can represent). 

#### Significand
- If you add up all the bits in the significand, they don't total `0.7853975`. They actually come out to `0.78539747`. 
- There aren't quite enough bits to store the value exactly. we can only store an approximation. 
- The number of bits in the significand determines the precision.
- 23-bits gives us roughly 6 decimal digits of precision. 64-bit floating point types give roughly 12 to 15 digits of precision. 


#### Strange, But Fact
- There are values that cannot be represented exactly no matter how many bits you use. Just as values like 1/3 cannot be represented in a finite number of decimal digits, values like 1/10 cannot be represented in a finite number of bits. 
- Since values are approximate, calculations with them are also approximate, and rounding errors accumulate. 


## Where decimal point is stored ?

- The decimal point is not explicitly stored anywhere. 
- As i written line `Floating-point numbers are encoded by storing the significand & the exponent (along with a sign bit)`, but you dont get it first time. Dont worry 99% people dont get it first including me.

## A bit more about repesenting numbers in memory

- According to `IEEE 754-1985` world wide standard, you can also store zero, negative/positive infinity and even NaN(Not a Number). Dont worry if you dont know what is NaN, i will give explanation shortly(But be worried, if you dont know infinity).

### Zero representation
- sign = 0 for positive zero, 1 for negative zero.
- biased exponent = 0.
- fraction = 0. 

### Positive & negative infinity representation
- sign = 0,	 for positive infinity, 1 for negative infinity.
- exponent = all 1 bits.
- fraction = all 0 bits. 

### NaN representation
- sign = either 0 or 1.
- exponent = all 1 bits.
- fraction = anything except all 0 bits (since all 0 bits represents infinity)

## Why we need NaN ?

- Some operations of floating-point arithmetic are invalid, such as dividing by zero or taking the square root of a negative number.
- The act of reaching an invalid result is called a floating-point exception(described in next topic). An exceptional result is represented by a special code called a NaN, for "Not a Number".

## Floating point exceptions

- The `IEEE 754-1985` standard defines five exceptions that can occur during a floating point calculation named as 

1. **Invalid Operation** : occurs due to many causes like multiplication of infinte with zero or infinite, division of infinte by zero or infinite & vice-versa, sqaure root of operand less than zero, etc.
2. **Division by Zero** : occures when "as its name sounds"
3. **Overflow** : occurs when result of operation is large enough which unable to preserve precision. Rounding of result will be performed. Note : Whenever the overflow exception is raised, the inexact exception is also raised. 
4. **Underflow** : occurs when result of operation is small enough which unable to preserve precision. Rounding of result will be performed.
5. **Inexact** : raised when rounded result is not exact. 

## Floating Point Rounding

- As we seen floating-point numbers have a limited number of digits, they cannot represent all real numbers accurately: when there are more digits than the format allows, the leftover ones are omitted - the number is rounded. 
- There are 4 rounding modes :
1. **Round to Nearest** : rounded to the nearest value with an even (zero) least significant bit, which occurs 50% of the time.
2. **Round toward 0** – simply truncate the extra digits.
3. **Round toward +∞** – rounding towards positive infinity
4. **Round toward −∞** – rounding towards negative infinity.
