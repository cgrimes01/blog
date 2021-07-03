---
title: JS Floating Point Numbers
date: "2021-05-01T22:12:03.284Z"
description: "Hello World"
---

Knowing how the Number JS primitive works is something that most devs don't think about, until they have to.

I came across a bug recently that could be reduced down to the user enters decimals into a table and when they try to save, it validates that the total of the numbers given is between 0 and 100. The particular issue that the user had is that from their perspective they had entered a series of decimals that added up to exactly 100 but the form was stopping them from saving, giving the error message that the total was over 100. 

This is a classic JS issue that most JS devs come across at some point. If you haven't done so already then go to the console and just do 0.1 + 0.2.

```javascript
0.1 + 0.2 // 0.30000000000000004
```

## Why?

Put simply JS uses a floating point format to store numbers that focuses on being able to store a wide range of values at the sacrifice of precision. Whereas in many other languages, such as C# or Java, you get multiple integer and floating point types, in JS you get just the one - [Number](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Number) (technically with the introduction of ES2020 you now have [BigInt](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/BigInt) but for now I'm going to pretend that doesn't exist). This is great as it means you don't have to think about data types and for the vast majority of cases everything will be fine, until it isn't...

If you are interested in how JS Numbers are stored then you can read more detail further down. But the important thing to be aware of is that JS Numbers might not be quite what you think. One way to figure out what is really going on is to use toPrecision to show the true value of a Number. 

```javascript
const x = 0.1 
x // 0.1
x.toPrecision(55) // 0.1000000000000000055511151231257827021181583404541015625
```

The reason the console gives you 0.1 for x above is due to the fact that toString is being applied to the Number before it is displayed. You can read the [ECMAScript Number::toString](https://tc39.es/ecma262/multipage/ecmascript-data-types-and-values.html#sec-numeric-types-number-tostring) specification for more detail but the key thing to be aware of is that the output will not be the exact Number but the lowest decimal representation that the Number value is closer to than any other Number representation.



```javascript
const x = 0.1 
const y = 0.2
x // 0.1
y // 0.2
x.toPrecision(55) // 0.1000000000000000055511151231257827021181583404541015625
y.toPrecision(55) // 0.2000000000000000111022302462515654042363166809082031250
const z = x + y
z // 0.30000000000000004                 
z.toPrecision(55) // 0.3000000000000000444089209850062616169452667236328125000
const p = 0.3 // 0.3
p // 0.30000000000000004                 
p.toPrecision(55) // 0.2999999999999999888977697537484345957636833190917968750
```

## Some Maths

What this means is that all JS numbers are stored using a double-precision 64-bit binary format [IEEE 754 - binary64](https://en.wikipedia.org/wiki/Double-precision_floating-point_format). So numbers are stored with 1 bit representing the sign, 11 bits for the exponent and 52 bits for the mantissa/significand/fraction, so all numbers are stored as:

$sign * 2^{exponent} * (1 + mantissa)$

### Sign bit

Encoded as 0 = positive and 1 = negative

### Exponent

Encoded with a 0 offset of 1023 (01111111111). So all numbers are stored as:

encoding - 1023 = exponent

| Exponent Value    |   Encoding                |Binary
|:-------------------|:--------------           |:------
|10                  |1033 (1033 - 1023 = 10)   |$10000001001$
|500                |1523 (1523 - 1023 = 500)   |$10111110011$
|-300               |723 (723 - 1023 = -300)    |$01011010011$
|0                  |1023 (1023 - 1023 = 0)     |$01111111111$

### Mantissa

This is the fractional part of the number with IEEE 754 assuming an integer of 1 on top of the fractional part. So the encoded value represents a decimal and is added to 1. The below shows some example mantissa encodings but only to 8 bits (in reality there would be 52 bits):

Mantissa encoding = $10000000$  
Mantissa value = $2^-1$ 
= $1/2$
= $0.5$

Mantissa encoding = $11110000$  
Mantissa value = $2^-1 + 2^-2 + 2^-3 + 2^-4$  
= $1/2 + 1/4 + 1/8 + 1/16$  
= $0.5 + 0.25 + 0.125 + 0.0625$  
= $0.9375$

### An Example

So taking the simple decimal example of the number 10, it is actually stored using IEEE 754 - binary64 as:

| Part      |   Value                   |   Binary (only showing 8 bit mantissa)    |
| --------- |   ---------               |   ---------                               |
| Sign bit  |   +1                      |   0                                       |
| Exponent  |   3 (encoded as 1026)     |   10000000010                             |
| Mantissa  |   1.25                    |   01000000                                |

x = +1 * $2^3$ * 1.25  
x = 8 * 1.25  
x = 10  

http://weitz.de/ieee/

For an integer like 10 this format can perfectly represent the number with no potential errors.

### Back to our original sum

Now we understand how Numbers are stored, it becomes much easier to understand how operators work and why 0.1 + 0.2 = 0.30000000000000004. The reason it is important to understand how Numbers are stored is because operators are applied on the binary expressions.

There is a great video that walks through adding IEEE-754 floating point numbers - https://www.youtube.com/watch?v=mKJiD2ZAlwM. The below uses this methodology, just with the 64 bit format.

Going back to our original example sum:

$0.1 + 0.2$

### 0.1

Sign bit = +1  
Exponent = 2^-4  
Binary Mantissa = 1.1001100110011001100110011001100110011001100110011010 
Decimal Mantissa = 1.60000000000000008882

x = +1 * 2^-4 * 1.60000000000000008882
x = 0.0625 * 1.60000000000000008882
x = 0.1000000000000000055511151231257827021181583404541015625  

Binary scientific notation:  

1.1001100110011001100110011001100110011001100110011010 x 2^-4

### 0.2

Sign bit = +1  
Exponent = 2^-3 
Binary Mantissa = 1.1001100110011001100110011001100110011001100110011010
Decimal Mantissa = 1.60000000000000008882

x = +1 * 2^-3 * 1.60000000000000008882
x = 0.125 * 1.60000000000000008882
x = 0.2000000000000000111022302462515654042363166809082031250

Binary scientific notation:  

1.1001100110011001100110011001100110011001100110011010 x 2^-3

### 0.1 + 0.2

Now we have the binary scientific notation for 0.1 and 0.2.

0.1 = 1.1001100110011001100110011001100110011001100110011010 x 2^-4
0.2 = 1.1001100110011001100110011001100110011001100110011010 x 2^-3

We need to standardise the exponents between these calculations, we do this by converting 0.1 to have an exponent of -3:

0.1 = 0.11001100110011001100110011001100110011001100110011010 x 2^-3
0.2 = 1.1001100110011001100110011001100110011001100110011010 x 2^-3

Now it is a matter of binary addition:

    0.1100110011001100110011001100110011001100110011001101
 +  1.1001100110011001100110011001100110011001100110011010
 ----------------------------------------------------------
   10.0110011001100110011001100110011001100110011001100111

And we end up with:

0.3 = 10.0110011001100110011001100110011001100110011001100111 x 2^-3

We can then convert this back into the standard notation by changing the exponent, so:

0.3 = 1.00110011001100110011001100110011001100110011001100111 x 2^-2

One important thing to be aware of here is that the mantissa now has 53 bits but we know that the format can only store 52 bits. We need to eliminate the additional bit by rounding it:

0.3 = 1.0011001100110011001100110011001100110011001100110100 x 2^-2

Finally we can convert this into the decimal format (the sign bit will still be 1):

0.3 = +1 * 2^-2 * 1.20000000000000017764
0.3 = 0.25 * 1.20000000000000017764
0.3 = 0.3000000000000000444089209850062616169452667236328125000


```javascript
const maxInt = Number.Max;
```


Over the years I've hit the same issue in a few different ways
