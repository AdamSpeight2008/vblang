## Event Statements

The `RaiseEvent`, `AddHandler`, and `RemoveHandler` statements raise events and handle events dynamically.

```antlr
EventStatement
    : RaiseEventStatement
    | AddHandlerStatement
    | RemoveHandlerStatement
    ;
```

### RaiseEvent Statement

A `RaiseEvent` statement notifies event handlers that a particular event has occurred.

```antlr
RaiseEventStatement
    : 'RaiseEvent' IdentifierOrKeyword
      ( OpenParenthesis ArgumentList? CloseParenthesis )? StatementTerminator
    ;
```

The simple name expression in a `RaiseEvent` statement is interpreted as a member lookup on `Me`. Thus, `RaiseEvent x` is interpreted as if it were `RaiseEvent Me.x`. The result of the expression must be classified as an event access for an event defined in the class itself; events defined on base types cannot be used in a `RaiseEvent` statement.

The `RaiseEvent` statement is processed as a call to the `Invoke` method of the event's delegate, using the supplied parameters, if any. If the delegate's value is `Nothing`, no exception is thrown. If there are no arguments, the parentheses may be omitted. For example:

```vb
Class Raiser
    Public Event E1(Count As Integer)

    Public Sub Raise()
        Static RaiseCount As Integer = 0

        RaiseCount += 1
        RaiseEvent E1(RaiseCount)
    End Sub
End Class

Module Test
    Private WithEvents x As Raiser

    Private Sub E1Handler(Count As Integer) Handles x.E1
        Console.WriteLine("Raise #" & Count)
    End Sub

    Public Sub Main()
        x = New Raiser
        x.Raise()        ' Prints "Raise #1".
        x.Raise()        ' Prints "Raise #2".
        x.Raise()        ' Prints "Raise #3".
    End Sub
End Module
```

The class `Raiser` above is equivalent to:

```vb
Class Raiser
    Public Event E1(Count As Integer)

    Public Sub Raise()
        Static RaiseCount As Integer = 0
        Dim TemporaryDelegate As E1EventHandler

        RaiseCount += 1

        ' Use a temporary to avoid a race condition.
        TemporaryDelegate = E1Event
        If Not TemporaryDelegate Is Nothing Then
            TemporaryDelegate.Invoke(RaiseCount)
        End If
    End Sub
End Class
```


### AddHandler and RemoveHandler Statements

Although most event handlers are automatically hooked up through `WithEvents` variables, it may be necessary to dynamically add and remove event handlers at run time. `AddHandler` and `RemoveHandler` statements do this.

```antlr
AddHandlerStatement
    : 'AddHandler' Expression Comma Expression StatementTerminator
    ;

RemoveHandlerStatement
    : 'RemoveHandler' Expression Comma Expression StatementTerminator
    ;
```

Each statement takes two arguments: the first argument must be an expression that is classified as an event access and the second argument must be an expression that is classified as a value. The second argument's type must be the delegate type associated with the event access. For example:

```vb
Public Class Form1
    Public Sub New()
        ' Add Button1_Click as an event handler for Button1's Click event.
        AddHandler Button1.Click, AddressOf Button1_Click
    End Sub 

    Private Button1 As Button = New Button()

    Sub Button1_Click(sender As Object, e As EventArgs)
        Console.WriteLine("Button1 was clicked!")
    End Sub

    Public Sub Disconnect()
        RemoveHandler Button1.Click, AddressOf Button1_Click
    End Sub
End Class
```

Given an event `E,` the statement calls the relevant `add_E` or `remove_E` method on the instance to add or remove the delegate as a handler for the event. Thus, the above code is equivalent to:

```vb
Public Class Form1
    Public Sub New()
        Button1.add_Click(AddressOf Button1_Click)
    End Sub 

    Private Button1 As Button = New Button()

    Sub Button1_Click(sender As Object, e As EventArgs)
        Console.WriteLine("Button1 was clicked!")
    End Sub

    Public Sub Disconnect()
        Button1.remove_Click(AddressOf Button1_Click)
    End Sub
End Class
```
