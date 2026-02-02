# Glossary

This glossary is meant to cover terms that aren't easily searched for via the search bar. E.g., it lists "arity" but not specific Forge language constructs like `pred` or `sig`. 

## Terms


The _arity_ of a relation is the number of columns in the relation; that is, the number of elements of a tuple in that relation. 

An _atom_ is a distinct object within an instance. 

An [_instance_]({{ docs_url }}/building-models/overview) is a concrete scenario that abides by the rules of a model, containing specific _atoms_ and their relationships to each other.

A [_model_]({{ docs_url }}/building-models/overview) is a representation of a system. In Forge, a model comprises a set of `sig` and `field` definitions, along with constraints.


## Glossary of Errors 

This section contains tips related to some error terminology in Forge. 

### Errors Related to Bounds

#### Please specify an upper bound for ancestors of ...

If `A` is a `sig`, and you get an error that says "Please specify an upper bound for ancestors of A", this means that, while you've defined the contents of `A`, Forge cannot infer corresponding contents for the parent `sig` of `A` and needs you to provide a binding. 


!!! example "Example"
    Given this definition:
    ```
    sig Course {}
    sig Intro, Intermediate, UpperLevel extends Course {} 
    ```
    and this example:
    ```
    example someIntro is {wellformed} for {
        Intro = `CSCI0150
    }
    ```
    the above error will be produced. Add a bound for `Course`:
    ```
    example someIntro is {wellformed} for {
        Intro = `CSCI0150
        Course = `CSCI0150
    }
    ```


### Errors Related to Testing


#### Invalid example ... the instance specified is impossible ... 

If an example fails, Forge will attempt to disambiguate between:

* it actually fails the _predicate under test_; and 
* it fails because it violates the type declarations for sigs and fields. 

Consider this example:

!!! example "Impossible Instance"

    ~~~
    #lang forge/bsl 

    abstract sig Grade {} 
    one sig A, B, C, None extends Grade {} 
    sig Course {} 
    sig Person { 
        grades: func Course -> Grade,
        spouse: lone Person 
    }

    pred wellformed { 
        all p: Person | p != p.spouse 
        all p1,p2: Person | p1 = p2.spouse implies p2 = p1.spouse
    }

    example selfloopNotWellformed is {wellformed} for {
        Person = `Tim + `Nim 
        Course = `CSCI1710 + `CSCI0320
        A = `A   B = `B   C = `C   None = `None 
        Grade = A + B + C + None

        -- this violates wellformed
        `Tim.spouse = `Tim 

        -- but this violates the definition: "grades" is a total function
        -- from courses to grades; there's no entry for `CSCI0320.
        `Tim.grades = (`CSCI1710) -> B

    }
    ~~~


If you receive this message, it means your example does something like the above, where some type declaration unrelated to the predicate under test is being violated.

### Errors related to syntax

#### Unexpected type or Contract Violation

In Forge there are 2 kinds of constraint syntax for use in predicates:

* formulas, which evaluate to true or false; and 
* expressions, which evaluate to values like specific atoms. 

If you write something like this:

!!! example "Contract Violation"
    ```
    #lang forge/bsl 
    sig Person {spouse: lone Person}
    run { some p: Person | p.spouse}
    ```
    produces:
    ```
    some: contract violation
      expected: formula?
      given: (join p (Relation spouse))
    ```


The syntax is invalid: the `some` quantifier expects a _formula_ after its such-that bar, but `p.spouse` is an expression. Something like `some p.spouse` is OK. (The phrase "contract violation" in this case just means "I was passed something I didn't expect.")

Likewise: 

!!! example "Unexpected Type"
    ```
    sig Person {spouse: lone Person}
    run { all p1,p2: Person | p1.spouse = p2.spouse implies p2.spouse}
    ```
    results in:
    ```
    =>: argument to => had unexpected type. 
      expected #<procedure:node/formula?>,
      got (join p2 (Relation spouse))
    ```


Since `implies` is a boolean operator, it takes a *formula* as its argument. Unfortunately, `p2.spouse` is an expression, not a formula. To fix this, express what you really meant was _implied_. 
