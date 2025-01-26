# Interesting Tidbits

Here are some things I've found noteworthy while learning.

## Rounding to Nearest Size Boundary

I originally saw this while studying Bloomberg's BSL library. In `bsl/bslstl/bsl_string.h`, there's
the following snippet that's similar to this:

```cpp
SIZE = ((20 + sizeof(TYPE) - 1) & ~(sizeof(TYPE) - 1));
```
The comment on that line (ln. 1029-1032), mentions that the code rounds the buffer to a word
boundary. I didn't know why it worked, so I did some research.
[This StackOverflow comment explains it for a similar code segment](https://stackoverflow.com/a/8178170):

>  `#define _INTSIZE(n) ((sizeof(n) + sizeof(int) - 1) & ~(sizeof(int) - 1))`
>
>> The code first adds three to the number.
>>
>> Then it zeroes out the last two bits to round down to a multiple of fours. Just like you can round down to the nearest multiple of 100 in decimal by replacing the last two digits with zeroes.)
>> 
>> If the number is already a multiple of four, adding three to it and then rounding down to the nearest multiple of four leaves it alone, as desired. If the number is 1, 2, or 3 more than a multiple of 4, adding 3 to it raises it above the next multiple of 4, which it then rounds down to, exactly as desired.

So, in other words, the act of adding `sizeof(TYPE) - 1` and then masking the value with the binary
inverse of `sizeof(TYPE) - 1` rounds the value to the nearest multiple of `sizeof(TYPE)`. 

The interesting takeaway for me was the use of bit operations to produce such a result.