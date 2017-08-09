* [**Statements**](/Statements/Statements.md)
  * [**Assignment Statements**](/Statements/Assignments/Assignment_Statements.md)
    * [**Regular Assignment**](/Statements/Assignments/Regular_Assignment_Statements.md)
    * [Compund Assignment](/Statements/Assignments/Compound_Assignment_Statements.md)  
    * [Mid Assignment Statement](/Statements/Assignments/Mid_Assignmnet_Statements.md)

-----
# Regular Assignment Statements

A simple assignment statement stores the result of an expression in a variable.

```antlr
RegularAssignmentStatement
    : Expression Equals Expression StatementTerminator
    ;
```

The expression on the left side of the assignment operator must be classified as a variable or a property access, while the expression on the right side of the assignment operator must be classified as a value. The type of the expression must be implicitly convertible to the type of the variable or property access.

If the variable being assigned into is an array element of a reference type, a run-time check will be performed to ensure that the expression is compatible with the array-element type. In the following example, the last assignment causes a `System.ArrayTypeMismatchException` to be thrown, because an instance of `ArrayList` cannot be stored in an element of a `String` array.

```vb
Dim sa(10) As String
Dim oa As Object() = sa
oa(0) = Nothing         ' This is allowed.
oa(1) = "Hello"         ' This is allowed.
oa(2) = New ArrayList() ' System.ArrayTypeMismatchException is thrown.
```

If the expression on the left side of the assignment operator is classified as a variable, then the assignment statement stores the value in the variable. If the expression is classified as a property access, then the assignment statement turns the property access into an invocation of the `Set` accessor of the property with the value substituted for the value parameter. For example:

```vb
Module Test
    Private PValue As Integer

    Public Property P As Integer
        Get
            Return PValue
        End Get

        Set (Value As Integer)
            PValue = Value
        End Set
    End Property

    Sub Main()
        ' The following two lines are equivalent.
        P = 10
        set_P(10)
    End Sub
End Module
```

If the target of the variable or property access is typed as a value type but not classified as a variable, a compile-time error occurs. For example:

```vb
Structure S
    Public F As Integer
End Structure

Class C
    Private PValue As S

    Public Property P As S
        Get
            Return PValue
        End Get

        Set (Value As S)
            PValue = Value
        End Set
    End Property
End Class

Module Test
    Sub Main()
        Dim ct As C = New C()
        Dim rt As Object = new C()

        ' Compile-time error: ct.P not classified as variable.
        ct.P.F = 10

        ' Run-time exception.
        rt.P.F = 10
    End Sub
End Module
```

Note that the semantics of the assignment depend on the type of the variable or property to which it is being assigned. If the variable to which it is being assigned is a value type, the assignment copies the value of the expression into the variable. If the variable to which it is being assigned is a reference type, the assignment copies the reference, not the value itself, into the variable. If the type of the variable is `Object`, the assignment semantics are determined by whether the value's type is a value type or a reference type at run time.


__Note.__ For intrinsic types such as `Integer` and `Date`, reference and value assignment semantics are the same because the types are immutable. As a result, the language is free to use reference assignment on boxed intrinsic types as an optimization. From a value perspective, the result is the same.

Because the equals character (`=`) is used both for assignment and for equality, there is an ambiguity between a simple assignment and an invocation statement in situations such as `x = y.ToString()`. In all such cases, the assignment statement takes precedence over the equality operator. This means that the example expression is interpreted as `x = (y.ToString())` rather than `(x = y).ToString()`.

