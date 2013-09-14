# nebraska [![build status](https://secure.travis-ci.org/thlorenz/nebraska.png)](http://travis-ci.org/thlorenz/nebraska)

Streams the state of another stream.

```js
var nebraska = require('nebraska');

var numbers = require('../test/fixtures/number-readable');
var powers = require('../test/fixtures/power-transform');

// the numbers readable emits numbers faster than the powers transfom
// can handle them (simulated via throttle settings)
var readable  =  numbers({ to: 10, throttle: 100, objectMode: true, highWaterMark: 0 })
  , transform =  powers({ throttle: 400, objectMode: true, highWaterMark: 4  })

var numbersState = nebraska(readable, { interval: 400 })
  , powersState  = nebraska(transform, { interval: 400 })

readable
  .on('data', function (d) { console.log('Emitted: ', d.toString()) })
  .on('end', numbersState.endSoon.bind(numbersState))
  .pipe(transform)
  .on('data', function (d) { console.log('Transformed: ', d.toString()) })
  .on('end', powersState.endSoon.bind(powersState))

numbersState.on('data', console.log.bind(console));
powersState.on('data', console.log.bind(console));
```

#### Output:

The below shows that buffer is filled to `highWaterMark` and drained while the faster number stream gets paused
due to backpressure.

**Note:** the last part of the output was omitted for brevity.

```sh
Emitted:  0
Emitted:  1
Emitted:  2
Emitted:  3
{ label: 'NumberReadable(S)',
  readable: { highWaterMark: 0, bufferLength: 0 } }
{ label: 'PowerTransform(S)',
  readable: { highWaterMark: 4, bufferLength: 0 },
  writable: { highWaterMark: 4, bufferLength: 3 } }
Transformed:  0
{ label: 'NumberReadable(S)',
  readable: { highWaterMark: 0, bufferLength: 1 } }
{ label: 'PowerTransform(S)',
  readable: { highWaterMark: 4, bufferLength: 0 },
  writable: { highWaterMark: 4, bufferLength: 2 } }
Transformed:  1
{ label: 'NumberReadable(S)',
  readable: { highWaterMark: 0, bufferLength: 1 } }
{ label: 'PowerTransform(S)',
  readable: { highWaterMark: 4, bufferLength: 0 },
  writable: { highWaterMark: 4, bufferLength: 1 } }
Transformed:  4
{ label: 'NumberReadable(S)',
  readable: { highWaterMark: 0, bufferLength: 1 } }
{ label: 'PowerTransform(S)',
  readable: { highWaterMark: 4, bufferLength: 0 },
  writable: { highWaterMark: 4, bufferLength: 0 } }
Transformed:  9
Emitted:  4
Emitted:  5
Emitted:  6
Emitted:  7
{ label: 'NumberReadable(S)',
  readable: { highWaterMark: 0, bufferLength: 0 } }
{ label: 'PowerTransform(S)',
  readable: { highWaterMark: 4, bufferLength: 0 },
  writable: { highWaterMark: 4, bufferLength: 3 } }
Transformed:  16
[ ... ]
```

## Installation

    npm install nebraska

## API


###*nebraska(stream[, opts])*
```
/**
 * Creates a readable stream that streams readable/writable state changes of the given stream.
 * 
 * @name WatcherReadable
 * @function
 * @param stream {Stream} a readable and/or writable stream whose states to stream
 * @param opts {Object} with the following properties
 *  - interval {Number} the millisecond interval at which state updates are streamed (default: 500)
 *  - readable {Array[String]} names of readable properties that should be included in the state stream (default: highWaterMark, bufferLength)
 *  - writable {Array[String]} names of writable properties that should be included in the state stream (default: highWaterMark, bufferLength)
 *  - label {String} label that is emitted with every state update (default: the name of the stream constructor + (S))
 * @return {Stream} WatcherReadable that emits state updates.
 */
```

###*WatcherReadable.endSoon*

```
/**
 * Call this in case you want to tell the state stream to end.
 * Useful for testing and/or when you want to end your debugging session and allow the program to exit.
 */
```

###*nebraska.properties*

```
/**
 * Arrays of all property names for each type of stream which make sense to be included in state updates.
 * @return {Object} with the following properties
 *  - readable: {Array[String]} properties of the stream.readableState
 *  - writable: {Array[String]} properties of the stream.writableState
 */
 ```


## Why the name Nebraska?

Because it has more streams than Montana.

## License

MIT
