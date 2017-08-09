###
* [Statements](../statements2.md)
  * [Methods](../Methods/Methods.md)
    * [Regular Methods](Regular_Methods.md)
    * [Async Methods](Async_Methods.md)
    * **Iterator Methods**

------

# Iterator Methods

Iterator methods are used as a convenient way to generate a sequence, one which can be consumed by the `For Each` statement. Iterator methods use the `Yield` statement (Section [Yield Statement](statements.md#yield-statement)) to provide elements of the sequence. (An iterator method with no `Yield` statements will produce an empty sequence). Here is an example of an iterator method:

```vb
Iterator Function Test() As IEnumerable(Of Integer)
    Console.WriteLine("hello")
    Yield 1
    Yield 2
End Function

Dim en = Test()
For Each x In en          ' prints "hello" before the first x
    Console.WriteLine(x)  ' prints "1" and then "2"
Next
```

When an iterator method is invoked whose return type is `IEnumerator(Of T)`,

1. First an instance of the iterator method is created specific to that invocation. This instance includes a copy of all parameters and local variables of the method.
2. Then all of its parameters are initialized to the supplied values, and all of its local variables to the default values of their types.
3. An implicit local variable is also initialized called the *iterator current variable*, whose type is `T` and whose initial value is the default of its type.
4. The method instance's control point is then set at the first statement of the method body.
5. An *iterator object* is then created, associated with this method instance. The iterator object implements the declared return type and has behavior as described below.
6. Control flow is then resumed *immediately* in the caller, and the result of the invocation is the iterator object. Note that this transfer is done without exiting the iterator method instance, and does not cause finally handlers to execute. The method instance is still referenced by the iterator object, and will not be garbage collected so long as there exists a live reference to the iterator object.

When the iterator object's `Current` property is accessed, the *current variable* of the invocation is returned.

When the iterator object's `MoveNext` method is invoked, the invocation does not create a new method instance. Instead the existing method instance is used (and its control point and local variables and parameters) - the instance that was created when the iterator method was first invoked. Control flow resumes execution at the control point of that method instance, and proceeds through the body of the iterator method as normal.

When the iterator object's `Dispose` method is invoked, again the existing method instance is used. Control flow resumes at the control point of that method instance, but then immediately behaves as if an `Exit Function` statement were the next operation.

The above descriptions of behavior for invocation of `MoveNext` or `Dispose` on an iterator object only apply if all previous invocations of `MoveNext` or `Dispose` on that iterator object have already returned to their callers. If they haven't, then the behavior is undefined.

When control flow exits the iterator method body normally -- through reaching the `End Function` that mark its end, or through an explicit `Return` or `Exit` statement -- it must have done so in the context of an invocation of `MoveNext` or `Dispose` function on an iterator object to resume the iterator method instance, and it will have been using the method instance that was created when the iterator method was first invoked. The control point of that instance is left at the `End Function` statement, and control flow resumes in the caller; and if it had been resumed by a call to `MoveNext` then the value `False` is returned to the caller.

When control flow exits the iterator method body through an unhandled exception, then the exception is propagated to the caller, which again will be either an invocation of `MoveNext` or of `Dispose`.

As for the other possible return types of an iterator function,

* When an iterator method is invoked whose return type is `IEnumerable(Of T)` for some `T`, an instance is first created -- specific to that invocation of the iterator method -- of all parameters in the method, and they are initialized with the supplied values. The result of the invocation is an an object which implements the return type. Should this object's `GetEnumerator` method be called, it creates an instance -- specific to that invocation of `GetEnumerator` -- of all parameters and local variables in the method. It initializes the parameters to the values already saved, and proceeds as for the iterator method above.
* When an iterator method is invoked whose return type is the non-generic interface `IEnumerator`, the behavior is exactly as for `IEnumerator(Of Object)`.
* When an iterator method is invoked whose return type is the non-generic interface `IEnumerable`, the behavior is exactly as for `IEnumerable(Of Object)`.
