## Conditional Statements

Conditional statements allow conditional execution of statements based on expressions evaluated at run time.

```antlr
ConditionalStatement
    : IfStatement
    | SelectStatement
    ;
```

### If...Then...Else Statements

An `If...Then...Else` statement is the basic conditional statement.

```antlr
IfStatement
    : BlockIfStatement
    | LineIfThenStatement
    ;

BlockIfStatement
    : 'If' BooleanExpression 'Then'? StatementTerminator
      Block?
      ElseIfStatement*
      ElseStatement?
      'End' 'If' StatementTerminator
    ;

ElseIfStatement
    : ElseIf BooleanExpression 'Then'? StatementTerminator
      Block?
    ;

ElseStatement
    : 'Else' StatementTerminator
      Block?
    ;

LineIfThenStatement
    : 'If' BooleanExpression 'Then' Statements ( 'Else' Statements )? StatementTerminator
    ;
```

Each expression in an `If...Then...Else` statement must be a Boolean expression, as per Section [Boolean Expressions](expressions.md#boolean-expressions). (Note: this does not require the expression to have Boolean type). If the expression in the `If` statement is true, the statements enclosed by the `If` block are executed. If the expression is false, each of the `ElseIf` expressions is evaluated. If one of the `ElseIf` expressions evaluates to true, the corresponding block is executed. If no expression evaluates to true and there is an `Else` block, the `Else` block is executed. Once a block finishes executing, execution passes to the end of the `If...Then...Else` statement.

The line version of the `If` statement has a single set of statements to be executed if the `If` expression is `True` and an optional set of statements to be executed if the expression is `False`. For example:

```vb
Module Test
    Sub Main()
        Dim a As Integer = 10
        Dim b As Integer = 20

        ' Block If statement.
        If a < b Then
            a = b
        Else
            b = a
        End If

        ' Line If statement
        If a < b Then a = b Else b = a
    End Sub
End Module
```

The line version of the If statement binds less tightly than ":", and its `Else` binds to the lexically nearest preceding `If` that is allowed by the syntax. For example, the following two versions are equivalent:

```vb
If True Then _
If True Then Console.WriteLine("a") Else Console.WriteLine("b") _
Else Console.WriteLine("c") : Console.WriteLine("d")

If True Then
    If True Then
        Console.WriteLine("a")
    Else
        Console.WriteLine("b")
    End If
    Console.WriteLine("c") : Console.WriteLine("d")
End If
```

All statements other than label declaration statements are allowed inside a line `If` statement, including block statements. However, they may not use LineTerminators as StatementTerminators except inside multi-line lambda expressions. For example:

```vb
' Allowed, since it uses : instead of LineTerminator to separate statements
If b Then With New String("a"(0),5) : Console.WriteLine(.Length) : End With

' Disallowed, since it uses a LineTerminator
If b then With New String("a"(0), 5)
              Console.WriteLine(.Length)
          End With

' Allowed, since it only uses LineTerminator inside a multi-line lambda
If b Then Call Sub()
                   Console.WriteLine("a")
               End Sub.Invoke()
```

### Select Case Statements

A `Select Case` statement executes statements based on the value of an expression.

```antlr
SelectStatement
    : 'Select' 'Case'? Expression StatementTerminator
      CaseStatement*
      CaseElseStatement?
      'End' 'Select' StatementTerminator
    ;

CaseStatement
    : 'Case' CaseClauses StatementTerminator
      Block?
    ;

CaseClauses
    : CaseClause ( Comma CaseClause )*
    ;

CaseClause
    : ( 'Is' LineTerminator? )? ComparisonOperator LineTerminator? Expression
    | Expression ( 'To' Expression )?
    ;

ComparisonOperator
    : '=' | '<' '>' | '<' | '>' | '>' '=' | '<' '='
    ;

CaseElseStatement
    : 'Case' 'Else' StatementTerminator
      Block?
    ;
```

The expression must be classified as a value. When a `Select Case` statement is executed, the `Select` expression is evaluated first, and the `Case` statements are then evaluated in order of textual declaration. The first `Case` statement that evaluates to `True` has its block executed. If no `Case` statement evaluates to `True` and there is a `Case Else` statement, that block is executed. Once a block has finished executing, execution passes to the end of the `Select` statement.

Execution of a `Case` block is not permitted to "fall through" to the next switch section. This prevents a common class of bugs that occur in other languages when a `Case` terminating statement is accidentally omitted. The following example illustrates this behavior:

```vb
Module Test
    Sub Main()
        Dim x As Integer = 10

        Select Case x
            Case 5
                Console.WriteLine("x = 5")
            Case 10
                Console.WriteLine("x = 10")
            Case 20 - 10
                Console.WriteLine("x = 20 - 10")
            Case 30
                Console.WriteLine("x = 30")
        End Select
    End Sub
End Module
```

The code prints:

```
x = 10
```

Although `Case 10` and `Case 20 - 10` select for the same value, `Case 10` is executed because it precedes `Case 20 - 10` textually. When the next `Case` is reached, execution continues after the `Select` statement.

A `Case` clause may take two forms. One form is an optional `Is` keyword, a comparison operator, and an expression. The expression is converted to the type of the `Select` expression; if the expression is not implicitly convertible to the type of the `Select` expression, a compile-time error occurs. If the `Select` expression is *E*, the comparison operator is *Op*, and the `Case` expression is *E1*, the case is evaluated as *E OP E1*. The operator must be valid for the types of the two expressions; otherwise a compile-time error occurs.

The other form is an expression optionally followed by the keyword `To` and a second expression. Both expressions are converted to the type of the `Select` expression; if either expression is not implicitly convertible to the type of the `Select` expression, a compile-time error occurs. If the `Select` expression is `E`, the first `Case` expression is `E1`, and the second `Case` expression is `E2`, the `Case` is evaluated either as `E = E1` (if no `E2` is specified) or `(E >= E1) And (E <= E2)`. The operators must be valid for the types of the two expressions; otherwise a compile-time error occurs.

