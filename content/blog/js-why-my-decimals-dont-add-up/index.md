---
title: JS - Why don't my decimals add up?
date: "2021-06-10"
description: "A look at potential issues when adding up decimals in JS"
---

I came across a bug recently for a simple table of inputs. A user is able to add a series of numbers, up to 2 decimal places, and then when a save button is clicked the total of the number is validated to make sure that it is between 0 and 100. The particular issue that this user had is that from their perspective they had entered a series of decimals that added up to exactly 100 but the form was stopping them from saving, giving the error message that the total was over 100.

If you have been working with JS for a while then it is quite likely that you have come across this problem, or a similar one, at some point. The most common form that a lot of people first come across this issue is:

```javascript
0.1 + 0.2 // 0.30000000000000004
```

But in this case the problem looks more like:

```javascript
11.1 + 33.17 + 11.3 + 11.32 + 33.11 // 100.00000000000001
```

I have reproduced this issue in the a [codesandbox](https://codesandbox.io/s/floating-points-maths-v89rv?file=/src/index.js) so that the numbers can be played around with a bit more.

## What's happening?

The fundamental issue is that the [JS Number type](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Number) uses a floating point format - [IEEE 754 - binary64](https://en.wikipedia.org/wiki/Double-precision_floating-point_format) - that is unable to precisely represent some decimal numbers. I'm not going to go into detail here - I started but it got too long for one post so you can read more about some of the maths involved in - [JS - Why does 0.1 + 0.2 ≠ 0.3](../js-why-does-0.1-plus-0.2-not-equal-0.3/).

For this post all that you need to know is that when working with integers the maths will be fine - as long as you don't go out of range:

```javascript
Number.MAX_SAFE_INTEGER //9007199254740991
Number.MIN_SAFE_INTEGER //-9007199254740991
```

When working with decimals though - as soon as you start doing any sort of arithmetic there is always the potential for unexpected results.

## What to do?

The most important thing is to simply be aware of this potential issue when working with JS. The majority of the time you'll probably find it will not be a problem for you.

By far the most common areas that I have seen this come up is when working with currencies or percentages (the particular example at the start of this post was a table of percentages), but there are other instances when performing arithmetic on decimals is important.

### Work only with integers

In a lot of cases the best solution is to make it so that we are working with integers only as opposed to decimals. As mentioned earlier, integer arithmetic will be precise as long as you don't exceed the min or max values.

This approach does rely on us knowing the number of decimal places required in advance though.

So going back to the numbers used earlier:

```javascript
11.1 + 33.17 + 11.3 + 11.32 + 33.11 // 100.00000000000001
(1110 + 3317 + 1130 + 1132 + 3311) / 100 // 100
```

As you can see, in this instance we know there can only be 2 decimal places so we can multiply each number by 100 to give make sure we always have integers, and then divide by 100 at the end to give us the result we are after. How you want to go about performing these conversions will depend on the codebase you are working with - you could do it case by case or create some reusable helper functions.

You can see an [integer only sandbox](https://codesandbox.io/s/wind-tuna-agt5ffsfs-40bw6?file=/src/index.js) that takes this approach. The only difference between the original solution, where we get the floating point issue, and the the integer only solution is the total line:

```javascript
total = inputs.reduce((total, current) => current + total, 0);
total = inputs.reduce((total, current) => current * 100 + total, 0) / 100;
```

In many cases it will be this simple to fix the problem.

### Currency

If you're working with currencies then you could use a library like [dinero.js](https://v2.dinerojs.com/docs) or [currency.js](https://currency.js.org/). These are both designed to solve floating point issues with currencies, although they go about it in very different ways.

I have created a [currency.js implementation](https://codesandbox.io/s/affectionate-maxwell-u18jc?file=/src/index.js) of the original codesandbox. In this example it takes in exactly the same numbers we started with but outputs our desired value of 100.00.

One thing to be concious of when working with currencies, is that you may need to vary the number of decimal places and this might impact sums that you need to carry out. If you look at the [Active ISO 4217 currencies](https://en.wikipedia.org/wiki/ISO_4217#Active_codes) you can see that although 2 decimal places is by far the most common, there are some currencies with 0, 3 and 4 decimal places.

### toFixed

One approach that I have seen in codebases and also recommended in forums is to use toFixed. In a lot of cases this might be fine but it can give people a false sense of security and lead to incorrect results.

If you are keeping the number of decimal places consistent (inputs have 2 decimal places and the total has 2 decimal places) then you shouldn't have any issues:

```javascript
const total = 11.10 + 33.17 + 11.30 + 11.32 + 33.11
total.toFixed(2); // "100.00"
```

However, it is very easy to end up with unexpected results using toFixed. For example, we would expect 0.14 + 0.21 + 0.30 to equal 0.65 and, if we wanted one decimal place, to round up to 0.7. If you do this in JS though, you end up with 0.6:

```javascript
const total = 0.14 + 0.21 + 0.30 // 0.6499999999999999
total.toFixed(1); // "0.6"
```

This goes back to the inability of the JS floating point format to precisely store some numbers that are easy to precisely represent in decimal format.

You see toFixed given as an answer to simple rounding questions in JS but you get this problem and the solution, again, is to make sure you are only working with integers and round by shifting your result by the number of decimal places you want to round to. You can see this below where we multiple the number by 100 as we want to round to 2 decimal places. This gives us the answer we might have expected from toFixed:

```javascript
const input = 0.365;
input.toFixed(2); // "0.36"
Math.round(input * 100) / 100; // 0.37
```

The other potential problem to mention with toFixed is that you end up working with Strings instead of Numbers, this can lead to concatenation as opposed to addition:

```javascript
const input = 0.365;
const fixed = input.toFixed(2); // "0.36"
fixed + 0.21; // "0.360.21"
```

It's easy to fix, using parseFloat, but your code starts to bloat and can get confusing.

Finally, every time you convert between types there is a performance cost - probably unnoticeable in most cases but it is still there. It's usually much faster to keep working with just Numbers if possible.

For these reasons I avoid toFixed and have not come across a situation that can't be solved working purely with integers. I find this code easier to read and less error prone.
