---
search:
  boost: 2
---

# Running

## How to Run 

You can either use the [Forge VSCode extension](https://marketplace.visualstudio.com/items?itemName=SiddharthaPrasad.forge-language-server)'s play button, or invoke Forge directly from the terminal via Racket:

```
racket <modelname.frg> 
```

You can provide Forge [options](../running-models/options.md) directly by adding command-line flags:

* The `-o` or `-option` flag will set the option, but if the file sets that option the file's version will be used.
* The `-O` or `-override` flag will set the option, and that option will hold regardless of what may be set in the file. 

!!! example "Setting options at the command line"
    If `ring_of_lights.frg` is in the current directory, running:
    ```
    racket ring_of_lights.frg -o run_sterling off -O verbose 0
    ```
    will:
    * disable Sterling _unless the option is set in the file_; and
    * disable verbose output entirely, regardless of any `verbose` option settings in the file.


## Commands to View Instances

There are two primary ways of running your model with the goal of getting instances. You can either as Forge to show you instances that satisfy a predicate you wrote with the `run` command, or ask Forge to look for counterexamples to a predicate you wrote with the `check` command. 

The various [testing commands](../testing-chapter/testing.md) will also execute your model.

### Run

The run command can be used in a few different ways, show below:

```
<run-name>: run <pred> for <bounds>
<run-name>: run { <expr> } for <bounds>
```

Note that the run-name is optional to provide, but is helpful to distinguish what different run commands are showing.

When using the run command, Forge will display possible worlds (instances) where the predicates or expressions you specified evaluate to true, within the given bounds. Instances are displayed in [Sterling](https://github.com/tnelson/Forge/wiki/Sterling-Visualizer) If no such instances are found, "UNSAT" is displayed.

When no more satisfying instances can be found, Sterling displays "No more instances found".

### Check

The check command is used to ask Forge to look for counterexamples to a given set of predicates, i.e. instances where the predicate or expression evaluates to false. The syntax is the same as for the run command, just with the keyword `check` instead:

```
<check-name>: check <pred> for <bounds>
<check-name>: check { <expr> } for <bounds>
```

If no counterexamples are found, Sterling displays "No counterexamples found. Assertion may be valid". When no more counterexamples can be found, Sterling displays "No more instances found".

!!! warning "Common Mistake!"
    Unless a predicate is explicitly used in the `run`, `check`, etc. command (or invoked by another predicate that is used in the command) it will not take effect. For example, if you have defined a `wellformed` predicate, but execute `run {}`, that predicate will not necessarily hold in instances Forge finds.


