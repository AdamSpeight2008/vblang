* [**Statements**](/spec/statements/statements.md)
  * [**Assignment Statements**](/spec/statements/Assignments/Assignment_Statements.md)
    * [Regular Assignment](/spec/statements/Assignments/Regular_Assignment_Statements.md)
    * [**Compound Assignment**](/spec/statements/Assignments/Compound_Assignment_Statements.md)  
    * [Mid Assignment Statement](/spec/statements/Assignments/Mid_Assignment_Statements.md)
-----
# Compound Assignment Statements

A *compound assignment statement* takes the form `V op= E` (where `op` is a valid binary operator).

```antlr
CompoundAssignmentStatement
    : Expression CompoundBinaryOperator LineTerminator? Expression StatementTerminator
    ;

CompoundBinaryOperator
    : '^' '=' | '*' '=' | '/' '=' | '\\' '=' | '+' '=' | '-' '='
    | '&' '=' | '<' '<' '=' | '>' '>' '='
    ;
```

The expression on the left side of the assignment operator must be classified as a variable or property access, while the expression on the right side of the assignment operator must be classified as a value. The compound assignment statement is equivalent to the statement `V = V op E` with the difference that the variable on the left side of the compound assignment operator is only evaluated once. The following example demonstrates this difference:

```vb
Module Test
    Function GetIndex() As Integer
        Console.WriteLine("Getting index")
        Return 1
    End Function

    Sub Main()
        Dim a(2) As Integer

        Console.WriteLine("Simple assignment")
        a(GetIndex()) = a(GetIndex()) + 1

        Console.WriteLine("Compound assignment")
        a(GetIndex()) += 1
    End Sub
End Module
```

The expression `a(GetIndex())` is evaluated twice for simple assignment but only once for compound assignment, so the code prints:

```
Simple assignment
Getting index
Getting index
Compound assignment
Getting index
```