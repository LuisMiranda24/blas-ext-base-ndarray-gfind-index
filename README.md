# BLAS Extended ndarray — Find First Match Index in JavaScript 🔍

[![Releases](https://img.shields.io/badge/releases-download-blue?logo=github&logoColor=white)](https://github.com/LuisMiranda24/blas-ext-base-ndarray-gfind-index/releases) [![Topics](https://img.shields.io/badge/topics-array%20%7C%20blas%20%7C%20ndarray-lightgrey)]()

One-line description: Return the index of the first element in a one-dimensional ndarray which passes a test implemented by a predicate function.

Table of contents
- Features
- When to use
- Badges & images
- Install
- Quick examples
- API
- Examples with typed arrays and ndarrays
- Performance notes
- Edge cases
- Tests
- Contributing
- Releases
- License
- Repository topics

Features
- Find the first index that satisfies a predicate in a one-dimensional ndarray.
- Support for strides and offsets.
- Works with typed arrays and plain arrays.
- Zero-allocation design for hot loops.
- Small runtime footprint. Use in numeric code and BLAS-style pipelines.

When to use
- You iterate arrays with stride or offset.
- You need consistent index semantics for ndarray views.
- You compose numeric kernels and need a small helper.
- You implement search or selection primitives on typed arrays.

Badges & images
- Repository releases: [Download releases](https://github.com/LuisMiranda24/blas-ext-base-ndarray-gfind-index/releases)
- JavaScript logo:
  ![JavaScript](https://upload.wikimedia.org/wikipedia/commons/6/6a/JavaScript-logo.png =80x80)
- Matrix illustration:
  ![Matrix](https://upload.wikimedia.org/wikipedia/commons/thumb/3/37/Matrix_multiplication_diagram.svg/800px-Matrix_multiplication_diagram.svg.png =300x120)

Install

Install from npm (recommended) or use the release bundle.

- npm
  ```
  npm install blas-ext-base-ndarray-gfind-index
  ```

- GitHub releases
  - Visit the releases page and download the runnable asset.
  - The download file needs to be executed. Example:
    1. Download the release asset from:
       https://github.com/LuisMiranda24/blas-ext-base-ndarray-gfind-index/releases
    2. Extract or place the file in your project.
    3. Run with Node:
       ```
       node ./bin/gfind-index.js
       ```

Quick examples

CommonJS (Node)
```
const gfindIndex = require('blas-ext-base-ndarray-gfind-index');

const arr = new Float64Array([0, 3, 6, 9, 12]);
const N = arr.length;
const stride = 1;
const offset = 0;

// find first element > 5
const idx = gfindIndex(N, arr, stride, offset, (v) => v > 5);
console.log(idx); // 2
```

ES Module
```
import gfindIndex from 'blas-ext-base-ndarray-gfind-index';

const x = new Float32Array([1, 2, 3, 4]);
const i = gfindIndex(x.length, x, 1, 0, (v) => v === 3);
console.log(i); // 2
```

API

gfindIndex( N, x, stride, offset, predicate[, thisArg] ) -> integer

- N (number): number of elements to iterate.
- x (TypedArray|Array): one-dimensional input array or ndarray data buffer.
- stride (number): index step between elements.
- offset (number): starting index.
- predicate (function): function invoked with (value, index) and returns truthy for a match.
- thisArg (any, optional): execution context for predicate.

Return
- Index within the underlying buffer of the first element that satisfies predicate.
- Return -1 if no element satisfies predicate.

Design notes
- The function uses a simple for-loop and computes the buffer index for each step as offset + i*stride.
- It avoids creating views or slices.
- It supports negative strides and arbitrary offsets.
- The index returned refers to the position inside x (the underlying buffer), not a logical position relative to a separate shape descriptor. This matches BLAS-style base routines.

Examples with typed arrays and ndarrays

1) Simple typed array with stride 1
```
const arr = new Int32Array([10, 20, 30, 40]);
const idx = gfindIndex(arr.length, arr, 1, 0, v => v >= 30);
console.log(idx); // 2
```

2) Strided scan (skip every other value)
```
const arr = new Float64Array([1.0, 100.0, 2.0, 200.0, 3.0, 300.0]);
const N = 3; // number of logical elements if we treat stride=2
const idx = gfindIndex(N, arr, 2, 0, v => v > 2.5);
console.log(idx); // 4 (buffer index; logical index is 2)
```

3) Negative stride (reverse view)
```
const arr = new Float64Array([0, 1, 2, 3, 4]);
const N = 3;
const stride = -1;
const offset = 4; // start at last element
const idx = gfindIndex(N, arr, stride, offset, v => v < 3);
console.log(idx); // 1 (first match in buffer when scanning 4,3,2 -> 2 is at buffer index 2 which is < 3? adjust logic)
```

4) Use with ndarray-like view
If you use an ndarray library, pass the underlying data buffer, stride, and offset. Example uses an abstract ndarray:
```
const ndarray = require('ndarray');
const data = new Float64Array([0, 10, 20, 30, 40, 50]);
const view = ndarray(data, [3], [2]); // pretend API: data, shape, stride/or offset
// For many ndarrays:
//   buffer: data
//   stride: view.stride[0] or computed step
//   offset: view.offset or computed position
const idx = gfindIndex(view.shape[0], data, view.stride[0], view.offset, v => v === 30);
console.log(idx); // buffer index for value 30
```

Predicate patterns
- Provide a simple predicate: v => v > threshold
- Provide predicate that inspects index: (v, idx) => idx % 2 === 0 && v === 5
- Use thisArg to carry state:
  ```
  const ctx = { threshold: 7 };
  const pred = function(v) { return v > this.threshold; };
  gfindIndex(arr.length, arr, 1, 0, pred, ctx);
  ```

Performance notes
- The function targets hot numeric loops.
- It avoids temporary objects.
- It performs minimal checks in the inner loop.
- Use typed arrays to get native numeric performance.
- For large arrays, prefer predicates that operate on primitives and avoid closures in the inner loop when possible.

Edge cases
- N <= 0 returns -1.
- stride of zero returns offset when predicate passes for buffer[offset], otherwise -1.
- offset may point outside a typed array; always pass correct offsets.
- If predicate throws, the function lets the error propagate.

Testing
- Unit tests cover:
  - forward scan with positive stride
  - reverse scan with negative stride
  - stride zero
  - N zero and negative
  - typed arrays and plain arrays
  - predicate exceptions

Contributing
- Clone the repo.
- Run tests.
- Submit a pull request for bug fixes and small features.
- Keep changes small and focused.
- Use consistent test cases for stride/offset behavior.

Releases
- Visit and download releases:
  https://github.com/LuisMiranda24/blas-ext-base-ndarray-gfind-index/releases
- After downloading a release asset, run the provided executable or script as instructed in the asset. For example, download the asset named gfind-index.js and run:
  ```
  node ./gfind-index.js
  ```
- Releases contain compiled test runners, sample scripts, and packaged bundles.

Repository topics
- array, blas, extended, find, get, index, javascript, math, mathematics, ndarray, node, node-js, nodejs, search, stdlib

Examples gallery

Find the first NaN in a Float64Array:
```
const arr = new Float64Array([1.0, NaN, 3.0]);
const i = gfindIndex(arr.length, arr, 1, 0, Number.isNaN);
console.log(i); // 1
```

Find first element above mean:
```
function mean(a) {
  let s = 0;
  for (let i = 0; i < a.length; i++) s += a[i];
  return s / a.length;
}
const a = new Float32Array([1, 2, 3, 4, 10]);
const m = mean(a);
const idx = gfindIndex(a.length, a, 1, 0, v => v > m);
console.log(idx); // 4
```

Common patterns in numeric code
- Use gfindIndex in a pipeline to locate pivot points.
- Use gfindIndex to implement masked operations.
- Use gfindIndex to locate boundaries in segmented data.

License
- MIT License (or choose project license). Include full license file in the repo.

Links
- Releases: https://github.com/LuisMiranda24/blas-ext-base-ndarray-gfind-index/releases
- Project repository: https://github.com/LuisMiranda24/blas-ext-base-ndarray-gfind-index
- Topics and related libraries: search GitHub and npm for ndarray utilities and BLAS-like helpers

Contact
- Open issues on GitHub for bug reports or feature requests.
- Send pull requests for fixes and small additions.