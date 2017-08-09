* [**Statements**](/spec/statements/statements.md)
  * [**Assignment Statements**](/spec/statements/Assignments/Assignment_Statements.md)
    * [Regular Assignment](/spec/statements/Assignments/Regular_Assignment_Statements.md)
    * [Compound Assignment](/spec/statements/Assignments/Compound_Assignment_Statements.md)  
    * [**Mid Assignment Statement**](/spec/statements/Assignments/Mid_Assignment_Statements.md)
-----
# Mid Assignment Statement

A `Mid` assignment statement assigns a string into another string. The left side of the assignment has the same syntax as a call to the function `Microsoft.VisualBasic.Strings.Mid`.

```antlr
MidAssignmentStatement
    : 'Mid' '$'? OpenParenthesis Expression Comma Expression
      ( Comma Expression )? CloseParenthesis Equals Expression StatementTerminator
    ;
```

The first argument is the target of the assignment and must be classified as a variable or a property access whose type is implicitly convertible to and from `String`. The second parameter is the 1-based start position that corresponds to where the assignment should begin in the target string and must be classified as a value whose type must be implicitly convertible to `Integer`. The optional third parameter is the number of characters from the right-side value to assign into the target string and must be classified as a value whose type is implicitly convertible to `Integer`. The right side is the source string and must be classified as a value whose type is implicitly convertible to `String`. The right side is truncated to the length parameter, if specified, and replaces the characters in the left-side string, starting at the start position. If the right side string contained fewer characters than the third parameter, only the characters from the right side string will be copied.

The following example displays `ab123fg`:

```vb
Module Test
    Sub Main()
        Dim s1 As String = "abcdefg"
        Dim s2 As String = "1234567"

        Mid$(s1, 3, 3) = s2
        Console.WriteLine(s1)
    End Sub
End Module
```

__Note.__ `Mid` is not a reserved word.
