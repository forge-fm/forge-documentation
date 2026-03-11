# Testing

Forge supports several different testing constructs. Chiefly:

- `example`, for expressing that _specific instances_ should satisfy (or not satisfy) a predicate; 
- `assert`, for expressing that a predicate should be...
  - sufficient to satisfy another (`is sufficient for`); 
  - necessary for another to be satisfied (`is necessary for`);
  - is consistent with another, i.e., that the combination of the two predicates can be satisfied by at least one instance (`is consistent with`);
  - is satisfiable or unsatisfiable (`is sat` and `is unsat`).   

Assertions largely subsume an older testing form, `test expect` blocks, which are less structured and used mostly within Forge's internal test cases.

**Note on terminology**: we will sometimes refer to tests, particularly assertions of necessity and sufficiency, as _property tests_. This is because they can be used to express sweeping expectations about predicates, rather than just the point-wise, single-instance expectations that `example` can. We also sometimes refer to a test as a _test of exclusion_ (if it asserts that some set of instances shouldn't be admitted by a predicate) or _test of inclusion_ (to assert that some set of instances should be admitted, or that such an instance exists). 

!!! note "Shared Context"
    All the subsections below contain tests for the same small model of tic-tac-toe:

    ```
    abstract sig Player {}
    one sig X, O extends Player {}
    sig Board { board: pfunc (Int -> Int) -> Player }
    pred wellformed {
      all b: Board | all row, col: Int | {
        (row < 0 or row > 2 or col < 0 or col > 2) implies
          no b.board[row][col] 
      } 
    }
    ```


## Examples

The `example` syntax lets you test whether a specific instance is satisfied by some predicate in your model. An `example` contains three parts:
* a name for the example;
* which predicate the instance should satisfy (or not satisfy); and 
* the instance itself. 

!!! example "Example"

    This `example`, named `diagonalPasses`, expresses that a board where X has moved in the upper-left, middle, and lower-right squares (and `O` has made no moves at all) should satisfy the `wellformed` predicate:

    ```
    example diagonalPasses is {wellformed} for {
      Board = `Board0
      X = `X0 
      O = `O0
      A = `A0
      B = `B0
      C = `C0
      `Board0.board = (0,0)->`X + (1,1)->`X + (2,2)->`X
    }
    ```

    Notice that the `example` needs to give a value for _all_ `sig`s and _all_ fields, even the `one sig`s.


Examples are analogous to unit tests in Forge. Every example says that if the instance were passed to the predicate, the predicate would return true (or false). This means that examples are a great way to explore potential design choices and map out your intent when modeling.

!!! warning "Temporal Forge"
    The `example` keyword is not supported in Temporal Forge.


#### How do examples work?

An example passes if and only if the instance given is consistent with the predicate given. 

!!! warning "Total vs. partial instances"
    If you leave a sig or field unbound in an `example`, Forge is free to assign that sig or field in any way to achieve consistency with the predicate. The consequence is that it is possible to write apparently contradictory examples that pass. E.g., in the above example, if we left out the binding for `board`:

    ```
    example exampleYes is {wellformed} for {
      Board = `Board0
      X = `X0 
      O = `O0
      A = `A0
      B = `B0
      C = `C0  
    }
    example exampleNo is {not wellformed} for {
      Board = `Board0
      X = `X0 
      O = `O0
      A = `A0
      B = `B0
      C = `C0  
    }
    ```
    **Both** of these examples would pass vs. the `wellformed` predicate, because Forge can find values for the `board` field that either satisfy or dissatisfy the `wellformed` predicate.



### Notes on Example Syntax

The block within the second pair of braces must always be a concrete instance. That is, a series of assignments for each sig and field to some set of tuples, defined over atom names. Atom names must be preceded by a backquote; this reinforces the idea that they are atoms in a specific instance, rather than names in the model. You will not be able to refer to these atoms in predicates and most other Forge syntax.

_Don't_ try to assign to the same field twice. If you want a field to contain multiple entries, use `+` instead. Remember that `=` in the context of an instance is _assignment_, not a constraint, and that most constraints won't work inside an instance.

Names of `sig`s may be used on the right-hand-side of an assignment only if the block has previously defined the value of that `sig`` exactly, allowing straightforward substitution.

## Assert

The `assert` syntax allows you to write tests at a more abstract level than examples do. An assert has these parts:

  - an optional name, followed by a colon;
  - an optional block of `all`-quantified variables;
  - the keyword `assert`, followed by a predicate name or `{}`-delimited constraint block;
  - the assertion type: `is necessary for`, `is sufficient for`, `is consistent with`, `is sat`, or `is unsat`; and 
  - for `is necessary for`, `is sufficient for`, and `is consistent with` asserts, a predicate name. 

!!! example "Assertions"

    If we first define these two predicates:

    ```
    pred fullFirstRow {some b: Board | b.board[0][0] = X and b.board[0][1] = X and b.board[0][2] = X}
    pred someMoveTaken {some b: Board, row, col: Int | some b.board[row][col] }
    ```

    we can then write two assertions:

    ```
    assert fullFirstRow is sufficient for winning for 1 Board
    assert someMoveTaken is necessary for winning for 1 Board
    ```

    which should both pass, since:
    * if `X` occupied the entire first row, it has won; and 
    * if someone has won the game, there must be moves taken on the board.

    But surely we also wish to make sure that these don't pass only because `fullFirstRow` and `someMoveTaken` are unsatisfiable!

    ```
    assert fullFirstRow is sat
    assert someMoveTaken is sat
    ```

    or even better (and with names, too):

    ```
    nonvacuous_fullFirstRow: assert fullFirstRow is consistent with wellformed
    nonvacuous_someMoveTaken: assert someMoveTaken is consistent with wellformed
    ```


Assertions also support universal quantification (i.e. `all`, but not `some`, `one`, `lone`, etc). For example, if you instead wrote the predicates:

!!! example "AssertionsQuant"
    ```
    pred fullFirstRow[b : Board] {b.board[0][0] = X and b.board[0][1] = X and b.board[0][2] = X}
    pred someMoveTaken[b : Board, row : Int, col : Int] {some b.board[row][col] }
    ```

    You could write the assertions
    ```
    assert all b : Board | fullFirstRow[b] is sufficient for winning for 1 Board
    assert all b : Board, row, col : Int | someMoveTaken[b, row, col] is necessary for winning for 1 Board
    ```



Assertions are an excellent way to check and document your goals and assumptions about your model. In a more complex setting, we might write assertions that enforce:

* Dijkstra's algorithm doesn't terminate until the destination vertex has been reached; 
* for a game of chess to be won, a king must be in check; 
* someone must first be logged into Gmail to read their mail;
or confirm that:
* it is possible to generate a run of Dijkstra's algorithm in the model;
* it is possible to generate a chess configuration where the king is in check; or
* someone can log into Gmail to begin with. 

### Notes on Assert Syntax

The right-hand-side of the assertion must be a predicate name. That is, you *cannot* provide arbitrary formulas enclosed in brackets to an `assert`. This restriction eases some analysis but also encourages you to create reusable predicates.


## Suites: Organizing Your Tests

You should organize your tests into `test suite`s for each predicate you plan to test.

!!! example "Test Suite"
    For example, you could combine all the above forms into one suite for the `winning` predicate:

    ```
    test suite for winning {
      assert fullFirstRow is sufficient for winning for 1 Board
      assert someMoveTaken is necessary for winning for 1 Board

      test expect { possibleToWin_withoutFullFirstRow: {winning and not fullFirstRow} is sat } 

      example diagonalWin is {winning} for {
        Board = `Board0
        X = `X0 
        O = `O0
        A = `A0
        B = `B0
        C = `C0
        `Board0.board = (0,0)->`X + (1,1)->`X + (2,2)->`X
      }
    }   
    ```



## Test-Expect Blocks

We generally suggest using assertions rather than test-expect blocks if you can do so. 

Every `test expect` contains a set of individual checks. Each has:
* an optional test name;
* a predicate block; and 
* an intention (`is sat`, `is unsat`, `is checked`, or `is forge_error`). 

The meaning of each intention is:

* `is sat`: the predicate block is satisfiable under the given bounds; 
* `is unsat`: the predicate block is unsatisfiable under the given bounds; and 
* `is checked`: the predicate block's *negation* is unsatisfiable under the given bounds.
* `is forge_error`: the predicate block produces a forge error when run.

Like the other test forms, each test may be accompanied by numeric scopes and `inst` bounds.

!!! example "Test expect"

    This expresses that it's possible to satisfy the `someMoveTaken` predicate:
    ```
    test expect { possibleToMove: {someMoveTaken} is sat }  
    ```

