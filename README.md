
# Parseq

Better living thru eventuality!

Parseq provides a straightforward functional approach to the management of eventuality. Parseq embraces the paradigm of distributed message passing.

You should structure your application as a set of requestor functions that each performs or manages a unit of work. This is a good design pattern in general. The workflow is specified by assembling the requestors into sets that are passed to the parseq factories. The units of work are kept distinct from the mechanics of control flow, leading to better programs.

Parseq is in the Public Domain.

[parseq.js](https://github.com/douglascrockford/parseq/blob/master/parseq.js) is a module that exports a parseq object that contains five factory functions.

### Factory

A factory function is any function that returns a requestor function. Parseq provides these factory functions:

    parseq.fallback(
        requestor_array,
        time_limit
    )

    parseq.parallel(
        required_array,
        optional_array,
        time_limit,
        time_option,
        throttle
    )

    parseq.parallel_object(
        required_object,
        optional_object,
        time_limit,
        time_option,
        throttle
    )

    parseq.race(
        requestor_array,
        time_limit,
        throttle
    )

    parseq.sequence(
        requestor_array,
        time_limit
    )

Each of these factories (except for `parallel_object`) takes an array of requestor functions. The `parallel` factory can take two arrays of requestor functions.

Each of these factory functions returns a requestor function. A factory function may throw an exception if it finds problems in its parameters.

### Requestor

A requestor function is any function that takes a callback and a value.

    my_little_requestor(callback, value)

A requestor will do some work or send a message to another process or system. When the work is done, the requestor signals the result by passing a value to its callback. The callback could be called in a future turn, so the requestor does not need to block, nor should it ever block.

The `value` may be of any type, including objects, arrays, and `undefined`.

A requestor will pass its `value` parameter to any requestors that it starts. A sequence will pass the `value` parameter to its first requestor. It will then pass the result of the previous requestor to the next requestor.

A requestor should not throw an exception. It should communicate all failures through its callback.

### Callback

A callback function takes two arguments: `value` and `reason`.

    my_little_callback(value, reason)

If `value` is `undefined`, then failure is being signalled. `reason` may contain information explaining the failure. If `value` is not `undefined`, then success is being signalled and `value` contains the result.

### Cancel

A requestor function may return a cancel function. A cancel function takes a reason argument that might be propagated as the `reason` of some callback.

    my_little_cancel(reason)

A cancel function attempts to stop the operation of the requestor. If a program decides that it no longer needs the result of a requestor, it can call the cancel function that the requestor returned. This is not an undo operation. It is just a way of stopping unneeded work. It is not guaranteed to stop the work.

### Time Limit

All of the factories can take a `time_limit` expressed in milliseconds. The requestor that the factory returns will fail if it can not finish its work in the specified time. If `time_limit` is `0` or `undefined`, then there will be no time limit.

Three of the factories (`parallel`, `parallel_object`, and `race`) can take a `throttle` argument. Normally these factories want to start all of their requestors at once. Unfortunately, that can cause some incompetent systems to fail due to resource exhaustion or other limitations. The `throttle` puts an upper limit on the number of requestors that can be running at once. Please be aware that some of your requestors might not start running until others have finished. You need to factor that delay into your time limits.