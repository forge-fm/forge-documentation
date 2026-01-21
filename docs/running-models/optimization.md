# Target-oriented model finding (TOMF) in Forge 

**CAUTION: This feature of Forge is experimental and subject to change. It will be added as a preview feature in version 4.3.**

Forge uses a modified version of the [Pardinus model finder](https://github.com/haslab/Pardinus) as its back end. Pardinus supports a form of optimization that targets a specific goal instance (which may or may not satisfy the given constraints). We are grateful to Alcino Cunha, Nuno Macedo, and Tiago Guimarães for their engineering work and [technical paper](https://link.springer.com/content/pdf/10.1007/978-3-642-54804-8_2.pdf) on target orientation.

## How to enable TOMF

To enable target-oriented mode, switch solvers to partial max-SAT and use the `target` problem type:
```
option problem_type target
option solver PMaxSAT4J
```
**This should only be done in Relational Forge (`#lang forge`).** Do not attempt to use `target` mode in Temporal Forge, which requires a different backend solver mode.

<!-- The `target_mode` option provides the global default for how the solver treats the target. There are 5 options currently available:
* `close_noretarget`: Get as close to the target as possible. When enumerating instances, keep targeting the original target.
* `far_noretarget`: Get as far from the target as possible (up to the given bounds). When enumerating instances, keep targeting the original target.
* `close_retarget`: Get as close to the target as possible. When enumerating instances, reset the target to the last produced instance. **(Not yet reliable)**
* `far_retarget`: Get as far from the target as possible (up to the given bounds). When enumerating instances, reset the target to the last produced instance. **(Not yet reliable)**
* `hamming_cover`: View instances as vectors of boolean variables, where each variable corresponds to a potential tuple membership in a relation. Define the distance between two instances to be the Hamming distance between their boolean vectors. Enumerate instances that maximize the distance from previously produced instances. **(Not yet reliable)** -->

<!-- Absent an explicit target, the engine will target the first instance generated. Therefore, use the global option if you don't have a specific target in mind, but want to customize the enumeration strategy that the solver follows.  -->

<!-- If you do have a specific target in mind, give it as part of a `run` command, following the target. E.g.:

```
tomf_close_fixed: run {} for 3 Node 
  target_pi {no Node} close_noretarget 
``` -->

## Mode: Targeting a partial instance 

To prioritize instances as close as possible to a target, use the `target_pi` keyword in a `run` command. The argument may be either be the name of a pre-defined `inst` or a `{}`-delimited partial-instance block. 

### Example: Minimal Instances

This example will produce the empty instance first, then a graph of 1 node, etc. 

```
#lang forge
option problem_type target
option solver PMaxSAT4J
sig Node { edges: pfunc Node -> Int }
inst emptyGraph { no Node }
tomf_test_close_noretarget_noNode: run {}
  target_pi emptyGraph
```

## Mode: Targeting an integer expression 

It can sometimes be useful to minimize an _integer_ expression, rather than a set. To do this, use either the `minimize_int` or `maximize_int` keywords, providing an integer-valued expression. 

When optimizing with respect to an integer expression, keep in mind the bitwidth. E.g., the default bitwidth of `4` will instantiate the $2^4 = 16$ integers in the interval `[-8, 7]`. Thus, minimizing an integer expression when the default bitwidth is in effect will target `-8`. 

### Example: Minimizing total edge weight 

This example will produce graphs with minimal total edge weight, modulo potential underflow. E.g., the example may produce a graph with a single edge of weight `-8`, but it might also produce a graph with three distinct `-8`-weight edges. 

```
#lang forge
option problem_type target
option solver PMaxSAT4J
sig Node { edges: pfunc Node -> Int }
tomf_test_close_noretarget_int_totalWeight4: run {} for exactly 2 Node
  minimize_int {sum m: Node | sum n: Node | m.edges[n]} 
```

If this example used `maximize_int` rather than `minimize_int`, the solver would anti-target `-8` and in effect target the maximum integer value `7`.



