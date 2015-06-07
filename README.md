# sffc-encoder

sffc-encoder is provides the encode/decode functions to and from SFFC (Significant Figures First Code) format.

SFFC format is a Integer to binary encoding format with a changing byte size (full bytes) based on the number of significant digits in base 10 are needed to represent the Integer.
This format benifits the more "rounded" (Higher powers of 10) Integer with a lower byte size encoding.

The decoding Scheme:

- First 2/3 bits are length flag:
 1. If first 2 bits are 1 then the flag is 2 bits.
 2. Else the flag is the first 3 bits.
- Remove the flag bits and concat the rest with the next extra amount of bytes using the conversion table (appendix A).
- Split the bits to mantis and exponent to their respective bits using table 1.
- Multiple the mantis by  10^exponent in base 10 to get the encoded Integer.

There are also some encoding optimizition, for example, even though 23 has 2 significant figures we can still encode it using 3 flag bits and 5 binary bits in only 1 byte so thats what we do by using the zero extra byte flag to encode 23 in 1 byte.

### Installation

```sh
$ npm install sffc-encoder
```


### Encode
Params:
- number - takes any integer between 0 to 9007199254740992

Returns a new Buffer holding the encoded Integer

```js
var sffc = require('sffc-encoder')
console.log(sffc.encode(1321321321)) // Will print: <Buffer 82 76 0e 1b 48>
console.log(sffc.encode(1)) // Will print: <Buffer 01>
```

### Decode

Params:
- consume - takes a consumable buffer (You can use [buffer-consumer] like in the example to create one)

Returns the Integer as number

```js
var sffc = require('sffc-encoder')
var consumer = require('buffer-consumer')

var codeBuffer = new Buffer([0x82,0x76,0x0e,0x1b,0x48])

console.log(sffc.decode(consumer(codeBuffer))) // Will print: 1321321321
```

### Testing

In order to test you need to install [mocha] globaly on your machine

```sh
$ cd /"module-path"/sffc-encoder
$ mocha
```

### Apendix A

 -------------------------------------------------------------------------------------------------------------------------
|Byte Flag | Significant Digits                          | Mantis bit size | Exponent bit size | Extra Bytes | Total Bytes|
|----------|---------------------------------------------|-----------------|-------------------|-------------|------------|
|000       | 1-31 (Number Value, not significant digits) | 5               | 0                 | 0           | 1          |
|001       | 2                                           | 9               | 4                 | 1           | 2          |
|010       | 5                                           | 17              | 4                 | 2           | 3          |
|011       | 7                                           | 25              | 4                 | 3           | 4          |
|100       | 10                                          | 34              | 3                 | 4           | 5          |
|101       | 12                                          | 42              | 3                 | 5           | 6          |
|11*       | 16                                          | 54              | 0                 | 6           | 7          |
 -------------------------------------------------------------------------------------------------------------------------

License
----

MIT


[mocha]:https://www.npmjs.com/package/mocha
[buffer-consumer]:https://www.npmjs.com/package/buffer-consumer