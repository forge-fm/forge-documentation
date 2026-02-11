---
search:
  boost: 3
---

# Functions

In the same way that [predicates](../formulas/predicates.md) define reusable formulas, _functions_ define reusable expressions in Relational and Temporal Forge. Define functions with the `fun` keyword:

```
fun <fun-name>[<args>]: <result-type> {
   <expr>
}
```

!!! example "Helper function"
    ```
    fun inLawA[p: Person]: one Person {
      p.spouse.parent1
    }
    ```


As with predicates, arguments will be evaluated via substitution. Functions may be used (with appropriate arguments) anywhere expressions can appear.

!!! example "Helper function use"
    ```
    all p: Person | some inLawA[p]
    ```

    This expands to:

    ```
    all p: Person | some (p.spouse.parent1)
    ```




