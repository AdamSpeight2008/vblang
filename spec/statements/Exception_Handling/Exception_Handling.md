## Exception-Handling Statements

Visual Basic supports structured exception handling and unstructured exception handling. Only one style of exception handling may be used in a method, but the `Error` statement may be used in structured exception handling. If a method uses both styles of exception handling, a compile-time error results.

```antlr
ErrorHandlingStatement
    : StructuredErrorStatement
    | UnstructuredErrorStatement
    ;
```

### Structured Exception-Handling Statements

Structured exception handling is a method of handling errors by declaring explicit blocks within which certain exceptions will be handled. Structured exception handling is done through a `Try` statement.

```antlr
StructuredErrorStatement
    : ThrowStatement
    | TryStatement
    ;

TryStatement
    : 'Try' StatementTerminator
      Block?
      CatchStatement*
      FinallyStatement?
      'End' 'Try' StatementTerminator
    ;
```


For example:

```vb
Module Test
    Sub ThrowException()
        Throw New Exception()
    End Sub

    Sub Main()
        Try
            ThrowException()
        Catch e As Exception
            Console.WriteLine("Caught exception!")
        Finally
            Console.WriteLine("Exiting try.")
        End Try
    End Sub
End Module
```

A `Try` statement is made up of three kinds of blocks: try blocks, catch blocks, and finally blocks. A *try block* is a statement block that contains the statements to be executed. A *catch block* is a statement block that handles an exception. A *finally block* is a statement block that contains statements to be run when the `Try` statement is exited, regardless of whether an exception has occurred and been handled. A `Try` statement, which can only contain one try block and one finally block, must contain at least one catch block or finally block. It is invalid to explicitly transfer execution into a try block except from within a catch block in the same statement.


#### Finally Blocks

A `Finally` block is always executed when execution leaves any part of the `Try` statement. No explicit action is required to execute the `Finally` block; when execution leaves the `Try` statement, the system will automatically execute the `Finally` block and then transfer execution to its intended destination. The `Finally` block is executed regardless of how execution leaves the `Try` statement: through the end of the `Try` block, through the end of a `Catch` block, through an `Exit Try` statement, through a `GoTo` statement, or by not handling a thrown exception.

Note that the `Await` expression in an async method, and the `Yield` statement in an iterator method, can cause flow of control to suspend in the async or iterator method instance and resume in some other method instance. However, this is merely a suspension of execution and does not involve exiting the respective async method or iterator method instance, and so does not cause `Finally` blocks to be executed.

It is invalid to explicitly transfer execution into a `Finally` block; it is also invalid to transfer execution out of a `Finally` block except through an exception.

```antlr
FinallyStatement
    : 'Finally' StatementTerminator
      Block?
    ;
```

#### Catch Blocks

If an exception occurs while processing the `Try` block, each `Catch` statement is examined in textual order to determine if it handles the exception.

```antlr
CatchStatement
    : 'Catch' ( Identifier ( 'As' NonArrayTypeName )? )?
	  ( 'When' BooleanExpression )? StatementTerminator
      Block?
    ;
```

The identifier specified in a `Catch` clause represents the exception that has been thrown. If the identifier contains an `As` clause, then the identifier is considered to be declared within the `Catch` block's local declaration space. Otherwise, the identifier must be a local variable (not a static variable) that was defined in a containing block.

A `Catch` clause with no identifier will catch all exceptions derived from `System.Exception`. A `Catch` clause with an identifier will only catch exceptions whose types are the same as or derived from the type of the identifier. The type must be `System.Exception`, or a type derived from `System.Exception`. When an exception is caught that derives from `System.Exception`, a reference to the exception object is stored in the object returned by the function `Microsoft.VisualBasic.Information.Err`.

A `Catch` clause with a `When` clause will only catch exceptions when the expression evaluates to `True`; the type of the expression must be a Boolean expression as per Section [Boolean Expressions](expressions.md#boolean-expressions). A `When` clause is only applied after checking the type of the exception, and the expression may refer to the identifier representing the exception, as this example demonstrates:

```vb
Module Test
    Sub Main()
        Dim i As Integer = 5

        Try
            Throw New ArgumentException()
        Catch e As OverflowException When i = 5
            Console.WriteLine("First handler")
        Catch e As ArgumentException When i = 4
            Console.WriteLine("Second handler")
        Catch When i = 5
            Console.WriteLine("Third handler")
        End Try

    End Sub
End Module
```

This example prints:

```
Third handler
```

If a `Catch` clause handles the exception, execution transfers to the `Catch` block. At the end of the `Catch` block, execution transfers to the first statement following the `Try` statement. The `Try` statement will not handle any exceptions thrown in a `Catch` block. If no `Catch` clause handles the exception, execution transfers to a location determined by the system.

It is invalid to explicitly transfer execution into a `Catch` block.

The filters in When clauses are normally evaluated prior to the exception being thrown. For instance, the following code will print "Filter, Finally, Catch".

```vb
Sub Main()
   Try
       Foo()
   Catch ex As Exception When F()
       Console.WriteLine("Catch")
   End Try
End Sub

Sub Foo()
    Try
        Throw New Exception
    Finally
        Console.WriteLine("Finally")
    End Try
End Sub

Function F() As Boolean
    Console.WriteLine("Filter")
    Return True
End Function
```

 However, Async and Iterator methods cause all finally blocks inside them to be executed prior to any filters outside. For instance, if the above code had `Async Sub Foo()`, then the output would be "Finally, Filter, Catch".


#### Throw Statement

The `Throw` statement raises an exception, which is represented by an instance of a type derived from `System.Exception`.

```antlr
ThrowStatement
    : 'Throw' Expression? StatementTerminator
    ;
```

If the expression is not classified as a value or is not a type derived from `System.Exception`, then a compile-time error occurs. If the expression evaluates to a null value at run time, then a `System.NullReferenceException` exception is raised instead.

A `Throw` statement may omit the expression within a catch block of a `Try` statement, as long as there is no intervening finally block. In that case, the statement rethrows the exception currently being handled within the catch block. For example:

```vb
Sub Test(x As Integer)
    Try
        Throw New Exception()
    Catch
        If x = 0 Then
            Throw    ' OK, rethrows exception from above.
        Else
            Try
                If x = 1 Then
                    Throw   ' OK, rethrows exception from above.
                End If
            Finally
                Throw    ' Invalid, inside of a Finally.
            End Try
        End If
    End Try
End Sub
```


### Unstructured Exception-Handling Statements

Unstructured exception handling is a method of handling errors by indicating statements to branch to when an exception occurs. Unstructured exception handling is implemented using three statements: the `Error` statement, the `On Error` statement, and the `Resume` statement.

```antlr
UnstructuredErrorStatement
    : ErrorStatement
    | OnErrorStatement
    | ResumeStatement
    ;
```

For example:

```vb
Module Test
    Sub ThrowException()
        Error 5
    End Sub

    Sub Main()
        On Error GoTo GotException

        ThrowException()
        Exit Sub

GotException:
        Console.WriteLine("Caught exception!")
        Resume Next
    End Sub
End Module
```

When a method uses unstructured exception handling, a single structured exception handler is established for the entire method that catches all exceptions. (Note that in constructors this handler does not extend over the call to the call to `New` at the beginning of the constructor.) The method then keeps track of the most recent exception-handler location and the most recent exception that has been thrown. At entry to the method, the exception-handler location and the exception are both set to `Nothing`. When an exception is thrown in a method that uses unstructured exception handling, a reference to the exception object is stored in the object returned by the function `Microsoft.VisualBasic.Information.Err`.

Unstructured error handling statements are not allowed in iterator or async methods.


#### Error Statement

An `Error` statement throws a `System.Exception` exception containing a Visual Basic 6 exception number. The expression must be classified as a value and its type must be implicitly convertible to `Integer`.

```antlr
ErrorStatement
    : 'Error' Expression StatementTerminator
    ;
```

#### On Error Statement

An `On Error` statement modifies the most recent exception-handling state.

```antlr
OnErrorStatement
    : 'On' 'Error' ErrorClause StatementTerminator
    ;

ErrorClause
    : 'GoTo' '-' '1'
    | 'GoTo' '0'
    | GoToStatement
    | 'Resume' 'Next'
    ;
```

It may be used in one of four ways:

* `On Error GoTo -1` resets the most recent exception to `Nothing`.

* `On Error GoTo 0` resets the most recent exception-handler location to `Nothing`.

* `On Error GoTo LabelName` establishes the label as the most recent exception-handler location. This statement cannot be used in a method that contains a lambda or query expression.

* `On Error Resume Next` establishes the `Resume Next` behavior as the most recent exception-handler location.


#### Resume Statement

A `Resume` statement returns execution to the statement that caused the most recent exception.

```antlr
ResumeStatement
    : 'Resume' ResumeClause? StatementTerminator
    ;

ResumeClause
    : 'Next'
    | LabelName
    ;
```

If the `Next` modifier is specified, execution returns to the statement that would have been executed after the statement that caused the most recent exception. If a label name is specified, execution returns to the label.

Because the `SyncLock` statement contains an implicit structured error-handling block, `Resume` and `Resume Next` have special behaviors for exceptions that occur in `SyncLock` statements. `Resume` returns execution to the beginning of the `SyncLock` statement, while `Resume Next` returns execution to the next statement following the `SyncLock` statement. For example, consider the following code:

```vb
Class LockClass
End Class

Module Test
    Sub Main()
        Dim FirstTime As Boolean = True
        Dim Lock As LockClass = New LockClass()

        On Error GoTo Handler

        SyncLock Lock
            Console.WriteLine("Before exception")
            Throw New Exception()
            Console.WriteLine("After exception")
        End SyncLock

        Console.WriteLine("After SyncLock")
        Exit Sub

Handler:
        If FirstTime Then
            FirstTime = False
            Resume
        Else
            Resume Next
        End If
    End Sub
End Module
```

It prints the following result.

```
Before exception
Before exception
After SyncLock
```

The first time through the `SyncLock` statement, `Resume` returns execution to the beginning of the `SyncLock` statement. The second time through the `SyncLock` statement, `Resume Next` returns execution to the end of the `SyncLock` statement. `Resume` and `Resume Next` are not allowed within a `SyncLock` statement.

In all cases, when a `Resume` statement is executed, the most recent exception is set to `Nothing`. If a `Resume` statement is executed with no most recent exception, the statement raises a `System.Exception` exception containing the Visual Basic error number `20` (Resume without error).

