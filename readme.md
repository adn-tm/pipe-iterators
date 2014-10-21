# pipe-iterators

Functions for iterating over object mode streams: `forEach`, `map`, `mapKey`, `reduce`, `filter`, `fromArray`, `toArray`, `devnull`, `pipe`, `head`, `tail`, `pipe`, `through`, `thru`, `writable`, `readable`, `duplex`, `pipeline`.

# Installation

    npm install --save pipe-iterators

Preamble:

```js
var pi = require('pipe-iterators');
```

## Iteration functions

The iterator functions closely follow the native `Array.*` iteration API (e.g. `forEach`, `map`, `filter`), but the functions return object mode streams instead of operating on arrays.

### forEach

```js
pi.forEach(callback, [thisArg])
```

Returns a duplex stream which calls a function for each element in the stream. `callback` is invoked with two arguments - `obj` (the element value) and `index` (the element index). The return value from the callback is ignored. 

If `thisArg` is provided, it is available as `this` within the callback.

```js
pi.fromArray(['a', 'b', 'c'])
  .pipe(pi.forEach(function(obj) { console.log(obj); }));
```

### map

```js
pi.map(callback, [thisArg])
```

Returns a duplex stream which produces a new stream of values by mapping each value in the stream through a transformation callback. The callback is invoked with two arguments, `obj` (the element value) and `index` (the element index). The return value from the callback is written back to the stream. 

If `thisArg` is provided, it is available as `this` within the callback.

Note: if you return `null` from your map function, core streams will interpret this as EOF for the stream.

```js
pi.fromArray([{ a: 'a' }, { b: 'b' }, { c: 'c' }])
  .pipe(pi.map(function(obj) { return _.defaults(obj, { foo: 'bar' }); }));
```

### reduce

```js
pi.reduce(callback, [initialValue])
```

Reduce returns a duplex stream which boils down a stream of values into a single value. `initialValue` is the initial value of the reduction, and each successive step of it should be returned by the `callback`. The callback is called with three arguments: `prev` (the accumulator value), `curr` (the current value) and `index` (the index).

When the input stream ends, the stream emits the value in the accumulator.

If `initialValue` is not provided, then `prev` will be equal to the first value in the array and `curr` will be equal to the second on the first call. 

```js
pi.fromArray(['a', 'b', 'c'])
  pipe(pi.reduce(function(posts, post) { return posts.concat(post); }, []));
```

### filter

```js
pi.filter(callback, [thisArg])
```

Returns a duplex stream which writes all values that pass (return `true` for) the test implemented by the provided `callback` function.

The callback is invoked with two arguments, `obj` (the element value) and `index` (the element index). If the callback returns `true`, the element is written to the next stream, otherwise the element is filtered out. 

```js
pi.filter(function(post) { return !post.draft; })
```

### mapKey

```js
pi.mapKey(key, callback, [thisArg])
pi.mapKey(hash, [thisArg])
```

Returns a duplex stream which produces a new stream of values by mapping a single key (when given `key` and `callback`) or multiple keys (when given `hash`) through a transformation callback. The callback is invoked with two arguments: `value` (the value `element[key]`), and `obj` (the element itself). The return value from the callback is set on the element, and the element itself is written back to the stream. 

If `thisArg` is provided, it is available as `this` within the callback.

```js
pi.fromArray([{ path: '/a/a' }, { path: '/a/b' }, { path: '/a/c' }])
  .pipe(pi.mapKey('path', function(p) { return p.replace('/a/', '/some/'); }));
```

You can also call the `mapKey` with a hash:

```js
pi.mapKey({
  a: function(value) { /* ... */},
  b: 'str',
  c: true
})
```

Each key in the hash is replaced with the return value of the function in the hash. When the value in the hash is not a function, it is simply assigned as the new value for that key.

## Input and output

These utility functions make it easy to provide input into a stream or capture output from a stream.

### fromArray

```js
pi.fromArray(arr)
```

Returns a readable stream given an array. The stream will emit one item for each item in the array, and then emit end.

### toArray

```js
pi.toArray(callback)
pi.toArray(array)
```

Returns a writable stream which buffers the input it receives into an array. When the stream emits `end`, the `callback` is called with one parameter - the array which contains the input elements written to the the stream.

You can also pass an instance of an array instead of a callback. The array's contents will be updated with the elements from the stream when the writable stream emits `finish`.

## Constructing streams

These functions make creating readable, writable and transform streams a bit less boilerplatey.

### thru & through

```js
pi.thru([options], [transformFn], [flushFn]);
pi.thru.obj([transformFn], [flushFn]);
pi.thru.ctor([options], [transformFn], [flushFn]);
```

Returns a Transform stream given a set of `options`, a `transformFn` and `flushFn`. You can call this function as `pi.through` or `pi.thru`. This uses the `through2` module, so you should [take a look at the documentation for that module](https://github.com/rvagg/through2). In short:

- The `options` hash is passed to `stream.Transform` to construct the stream. See the [core docs](http://nodejs.org/api/stream.html#stream_new_stream_duplex_options).
- The `transformFn` has the signature: `function (chunk, encoding, onDone) {}`. See the [core docs](http://nodejs.org/api/stream.html#stream_transform_transform_chunk_encoding_callback) for details.
- The `flushFn` has the signature `function(onDone)`. See the [core docs](http://nodejs.org/api/stream.html#stream_transform_flush_callback) for details.
- `thru.obj(fn)` is a convenience wrapper around `thru({ objectMode: true }, fn)`. 
- `thru.ctor()` returns a constructor for a custom Transform. This is useful when you want to use the same transform logic in multiple instances.

### writable

```js
pi.writable([options], writeFn)
pi.writable.obj(writeFn)
pi.writable.ctor([options], writeFn)
```

Returns a Writable stream given a set of `options` and a `writeFn`.

Has the same options as `thru`:

- The `options` hash is passed to `stream.Writable` to construct the stream. See the [core docs](http://nodejs.org/api/stream.html#stream_new_stream_writable_options).
- The `writeFn` has the signature: `function(chunk, encoding, callback) {}`. See the [core docs](http://nodejs.org/api/stream.html#stream_writable_write_chunk_encoding_callback_1) for details.
- `writable.obj()` is a convenience wrapper for `writable({ objectMode: true })`.
- `writable.ctor()` returns a constructor for the writable stream. 

### readable

```js
pi.readable([options], [readFn])
pi.readable.obj([readFn])
pi.readable.ctor([options], [readFn])
```

Returns a Readable stream given a set of `options` and a `readFn`.

Has the same options as `thru`:

- The `options` hash is passed to `stream.Readable` to construct the stream. See the [core docs](http://nodejs.org/api/stream.html#stream_new_stream_readable_options).
- The `readFn` has the signature: `function(size) {}`. See the [core docs](http://nodejs.org/api/stream.html#stream_readable_read_size_1) for details.
- `readable.obj()` is a convenience wrapper for `readable({ objectMode: true })`.
- `readable.ctor()` returns a constructor for the readable stream.

### duplex

```js
pi.duplex([options], writeFn, readFn)
pi.duplex.obj(writeFn, readFn)
pi.duplex.ctor([options], writeFn, readFn)
```

Returns a Duplex stream given a set of `options`, a `writeFn` and a `readFn`.

Has the same options as `thru`:

- The `options` hash is passed to `stream.Duplex` to construct the stream. See the [core docs](http://nodejs.org/api/stream.html#stream_new_stream_duplex_options).
- The `writeFn` has the signature: `function(chunk, encoding, callback) {}`. See the [core docs](http://nodejs.org/api/stream.html#stream_writable_write_chunk_encoding_callback_1) for details. 
- The `readFn` has the signature: `function(size) {}`. See the [core docs](http://nodejs.org/api/stream.html#stream_readable_read_size_1) for details.
- `duplex.obj()` is a convenience wrapper for `duplex({ objectMode: true })`.
- `duplex.ctor()` returns a constructor for the duplex stream.

### combine

```js
pi.combine(writableStream, readableStream)
```

Takes a readable stream and a writable stream and returns a duplex stream. 

Note: the two streams ARE NOT piped together. If you want to construct a pipeline with multiple streams, you can, but you need to perform the pipe operations yourself (or use the `.pipeline` function instead). This makes `.combine` work with streams where the connections is not via a pipe mechanism, like with `child_process.spawn`:

```js
var child = require('child_process').spawn('wc', ['-c']);
pi.fromArray(['a', 'b', 'c'])
  .pipe(pi.combine(child.stdin, child.stdout))
  .pipe(process.stdout);
```

Listeners for the `error` event will receive errors that are emitted in either stream, or that are emitted as a result of piping into the duplex stream.

### devnull

```js
pi.devnull()
```

Returns a writable stream which consumes any input and produces no output. Useful for consuming output from duplex streams when prototyping or when you want to run the processing but discard the final output.

### cap

```js
pi.cap(duplex)
```

Returns a writable stream given a duplex stream. Any input written into the stream is written to the duplex stream.

### clone

```js
pi.clone()
```

Returns a duplex stream. Inputs written to the stream are cloned and then written out. This is useful if you need to ensure that concurrent modifications to objects written into multiple streams do not influence each other.

## Control flow

These functions allow you to write more advanced streams, going from one linear sequence of transformation steps to multiple pipelines.

### fork

```js
pi.fork(stream1, [stream2], [...])
pi.fork([ stream1, stream2, ... ])
```

Returns a duplex stream. Inputs written to the stream are written to all of the streams passed as arguments to `fork`.

Every forked stream receives a clone of the original input object. Cloning prevents annoying issues that might occur when one fork stream modifies an object that is shared among multiple forked streams.

Also accepts a single array of streams as the first parameter. 

### match

```js
pi.match(condition1, stream1, [condition2], [stream2], [...], [rest])
pi.match([ condition1, stream1, condition2, stream2, ..., rest ])
```

Allows you to construct `if-else` style conditionals which split a stream into multiple substreams based on a condition.

Returns a writable stream given a series of `condition` function and `stream` pairs. When elements are written to the stream, they are matched against each condition function in order. 

The `condition` function is called with two arguments - `obj` (the element value) and `index` (the element index). If the condition returns `true`, the element is written to the associated stream and no further matches are performed.

The last argument, `rest` is optional. It should be a writable stream (without a preceding condition function). Any elements not matching the other conditions will be written into it.

```js
pi.fromArray([
  { url: '/people' }, 
  { url: '/posts/1' }, { url: '/posts' },
  { url: '/comments/2' }])
  .pipe(pi.match(
    function(req) { return /^\/people.*$/.test(req.url); },
    pi.pipeline(
        pi.forEach(function(obj) { console.log('person!', obj); }), 
        pi.devNull()
    ),
    function(req) { return /^\/posts.*$/.test(req.url); },
    pi.pipeline(
        pi.forEach(function(obj) { console.log('post!', obj); }), 
        pi.devNull()
    ),
    pi.pipeline(
        pi.forEach(function(obj) { console.log('other:', obj); }), 
        pi.devNull()
    )
  ));
```

TODO: Listening for 'error' will recieve errors from all streams inside the pipe.


## Constructing pipelines from individual elements

These functions apply `pipe` in various ways to make it easier to go from an array of streams to a pipeline.

### pipe()

```js
pi.pipe(stream1, [stream2], [...])
pi.pipe([ stream1, stream2, ...])
```

Given a series of streams, calls `.pipe()` for each stream in sequence and returns an array which contains all the streams. Used by `head()` and `tail()`.

Also accepts a single array of streams as the first parameter. 

### head()

```js
pi.head(stream1, [stream2], [...])
pi.head([ stream1, stream2, ... ])
```

Given a series of streams, calls `.pipe()` for each stream in sequence and returns the first stream in the series.

Also accepts a single array of streams as the first parameter. 

Similar to `a.pipe(b).pipe(c)`, but `.head()` returns the first stream (`a`) rather than the last stream (`c`). 

### tail()

```js
pi.tail(stream1, [stream2], [...])
pi.tail([ stream1, stream2, ... ])
```

Given a series of streams, calls `.pipe()` for each stream in sequence and returns the last stream in the series.

Also accepts a single array of streams as the first parameter. 

Just like calling `a.pipe(b).pipe(c)`.

### pipeline

```js
pi.pipeline(stream1, stream2, ...)
pi.pipeline([ stream1, stream2, ... ])
```

Constructs a pipeline from a series of streams. Always returns a single stream object, which is either duplex or writable. Pipelines are series of streams that either:

- start with a duplex stream and end with a duplex stream or
- start with a duplex stream and end with a writable stream

Given a pipeline that starts with a duplex stream and ends with a duplex stream, `pipeline` returns a single duplex stream in which any writes go the first stream and any reads/pipes etc. are done from the last stream.

Normally, when just manually applying `pipe` you have to pick whether to return the first stream or the last stream in the pipeline. Returning the first stream has the benefit that writes to it will correctly go into the pipeline, but of course any reads/pipes from it will skip the rest of the pipeline. Returning the last stream has the opposite problem: you can read from the pipeline but cannot pipe to the first stream anymore. With `pipeline` you don't need to choose, since the return result works as you would expect.

Given a pipeline that starts with a duplex stream and ends with a writable stream, `pipeline` returns a single writable stream in which any writes go the first stream. Since the last stream in the pipeline is writable but not readable, the pipeline is also only writable but not readable. This helps stop errors where you accidentally pipe the first stream of a pipeline out (which will not work as expected since outputs do not pass through the whole pipeline).

With `.pipeline()`, writes into the return value go to the first stream but reads (and pipe calls) are applied to the last value in the stream:

```js
module.exports = function() {
  return pi.pipeline(a, a2, a3);
};
```

works as expected and `input.pipe(myPipeline).pipe(b)` writes to `a` but reads from `a3`.

### Checking stream instances

These functions are like [rvagg/isstream](https://github.com/rvagg/isstream), but they work correctly on Node 0.8. The main differences are that 1) the 0.8 core streams from things like fs and child_process are correctly detected and 2) the functions use duck typing (checking for conformance to an API) rather than `instanceof` checks which can be problematic in a browser environment or when using modules that are compatible from an API perspective but do not descend from the native `stream`.

#### isStream

```js
pi.isStream(obj)
```

Returns true if a stream provides either the Readable stream interface or the Writable stream interface.

#### isReadable

```js
pi.isReadable(obj)
```

Returns true if a stream provides the Readable stream interface.

#### isWritable

```js
pi.isWritable(obj)
```

Returns true if a stream provides the Writable stream interface.

#### isDuplex

```js
pi.isDuplext(obj)
```

Returns true if a stream provides both the Readable and Writable stream interfaces.

## What about asynchronous iteration?

Meh, `through2` streams already make writing async iteration quite easy. 

## What about splitting strings?

Best handled by something that can do that in an efficient manner, such as [binary-split](https://github.com/maxogden/binary-split).

## Related

- [Raynos/duplexer](https://github.com/Raynos/duplexer)
- [dominictarr/event-stream](https://github.com/dominictarr/event-stream)
- [Medium/sculpt](https://github.com/Medium/sculpt)
- Duplexing:
  - [deoxxa/duplexer2](https://github.com/deoxxa/duplexer2)
  - [naomik/bun](https://github.com/naomik/bun)
  - [KylePDavis/node-stream-utils](https://github.com/KylePDavis/node-stream-utils): duplex() for old (0.8.x) style streams
  - [sterpe/composite-pipes](https://github.com/sterpe/composite-pipes)
- [rvagg/isstream](https://github.com/rvagg/isstream)
