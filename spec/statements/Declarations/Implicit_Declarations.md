### Implicit Local Declarations

In addition to local declaration statements, local variables can also be declared implicitly through use. A simple name expression that uses a name that does not resolve to something else declares a local variable by that name. For example:

```vb
Option Explicit Off

Module Test
    Sub Main()
        x = 10
        y = 20
        Console.WriteLine(x + y)
    End Sub
End Module
```

Implicit local declaration only occurs in expression contexts that can accept an expression classified as a variable. The exception to this rule is that a local variable may not be implicitly declared when it is the target of a function invocation expression, indexing expression, or a member access expression.

Implicit locals are treated as if they are declared at the beginning of the containing method. Thus, they are always scoped to the entire method body, even if declared inside of a lambda expression. For example, the following code:

```vb
Option Explicit Off 

Module Test
    Sub Main()
        Dim x = Sub()
                    a = 10
                End Sub
        Dim y = Sub()
                    Console.WriteLine(a)
                End Sub

        x()
        y()
    End Sub
End Module
```

will print the value `10`. Implicit locals are typed as `Object` if no type character was attached to the variable name; otherwise the type of the variable is the type of the type character. Local variable type inference is not used for implicit locals.

If explicit local declaration is specified by the compilation environment or by `Option Explicit`, all local variables must be explicitly declared and implicit variable declaration is disallowed.