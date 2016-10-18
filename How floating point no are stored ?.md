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
- How this encoding work ? go to [exponent bias](https://en.wikipedia.org/wiki/Exponent_bias) or learn it in next topic practically.

#### Significand
- The remaining 23-bits are used for the significand(AKA mantissa). Each bit represents a negative power of 2 counting from the left, so:

```
01101 = 0 * 2^-1 + 1 * 2^-2 + 1 * 2^-3 + 0 * 2^-4 + 1 * 2^-5 
      = 0.25 + 0.125 + 0.03125 
      = 0.40625
```

OK ! We are done with basics.

## Let's Understand Practically

- So, we consider very famous float value `3.14`(PI) example.


#### Sign

- Zero here, as PI is positive!

#### Exponent

- 3 is easy: `0011` in binary

`11`

The rest, 0.14

0.14 x 2 = 0.28, 		0

0.28 x 2 = 0.56, 		0

0.56 x 2 = 1.12, 		1

0.12 x 2 = 0.24, 		0

0.24 x 2 = 0.48, 		0

0.48 x 2 = 0.96, 		0

0.96 x 2 = 1.92, 		1

0.92 x 2 = 1.84, 		1

0.84 x 2 = 1.68, 		1

And so on . . . 

`0.14 = 001000111...`

If you dont know how to convert decimal no in binary then refer this [float to binary](http://stackoverflow.com/questions/3954498/how-to-convert-float-number-to-binary).

- Now add 3,

`11.001000111...	   with exp 0    (3.14 * 2^0)`

- Now shift it (normalize it) and adjust the exponent accordingly 

`1.1001000111...    with exp +1   	(1.57 * 2^1)`

- Now you only have to add the bias of 127 to the exponent 1 and store it(i.e. 128 = 1000 0000)

`0 1000 0000 1100 1000 111...`

- Forget the top 1 of the mantissa (which is always supposed to be 1, except for some special values, so it is not stored), and you get:

`0 10000000 1001 0001 111...`

- So our value of `3.14` would be represented as something like

```
    0 10000000 10010001111010111000011
    ^     ^               ^
    |     |               |
    |     |               +--- significand = 0.7853975
    |     |
    |     +------------------- exponent = 1
    |
    +------------------------- sign = 0 (positive)
```

- The number of bits in the exponent determines the range (the minimum and maximum values you can represent). 

#### Significand
- If you add up all the bits in the significand, they don't total `0.7853975`(which should be, according to 7 digit precision). They actually come out to `0.78539747`. 
- There aren't quite enough bits to store the value exactly. we can only store an approximation. 
- The number of bits in the significand determines the precision.
- 23-bits gives us roughly 6 decimal digits of precision. 64-bit floating point types give roughly 12 to 15 digits of precision. 


#### Strange, But Fact
- There are values that cannot be represented exactly no matter how many bits you use. Just as values like 1/3 cannot be represented in a finite number of decimal digits, values like 1/10 cannot be represented in a finite number of bits. 
- Since values are approximate, calculations with them are also approximate, and rounding errors accumulate. 

## Let's See Practically

```
#include <stdio.h>
#include <string.h>

void intToBinary(unsigned int n)// Print binary stored in plain 32 bit block
{
        int c, k;

        for (c = 31; c >= 0; c--)
        {
                k = n >> c;

                if (k & 1)
                        printf("1");
                else
                        printf("0");
        }

        printf("\n");
}

int main(void) {

        float f = 3.14;

        //printf("f = %a\n", f);  //See hex value in scientific(with exponential) form

        unsigned int m;
        memcpy(&m, &f, sizeof (m));     // Copy memory representation of float to plain 32 bit block

        intToBinary(m);

        return 0;
}
```

- This code will print binary representation of float on console.

## Where decimal point is stored ?

- The decimal point is not explicitly stored anywhere. 
- As i written line `Floating-point numbers are encoded by storing the significand & the exponent (along with a sign bit)`, but you dont get it first time. Dont worry 99% people dont get it first, including me.

## A bit more about repesenting numbers in memory

- According to `IEEE 754-1985` world wide standard, you can also store zero, negative/positive infinity and even NaN(Not a Number). Dont worry if you dont know what is NaN, i will give explanation shortly(But be worried, if you dont know infinity).

### Zero representation
- sign = 0 for positive zero, 1 for negative zero.
- exponent = 0.
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
3. **Round toward +∞** – rounding towards positive infinity.
4. **Round toward −∞** – rounding towards negative infinity.

## Extra Knowledge Points

- In older time, embedded system processors does not use floating point numbers as they dont have such hardware capabilities.
- So there is some alternative to floating point number, called Fixed Point Numbers.
- Fixed point number is usually used in special-purpose applications on embedded processors that can only do integer arithmetic, but decimal fixed point('.') is manipulated by software library.
- But nowadays, microcontrollers have separate FPU's too, like STM32F series.
