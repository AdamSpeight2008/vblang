
## Using statement

Instances of types are automatically released by the garbage collector when a collection is run and no live references to the instance are found. If a type holds on a particularly valuable and scarce resource (such as database connections or file handles), it may not be desirable to wait until the next garbage collection to clean up a particular instance of the type that is no longer in use. To provide a lightweight way of releasing resources before a collection, a type may implement the `System.IDisposable` interface. A type that does so exposes a `Dispose` method that can be called to force valuable resources to be released immediately, as such:

```vb
Module Test
    Sub Main()
        Dim x As DBConnection = New DBConnection("...")

        ' Do some work
        ...

        x.Dispose()        ' Free the connection
    End Sub
End Module
```

The `Using` statement automates the process of acquiring a resource, executing a set of statements, and then disposing of the resource. The statement can take two forms: in one, the resource is a local variable declared as a part of the statement and treated as a regular local variable declaration statement; in the other, the resource is the result of an expression.

```antlr
UsingStatement
    : 'Using' UsingResources StatementTerminator
      Block?
      'End' 'Using' StatementTerminator
    ;

UsingResources
    : VariableDeclarators
    | Expression
    ;
```

If the resource is a local variable declaration statement then the type of the local variable declaration must be a type that can be implicitly converted to `System.IDisposable`. The declared local variables are read-only, scoped to the `Using` statement block and must include an initializer. If the resource is the result of an expression then the expression must be classified as a value and must be of a type that can be implicitly converted to `System.IDisposable`. The expression is only evaluated once, at the beginning of the statement.

The `Using` block is implicitly contained by a `Try` statement whose finally block calls the method `IDisposable.Dispose` on the resource. This ensures the resource is disposed even when an exception is thrown. As a result, it is invalid to branch into a `Using` block from outside of the block, and a `Using` block is treated as a single statement for the purposes of `Resume` and `Resume Next`. If the resource is `Nothing`, then no call to `Dispose` is made. Thus, the example:

```vb
Using f As C = New C()
    ...
End Using
```

is equivalent to:

```vb
Dim f As C = New C()
Try
    ...
Finally
    If f IsNot Nothing Then
        f.Dispose()
    End If
End Try
```

A `Using` statement that has a local variable declaration statement may acquire multiple resources at a time, which is equivalent to nested `Using` statements.  For example, a `Using` statement of the form:

```vb
Using r1 As R = New R(), r2 As R = New R()
    r1.F()
    r2.F()
End Using
```

is equivalent to:

```vb
Using r1 As R = New R()
    Using r2 As R = New R()
        r1.F()
        r2.F()
    End Using
End Using
```

