## Array-Handling Statements

Two statements simplify working with arrays: `ReDim` statements and `Erase` statements.

```antlr
ArrayHandlingStatement
    : RedimStatement
    | EraseStatement
    ;
```

### ReDim Statement

A `ReDim` statement instantiates new arrays.

```antlr
RedimStatement
    : 'ReDim' 'Preserve'? RedimClauses StatementTerminator
    ;

RedimClauses
    : RedimClause ( Comma RedimClause )*
    ;

RedimClause
    : Expression ArraySizeInitializationModifier
    ;
```

Each clause in the statement must be classified as a variable or a property access whose type is an array type or `Object`, and be followed by a list of array bounds. The number of the bounds must be consistent with the type of the variable; any number of bounds is allowed for `Object`. At run time, an array is instantiated for each expression from left to right with the specified bounds and then assigned to the variable or property. If the variable type is `Object`, the number of dimensions is the number of dimensions specified, and the array element type is `Object`. If the given number of dimensions is incompatible with the variable or property at run time a compile-time error occurs. For example:

```vb
Module Test
    Sub Main()
        Dim o As Object
        Dim b() As Byte
        Dim i(,) As Integer

        ' The next two statements are equivalent.
        ReDim o(10,30)
        o = New Object(10, 30) {}

        ' The next two statements are equivalent.
        ReDim b(10)
        b = New Byte(10) {}

        ' Error: Incorrect number of dimensions.
        ReDim i(10, 30, 40)
    End Sub
End Module
```

If the `Preserve` keyword is specified, then the expressions must also be classifiable as a value, and the new size for each dimension except for the rightmost one must be the same as the size of the existing array. The values in the existing array are copied into the new array: if the new array is smaller, the existing values are discarded; if the new array is bigger, the extra elements will be initialized to the default value of the element type of the array. For example, consider the following code:

```vb
Module Test
    Sub Main()
        Dim x(5, 5) As Integer

        x(3, 3) = 3

        ReDim Preserve x(5, 6)
        Console.WriteLine(x(3, 3) & ", " & x(3, 6))
    End Sub
End Module
```

It prints the following result:

```
3, 0
```

If the existing array reference is a null value at run time, no error is given. Other than the rightmost dimension, if the size of a dimension changes, a `System.ArrayTypeMismatchException` will be thrown.

__Note.__ `Preserve` is not a reserved word.


### Erase Statement

An `Erase` statement sets each of the array variables or properties specified in the statement to `Nothing`. Each expression in the statement must be classified as a variable or property access whose type is an array type or `Object`. For example:

```vb
Module Test
    Sub Main()
        Dim x() As Integer = New Integer(5) {}

        ' The following two statements are equivalent.
        Erase x
        x = Nothing
    End Sub
End Module
```

```antlr
EraseStatement
    : 'Erase' EraseExpressions StatementTerminator
    ;

EraseExpressions
    : Expression ( Comma Expression )*
    ;
```