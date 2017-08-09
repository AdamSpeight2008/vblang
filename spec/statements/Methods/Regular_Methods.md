* [Statements](../statements2.md)
  * [Methods](../Methods/Methods.md)
    * **Regular Methods**
    * [Async Methods](Async_Methods.md)
    * [Iterator Methods](Iterator_Methods.md)

------

# Regular Methods

Here is an example of a regular method

```vb
Function Test() As Integer
    Console.WriteLine("hello")
    Return 1
End Sub

Dim x = Test()    ' invokes the function, prints "hello", assigns 1 to x
```

When a regular method is invoked,

1. First an instance of the method is created specific to that invocation. This instance includes a copy of all parameters and local variables of the method.
2. Then all of its parameters are initialized to the supplied values, and all of its local variables to the default values of their types.
3. In the case of a `Function`, an implicit local variable is also initialized called the *function return variable* whose name is the function's name, whose type is the return type of the function and whose initial value is the default of its type.
4. The method instance's control point is then set at the first statement of the method body, and the method body immediately starts to execute from there (Section [Blocks and Labels](statements.md#blocks-and-labels)).

When control flow exits the method body normally - through reaching the `End Function` or `End Sub` that mark its end, or through an explicit `Return` or `Exit` statement - control flow returns to the caller of the method instance. If there is a function return variable, the result of the invocation is the final value of this variable.

When control flow exits the method body through an unhandled exception, that exception is propagated to the caller.

After control flow has exited, there are no longer any live references to the method instance. If the method instance held the only references to its copy of local variables or parameters, then they may be garbage collected.
