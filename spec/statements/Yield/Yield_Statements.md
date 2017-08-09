## Yield Statement

Yield statements are related to iterator methods, which are described in Section [Iterator Methods](/statements/methods/iterator_methods.md).

```antlr
YieldStatement
    : 'Yield' Expression StatementTerminator
    ;
```

`Yield` is a reserved word if the immediately enclosing method or lambda expression in which it appears has an `Iterator` modifier, and if the `Yield` appears after that `Iterator` modifier; it is unreserved elsewhere. It is also unreserved in preprocessor directives. The yield statement is only allowed in the body of a method or lambda expression where it is a reserved word. Within the immediately enclosing method or lambda, the yield statement may not occur inside the body of a `Catch` or `Finally` block, nor inside the body of a `SyncLock` statement.

The yield statement takes a single expression which must be classified as a value and whose type is implicitly convertible to the type of the *iterator current variable* (Section [Iterator Methods](statements.md#iterator-methods)) of its enclosing iterator method.

Control flow only ever reaches a `Yield` statement when the `MoveNext` method is invoked on an iterator object. (This is because an iterator method instance only ever executes its statements due to the `MoveNext` or `Dispose` methods being called on an iterator object; and the `Dispose` method will only ever execute code in `Finally` blocks, where `Yield` is not allowed).

When a `Yield` statement is executed, its expression is evaluated and stored in the *iterator current variable* of the iterator method instance associated with that iterator object. The value `True` is returned to the invoker of `MoveNext`, and the control point of this instance stops advancing until the next invocation of `MoveNext` on the iterator object.

