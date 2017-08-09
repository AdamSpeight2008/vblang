###
* [Statements](../statements2.md)
  * [Methods](../Methods/Methods.md)
    * [Regular Methods](Regular_Methods.md)
    * **Async Methods**
    * [Iterator Methods](Iterator_Methods.md)

------
### Async Methods

Async methods are a convenient way to do long-running work without for example blocking the UI of an application. Async is short for *Asynchronous* - it means that the caller of the async method will resume its execution promptly, but the eventual completion of the async method's instance may happen at some later time in the future. By convention async methods are named with the suffix "Async".

```vb
Async Function TestAsync() As Task(Of String)
    Console.WriteLine("hello")
    Await Task.Delay(100)
    Return "world"
End Function

Dim t = TestAsync()         ' prints "hello"
Console.WriteLine(Await t)  ' prints "world"
```

__Note.__ Async methods are *not* run on a background thread. Instead they allow a method to suspend itself through the `Await` operator, and schedule itself to be resumed in response to some event.

When an async method is invoked

1. First an instance of the async method is created specific to that invocation. This instance includes a copy of all parameters and local variables of the method.
2. Then all of its parameters are initialized to the supplied values, and all of its local variables to the default values of their types.
3. In the case of an async method with return type `Task(Of T)` for some `T`, an implicit local variable is also initialized called the *task return variable*, whose type is `T` and whose initial value is the default of `T`.
4. If the async method is a `Function` with return type `Task` or `Task(Of T)` for some `T`, then an object of that type implicitly created, associated with the current invocation. This is called an *async object* and represents the future work that will be done by executing the instance of the async method. When control resumes in the caller of this async method instance, the caller will receive this async object as the result of its invocation.
5. The instance's control point is then set at the first statement of the async method body, and immediately starts to execute the method body from there (Section [Blocks and Labels](statements.md#blocks-and-labels)).

__Resumption Delegate and Current Caller__

As detailed in Section [Await Operator](expressions.md#await-operator), execution of an `Await` expression has the ability to suspend the method instance's control point leaving control flow to go elsewhere. Control flow can later resume at the same instance's control point through invocation of a *resumption delegate*. Note that this suspension is done without exiting the async method, and does not cause finally handlers to execute. The method instance is still referenced by both the resumption delegate and the `Task` or `Task(Of T)` result (if any), and will not be garbage collected so long as there exists a live reference to either delegate or result.

It is helpful to imagine the statement `Dim x = Await WorkAsync()` approximately as syntactic shorthand for the following:

```vb
Dim temp = WorkAsync().GetAwaiter()
If Not temp.IsCompleted Then
       temp.OnCompleted(resumptionDelegate)
       Return [task]
       CONT:   ' invocation of 'resumptionDelegate' will resume here
End If
Dim x = temp.GetResult()
```

In the following, the *current caller* of the method instance is defined as either the original caller, or the caller of the resumption delegate, whichever is more recent.

When control flow exits the async method body -- through reaching the `End Sub` or `End Function` that mark its end, or through an explicit `Return` or `Exit` statement, or through an unhandled exception -- the instance's control point is set to the end of the method. Behavior then depends on the return type of the async method.

* In the case of an `Async Function` with return type `Task`:
  1. If control flow exits through an unhandled exception, then the async object's status is set to `TaskStatus.Faulted` and its `Exception.InnerException` property is set to the exception (except: certain implementation-defined exceptions such as `OperationCanceledException` change it to `TaskStatus.Canceled`). Control flow resumes in the current caller.
  2. Otherwise, the async object's status is set to `TaskStatus.Completed`. Control flow resumes in the current caller.

     (__Note.__ The whole point of Task, and what makes async methods interesting, is that when a task becomes Completed then any methods that were awaiting it will presently have their resumption delegates executed, i.e. they will become unblocked.)

* In the case of an `Async Function` with return type `Task(Of T)` for some `T`: the behavior is as above, except that in non-exception cases the async object's `Result` property is also set to the final value of the task return variable.

* In the case of an `Async Sub`:
  1. If control flow exits through an unhandled exception, then that exception is propagated to the environment in some implementation-specific manner. Control flow resumes in the current caller.
  2. Otherwise, control flow simply resumes in the current caller.

#### Async Sub

There is some Microsoft-specific behavior of an `Async Sub`.

If `SynchronizationContext.Current` is `Nothing` at the start of the invocation, then any unhandled exceptions from an Async Sub will be posted to the Threadpool.

If `SynchronizationContext.Current` is not `Nothing` at the start of the invocation, then `OperationStarted()` is invoked on that context before the start of the method and `OperationCompleted()` after the end. Additionally, any unhandled exceptions will be posted to be rethrown on the synchronization context.

This means that in UI applications, for an `Async Sub` that is invoked on the UI thread, any exceptions it fails to handle will be reposted the UI thread.

#### Mutable structures in async and iterator methods

Mutable structures in general are considered bad practice, and they are not supported by async or iterator methods. In particular, each invocation of an async or iterator method in a structure will implicitly operate on a *copy* of that structure that is copied at its moment of invocation. Thus, for example,

```vb
Structure S
       Dim x As Integer
       Async Sub Mutate()
           x = 2
       End Sub
End Structure

Dim s As New S With {.x = 1}
s.Mutate()
Console.WriteLine(s.x)   ' prints "1"
```
