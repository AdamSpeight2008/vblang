## Await Statement

An await statement has the same syntax as an await operator expression (Section [Await Operator](expressions.md#await-operator)), is allowed only in methods that also allow await expressions, and has the same behavior as an await operator expression.

However, it may be classified as either a value or void. Any value resulting from evaluation of the await operator expression is discarded.

```antlr
AwaitStatement
    : AwaitOperatorExpression StatementTerminator
    ;
```