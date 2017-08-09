# Statements

Statements represent executable code.

```antlr
Statement ::= 
   LabelDeclarationStatement
 | LocalDeclarationStatement
 | WithStatement
 | SyncLockStatement
 | EventStatement
 | AssignmentStatement
 | InvocationStatement
 | ConditionalStatement
 | LoopStatement
 | ErrorHandlingStatement
 | BranchStatement
 | ArrayHandlingStatement
 | UsingStatement
 | AwaitStatement
 | YieldStatement
```

__Note.__ The Microsoft Visual Basic Compiler only allows statements which start with a keyword or an identifier. Thus, for instance, the invocation statement "`Call (Console).WriteLine`" is allowed, but the invocation statement "`(Console).WriteLine`" is not.

-------

## Control Flow

*Control flow* is the sequence in which statements and expressions are executed. The order of execution depends on the particular statement or expression.

For example, when evaluating an addition operator (Section [Addition Operator](expressions.md#addition-operator)), first the left operand is evaluated, then the right operand, and then the operator itself. Blocks are executed (Section [Blocks and Labels](statements.md#blocks-and-labels)) by first executing their first substatement, and then proceeding one by one through the statements of the block.

Implicit in this ordering is the concept of a *control point*, which is the next operation to be executed. When a method is invoked (or "called"), we say it creates an *instance* of the method. A method instance consists of its own copy of the method's parameters and local variables, and its own control point.

-----

* [Methods](/statements/Methods.md)
  * [Reqular Methods](/statements/methods/Regular_Methods.md)    
  * [Iterator Methods](/statements/methods/Iterator_Methods.md)    
  * [Async Methods](/statements/methods/Async_Methods.md)
* [Blocks and Labels](/statements/Blocks_Labels.md)
* [Local Declaration Statements](/statements/Declarations/Local.md)
  * [Implicit Local Declarations](/statements/Declarations/Implicit.md)
* [SyncLock Statement](/statements/SyncLock.md)
* [Event Statements](/statements/Events/Event_Statements.md)
  * RaiseEvent Statement
  * AddHandler and RemoveHandler Statements
* [Assignment Statments](/statements/Assignment/Assignment_Statements.md)
  * Regular Assignment Statements
  * Compound Assignment Statements
  * Mid Assignment Statement
* [Invocation Statements](/statements/Invocation/Invocation_Statments.md)
* [Conditional Statements](/statements/Conditional/Conditional_Statements.md)
  * If...Then...Else Statements
  * Select Case Statements
* [Loop Statements](statements/Loop/Loop_Statements.md)
  * While...End While and Do...Loop Statements
  * For Each...Next Statements
* [Exception-Handling Statements](/statements/Exception_Handling/Exception_Handling_Statements.md)
  * Structured Exception-Handling Statements
    * Finally Blocks
    * Catch Blocks
    * Throw Statement
  * Unstructured Error Handling Statements
    * Error Statement
    * On Error Statement
    * Resume Statement
* [Branch Statements](/statement/Branch/Branch_Statements.md)
* [Array-Handling Statements](/statements/Array_Handling/Array_Handling_Statements.md)
  * ReDim Statement
  * Erase Statement
* [Using Statements](/statements/Using/Using_Statements.md)
* [Await Statements](/statements/Await/Await_Statements.md)
* [Yield Statements](/statements/Yield/Yield_Statements.md)
