### Blocks and Labels

A group of executable statements is called a statement block. Execution of a statement block begins with the first statement in the block. Once a statement has been executed, the next statement in lexical order is executed, unless a statement transfers execution elsewhere or an exception occurs.

Within a statement block, the division of statements on logical lines is not significant with the exception of label declaration statements. A label is an identifier that identifies a particular position within the statement block that can be used as the target of a branch statement such as `GoTo`.

```antlr
Block
    : Statements*
    ;

LabelDeclarationStatement
    : LabelName ':'
    ;

LabelName
    : Identifier
    | IntLiteral
    ;

Statements
    : Statement? ( ':' Statement? )*
    ;
```


Label declaration statements must appear at the beginning of a logical line and labels may be either an identifier or an integer literal. Because both label declaration statements and invocation statements can consist of a single identifier, a single identifier at the beginning of a local line is always considered a label declaration statement. Label declaration statements must always be followed by a colon, even if no statements follow on the same logical line.

Labels have their own declaration space and do not interfere with other identifiers. The following example is valid and uses the name variable `x` both as a parameter and as a label.

```vb
Function F(x As Integer) As Integer
    If x >= 0 Then
        GoTo x
    End If
    x = -x
x: 
    Return x
End Function
```

The scope of a label is the body of the method containing it.

For the sake of readability, statement productions that involve multiple substatements are treated as a single production in this specification, even though the substatements may each be by themselves on a labeled line.

