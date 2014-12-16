# buffer [![travis][travis-image]][travis-url] [![npm][npm-image]][npm-url] [![downloads][downloads-image]][npm-url] [![gratipay][gratipay-image]][gratipay-url]

#### The buffer module from [node.js](http://nodejs.org/), for the browser.

[![saucelabs][saucelabs-image]][saucelabs-url]

[travis-image]: https://img.shields.io/travis/feross/buffer.svg?style=flat
[travis-url]: https://travis-ci.org/feross/buffer
[npm-image]: https://img.shields.io/npm/v/buffer.svg?style=flat
[npm-url]: https://npmjs.org/package/buffer
[downloads-image]: https://img.shields.io/npm/dm/buffer.svg?style=flat
[gratipay-image]: https://img.shields.io/gittip/feross.svg?style=flat
[gratipay-url]: https://www.gittip.com/feross
[saucelabs-image]: https://saucelabs.com/browser-matrix/buffer.svg
[saucelabs-url]: https://saucelabs.com/u/buffer

With [browserify](http://browserify.org), simply `require('buffer')` or use the `Buffer` global and you will get this module.

The goal is to provide an API that is 100% identical to
[node's Buffer API](http://nodejs.org/api/buffer.html). Read the
[official docs](http://nodejs.org/api/buffer.html) for the full list of properties,
instance methods, and class methods that are supported.

## features

- Manipulate binary data like a boss, in all browsers -- even IE6!
- Super fast. Backed by Typed Arrays (`Uint8Array`/`ArrayBuffer`, not `Object`)
- Extremely small bundle size (**5.04KB minified + gzipped**, 35.5KB with comments)
- Excellent browser support (IE 6+, Chrome 4+, Firefox 3+, Safari 5.1+, Opera 11+, iOS, etc.)
- Preserves Node API exactly, with one important difference (see below)
- `.slice()` returns instances of the same type (Buffer)
- Square-bracket `buf[4]` notation works, even in old browsers like IE6!
- Does not modify any browser prototypes or put anything on `window`
- Comprehensive test suite


## install

To use this module directly (without browserify), install it:

```bash
npm install buffer
```

This module was previously called **native-buffer-browserify**, but please use **buffer**
from now on.


## usage

The module's API is identical to node's `Buffer` API. Read the
[official docs](http://nodejs.org/api/buffer.html) for the full list of properties,
instance methods, and class methods that are supported.

As mentioned above, `require('buffer')` or use the `Buffer` global with
[browserify](http://browserify.org) and this module will automatically be included
in your bundle. Almost any npm module will work in the browser, even if it assumes that
the node `Buffer` API will be available.

To depend on this module explicitly (without browserify), require it like this:

```js
var Buffer = require('buffer/').Buffer  // note: the trailing slash is important!
```

To require this module explicitly, use `require('buffer/')` which tells the node.js module
lookup algorithm (also used by browserify) to use the **npm module** named `buffer`
instead of the **node.js core** module named `buffer`!


## how does it work?

The `Buffer` constructor returns instances of `Uint8Array` that are augmented with function properties for all the `Buffer` API functions. We use `Uint8Array` so that square bracket notation works as expected -- it returns a single octet. By augmenting the instances, we can avoid modifying the `Uint8Array` prototype.


## differences

#### IMPORTANT: always use `Buffer.isBuffer` instead of `instanceof Buffer`

The Buffer constructor returns a `Uint8Array` (with all the Buffer methods added as
properties on the instance) for performance reasons, so `instanceof Buffer` won't work. In
node, you can use either `Buffer.isBuffer` or `instanceof Buffer` to check if an object
is a `Buffer`. But, in the browser you must use `Buffer.isBuffer` to detect the special
`Uint8Array`-based Buffers.

#### Minor: `buf.slice()` does not modify parent buffer's memory in old browsers

If you only support modern browsers (specifically, those with typed array support), then
this issue does not affect you.

In node, the `slice()` method returns a new `Buffer` that shares underlying memory with
the original Buffer. When you modify one buffer, you modify the other. [Read more.](http://nodejs.org/api/buffer.html#buffer_buf_slice_start_end)

This works correctly in browsers with typed array support (\* with the exception of Firefox older than version 30). Browsers that lack typed arrays get an alternate buffer implementation based on `Object` which has no mechanism to point separate `Buffer`s to the same underlying slab of memory.

\* *Firefox older than version 30 gets the `Object` implementation -- not the typed arrays one -- because of [this
bug](https://bugzilla.mozilla.org/show_bug.cgi?id=952403) (now fixed!) that made it impossible to add properties to a typed array.*


## tracking the latest node api

This module tracks the Buffer API in the latest (unstable) version of node.js. The Buffer
API is considered **stable** in the
[node stability index](http://nodejs.org/docs/latest/api/documentation.html#documentation_stability_index),
so it is unlikely that there will ever be breaking changes.
Nonetheless, when/if the Buffer API changes in node, this module's API will change
accordingly.

## performance

See perf tests in `/perf`.

`BrowserBuffer` is the browser `buffer` module (this repo). `Uint8Array` is included as a
sanity check (since `BrowserBuffer` uses `Uint8Array` under the hood, `Uint8Array` will
always be at least a bit faster). Finally, `NodeBuffer` is the node.js buffer module,
which is included to compare against.

### Chrome 38

| Method | Operations | Accuracy | Sampled | Fastest |
|:-------|:-----------|:---------|:--------|:-------:|
| BrowserBuffer#bracket-notation | 11,457,464 ops/sec | ±0.86% | 66 | ✓ |
| Uint8Array#bracket-notation | 10,824,332 ops/sec | ±0.74% | 65 | |
| | | | |
| BrowserBuffer#concat | 450,532 ops/sec | ±0.76% | 68 | |
| Uint8Array#concat | 1,368,911 ops/sec | ±1.50% | 62 | ✓ |
| | | | |
| BrowserBuffer#copy(16000) | 903,001 ops/sec | ±0.96% | 67 | |
| Uint8Array#copy(16000) | 1,422,441 ops/sec | ±1.04% | 66 | ✓ |
| | | | |
| BrowserBuffer#copy(16) | 11,431,358 ops/sec | ±0.46% | 69 | |
| Uint8Array#copy(16) | 13,944,163 ops/sec | ±1.12% | 68 | ✓ |
| | | | |
| BrowserBuffer#new(16000) | 106,329 ops/sec | ±6.70% | 44 | |
| Uint8Array#new(16000) | 131,001 ops/sec | ±2.85% | 31 | ✓ |
| | | | |
| BrowserBuffer#new(16) | 1,554,491 ops/sec | ±1.60% | 65 | |
| Uint8Array#new(16) | 6,623,930 ops/sec | ±1.66% | 65 | ✓ |
| | | | |
| BrowserBuffer#readDoubleBE | 112,830 ops/sec | ±0.51% | 69 | ✓ |
| DataView#getFloat64 | 93,500 ops/sec | ±0.57% | 68 | |
| | | | |
| BrowserBuffer#readFloatBE | 146,678 ops/sec | ±0.95% | 68 | ✓ |
| DataView#getFloat32 | 99,311 ops/sec | ±0.41% | 67 | |
| | | | |
| BrowserBuffer#readUInt32LE | 843,214 ops/sec | ±0.70% | 69 | ✓ |
| DataView#getUint32 | 103,024 ops/sec | ±0.64% | 67 | |
| | | | |
| BrowserBuffer#slice | 1,013,941 ops/sec | ±0.75% | 67 | |
| Uint8Array#subarray | 1,903,928 ops/sec | ±0.53% | 67 | ✓ |
| | | | |
| BrowserBuffer#writeFloatBE | 61,387 ops/sec | ±0.90% | 67 | |
| DataView#setFloat32 | 141,249 ops/sec | ±0.40% | 66 | ✓ |


### Firefox 33

| Method | Operations | Accuracy | Sampled | Fastest |
|:-------|:-----------|:---------|:--------|:-------:|
| BrowserBuffer#bracket-notation | 20,800,421 ops/sec | ±1.84% | 60 | |
| Uint8Array#bracket-notation | 20,826,235 ops/sec | ±2.02% | 61 | ✓ |
| | | | |
| BrowserBuffer#concat | 153,076 ops/sec | ±2.32% | 61 | |
| Uint8Array#concat | 1,255,674 ops/sec | ±8.65% | 52 | ✓ |
| | | | |
| BrowserBuffer#copy(16000) | 1,105,312 ops/sec | ±1.16% | 63 | |
| Uint8Array#copy(16000) | 1,615,911 ops/sec | ±0.55% | 66 | ✓ |
| | | | |
| BrowserBuffer#copy(16) | 16,357,599 ops/sec | ±0.73% | 68 | |
| Uint8Array#copy(16) | 31,436,281 ops/sec | ±1.05% | 68 | ✓ |
| | | | |
| BrowserBuffer#new(16000) | 52,995 ops/sec | ±6.01% | 35 | |
| Uint8Array#new(16000) | 87,686 ops/sec | ±5.68% | 45 | ✓ |
| | | | |
| BrowserBuffer#new(16) | 252,031 ops/sec | ±1.61% | 66 | |
| Uint8Array#new(16) | 8,477,026 ops/sec | ±0.49% | 68 | ✓ |
| | | | |
| BrowserBuffer#readDoubleBE | 99,871 ops/sec | ±0.41% | 69 | |
| DataView#getFloat64 | 285,663 ops/sec | ±0.70% | 68 | ✓ |
| | | | |
| BrowserBuffer#readFloatBE | 115,540 ops/sec | ±0.42% | 69 | |
| DataView#getFloat32 | 288,722 ops/sec | ±0.82% | 68 | ✓ |
| | | | |
| BrowserBuffer#readUInt32LE | 633,926 ops/sec | ±1.08% | 67 | ✓ |
| DataView#getUint32 | 294,808 ops/sec | ±0.79% | 64 | |
| | | | |
| BrowserBuffer#slice | 349,425 ops/sec | ±0.46% | 69 | |
| Uint8Array#subarray | 5,965,819 ops/sec | ±0.60% | 65 | ✓ |
| | | | |
| BrowserBuffer#writeFloatBE | 59,980 ops/sec | ±0.41% | 67 | |
| DataView#setFloat32 | 317,634 ops/sec | ±0.63% | 68 | ✓ |

### Safari 8

| Method | Operations | Accuracy | Sampled | Fastest |
|:-------|:-----------|:---------|:--------|:-------:|
| BrowserBuffer#bracket-notation | 10,279,729 ops/sec | ±2.25% | 56 | ✓ |
| Uint8Array#bracket-notation | 10,030,767 ops/sec | ±2.23% | 59 | |
| | | | |
| BrowserBuffer#concat | 144,138 ops/sec | ±1.38% | 65 | |
| Uint8Array#concat | 4,950,764 ops/sec | ±1.70% | 63 | ✓ |
| | | | |
| BrowserBuffer#copy(16000) | 1,058,548 ops/sec | ±1.51% | 64 | |
| Uint8Array#copy(16000) | 1,409,666 ops/sec | ±1.17% | 65 | ✓ |
| | | | |
| BrowserBuffer#copy(16) | 6,282,529 ops/sec | ±1.88% | 58 | |
| Uint8Array#copy(16) | 11,907,128 ops/sec | ±2.87% | 58 | ✓ |
| | | | |
| BrowserBuffer#new(16000) | 101,663 ops/sec | ±3.89% | 57 | |
| Uint8Array#new(16000) | 22,050,818 ops/sec | ±6.51% | 46 | ✓ |
| | | | |
| BrowserBuffer#new(16) | 176,072 ops/sec | ±2.13% | 64 | |
| Uint8Array#new(16) | 24,385,731 ops/sec | ±5.01% | 51 | ✓ |
| | | | |
| BrowserBuffer#readDoubleBE | 41,341 ops/sec | ±1.06% | 67 | |
| DataView#getFloat64 | 322,280 ops/sec | ±0.84% | 68 | ✓ |
| | | | |
| BrowserBuffer#readFloatBE | 46,141 ops/sec | ±1.06% | 65 | |
| DataView#getFloat32 | 337,025 ops/sec | ±0.43% | 69 | ✓ |
| | | | |
| BrowserBuffer#readUInt32LE | 151,551 ops/sec | ±1.02% | 66 | |
| DataView#getUint32 | 308,278 ops/sec | ±0.94% | 67 | ✓ |
| | | | |
| BrowserBuffer#slice | 197,365 ops/sec | ±0.95% | 66 | |
| Uint8Array#subarray | 9,558,024 ops/sec | ±3.08% | 58 | ✓ |
| | | | |
| BrowserBuffer#writeFloatBE | 17,518 ops/sec | ±1.03% | 63 | |
| DataView#setFloat32 | 319,751 ops/sec | ±0.48% | 68 | ✓ |


### Node 0.11.14

| Method | Operations | Accuracy | Sampled | Fastest |
|:-------|:-----------|:---------|:--------|:-------:|
| BrowserBuffer#bracket-notation | 10,489,828 ops/sec | ±3.25% | 90 | |
| Uint8Array#bracket-notation | 10,534,884 ops/sec | ±0.81% | 92 | ✓ |
| NodeBuffer#bracket-notation | 10,389,910 ops/sec | ±0.97% | 87 | |
| | | | |
| BrowserBuffer#concat | 487,830 ops/sec | ±2.58% | 88 | |
| Uint8Array#concat | 1,814,327 ops/sec | ±1.28% | 88 | ✓ |
| NodeBuffer#concat | 1,636,523 ops/sec | ±1.88% | 73 | |
| | | | |
| BrowserBuffer#copy(16000) | 1,073,665 ops/sec | ±0.77% | 90 | |
| Uint8Array#copy(16000) | 1,348,517 ops/sec | ±0.84% | 89 | ✓ |
| NodeBuffer#copy(16000) | 1,289,533 ops/sec | ±0.82% | 93 | |
| | | | |
| BrowserBuffer#copy(16) | 12,782,706 ops/sec | ±0.74% | 85 | |
| Uint8Array#copy(16) | 14,180,427 ops/sec | ±0.93% | 92 | ✓ |
| NodeBuffer#copy(16) | 11,083,134 ops/sec | ±1.06% | 89 | |
| | | | |
| BrowserBuffer#new(16000) | 141,678 ops/sec | ±3.30% | 67 | |
| Uint8Array#new(16000) | 161,491 ops/sec | ±2.96% | 60 | |
| NodeBuffer#new(16000) | 292,699 ops/sec | ±3.20% | 55 | ✓ |
| | | | |
| BrowserBuffer#new(16) | 1,655,466 ops/sec | ±2.41% | 82 | |
| Uint8Array#new(16) | 14,399,926 ops/sec | ±0.91% | 94 | ✓ |
| NodeBuffer#new(16) | 3,894,696 ops/sec | ±0.88% | 92 | |
| | | | |
| BrowserBuffer#readDoubleBE | 109,582 ops/sec | ±0.75% | 93 | ✓ |
| DataView#getFloat64 | 91,235 ops/sec | ±0.81% | 90 | |
| NodeBuffer#readDoubleBE | 88,593 ops/sec | ±0.96% | 81 | |
| | | | |
| BrowserBuffer#readFloatBE | 139,854 ops/sec | ±1.03% | 85 | ✓ |
| DataView#getFloat32 | 98,744 ops/sec | ±0.80% | 89 | |
| NodeBuffer#readFloatBE | 92,769 ops/sec | ±0.94% | 93 | |
| | | | |
| BrowserBuffer#readUInt32LE | 710,861 ops/sec | ±0.82% | 92 | |
| DataView#getUint32 | 117,893 ops/sec | ±0.84% | 91 | |
| NodeBuffer#readUInt32LE | 851,412 ops/sec | ±0.72% | 93 | ✓ |
| | | | |
| BrowserBuffer#slice | 1,673,877 ops/sec | ±0.73% | 94 | |
| Uint8Array#subarray | 6,919,243 ops/sec | ±0.67% | 90 | ✓ |
| NodeBuffer#slice | 4,617,604 ops/sec | ±0.79% | 93 | |
| | | | |
| BrowserBuffer#writeFloatBE | 66,011 ops/sec | ±0.75% | 93 | |
| DataView#setFloat32 | 127,760 ops/sec | ±0.72% | 93 | ✓ |
| NodeBuffer#writeFloatBE | 103,352 ops/sec | ±0.83% | 93 | |


## credit

This was originally forked from [buffer-browserify](https://github.com/toots/buffer-browserify).


## license

MIT. Copyright (C) [Feross Aboukhadijeh](http://feross.org), and other contributors. Originally forked from an MIT-licensed module by Romain Beauxis.
