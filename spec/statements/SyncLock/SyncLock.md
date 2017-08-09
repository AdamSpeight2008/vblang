## SyncLock Statement

A `SyncLock` statement allows statements to be synchronized on an expression, which ensures that multiple threads of execution do not execute the same statements at the same time.

```antlr
SyncLockStatement
    : 'SyncLock' Expression StatementTerminator
      Block?
      'End' 'SyncLock' StatementTerminator
    ;
```

The expression must be classified as a value and is evaluated once, upon entry to the block. When entering the `SyncLock` block, the `Shared` method `System.Threading.Monitor.Enter` is called on the specified expression, which blocks until the thread of execution has an exclusive lock on the object returned by the expression. The type of the expression in a `SyncLock` statement must be a reference type. For example:

```vb
Class Test
    Private count As Integer = 0

    Public Function Add() As Integer
        SyncLock Me
            count += 1
            Add = count
        End SyncLock
    End Function

    Public Function Subtract() As Integer
        SyncLock Me
            count -= 1
            Subtract = count
        End SyncLock
    End Function
End Class
```

The example above synchronizes on the specific instance of the class `Test` to ensure that no more than one thread of execution can add or subtract from the count variable at a time for a particular instance.

The `SyncLock` block is implicitly contained by a `Try` statement whose `Finally` block calls the `Shared` method `System.Threading.Monitor.Exit` on the expression. This ensures the lock is freed even when an exception is thrown. As a result, it is invalid to branch into a `SyncLock` block from outside of the block, and a `SyncLock` block is treated as a single statement for the purposes of `Resume` and `Resume Next`. The above example is equivalent to the following code:

```vb
Class Test
    Private count As Integer = 0

    Public Function Add() As Integer
        Try
            System.Threading.Monitor.Enter(Me)

            count += 1
            Add = count
        Finally
            System.Threading.Monitor.Exit(Me)
        End Try
    End Function

    Public Function Subtract() As Integer
        Try
            System.Threading.Monitor.Enter(Me)

            count -= 1
            Subtract = count
        Finally
            System.Threading.Monitor.Exit(Me)
        End Try
    End Function
End Class
```
