# Enhanced Iteration Syntax for Custom Iterators in Zig
## Introduction

This proposal suggests introducing an enhanced iteration syntax to Zig for iterating over custom iterators using for loops directly, aiming to improve code readability and align with iteration patterns common in other programming languages.

## Motivation
Currently, iterating over custom iterators in Zig requires a manual while loop with explicit calls to the iterator's next method and handling the end of iteration with orelse break. This approach, while explicit, can lead to verbose code, especially in scenarios where simplified iteration syntax could achieve the same with less boilerplate.

current syntax
```zig
var range = IntRange(1, 4){ .current = 1, .end = 4 };
    
while (true) {
    const next = range.next() orelse break;
    std.debug.print("Value: {}\n", .{next});
}
```


Proposed Syntax:
```zig
var range = IntRange(1, 4){ .current = 1, .end = 4 };
for (range) |next| {
    std.debug.print("Value: {}\n", .{next});
}
```
The proposed syntax simplifies iteration, making code more concise and readable, and brings Zig's iteration syntax closer to that of other languages, potentially making Zig more approachable for newcomers.

## Detailed Design

The proposal involves the following changes to the Zig language:

1. Introduce a protocol for custom iterators where any type that implements a next method returning an optional value can be directly used in a for loop.
2. The for loop syntax would implicitly call the next method on the iterator until a null value is encountered, indicating the end of the iteration.
3. This change would be backward-compatible, as it introduces new syntax without altering the behavior of existing code.

## Impact on Existing Code

The proposed enhancement is designed to be an addition to the existing iteration mechanisms in Zig, posing no breaking changes to current codebases. It offers an alternative, more concise way to write iteration loops, especially beneficial for scenarios involving custom iterators.

## Drawbacks
**Explicitness**: The proposal might obscure the explicit control flow and error handling that Zig aims for, as the iteration and potential error handling in `next` calls are implicit.
