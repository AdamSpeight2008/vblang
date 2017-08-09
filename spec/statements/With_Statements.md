
## With Statement

A `With` statement allows multiple references to an expression's members without specifying the expression multiple times.

```antlr
WithStatement
    : 'With' Expression StatementTerminator
      Block?
      'End' 'With' StatementTerminator
    ;
```

The expression must be classified as a value and is evaluated once, upon entry into the block. Within the `With` statement block, a member access expression or dictionary access expression starting with a period or an exclamation point is evaluated as if the `With` expression preceded it. For example:

```vb
Structure Test
    Public x As Integer

    Function F() As Integer
        Return 10
    End Sub
End Structure

Module TestModule
    Sub Main()
        Dim y As Test

        With y
            .x = 10
            Console.WriteLine(.x)
            .x = .F()
        End With
    End Sub
End Module
```

It is invalid to branch into a `With` statement block from outside of the block.