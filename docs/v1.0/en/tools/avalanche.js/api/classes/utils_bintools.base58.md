[avalanche](../README.md) › [Utils-BinTools](../modules/utils_bintools.md) › [Base58](utils_bintools.base58.md)

# Class: Base58

A Base58 class that uses the cross-platform Buffer module. Built so that Typescript will accept the code.

```js
let b58:Base58 = new Base58();
let str:string = b58.encode(somebuffer);
let buff:Buffer = b58.decode(somestring);
```

## Hierarchy

* **Base58**

## Index

### Properties

* [alphabetIdx0](utils_bintools.base58.md#protected-alphabetidx0)
* [b58](utils_bintools.base58.md#protected-b58)
* [b58alphabet](utils_bintools.base58.md#protected-b58alphabet)
* [big58Radix](utils_bintools.base58.md#protected-big58radix)
* [bigZero](utils_bintools.base58.md#protected-bigzero)

### Methods

* [decode](utils_bintools.base58.md#decode)
* [encode](utils_bintools.base58.md#encode)

## Properties

### `Protected` alphabetIdx0

• **alphabetIdx0**: *string* = "1"

*Defined in [utils/bintools.ts:235](https://github.com/ava-labs/avalanche.js/blob/c723742/src/utils/bintools.ts#L235)*

___

### `Protected` b58

• **b58**: *number[]* = [
        255, 255, 255, 255, 255, 255, 255, 255,
        255, 255, 255, 255, 255, 255, 255, 255,
        255, 255, 255, 255, 255, 255, 255, 255,
        255, 255, 255, 255, 255, 255, 255, 255,
        255, 255, 255, 255, 255, 255, 255, 255,
        255, 255, 255, 255, 255, 255, 255, 255,
        255, 0, 1, 2, 3, 4, 5, 6,
        7, 8, 255, 255, 255, 255, 255, 255,
        255, 9, 10, 11, 12, 13, 14, 15,
        16, 255, 17, 18, 19, 20, 21, 255,
        22, 23, 24, 25, 26, 27, 28, 29,
        30, 31, 32, 255, 255, 255, 255, 255,
        255, 33, 34, 35, 36, 37, 38, 39,
        40, 41, 42, 43, 255, 44, 45, 46,
        47, 48, 49, 50, 51, 52, 53, 54,
        55, 56, 57, 255, 255, 255, 255, 255,
        255, 255, 255, 255, 255, 255, 255, 255,
        255, 255, 255, 255, 255, 255, 255, 255,
        255, 255, 255, 255, 255, 255, 255, 255,
        255, 255, 255, 255, 255, 255, 255, 255,
        255, 255, 255, 255, 255, 255, 255, 255,
        255, 255, 255, 255, 255, 255, 255, 255,
        255, 255, 255, 255, 255, 255, 255, 255,
        255, 255, 255, 255, 255, 255, 255, 255,
        255, 255, 255, 255, 255, 255, 255, 255,
        255, 255, 255, 255, 255, 255, 255, 255,
        255, 255, 255, 255, 255, 255, 255, 255,
        255, 255, 255, 255, 255, 255, 255, 255,
        255, 255, 255, 255, 255, 255, 255, 255,
        255, 255, 255, 255, 255, 255, 255, 255,
        255, 255, 255, 255, 255, 255, 255, 255,
        255, 255, 255, 255, 255, 255, 255, 255
    ]

*Defined in [utils/bintools.ts:236](https://github.com/ava-labs/avalanche.js/blob/c723742/src/utils/bintools.ts#L236)*

___

### `Protected` b58alphabet

• **b58alphabet**: *string* = "123456789ABCDEFGHJKLMNPQRSTUVWXYZabcdefghijkmnopqrstuvwxyz"

*Defined in [utils/bintools.ts:234](https://github.com/ava-labs/avalanche.js/blob/c723742/src/utils/bintools.ts#L234)*

___

### `Protected` big58Radix

• **big58Radix**: *BN* = new BN(58)

*Defined in [utils/bintools.ts:270](https://github.com/ava-labs/avalanche.js/blob/c723742/src/utils/bintools.ts#L270)*

___

### `Protected` bigZero

• **bigZero**: *BN* = new BN(0)

*Defined in [utils/bintools.ts:271](https://github.com/ava-labs/avalanche.js/blob/c723742/src/utils/bintools.ts#L271)*

## Methods

###  decode

▸ **decode**(`b`: string): *Buffer*

*Defined in [utils/bintools.ts:305](https://github.com/ava-labs/avalanche.js/blob/c723742/src/utils/bintools.ts#L305)*

Dencodes a base-58 into a [Buffer](https://github.com/feross/buffer)

**Parameters:**

Name | Type | Description |
------ | ------ | ------ |
`b` | string | A base-58 string to decode  |

**Returns:** *Buffer*

A [Buffer](https://github.com/feross/buffer) from the decoded string.

___

###  encode

▸ **encode**(`buff`: Buffer): *string*

*Defined in [utils/bintools.ts:280](https://github.com/ava-labs/avalanche.js/blob/c723742/src/utils/bintools.ts#L280)*

Encodes a [Buffer](https://github.com/feross/buffer) as a base-58 string

**Parameters:**

Name | Type | Description |
------ | ------ | ------ |
`buff` | Buffer | A [Buffer](https://github.com/feross/buffer) to encode  |

**Returns:** *string*

A base-58 string.
