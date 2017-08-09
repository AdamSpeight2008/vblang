### Local Variables and Parameters

The preceding sections detail how and when method instances are created, and with them the copies of a method's local variables and parameters. In addition, each time the body of a loop is entered, a new copy is made of each local variable declared inside that loop as described in Section [Loop Statements](statements.md#loop-statements), and the method instance now contains this copy of its local variable rather than the previous copy.

All locals are initialized to their type's default value. Local variables and parameters are always publicly accessible. It is an error to refer to a local variable in a textual position that precedes its declaration, as the following example illustrates:

```vb
Class A
    Private i As Integer = 0

    Sub F()
        i = 1
        Dim i As Integer       ' Error, use precedes declaration.
        i = 2
    End Sub

    Sub G()
        Dim a As Integer = 1
        Dim b As Integer = a   ' This is valid.
    End Sub
End Class
```

In the `F` method above, the first assignment to `i` specifically does not refer to the field declared in the outer scope. Rather, it refers to the local variable, and it is in error because it textually precedes the declaration of the variable. In the `G` method, a subsequent variable declaration refers to a local variable declared in an earlier variable declaration within the same local variable declaration.

Each block in a method creates a declaration space for local variables. Names are introduced into this declaration space through local variable declarations in the method body and through the parameter list of the method, which introduces names into the outermost block's declaration space. Blocks do not allow shadowing of names through nesting: once a name has been declared in a block, the name may not be redeclared in any nested blocks.

Thus, in the following example, the `F` and `G` methods are in error because the name `i` is declared in the outer block and cannot be redeclared in the inner block. However, the `H` and `I` methods are valid because the two `i`'s are declared in separate non-nested blocks.

```vb
Class A
    Sub F()
        Dim i As Integer = 0
        If True Then
               Dim i As Integer = 1
        End If
    End Sub

    Sub G()
        If True Then
            Dim i As Integer = 0
        End If
        Dim i As Integer = 1
    End Sub 

    Sub H()
        If True Then
            Dim i As Integer = 0
        End If
        If True Then
            Dim i As Integer = 1
        End If
    End Sub

    Sub I() 
        For i As Integer = 0 To 9
            H()
        Next i

        For i As Integer = 0 To 9
            H()
        Next i
    End Sub 
End Class
```

When the method is a function, a special local variable is implicitly declared in the method body's declaration space with the same name as the method representing the return value of the function. The local variable has special name resolution semantics when used in expressions. If the local variable is used in a context that expects an expression classified as a method group, such as an invocation expression, then the name resolves to the function rather than to the local variable. For example:

```vb
Function F(i As Integer) As Integer
    If i = 0 Then
        F = 1        ' Sets the return value.
    Else
        F = F(i - 1) ' Recursive call.
    End If
End Function
```

The use of parentheses can cause ambiguous situations (such as `F(1)`, where `F` is a function whose return type is a one-dimensional array); in all ambiguous situations, the name resolves to the function rather than the local variable. For example:

```vb
Function F(i As Integer) As Integer()
    If i = 0 Then
        F = new Integer(2) { 1, 2, 3 }
    Else
        F = F(i - 1) ' Recursive call, not an index.
    End If
End Function
```

When control flow leaves the method body, the value of the local variable is passed back to the invocation expression. If the method is a subroutine, there is no such implicit local variable, and control simply returns to the invocation expression.