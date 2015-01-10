thunks
====
A small and magical async control flow tool, wrap promise, generator and anything to thunk.

[![NPM version][npm-image]][npm-url]
[![Build Status][travis-image]][travis-url]
[![Talk topic][talk-image]][talk-url]

# [toa](https://github.com/toajs/toa): A web app framework rely on thunks.

[中文说明](https://github.com/thunks/thunks/blob/master/README_zh.md)

[thunks 的作用域和异常处理设计](https://github.com/thunks/thunks/blob/master/docs/scope-and-error-catch.md)

## Compatibility

ES3+, support node.js and all browsers.

## What is a thunk?

1. **`thunk`** is a function that encapsulates synchronous or asynchronous code inside.

2. **`thunk`** accepts only one `callback` function as an arguments, which is a CPS function;

3. **`thunk`** returns another **`thunk`** function after being called, for chaining operations;

4. **`thunk`** would passing the results into a `callback` function after excuted.

5. If `callback` returns a new **`thunk`** function, then it would be send to another **`thunk`** to excute,
or it would be send to another new **`thunk`** function as the value of the computation.

## Benchmark

```js
➜  thunks git:(master) ✗ node --harmony benchmark/index
Sync Benchmark...

JSBench Start (1000 cycles, async mode):
Test Promise...
Test co...
Test thunks-generator...
Test bluebird...
Test when...
Test RSVP...
Test async...
Test thenjs...
Test thunks...

JSBench Results:
co: 1000 cycles, 25.454 ms/cycle, 39.287 ops/sec
Promise: 1000 cycles, 24.594 ms/cycle, 40.660 ops/sec
thunks: 1000 cycles, 7.276 ms/cycle, 137.438 ops/sec
thunks-generator: 1000 cycles, 5.577 ms/cycle, 179.308 ops/sec
async: 1000 cycles, 2.788 ms/cycle, 358.680 ops/sec
RSVP: 1000 cycles, 2.225 ms/cycle, 449.438 ops/sec
when: 1000 cycles, 1.816 ms/cycle, 550.661 ops/sec
bluebird: 1000 cycles, 1.67 ms/cycle, 598.802 ops/sec
thenjs: 1000 cycles, 1.345 ms/cycle, 743.494 ops/sec

co: 100%; Promise: 103.50%; thunks: 349.84%; thunks-generator: 456.41%; async: 912.98%; RSVP: 1144.00%; when: 1401.65%; bluebird: 1524.19%; thenjs: 1892.49%;

JSBench Completed!

```

```js
➜  thunks git:(master) ✗ node --harmony benchmark/index
Async Benchmark...

JSBench Start (1000 cycles, async mode):
Test Promise...
Test co...
Test thunks-generator...
Test bluebird...
Test when...
Test RSVP...
Test async...
Test thenjs...
Test thunks...

JSBench Results:
co: 1000 cycles, 32.755 ms/cycle, 30.530 ops/sec
Promise: 1000 cycles, 31.139 ms/cycle, 32.114 ops/sec
thunks: 1000 cycles, 11.388 ms/cycle, 87.812 ops/sec
RSVP: 1000 cycles, 10.155 ms/cycle, 98.474 ops/sec
thunks-generator: 1000 cycles, 10.126 ms/cycle, 98.756 ops/sec
when: 1000 cycles, 8.835 ms/cycle, 113.186 ops/sec
bluebird: 1000 cycles, 8.352 ms/cycle, 119.732 ops/sec
async: 1000 cycles, 6.477 ms/cycle, 154.392 ops/sec
thenjs: 1000 cycles, 4.939 ms/cycle, 202.470 ops/sec

co: 100%; Promise: 105.19%; thunks: 287.63%; RSVP: 322.55%; thunks-generator: 323.47%; when: 370.74%; bluebird: 392.18%; async: 505.71%; thenjs: 663.19%;

JSBench Completed!
```

## Demo

```js
var Thunk = require('../thunks.js')();
var fs = require('fs');

var size = Thunk.thunkify(fs.stat);

// sequential
size('.gitignore')(function (error, res) {
  console.log(error, res);
  return size('thunks.js');

})(function (error, res) {
  console.log(error, res);
  return size('package.json');

})(function (error, res) {
  console.log(error, res);
})

// parallel
Thunk.all([size('.gitignore'), size('thunks.js'), size('package.json')])(function (error, res) {
  console.log(error, res);
})

// sequential
Thunk.seq([size('.gitignore'), size('thunks.js'), size('package.json')])(function (error, res) {
  console.log(error, res);
})
```

```js
var Thunk = require('../thunks.js')();
var fs = require('fs');

var size = Thunk.thunkify(fs.stat);


// generator
Thunk(function* () {

  // sequential
  console.log(yield size('.gitignore'));
  console.log(yield size('thunks.js'));
  console.log(yield size('package.json'));

})(function* (error, res) {
  //parallel
  console.log(yield [size('.gitignore'), size('thunks.js'), size('package.json')]);
})();
```

## Installation

**Node.js:**

    npm install thunks

**Bower:**

    bower install thunks

**browser:**

```html
<script src="/pathTo/thunks.js"></script>
```

## API

```js
var thunks = require('thunks');
```

### thunks([options])

Generator of `thunks`, it generates the main function of `Thunk` with its scope.
"scope" refers to the running evironments `Thunk` generated(directly or indirectly) for all `thunk` functions.

1. Here's how you create a basic `Thunk`, any exceptions would be passed the next `thunk` function:

    ```js
    var Thunk = thunks();
    ```

2. Here's the way to create a `Thunk` listening to all exceptions in current scope with `onerror`,
and it will make sure the exeptions not being passed to the followed `thunk` function, unless `onerror` function return `true`.

    ```js
    var Thunk = thunks(function (error) { console.error(error); });
    ```

3. Create a `Thunk` with `onerror` and `debug` listeners.
Results of this `Thunk` would be passed to `debug` function first before passing to the followed `thunk` function.

    ```js
    var Thunk = thunks({
      onerror: function (error) { console.error(error); },
      debug: function () { console.log.apply(console, arguments); }
    });
    ```

Even multiple `Thunk` main functions with diferent scope are composed,
each scope would be seperated from each other,
which means, `onerror` and `debug` would not run in other scopes.

### Thunk(start)

This is the main function, to create new `thunk` functions.

The parameter `start` could be:

1. a `thunk` function, by calling this function a new `thunk` function will be returned

    ```js
    var thunk1 = Thunk(1);
    var thunk2 = Thunk(thunk1); // thunk2 equals to thunk1;
    ```

2. `function (callback) {}`, by calling it, results woule be gathered and be passed to the next `thunk` function

    ```js
    Thunk(function (callback) {
      callback(null, 1)
    })(function (error, value) {
      console.log(error, value); // null 1
    });
    ```

3. a Promise object, results of Promise would be passed to a new `thunk` function

    ```js
    var promise = Promise.resolve(1);

    Thunk(promise)(function (error, value) {
      console.log(error, value); // null 1
    });
    ```

4. objects which implements methods of `toThunk`

    ```js
    var then = Thenjs(1); // then.toThunk() return a thunk function

    Thunk(then)(function (error, value) {
      console.log(error, value); // null 1
    });
    ```

5. Generator and Generator Function, like `co`, and `yield` anything

    ```js
    Thunk(function* () {
      var x = yield 10;
      return 2 * x;
    })(function* (error, res) {
      console.log(error, res); // null, 20

      return yield [1, 2, Thunk(3)];
    })(function* (error, res) {
      console.log(error, res); // null, [1, 2, 3]
      return yield {
        name: 'test',
        value: Thunk(1)
      };
    })(function (error, res) {
      console.log(error, res); // null, {name: 'test', value: 1}
    });
    ```

6. values in other types would be valid results passing to a new `thunk` function

    ```js
    Thunk(1)(function (error, value) {
      console.log(error, value); // null 1
    });

    Thunk([1, 2, 3])(function (error, value) {
      console.log(error, value); // null [1, 2, 3]
    });
    ```

You can also run with `this`:

    ```js
    Thunk.call({x: 123}, 456)(function (error, value) {
      console.log(error, this.x, value); // null 123 456
      return 'thunk!';
    })(function (error, value) {
      console.log(error, this.x, value); // null 123 'thunk!'
    });
    ```

### Thunk.all(obj)
### Thunk.all(thunk1, ..., thunkX)

Returns a `thunk` function.

`obj` can be an array or an object that contains any value. `Thunk.all` will transform value to a `thunk` function and excuted it in parallel. After all of them are finished, an array containing results(in its original order) would be passed to the a new `thunk` function.

```js
Thunk.all([
  Thunk(0),
  function* () { return yield 1; },
  2,
  Thunk(function (callback) { callback(null, [3]); })
])(function (error, value) {
  console.log(error, value); // null [0, 1, 2, [3]]
});

Thunk.all({
  a: Thunk(0),
  b: Thunk(1),
  c: 2,
  d: Thunk(function (callback) { callback(null, [3]); })
})(function (error, value) {
  console.log(error, value); // null {a: 0, b: 1, c: 2, d: [3]}
});
```

You may also write code like this:

```js
Thunk.all.call({x: [1, 2, 3]}, [4, 5, 6])(function (error, value) {
  console.log(error, this.x, value); // null [1, 2, 3] [4, 5, 6]
  return 'thunk!';
})(function (error, value) {
  console.log(error, this.x, value); // null [1, 2, 3] 'thunk!'
});
```

### Thunk.seq([thunk1, ..., thunkX])
### Thunk.seq(thunk1, ..., thunkX)

Returns a `thunk` function.

`thunkX` can be any value, `Thunk.seq` will transform value to a `thunk` function and excuted it in order. After all of them are finished, an array containing results(in its original order) would be passed to the a new `thunk` function.

```js
Thunk.seq([
  function (callback) {
    setTimeout(function () {
      callback(null, 'a', 'b');
    }, 100);
  },
  Thunk(function (callback) {
    callback(null, 'c');
  }),
  [Thunk('d'), function* () { return yield 'e'; }], // thunk in array will be excuted in parallel
  function (callback) {
    should(flag).be.eql([true, true]);
    flag[2] = true;
    callback(null, 'f');
  }
])(function (error, value) {
  console.log(error, value); // null [['a', 'b'], 'c', ['d', 'e'], 'f']
});
```
or

```js
Thunk.seq(
  function (callback) {
    setTimeout(function () {
      callback(null, 'a', 'b');
    }, 100);
  },
  Thunk(function (callback) {
    callback(null, 'c');
  }),
  [Thunk('d'), Thunk('e')], // thunk in array will be excuted in parallel
  function (callback) {
    should(flag).be.eql([true, true]);
    flag[2] = true;
    callback(null, 'f');
  }
)(function (error, value) {
  console.log(error, value); // null [['a', 'b'], 'c', ['d', 'e'], 'f']
});
```

You may also write code like this:

```js
Thunk.seq.call({x: [1, 2, 3]}, 4, 5, 6)(function (error, value) {
  console.log(error, this.x, value); // null [1, 2, 3] [4, 5, 6]
  return 'thunk!';
})(function (error, value) {
  console.log(error, this.x, value); // null [1, 2, 3] 'thunk!'
});
```

### Thunk.race([thunk1, ..., thunkX])
### Thunk.race(thunk1, ..., thunkX)

Returns a `thunk` function with the value or error from one first completed.

### Thunk.digest(error, val1, val2, ...)

Returns a `thunk` function.

Transform a Node.js callback function into a `thunk` function.
This `thunk` function retuslts in `(error, val1, val2, ...)`, which is just being passed to a new `thunk` function,
like:

```js
Thunk(function (callback) {
  callback(error, val1, val2, ...);
})
```

One use case:

```js
Thunk(function (callback) {
  //...
  callback(error, result);
})(function (error, value) {
  //...
  return Thunk.digest(error, value);
})(function (error, value) {
  //...
});
```

You may also write code with `this`：

```js
var a = {x: 1};
Thunk.digest.call(a, null, 1, 2)(function (error, value1, value2) {
  console.log(this, error, value1, value2) // { x: 1 } null 1 2
});
```

### Thunk.thunkify(fn)

Returns a new function that would return a `thunk` function

Transform a `fn` function which is in Node.js style into a new function.
This new function does not accept `callback` as arguments, but accepts `thunk` functions.

```js
var Thunk = require('../thunks.js')();
var fs = require('fs');
var fsStat = Thunk.thunkify(fs.stat);

fsStat('thunks.js')(function (error, result) {
  console.log('thunks.js: ', result);
});
fsStat('.gitignore')(function (error, result) {
  console.log('.gitignore: ', result);
});
```

You may also write code with `this`:

```js
var obj = {a: 8};
function run(x, callback) {
  //...
  callback(null, this.a * x);
};

var run = Thunk.thunkify.call(obj, run);

run(1)(function (error, result) {
  console.log('run 1: ', result);
});
run(2)(function (error, result) {
  console.log('run 2: ', result);
});
```

### Thunk.delay(delay)

Return a `thunk` function, this `thunk` function will be called after `delay` milliseconds.

```js
console.log('Thunk.delay 500: ', Date.now());
Thunk.delay(500)(function () {
  console.log('Thunk.delay 1000: ', Date.now());
  return Thunk.delay(1000);
})(function () {
  console.log('Thunk.delay end: ', Date.now());
});
```

You may also write code with `this`:

```js
console.log('Thunk.delay start: ', Date.now());
Thunk.delay.call(this, 1000)(function () {
  console.log('Thunk.delay end: ', Date.now());
});
```

### Thunk.stop([messagge])

This will stop thunk function with a message similar to Promise's cancelable(not implement yet). It will throw a stop signal.
Stop signal is a Error object with a message and `status === 19`(POSIX signal SIGSTOP) and a special code, stop signal can be caught by `onerror`.

```js
var Thunk = require('thunks')(function(err) {
  console.log(err); // { [Error: Stop now!] code: {}, status: 19 }
});

Thunk(function(callback) {
  Thunk.stop('Stop now!');
  console.log('It will not be run!');
})(function(error, value) {
  console.log('It will not be run!');
});
```

```js
Thunk.delay(100)(function() {
  console.log('Hello');
  return Thunk.delay(100)(function() {
    Thunk.stop('Stop now!');
    console.log('It will not be run!');
  });
})(function(error, value) {
  console.log('It will not be run!');
});
```

[npm-url]: https://npmjs.org/package/thunks
[npm-image]: http://img.shields.io/npm/v/thunks.svg

[travis-url]: https://travis-ci.org/thunks/thunks
[travis-image]: http://img.shields.io/travis/thunks/thunks.svg

[talk-url]: https://guest.talk.ai/rooms/d1ccbf802n
[talk-image]: https://img.shields.io/talk/t/d1ccbf802n.svg
