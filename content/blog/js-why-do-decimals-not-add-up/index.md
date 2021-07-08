---
title: JS - Why does 0.1 + 0.2 ≠ 0.3
date: "2021-07-03"
description: "Why floating point numbers don't always add up in JS"
---

This is a classic JS issue that most JS devs come across at some point. If you haven't done so already then go to the console and just type 0.1 + 0.2.

```javascript
0.1 + 0.2 // 0.30000000000000004
0.1 + 0.2 === 0.3 // false
```

If you haven't done this before then this might not be exactly what you excpected, so what is going on?

## Numbers in JS

JS uses a floating point format to store numbers that focuses on being able to store a wide range of values at the sacrifice of precision. Whereas in many other languages, such as C# or Java, you get multiple integer and floating point types, in JS you get just the one - [Number](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Number) (technically with the introduction of ES2020 you now have [BigInt](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/BigInt) but for now I'm going to pretend that doesn't exist). This is great as it means you don't have to think about data types and for the vast majority of cases everything will be fine, until it isn't...

One of the confusing things when working with Numbers in JS is that the console or output method you are using to view the results of calculations may well be misleading you. If you take the simple one decimal place value 0.1: if you enter it into a JS console; store it against the variable x and return it then you get 0.1. 

```javascript
const x = 0.1 
x // 0.1
```
Brilliant! Except for the fact that the precise Number value that has been stored against x is not in fact 0.1. A way to find out exactly what has been stored is to force a greater precision to be returned.

```javascript
const x = 0.1 
x // 0.1
x.toPrecision(55) // 0.1000000000000000055511151231257827021181583404541015625
```

So we asked JS to store 0.1, it made it look like it had but then secretly had stored a much more specific decimal. I'm still confused!

### A Repeating Problem

As mentioned earlier the floating point format allows for a wide range but sacrifices precision. The format that JS uses is [IEEE 754 - binary64](https://en.wikipedia.org/wiki/Double-precision_floating-point_format), meaning it has 64 bits to represent the widest range possible. How Numbers are stored is shown in more detail later, but the important thing to be aware of is that numbers are stored as rationals (can be represented as a fraction) and using the binary format it is impossible to exactly represent some numbers. 

We have exactly the same problem with base 10 - if you try to represent $1/3$ as a decimal you end up with a repeating pattern $0.3333333\.{3}$. The format used to store 0.1 using IEEE 754 - binary64, we end up with the repeating binary pattern $1.1\.{0}01\.{1}$ used in part of the representation and therefore an imprecise representation of 0.1.

It is impossible for 0.1 to be accurately stored using IEEE 754 - binary64!

### JS Number::toString

So that covers why JS struggles to store some numbers but it doesn't explain why, on the fact of it, it looks like a value has been accurately stored. 

The reason the console gives you 0.1 when you print x above is due to the fact that toString is being applied to the Number before it is displayed. You can read the [ECMAScript Number::toString specification](https://tc39.es/ecma262/multipage/ecmascript-data-types-and-values.html#sec-numeric-types-number-tostring) for more detail but the key thing to be aware of is that the output will not be the exact Number but the lowest precision decimal representation that the Number value is closer to than any other Number representation (or at least I think that is what it means).

It does this to try and get the best decimal representation of the Number being stored. The success of this is evidenced by the fact that you enter 0.1 and you get 0.1 out!

## So then why is 0.1 + 0.2 ≠ 0.3?

We now know that some numbers can not be precisely represented in the format that JS uses to store Numbers and we also know that when we console.log numbers we aren't necessarily getting the precise Number that JS has stored.

So what is really happening is that an imperfect representation of 0.1 is being added to an imperfect representation of 0.2 - these are being added together to get a value that is close to 0.3. However this value is not the closest possible value to 0.3 so instead of 0.3 you get 0.30000000000000004 when toString is applied. If you just directly assign 0.3 it is still not possible to store a precise value for this but the value stored is the closest possible so when you log it you do see 0.3.

So as can be seen in the JS below what is really happening is not:

0.1 + 0.2 === 0.3

But instead:

0.3000000000000000444089209850062616169452667236328125000 === 0.2999999999999999888977697537484345957636833190917968750

It seems much more reasonable for false to be the return value when viewed like this!

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

So hopefully this gives a much better understand of why the original snippet happens:

```javascript
0.1 + 0.2 // 0.30000000000000004
0.1 + 0.2 === 0.3 // false
```

However, it doesn't entirely explain why 0.1 + 0.2 = 0.30000000000000004. 

If you take the decimal for 0.1 that we worked out above:

0.1000000000000000055511151231257827021181583404541015625

And the decimal for 0.2:

0.2000000000000000111022302462515654042363166809082031250

If we were working with decimals then the expectation would be that our value for 0.3 would be equal to the sum of these values. However:

$$
    0.1000000000000000055511151231257827021181583404541015625  \\
+   0.2000000000000000111022302462515654042363166809082031250  \\
-------------------------------------------------------------- \\
  0.3000000000000000166533453693773481063544750213623046875  \\ 
$$

Which is different to the 0.3000000000000000444089209850062616169452667236328125000 we get:

```javascript
const x = 0.1 + 0.2
z // 0.30000000000000004                 
z.toPrecision(55) // 0.3000000000000000444089209850062616169452667236328125000
```

The reason this happens is that the arithmetic is done on the binary representation. To really be able to follow through the simple example of 0.1 + 0.2 you need to understand how Numbers are stored.

### Storage

JS numbers are stored using a double-precision 64-bit binary format [IEEE 754 - binary64](https://en.wikipedia.org/wiki/Double-precision_floating-point_format). This means that out of the 64 bits available, the first 1 bit represents the sign, the next 11 bits represent the exponent and the final 52 bits represent the mantissa/significand/fraction. The formula to calculate the number being represented using these 3 parts is:

$sign * 2^{exponent} * (1 + mantissa)$

#### Sign bit

Encoded as 0 = positive and 1 = negative.

So if the first bit is 0 then the sign in the formula will be +1 and if the first bit is 1 then the sign in the formulat will be -1.

#### Exponent

This can be slightly more complex to understand as we want to be able to store a range of positive and negative numbers for the exponent. The exponent in this formula is used to essentially tell us how many places to move the binary point and so we can store from -1022 to +1023 (there are some special cases that mean we can't quite use the full range).

The way the encoding allows for this range is by using a 0 offset of 1023 (01111111111). So all numbers are stored as:

encoding - 1023 = exponent

| Exponent Value    |   Encoding                |Binary
|:-------------------|:--------------           |:------
|0                  |1023 (1023 - 1023 = 0)     |$01111111111$
|10                  |1033 (1033 - 1023 = 10)   |$10000001001$
|500                |1523 (1523 - 1023 = 500)   |$10111110011$
|-300               |723 (723 - 1023 = -300)    |$01011010011$
|-1023              |1023 (1023 - 1023 = 0)     |$00000000001$

#### Mantissa

This is the fractional part of the number with IEEE 754 assuming an integer of 1 on top of the fractional part. So the encoded value represents a fraction and is added to 1. The below shows some example mantissa encodings but only to 8 bits (in reality there would be 52 bits):

Mantissa encoding = $10000000$  
Mantissa value = $2^-1$ 
= $1/2$
= $0.5$

Mantissa encoding = $11110000$  
Mantissa value = $2^-1 + 2^-2 + 2^-3 + 2^-4$  
= $1/2 + 1/4 + 1/8 + 1/16$  
= $0.5 + 0.25 + 0.125 + 0.0625$  
= $0.9375$

#### An Example

So taking the simple example of the number 10, it is actually stored using IEEE 754 - binary64 as:

| Part      |   Value                   |   Binary (only showing 8 bit mantissa)    |
| --------- |   ---------               |   ---------                               |
| Sign bit  |   +1                      |   0                                       |
| Exponent  |   3 (encoded as 1026)     |   10000000010                             |
| Mantissa  |   1.25                    |   01000000                                |

x = +1 * $2^3$ * 1.25  
x = 8 * 1.25  
x = 10  

For an integer, like 10, this format can perfectly represent the number with no potential errors. It is when trying to represent decimals that we get imprecision.

If you'd like to play around with 

### Back to our original sum

Now we understand how Numbers are stored, it becomes much easier to understand how operators work and why 0.1 + 0.2 = 0.30000000000000004. The reason it is important to understand how Numbers are stored is because operators are applied on the binary expressions.

There is a great video that walks through adding IEEE-754 floating point numbers - https://www.youtube.com/watch?v=mKJiD2ZAlwM. The below uses this methodology, just with the 64 bit format.

Going back to our original example sum:

$0.1 + 0.2$

#### 0.1

Sign bit = +1  
Exponent = 2^-4  
Binary Mantissa = 1.1001100110011001100110011001100110011001100110011010 
Decimal Mantissa = 1.60000000000000008882

x = +1 * 2^-4 * 1.60000000000000008882
x = 0.0625 * 1.60000000000000008882
x = 0.1000000000000000055511151231257827021181583404541015625  

Binary notation:  

1.1001100110011001100110011001100110011001100110011010 x 2^-4

#### 0.2

Sign bit = +1  
Exponent = 2^-3 
Binary Mantissa = 1.1001100110011001100110011001100110011001100110011010
Decimal Mantissa = 1.60000000000000008882

x = +1 * 2^-3 * 1.60000000000000008882
x = 0.125 * 1.60000000000000008882
x = 0.2000000000000000111022302462515654042363166809082031250

Binary notation:  

1.1001100110011001100110011001100110011001100110011010 x 2^-3

#### 0.1 + 0.2

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

And finally we have managed to get to the exact number that JS gives us:

```javascript
const x = 0.1 + 0.2
z // 0.30000000000000004                 
z.toPrecision(55) // 0.3000000000000000444089209850062616169452667236328125000
```