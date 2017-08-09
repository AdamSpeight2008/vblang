# Local Declaration Statements

A local declaration statement declares a new local variable, local constant, or static variable. *Local variables* and *local constants* are equivalent to instance variables and constants scoped to the method and are declared in the same way. *Static variables* are similar to `Shared` variables and are declared using the `Static` modifier.

```antlr
LocalDeclarationStatement
    : LocalModifier VariableDeclarators StatementTerminator
    ;

LocalModifier
    : 'Static' | 'Dim' | 'Const'
    ;
```

Static variables are locals that retain their value across invocations of the method. Static variables declared within non-shared methods are per instance: each instance of the type that contains the method has its own copy of the static variable. Static variables declared within `Shared` methods are per type; there is only one copy of the static variable for all instances. While local variables are initialized to their type's default value upon each entry into the method, static variables are only initialized to their type's default value when the type or type instance is initialized. Static variables may not be declared in structures or generic methods.

Local variables, local constants, and static variables always have public accessibility and may not specify accessibility modifiers. If no type is specified on a local declaration statement, then the following steps determine the type of the local declaration:

1. If the declaration has a type character, the type of the type character is the type of the local declaration.

2. If the local declaration is a local constant, or if the local declaration is a local variable with an initializer and local variable type inference is being used, the type of the local declaration is inferred from the type of the initializer. If the initializer refers to the local declaration, a compile-time error occurs. (Local constants are required to have initializers.)

3. If strict semantics are not being used, the type of the local declaration statement is implicitly `Object`.

4. Otherwise, a compile-time error occurs.

If no type is specified on a local declaration statement that has an array size or array type modifier, then the type of the local declaration is an array with the specified rank and the previous steps are used to determine the element type of the array. If local variable type inference is used, the type of the initializer must be an array type with the same array shape (i.e. array type modifiers) as the local declaration statement. Note that it is possible that the inferred element type may still be an array type. For example:

```vb
Option Infer On

Module Test
    Sub Main()
        ' Error: initializer is not an array type
        Dim x() = 1

        ' Type is Integer()
        Dim y() = New Integer() {}

        ' Type is Integer()()
        Dim z() = New Integer()() {}

        ' Type is Integer()()()

        Dim a()() = New Integer()()() {}

        ' Error: initializer does not have same array shape
        Dim b()() = New Integer(,)() {}
    End Sub
End Module
```

If no type is specified on a local declaration statement that has a nullable type modifier, then the type of the local declaration is the nullable version of the inferred type or the inferred type itself if it a nullable value type already.  If the inferred type is not a value type that can be made nullable, a compile-time error occurs. If both a nullable type modifier and an array size or array type modifier are placed on a local declaration statement with no type, then the nullable type modifier is considered to apply to the element type of the array and the previous steps are used to determine the element type.

Variable initializers on local declaration statements are equivalent to assignment statements placed at the textual location of the declaration. Thus, if execution branches over the local declaration statement, the variable initializer is not executed. If the local declaration statement is executed more than once, the variable initializer is executed an equal number of times. Static variables only execute their initializer the first time. If an exception occurs while initializing a static variable, the static variable is considered initialized with the default value of the static variable's type.

The following example shows the use of initializers:

```vb
Module Test
    Sub F()
        Static x As Integer = 5

        Console.WriteLine("Static variable x = " & x)
        x += 1
    End Sub

    Sub Main()
        Dim i As Integer

        For i = 1 to 3
            F()
        Next i

        i = 3
label:
        Dim y As Integer = 8

        If i > 0 Then
            Console.WriteLine("Local variable y = " & y)
            y -= 1
            i -= 1
            GoTo label
        End If
    End Sub
End Module
```

This program prints:

```
Static variable x = 5
Static variable x = 6
Static variable x = 7
Local variable y = 8
Local variable y = 8
Local variable y = 8
```

Initializers on static locals are thread-safe and protected against exceptions during initialization. If an exception occurs during a static local initializer, the static local will have its default value and not be initialized. A static local initializer

```vb
Module Test
    Sub F()
        Static x As Integer = 5
    End Sub
End Module
```

is equivalent to

```vb
Imports System.Threading
Imports Microsoft.VisualBasic.CompilerServices

Module Test
    Class InitFlag
        Public State As Short
    End Class

    Private xInitFlag As InitFlag = New InitFlag()

    Sub F()
        Dim x As Integer

        If xInitFlag.State <> 1 Then
            Monitor.Enter(xInitFlag)
            Try
                If xInitFlag.State = 0 Then
                    xInitFlag.State = 2
                    x = 5
                Else If xInitFlag.State = 2 Then
                    Throw New IncompleteInitialization()
                End If
            Finally
                xInitFlag.State = 1
                Monitor.Exit(xInitFlag)
            End Try
        End If
    End Sub
End Module
```

Local variables, local constants, and static variables are scoped to the statement block in which they are declared. Static variables are special in that their names may only be used once throughout the entire method. For example, it is not valid to specify two static variable declarations with the same name even if they are in different blocks.
