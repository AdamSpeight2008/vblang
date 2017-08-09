
## Loop Statements

Loop statements allow repeated execution of the statements in their body.

```antlr
LoopStatement
    : WhileStatement
    | DoLoopStatement
    | ForStatement
    | ForEachStatement
    ;
```

Each time a loop body is entered, a fresh copy is made of all local variables declared in that body, initialized to the previous values of the variables. Any reference to a variable within the loop body will use the most recently made copy. This code shows an example:

```vb
Module Test
    Sub Main()
        Dim lambdas As New List(Of Action)
        Dim x = 1

        For i = 1 To 3
            x = i
            Dim y = x
            lambdas.Add(Sub() Console.WriteLine(x & y))
        Next

        For Each lambda In lambdas
            lambda()
        Next
    End Sub
End Module
```

The code produces the output:

```
31    32    33
```

When the loop body is executed, it uses whichever copy of the variable is current. For example, the statement  `Dim y = x` refers to the latest copy of `y` and the original copy of `x`. And when a lambda is created, it remembers whichever copy of a variable was current at the time it was created. Therefore, each lambda uses the same shared copy of `x`, but a different copy of `y`. At the end of the program, when it executes the lambdas, that shared copy of `x` that they all refer to is now at its final value 3.

Note that if there are no lambdas or LINQ expressions, then it's impossible to know that a fresh copy is made on loop entry. Indeed, compiler optimizations will avoid making copies in this case. Note too that it's illegal to `GoTo` into a loop that contains lambdas or LINQ expressions.


### While...End While and Do...Loop Statements

A `While` or `Do` loop statement loops based on a Boolean expression.

```antlr
WhileStatement
    : 'While' BooleanExpression StatementTerminator
      Block?
      'End' 'While' StatementTerminator
    ;

DoLoopStatement
    : DoTopLoopStatement
    | DoBottomLoopStatement
    ;

DoTopLoopStatement
    : 'Do' ( WhileOrUntil BooleanExpression )? StatementTerminator
      Block?
      'Loop' StatementTerminator
    ;

DoBottomLoopStatement
    : 'Do' StatementTerminator
      Block?
      'Loop' WhileOrUntil BooleanExpression StatementTerminator
    ;

WhileOrUntil
    : 'While' | 'Until'
    ;
```

A `While` loop statement loops as long as the Boolean expression evaluates to true; a `Do` loop statement may contain a more complex condition. An expression may be placed after the `Do` keyword or after the `Loop` keyword, but not after both. The Boolean expression is evaluated as per Section [Boolean Expressions](expressions.md#boolean-expressions). (Note: this does not require the expression to have Boolean type). It is also valid to specify no expression at all; in that case, the loop will never exit. If the expression is placed after `Do`, it will be evaluated before the loop block is executed on each iteration. If the expression is placed after `Loop`, it will be evaluated after the loop block has executed on each iteration. Placing the expression after `Loop` will therefore generate one more loop than placement after `Do`. The following example demonstrates this behavior:

```vb
Module Test
    Sub Main()
        Dim x As Integer

        x = 3
        Do While x = 1
            Console.WriteLine("First loop")
        Loop

        Do
            Console.WriteLine("Second loop")
        Loop While x = 1
    End Sub
End Module
```

The code produces the output:

```
Second Loop
```

In the case of the first loop, the condition is evaluated before the loop executes. In the case of the second loop, the condition is executed after the loop executes. The conditional expression must be prefixed with either a `While` keyword or an `Until` keyword. The former breaks the loop if the condition evaluates to false, the latter when the condition evaluates to true.

__Note.__ `Until` is not a reserved word.


### For...Next Statements

A `For...Next` statement loops based on a set of bounds. A `For` statement specifies a loop control variable, a lower bound expression, an upper bound expression, and an optional step value expression. The loop control variable is specified either through an identifier followed by an optional `As` clause or an expression.

```antlr
ForStatement
    : 'For' LoopControlVariable Equals Expression 'To' Expression
      ( 'Step' Expression )? StatementTerminator
      Block?
      ( 'Next' NextExpressionList? StatementTerminator )?
    ;

LoopControlVariable
    : Identifier ( IdentifierModifiers 'As' TypeName )?
    | Expression
    ;

NextExpressionList
    : Expression ( Comma Expression )*
    ;
```

As per the following rules, the loop control variable refers either to a new local variable specific to this `For...Next` statement, or to a pre-existing variable, or to an expression.

* If the loop control variable is an identifier with an `As` clause, the identifier defines a new local variable of the type specified in the `As` clause, scoped to the entire `For` loop.

* If the loop control variable is an identifier without an `As` clause, then the identifier is first resolved using the simple name resolution rules (see Section [Simple Name Expressions](expressions.md#simple-name-expressions)), excepting that this occurrence of the identifier would not in and of itself cause an implicit local variable to be created (Section [Implicit Local Declarations](statements.md#implicit-local-declarations)).

  * If this resolution succeeds and the result is classified as a variable, then the loop control variable is that pre-existing variable.

  * If resolution fails, or if resolution succeeds and the result is classified as a type, then:
    * if local variable type inference is being used, the identifier defines a new local variable whose type is inferred from the bound and step expressions, scoped to the entire `For` loop;
    * if local variable type inference is not being used but implicit local declaration is, then an implicit local variable is created whose scope is the entire method (Section [Implicit Local Declarations](statements.md#implicit-local-declarations)), and the loop control variable refers to this pre-existing variable;
    * if neither local variable type inference nor implicit local declarations are used, it is an error.

  * If resolution succeeds with something classified as neither a type nor a variable, it is an error.

* If the loop control variable is an expression, the expression must be classified as a variable.

A loop control variable cannot be used by another enclosing `For...Next` statement. The type of the loop control variable of a `For` statement determines the type of the iteration and must be one of:

* `Byte`, `SByte`, `UShort`, `Short`, `UInteger`, `Integer`, `ULong`, `Long`, `Decimal`, `Single`, `Double`
* An enumerated type
* `Object`
* A type `T` that has the following operators, where `B` is a type that can be used in a Boolean expression:

```vb
Public Shared Operator >= (op1 As T, op2 As T) As B
Public Shared Operator <= (op1 As T, op2 As T) As B
Public Shared Operator - (op1 As T, op2 As T) As T
Public Shared Operator + (op1 As T, op2 As T) As T
```

The bound and step expressions must be implicitly convertible to the type of the loop control variable and must be classified as values. At compile time, the type of the loop control variable is inferred by choosing the widest type among the lower bound, upper bound, and step expression types. If there is no widening conversion between two of the types, then a compile-time error occurs.

At run time, if the type of the loop control variable is `Object`, then the type of the iteration is inferred the same as at compile time, with two exceptions. First, if the bound and step expressions are all of integral types but have no widest type, then the widest type that encompasses all three types will be inferred. And second, if the type of the loop control variable is inferred to be `String`, `Double` will be inferred instead. If, at run time, no loop control type can be determined or if any of the expressions cannot be converted to the loop control type, a `System.InvalidCastException` will occur. Once a loop control type has been chosen at the beginning of the loop, the same type will be used throughout the iteration, regardless of changes made to the value in the loop control variable.

A `For` statement must be closed by a matching `Next` statement. A `Next` statement without a variable matches the innermost open `For` statement, while a `Next` statement with one or more loop control variables will, from left to right, match the `For` loops that match each variable. If a variable matches a `For` loop that is not the most nested loop at that point, a compile-time error results.

At the beginning of the loop, the three expressions are evaluated in textual order and the lower bound expression is assigned to the loop control variable. If the step value is omitted, it is implicitly the literal `1`, converted to the type of the loop control variable. The three expressions are only ever evaluated at the beginning of the loop.

At the beginning of each loop, the control variable is compared to see if it is greater than the end point if the step expression is positive, or less than the end point if the step expression is negative. If it is, the `For` loop terminates; otherwise the loop block executes. If the loop control variable is not a primitive type, the comparison operator is determined by whether the expression `step >= step - step` is true or false. At the `Next` statement, the step value is added to the control variable and execution returns to the top of the loop.

Note that a new copy of the loop control variable is *not* created on each iteration of the loop block. In this respect, the `For` statement differs from `For Each` (Section [For Each...Next Statements](statements.md#for-eachnext-statements)).

It is not valid to branch into a `For` loop from outside the loop.


### For Each...Next Statements

A `For Each...Next` statement loops based on the elements in an expression. A `For Each` statement specifies a loop control variable and an enumerator expression. The loop control variable is specified either through an identifier followed by an optional `As` clause or an expression.

```antlr
ForEachStatement
    : 'For' 'Each' LoopControlVariable 'In' LineTerminator? Expression StatementTerminator
      Block?
      ( 'Next' NextExpressionList? StatementTerminator )?
    ;
```

Following the same rules as `For...Next` statements (Section [For...Next Statements](statements.md#fornext-statements)), the loop control variable refers either to a new local variable specific to this For Each...Next statement, or to a pre-existing variable, or to an expression.

The enumerator expression must be classified as a value and its type must be a collection type or `Object`. If the type of the enumerator expression is `Object`, then all processing is deferred until run-time. Otherwise, a conversion must exist from the element type of the collection to the type of the loop control variable.

The loop control variable cannot be used by another enclosing `For Each` statement. A `For Each` statement must be closed by a matching `Next` statement. A `Next` statement without a loop control variable matches the innermost open `For Each`. A `Next` statement with one or more loop control variables will, from left to right, match the `For Each` loops that have the same loop control variable. If a variable matches a `For Each` loop that is not the most nested loop at that point, a compile-time error occurs.

A type `C` is said to be a *collection type* if one of:

* All of the following are true:
  * `C` contains an accessible instance, shared or extension method with the signature `GetEnumerator()` that returns a type `E`.
  * `E` contains an accessible instance, shared or extension method with the signature `MoveNext()` and the return type `Boolean`.
  * `E` contains an accessible instance or shared property named `Current` that has a getter. The type of this property is the element type of the collection type.

* It implements the interface `System.Collections.Generic.IEnumerable(Of T)`, in which case the element type of the collection is considered to be `T`.

* It implements the interface `System.Collections.IEnumerable`, in which case the element type of the collection is considered to be `Object`.

Following is an example of a class that can be enumerated:

```vb
Public Class IntegerCollection
    Private integers(10) As Integer

    Public Class IntegerCollectionEnumerator
        Private collection As IntegerCollection
        Private index As Integer = -1

        Friend Sub New(c As IntegerCollection)
            collection = c
        End Sub

        Public Function MoveNext() As Boolean
            index += 1

            Return index <= 10
        End Function

        Public ReadOnly Property Current As Integer
            Get
                If index < 0 OrElse index > 10 Then
                    Throw New System.InvalidOperationException()
                End If

                Return collection.integers(index)
            End Get
        End Property
    End Class

    Public Sub New()
        Dim i As Integer

        For i = 0 To 10
            integers(i) = I
        Next i
    End Sub

    Public Function GetEnumerator() As IntegerCollectionEnumerator
        Return New IntegerCollectionEnumerator(Me)
    End Function
End Class
```

Before the loop begins, the enumerator expression is evaluated. If the type of the expression does not satisfy the design pattern, then the expression is cast to `System.Collections.IEnumerable` or `System.Collections.Generic.IEnumerable(Of T)`. If the expression type implements the generic interface, the generic interface is preferred at compile-time but the non-generic interface is preferred at run-time. If the expression type implements the generic interface multiple times, the statement is considered ambiguous and a compile-time error occurs.

__Note.__ The non-generic interface is preferred in the late bound case, because picking the generic interface would mean that all the calls to the interface methods would involve type parameters. Since it is not possible to know the matching type arguments at run-time, all such calls would have to be made using late-bound calls. This would be slower than calling the non-generic interface because the non-generic interface could be called using compile-time calls.

`GetEnumerator` is called on the resulting value and the return value of the function is stored in a temporary. Then at the beginning of each iteration, `MoveNext` is called on the temporary. If it returns `False`, the loop terminates. Otherwise, each iteration of the loop is executed as follows:

1. If the loop control variable identified a new local variable (rather than a pre-existing one), then a fresh copy of this local variable is created. For the current iteration, all references within the loop block will refer to this copy.
2. The `Current` property is retrieved, coerced to the type of the loop control variable (regardless of whether the conversion is implicit or explicit), and assigned to the loop control variable.
3. The loop block executes.

__Note.__ There is a slight change in behavior between version 10.0 and 11.0 of the language. Prior to 11.0, a fresh iteration variable was *not* created for each iteration of the loop. This difference is observable only if the iteration variable is captured by a lambda or a LINQ expression which is then invoked after the loop:

```vb
Dim lambdas As New List(Of Action)
For Each x In {1,2,3}
   lambdas.Add(Sub() Console.WriteLine(x)
Next
lambdas(0).Invoke()
lambdas(1).Invoke()
lambdas(2).Invoke()
```

Up to Visual Basic 10.0, this produced a warning at compile-time and printed "3" three times. That was because there was only a single variable "x" shared by all iterations of the loop, and all three lambdas captured the same "x", and by the time the lambdas were executed it then held the number 3. As of Visual Basic 11.0, it prints "1, 2, 3". That is because each lambda captures a different variable "x".


__Note.__ The current element of the iteration is converted to the type of the loop control variable even if the conversion is explicit because there is no convenient place to introduce a conversion operator in the statement. This became particularly troublesome when working with the now-obsolete type `System.Collections.ArrayList`, because its element type is `Object`. This would have required casts in a great many loops, something we felt was not ideal. Ironically, generics enabled the creation of a strongly-typed collection, `System.Collections.Generic.List(Of T)`, which might have made us rethink this design point, but for compatibility's sake, this cannot be changed now.


When the `Next` statement is reached, execution returns to the top of the loop. If a variable is specified after the `Next` keyword, it must be the same as the first variable after the `For Each`. For example, consider the following code:

```vb
Module Test
    Sub Main()
        Dim i As Integer
        Dim c As IntegerCollection = New IntegerCollection()

        For Each i In c
            Console.WriteLine(i)
        Next i
    End Sub
End Module
```

It is equivalent to the following code:

```vb
Module Test
    Sub Main()
        Dim i As Integer
        Dim c As IntegerCollection = New IntegerCollection()

        Dim e As IntegerCollection.IntegerCollectionEnumerator

        e = c.GetEnumerator()
        While e.MoveNext()
            i = e.Current

            Console.WriteLine(i)
        End While
    End Sub
End Module
```

If the type `E` of the enumerator implements `System.IDisposable`, then the enumerator is disposed upon exiting the loop by calling the `Dispose` method. This ensures that resources held by the enumerator are released. If the method containing the `For Each` statement does not use unstructured error handling, then the `For Each` statement is wrapped in a `Try` statement with the `Dispose` method called in the `Finally` to ensure cleanup.

__Note.__ The `System.Array` type is a collection type, and since all array types derive from `System.Array`, any array type expression is permitted in a `For Each` statement. For single-dimensional arrays, the `For Each` statement enumerates the array elements in increasing index order, starting with index 0 and ending with index Length - 1. For multidimensional arrays, the indices of the rightmost dimension are increased first.

For example, the following code prints `1 2 3 4`:

```vb
Module Test
    Sub Main()
        Dim x(,) As Integer = { { 1, 2 }, { 3, 4 } }
        Dim i As Integer

        For Each i In x
            Console.Write(i & " ")
        Next i
    End Sub
End Module
```

It is not valid to branch into a `For Each` statement block from outside the block.
