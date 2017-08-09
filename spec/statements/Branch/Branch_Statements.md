
## Branch Statements

Branch statements modify the flow of execution in a method. There are six branch statements:

1. A `GoTo` statement causes execution to transfer to the specified label in the method. It is not allowed to `GoTo` into a `Try`, `Using`, `SyncLock`, `With`, `For` or `For Each` block, nor into any loop block if a local variable of that block is captured in a lambda or LINQ expression.
2. An `Exit` statement transfers execution to the next statement after the end of the immediately containing block statement of the specified kind. If the block is the method block, then control flow exits the method as described in Section [Control Flow](statements.md#control-flow). If the `Exit` statement is not contained within the kind of block specified in the statement, a compile-time error occurs.
3. A `Continue` statement transfers execution to the end of the immediately containing block loop statement of the specified kind. If the `Continue` statement is not contained within the kind of block specified in the statement, a compile-time error occurs.
4. A `Stop` statement causes a debugger exception to occur.
5. An `End` statement terminates the program. Finalizers are run before shutdown, but the finally blocks of any currently executing `Try` statements are not executed. This statement may not be used in programs that are not executable (for example, DLLs).
6. A `Return` statement with no expression is equivalent to an `Exit Sub` or `Exit Function` statement. A `Return` statement with an expression is only allowed in a regular method that is a function, or in an async method that is a function with return type `Task(Of T)` for some `T`. The expression must be classified as a value which is implicitly convertible to the *function return variable* (in the case of regular methods) or to the *task return variable* (in the case of async methods). Its behavior is to evaluate its expression, then store it in the return variable, then execute an implicit `Exit Function` statement.

```antlr
BranchStatement
    : GoToStatement
    | ExitStatement
    | ContinueStatement
    | StopStatement
    | EndStatement
    | ReturnStatement
    ;

GoToStatement
    : 'GoTo' LabelName StatementTerminator
    ;

ExitStatement
    : 'Exit' ExitKind StatementTerminator
    ;

ExitKind
    : 'Do' | 'For' | 'While' | 'Select' | 'Sub' | 'Function' | 'Property' | 'Try'
    ;

ContinueStatement
    : 'Continue' ContinueKind StatementTerminator
    ;

ContinueKind
    : 'Do' | 'For' | 'While'
    ;

StopStatement
    : 'Stop' StatementTerminator
    ;

EndStatement
    : 'End' StatementTerminator
    ;

ReturnStatement
    : 'Return' Expression? StatementTerminator
    ;
```
