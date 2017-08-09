* [**Statements**](/spec/statements/statements.md)
  * [**Assignment Statements**](/spec/statements/Assignments/Assignment_Statements.md)
    * [Regular Assignment](/spec/statements/Assignments/Regular_Assignment_Statements.md)
    * [Compound Assignment](/spec/statements/Assignments/Compound_Assignment_Statements.md)  
    * [Mid Assignment Statement](/spec/statements/Assignments/Mid_Assignment_Statements.md)

-----

# Assignment Statements

An assignment statement assigns the value of an expression to a variable. There are several types of assignment.

```antlr
AssignmentStatement
    : RegularAssignmentStatement
    | CompoundAssignmentStatement
    | MidAssignmentStatement
    ;
```

