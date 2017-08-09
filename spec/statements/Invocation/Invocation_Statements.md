## Invocation Statements

An invocation statement invokes a method preceded by the optional keyword `Call`. The invocation statement is processed in the same way as the function invocation expression, with some differences noted below. The invocation expression must be classified as a value or void. Any value resulting from the evaluation of the invocation expression is discarded.

If the `Call` keyword is omitted, then the invocation expression must start with an identifier or keyword, or with `.` inside a `With` block. Thus, for instance, "`Call 1.ToString()`" is a valid statement but "`1.ToString()`" is not. (Note that in an expression context, invocation expressions also need not start with an identifier. For example, "`Dim x = 1.ToString()`" is a valid statement).

There is another difference between the invocation statements and invocation expressions: if an invocation statement includes an argument list, then this is always taken as the argument list of the invocation. The following example illustrates the difference:

```vb
Module Test
    Sub Main()
        Call {Function() 15}(0)
        ' error: (0) is taken as argument list, but array is not invokable

        Call ({Function() 15}(0))
        ' valid, since the invocation statement has no argument list

        Dim x = {Function() 15}(0)
        ' valid as an expression, since (0) is taken as an array-indexing

        Call f("a")
        ' error: ("a") is taken as argument list to the invocation of f

        Call f()("a")
        ' valid, since () is the argument list for the invocation of f

        Dim y = f("a")
        ' valid as an expression, since f("a") is interpreted as f()("a")
    End Sub

    Sub f() As Func(Of String,String)
        Return Function(x) x
    End Sub
End Module
```

```antlr
InvocationStatement
    : 'Call'? InvocationExpression StatementTerminator
    ;
```