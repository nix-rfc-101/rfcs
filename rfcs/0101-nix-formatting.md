---
feature: nix_formatting
start-date: 2021-08-17
author: \@piegamesde
co-authors: \@infinisil
shepherd-team:
shepherd-leader:
related-issues: https://github.com/serokell/nixfmt/pull/118
---

# Nix formatting

## Summary

[summary]: #summary

- Decide on basic Nix formatting style guidelines and pick a default formatter for Nix code.
- `nix fmt` (or any future replacement) has to use this formatter as a default
- Automatically format Nixpkgs and enforce the format using CI from then on.
- Establish the Nix format team to take care of the long-term maintenance of the code style and formatter.

## Motivation

[motivation]: #motivation

Currently, there is no authoritative formatting style guide for Nix code, including in Nixpkgs.
The current code style in Nixpkgs has evolved organically over time, leading to inconsistencies both across files and language features.
Things are occasionally improved as people touch old files, at the cost of muddying the diff with cosmetic changes.
There are several auto-formatters for Nix code, each with their own style, and none currently used for Nixpkgs.

The goals of this RFC are:

- We want to prevent future debate around how things should be formatted.
- We want to make it as easy as possible for contributors (especially new ones) to make changes without having to worry about formatting.
- Conversely, reviewers should not be bothered with ill-formatted contributions.
- We want a unified Nix code style that's consistent with itself, easily readable, accessible and results in readable diffs.

Non-goals of this RFC:

- Code style aspects that are not purely syntactic (e.g. optional/redundant parentheses, adding `inherit` statements, swapping argument order with `flip`, …)
- Nixpkgs-specific tweaks to the output format (e.g. using attribute names and other context for heuristics about how to best format the contents)
- Extended Nixpkgs-specific linting like nixpkgs-hammering
- Formatting non-Nix files in Nixpkgs
- Applying the format to other repositories within the NixOS organization containing Nix code. It is up to their respective maintainers to make the transition.

## Goals and approach

There are several goals that the formatting style should match.
These are inherently at conflict with each other, requiring priorisation and making trade-offs.
The resulting choice is always a compromise.

In general, we want the code to be (in no particular order):

- **Short and concise.** Code should not be spread across too many lines, but also without being crammed
- **Readable.** The output format should reflect the semantic flow of the program. It should be clear where expressions start and end. The amount of information per line should be limited.
- **Consistent.** Similar syntax constructs should be formatted similarly.
    - The number of special cases in the formatting rules should be minimized.
- **Diffable and stable.** Small changes to the code should not result in excessive changes in the output.

The general approach taken here is to liberally expand expressions by default, with the goal of being stable, diffable and consistent.
Then, special cases with more compact output for the most common patterns are introduced as needed,
sacrificing those properties in favor of conciseness.
The idea is that this results in a format that is wide by default but compact where it matters.

The interactions between different language features are complex and producing a style that matches the expectations involves a lot of special cases.
Any attempt at creating an exhaustive rule set would be futile.
Therefore, the formatting rules are intentionally under-specified, leaving room for the formatter implementation.
However, the most important or potentially controversial rules are included, as well as some general meta-rules.

When deciding between two *equally good* options, currently prevalent formatting style in Nixpkgs should be followed.
The emphasis here is on "equally good".
We should not fear of making radical changes to the current style if there are sufficient arguments in favor of it.

*Bad code does not deserve good formatting.*

## Detailed design
[design]: #detailed-design

The RFC consist of these main parts, see the following sections for more information:

- Define the initial _standard Nix format_
- Establish the _Nix format team_
- Create the _official Nix formatter_ implementation
- Reformat Nixpkgs with the official Nix formatter
- Specify that any default formatting in the Nix CLI must use the official Nix formatter

### Standard Nix format

The initial version of the standard Nix format is defined in a section towards the end:

[Initial standard Nix format](#initial-standard-nix-format).

Significant changes to the standard Nix format must go through another RFC.

The latest version of the standard Nix format must be in a file on the main branch of the [official Nix formatter](#official-nix-formatter).

### Establishing the Nix format team

A new team is created, initially consisting of:
- @piegames (author of this RFC, shepherd of the original formatting RFC)
- @infinisil (from Tweag, co-author of this RFC, shepherd of the original formatting RFC)
- @tomberek (from Flox, shepherd of the original formatting RFC)
- @0x4A6F (shepherd of the original formatting RFC)
- @Sereja313 (from Serokell)

Team member updates are left for the team itself to decide.

### Official Nix formatter

The Nix format team is given the authority and responsibility of
creating and maintaining the _official Nix formatter_ implementation.
This is a repository in the NixOS GitHub organisation.
The repository will initially be based on [this nixfmt pull request](https://github.com/serokell/nixfmt/pull/118).

Any release of the official Nix formatter must conform to the latest version of the [standard Nix format](#standard-nix-format).

The latest release of the official Nix formatter should support the Nix language syntax of the latest Nix release.
The Nix format team should be consulted before the Nix language syntax is changed.

For changes that maintain conformity to the standard Nix format,
the team has the authority to accept or reject them.

### Reformat Nixpkgs

For formatting Nixpkgs itself, a pinned release version of the official Nix formatter must be used.
CI must generally enforce all files in Nixpkgs to be formatted with this version at all times.
Automatically generated files may be handled differently, see the next section.

For every bump of the pinned formatter in Nixpkgs,
its files must thus be re-formatted accordingly.
The Nix format team is responsible for this task.

In order to minimize conflicts especially when back-porting, it is preferable to only be updated and thus re-formatted shortly before release branch-off.
This must be done in coordination with the NixOS release managers.

#### Automatically generated files

There are automatically generated files in Nixpkgs, with a potentially different format.
This RFC makes no decisions on how to handle such cases, but there are some options:
- Exclude them from the CI via some tooling (e.g. treefmt if that is being used)
- Format them anyway, either after-the-fact or ideally already in the generator tooling itself

### Default Nix CLI formatting

In case the Nix CLI ever gets support for running a default Nix formatter,
the official Nix formatter must be used.

## Examples and Interactions

[examples-and-interactions]: #examples-and-interactions

### Git blames

Commits that re-format Nixpkgs must be added to `.git-blame-ignore-revs`,
so it [won't get shown](https://docs.github.com/en/repositories/working-with-files/using-files/viewing-a-file#ignore-commits-in-the-blame-view) in blames on GitHub,
and can be ignored in the `git blame` command using [`--ignore-revs-file`](https://www.git-scm.com/docs/git-blame#Documentation/git-blame.txt---ignore-revs-fileltfilegt).

### Documentation

TODO move up to detailed design section?

Update [Section 20.1 (Coding conventiones → Syntax)](https://nixos.org/manual/nixpkgs/stable/#sec-syntax) of the Nixpkgs manual. All formatting-specific guidance is removed and replaced with instructions on how to automatically format Nixpkgs instead.

### Formatting gotchas requiring manual intervention

Some code patterns in Nixpkgs will result in a sub-optimal format,
because an auto-formatter cannot do exceptions based on context.
A lot of the times though, the same program can be equivalently expressed in a prettier way.

#### CLI argument flags

It is ommon to pass in CLI flags (e.g. to builders) like this:

```nix
[
  "--some-flag" "some-value"
]
```

However, this will be formatted sub-optimally:

```
[
  "--some-flag"
  "some-value"
]
```

The solution is to use a more structured helper function:

```
lib.cli.toGNUCommandLine {} {
  some-flag = "some-value";
}
```

#### Badly placed comments

TODO

#### Long second-to-last function argument

TODO

#### Singleton lists

TODO singleton lists should use `lib.singleton`

## Drawbacks

[drawbacks]: #drawbacks

- No automatic code format can be as pretty as a carefully crafted manual one. There will always be "ugly" edge cases.
    - However, we argue that on average the new format will be an improvement, and that contributors should not waste their precious time making the code slightly more pretty.
- Every formatter will have bugs, but the Nix format team will be able make fixes
- Having a commit that changes the formatting, can make git blame harder to use. It will need the `--ignore-rev` flag.
    - GitHub also has [builtin functionality](https://docs.github.com/en/repositories/working-with-files/using-files/viewing-a-file#ignore-commits-in-the-blame-view) for this

## Alternatives

[alternatives]: #alternatives

- Keep the status quo of not having an official formatter. The danger is that this creates discord within the Nix community. The current friction and maintainer churn due to bad formatting may arguably be small, but not negligible.
- Pick a different formatter, or a different format
- Pick a formatter and/or format, but don't enforce it in CI, to allow manually tweaking the output if necessary.
- Apply the format incrementally, i.e. only changed files get formatted, to make a more graceful transition. However, such an approach would not improve the amount of merge conflicts, and increase the workload for contributors during that time significantly.

## Unresolved questions

[unresolved]: #unresolved-questions

## Future work

[future]: #future-work

- General style guidelines beyond AST reformatting
- Making widespread use of linters like Nixpkgs-hammering
- Applying the formatter to other repositories within the Nix community

-----

## Initial standard Nix format

- Newlines are not guaranteed to be preserved, but empty lines are.
  - This allows the formatter to compact down multi-line expressions if necessary, while still allowing to structure the code appropriately.
- Expressions of the same kind that can be treated as a sequence of expressions on the same level should be treated as such, even though they are technically parsed as a nested tree.
  - This applies to else-if chains, functions with multiple arguments, some operators, etc.
  - Example:
    ```nix
    # This is treated as a sequence of if-then-elsa's chains
    if cond1 then
      foo
    else if cond2 then
      bar
    else
      baz
    ```
- Any two (sub-)expressions that are fully on a common single line must have a common (transitive) parent expression which is also fully on that line.
  - Equivalently: If a maximally parenthesized form of a line fully contains a parenthesis pair, there must be a single outermost pair on that line, meaning it contains all of the others.
  - Example:
    ```nix
    # Bad, because cond and foo are two expressions but they don't have a common parent on the same line
    if cond then foo
    else bar
    ```

### Terms and definitions

- Brackets: `[]`
- Braces: `{}`
- Parentheses: `()`

### Indentation

- Two spaces are used for each indentation level.
  - This may be revisited should Nix get proper support for [using tabs for indentation](https://github.com/NixOS/nix/issues/7834) in the future.
- There is no vertical alignment, neither at the start of the line nor within lines.
  - Examples:
    ```nix
    {
      # Within-line vertical alignment
      linux    = { execFormat = elf;     families = {              }; };
      netbsd   = { execFormat = elf;     families = { inherit bsd; }; };
      none     = { execFormat = unknown; families = {              }; };
      openbsd  = { execFormat = elf;     families = { inherit bsd; }; };
      # Start of line vertical alignment
      optExecFormat =
        lib.optionalString (kernel.name == "netbsd" &&
                            gnuNetBSDDefaultExecFormat cpu != kernel.execFormat
                           )
                           kernel.execFormat.name;
    }
    ```
- Indentation levels *must not* be "skipped", i.e. on subsequent lines, indentation can only increase by at most one level, but may decrease arbitrarily many levels.
  - In other words: a line on indentation level 6 could be followed by a line on indentation level 1, but not the other way around.
  - Examples:
    ```nix
    buildInputs = [
        foo # <--
      ] // lib.optionals cond [
        bar
      ];

    attribute = { args }: let
        foo = "bar" # <--
      in
        foo;

    (callFunction {
        foo = "bar"; # <--
      }
      arg
    )

    # This is okay, indentation increases only one level per line
    let
      x = {
        a = foo
          bar
          baz;
      }; # <-- The decrease by two levels here is okay though
    in
    null
    ```

### Expansion of expressions

**Description**

Unless stated otherwise, any expression that fits onto one single line will be trivially formatted as such.

For sequences of items in expressions, like elements in a list, key-value pairs in attribute sets, or function arguments, the following applies:

- If expanded into with multiple lines, each item should be on its own line.
  - Grouping similar items together can be done by adding blank lines or comments between the groups instead.
  - This also applies to the first item, so e.g. `[ firstElement` in a multi line list is not allowed.
- Long sequences of items may be liberally expanded, even if they would fit onto one line character-wise.
  - The motivation is to keep the information per line manageable. Usually "number of elements" is a better metric for that than "line length".
  - The cutoff is usually determined empirically based on common usage patterns.

**Examples:**

```nix
{
  #1
  buildInputs = [
    foo
    bar
    baz

    somethingElse
  ];

  #2
  systemd.services = {
    foo = { };
    bar = { };
  };

  #3
  inherit
    lib
    foo
    bar
    baz
  ;
}
```

### Function application

**Description:**

- In a function application chain, the first element is treated as the "function" and the remaining ones as "arguments".
- The last argument receives special treatment, to better represent common coding patterns.
- As much arguments as possible are fit onto the first line.
  - If all but the last argument do fit, then the last argument may start on the same line.
  - If an earlier argument does not fit onto the first line, then itself and all the following ones start on a new line. This is called the expanded form.
  - All arguments that are not on the same line as the function are indented by one level.

**Examples:**

```nix
#1
function arg1 arg2

#2
function arg1 arg2 {
  more = "things";
}

#3
function arg1 arg2 arg3 # reached line limit here
  arg4
  arg5
  [
    1
    2
  ]
  arg7

#4
function arg1
  {
    # stuff
  }
  arg3

#5
function
  {
   # …
  }
  {
   # …
  }

#6
function arg1 (
  function2 args
)
```

**Drawbacks**

- This style sometimes forces lists or attribute sets to start on a new line, with additional indentation of their items.

**Rationale and alternatives**

- Not indenting the arguments, to save some indentation depth. This would be consistent with other constructs like function declarations and let bindings.
- Compacting multiline arguments like this:
  ```nix
  #4b
  function arg1 {
    # stuff
  } arg3

  #5b
  function {
    # …
  } {
    # …
  }
  ```
  - This violates the guideline of the indentation representing the expression structure, and thus reduces readability.
  - This does not work well with line length limits on short arguments like in example #3.

### Function declaration

**Description**

- The body of the function is not indented relative to its arguments.
- Multiple ("simple") identifier arguments are written onto the same line if possible.
- Attribute set arguments always start on a new line; they are not mixed with identifier arguments.
  - If they have few attributes, the argument may be written on a single line, otherwise the expanded form is used.
- Attribute set arguments have their attributes on a new line each with indentation, followed by a trailing comma.

**Examples**

```nix
#1
name: value: name ++ value

#2
name: value:
{
  "${name}-foo" = value;
}

#3
{ pkgs }: pkgs.hello

#4
args@{
  some,
  argument,
  default ? value,
  ...
}:
{
  # body
}

#5
{ pkgs }:
name: value: 
{
  # body
}
```

**Rationale and alternatives**

- Have leading commas for parameters in attribute set arguments, like currently done in Nixpkgs
  ```nix
  #6
  { some
  , arg
  }:
  #7
  args@{
    some
  , argument
  # Single line comment
  , commentedArgument
  , # Comment on the value
    # multiline comment
    default ? value
  , ...
  }:
  # …
  ```
  - This leads to problems with the first argument, as leading commas are not allowed. `{ some` is discouraged by the the style guidelines; `some` should start on a new line instead. Also, this does not work well with `@` bindings.
  - The currently suggested style for commenting items in the Nixpkgs manual (depicted here in `#7`) is not great. However, there are no other good solutions with leading comma style that don't run into other problems.
  - The leading comma style was a lesser-evil workaround for the lack of trailing commas in the Nix language. Now that the language has this feature, there is no reason to keep it that way anymore.


### Operations

**Description**

Chained operations of an operator with the same binding strength are treated as one.
If an operation chain does not fit onto one line, it is expanded such that every operator starts a new line.
Usually, the operands start on the same line as their operator.
Notable exception to this are other nested operators and wide function calls.
The right hand side of an operation (or, if chained, all but the first operand) is indented.

Binary operators (which cannot be chained) use a more compact representation,
where the operator is not required to start a new line even when the operands span multiple lines.

The `//` operator is special cased to such a more compact representation too,
even though this results in multiple violations of the style guidelines.
The motivation for this is that it is often used in places that are very sensitive to the indentation of large attribute sets.

**Examples**

```nix
```

**Drawbacks**

**Rationale and alternatives**

### if

**Desciption**

- `if` and `else` keywords always start a line, the if and else bodies are indented.
- If the condition does not fit onto one line, then it will start on the next line with indentation, and `then` will be on the start of the line following the condition.
- `else if` chains are treated as one long sequence, with no indentation creep on each step.
- Only simple `if` statement can be single-line, no `else if` chains.

**Examples**

```nix
#1
if builtins.length matches != 0 then
  { inherit path matches; }
else if path == /. then
  [
    1
    2
  ]
else
  go (dirOf path);

#2
if
  matches != null
  && builtins.length matches != 0
then
  { inherit path matches; }
else if path == /. then
  null
else
  go (dirOf path);
```

**Rationale and alternatives**

- Attribute sets and lists could start on the same line as the if keywords, saving an indentation level on their body:
  ```nix  
  #1a
  if builtins.length matches != 0 then {
    inherit path matches;
  } else if path == /. then [
    1
    2
  ] else
    go (dirOf path);
  ```
  - This results in inconsistent vertical start of the keywords, making the structure harder to follow
- Have the `then` on the start of the next line, directly followed by the if body:
  ```nix
  #1b
  if builtins.length matches != 0
    then { inherit path matches; }
  else if path == /.
    then [
      1
      2
    ]
  else go (dirOf path);

  #1c
  if builtins.length matches != 0
  then { inherit path matches; }
  else if path == /.
  then [
    1
    2
  ]
  else go (dirOf path);
  ```

### with, assert

**Description**

- The body after the statement starts on a new line, without indentation.
- For `with` expressions there may be exceptions for common idioms, in which the body already starts on the same line.

**Examples**

```nix
with pkgs;
assert foo == bar;
{
  meta.maintainers = with lib.maintainers; [
    some
    people
  ];
}
```

### let

**Description**

- Let bindings are *always* multiline.
- The "let" part is indented one level, but not the "in" part.
- Each item in the "let" part is indented and starts on its own line. For more details, see the section TODO.
- The "in" part starts on a new line.

**Examples**

```nix
let
  foo = "bar";
in
if foo == "bar" then
  "hello"
else
  "world"
```

### Attribute sets and lists

**Description**

- Brackets and braces are generally written with a space on the inside, like `[ `, ` ]`, `{ ` and ` }`.
  - Empty lists and attribute sets are written as`[ ]` and`{ }`, respectively.
- Lists and attribute sets with multiple items are liberally expanded.
  - They can only be on a single line if:
    - They contain at most one element
    - Fit on the line
  - As described under Binders below (TODO section link), nested attribute sets are always expanded.
- Lists with a single element may be written compactly if that element is another list or attribute set

**Examples**

```nix
#1
[
  { }
  { foo = "bar"; }
  {
    foo = {
      bar = "baz";
    };
  }
  { foo.bar = "baz"; }
]

#2
[
  [ 1 ]
  [
    2
    3
  ]
]

#3
[ [
  1
  2
  3
] ]

#4
[ {
  mySingletons = [ [ ({
    # stuff in there
  }) ] ];
  
  foo = [
    {
      bar = 10;
    }
  ];

  mySingletons = [ [
    (function call)
  ] ];
} ]
```

**Drawbacks**

- This special casing of singleton lists can result to weird spacing when combined with parentheses: `([ [`

**Rationale and alternatives**

- Don't have a special compact form for singleton lists, at the cost of an indentation level and two additional lines
- Be a bit less eager about expanding lists and attribute sets (for example by allowing up to n elements).

- Argument in favor of not doing the special casing:
  - If you have a singleton list, it's likely that you'll want to add more elements later anyways
  - Otherwise your code could use a refactor to not need

### Binders and inherit

Let bindings and attribute sets share the same syntax for their items, which is discussed here together.

#### Binders

**Description**

Binders have the most special cases to accomodate for many common Nixpkgs idioms.
Generally, the following styles exist, which are used depending on the kind and size of the value:

```nix
#1 single line
foo = "bar";

#2 single line, on a new line
very.long.foo =
  function arg1 arg2 arg3;

#3 multi line, starting on the same line
foo = function {
  # args
};

#4 multi line, starting on a new line
foo =
  function
    arg1
    arg2
    arg3
;
```

Notable special cases are:

- Single line values that would not benefit from style #2 keep using #1, even if this makes it go above the line limit. This mostly applies to simple strings and paths.
- Attribute set values are *always* expanded. This has the consequence of always forcing nested attribute sets to be multiline (even if they would be single line otherwise because they only contain a single item), which usually is desired.
  ```nix
  {
    foo.bar.baz = "qux";
    foo = {
      bar.baz = "qux";
    };
  }
  ```
- "statement-like" expressions like "let", "if" and "assert" always use #4 (or #1).
- If the value is a `with` followed by a function application or list or attribute set, try to use #3.
  ```nix
  buildInputs = with pkgs; [
    some
    dependencies
  ];
  ```

**Alternatives**

One could eliminate style #2 by having #4 always start on the first line. This would even reduce indentation in some cases. However, this may look really weird in other cases, especially when the binder is very long:

```nix
some.very.long.attr = callFunction
  arg1
  arg2
  arg3;
```

#### inherit

**Description**

The items are either all on the same line, or all on a new line each (with indentation).

**Examples**

```nix
inherit foo bar baz;
inherit
  foo
  bar
  baz
;
```

#### inherit from

**Description**

- If the inherit target is single-line, it is placed on the same line as the `inherit`, even if the following items do not fit onto one line.
- Otherwise, it starts on a new line with indentation, like the others.
  - In that case, the remaining items are force-expanded too, even if they would have fit onto one line in the first place.

**Examples**

```nix
inherit (pkgs) app1 app2 app3;
inherit (pkgs)
  app1
  app2
  # …
  app42
;
inherit
  (pkgs.callPackage ./foo.nix {
    arg = "val";
  })
  attr1
  attr2
;
```

#### Semicolon placement

**Description**

The semicolon is always placed on the same line as the expression it concludes.

**Examples**

```nix
{
  attr1 = bar;
  attr2 = function call {
    # stuff
  };
  attr3 =
    function call
      many
      arguments;
  attr3 =
    let
      foo = "bar"
    in
    some statement;
  attr4 =
    if foo then
      "bar"
    else
      "baz";
  attr5 =
    let
      foo = false;
    in
    if foo then "bar" else "baz";
  attr6 = function (
    if foo then
      "bar"
    else
      "baz"
  );
  attr7 =
    cond1
    || cond2
    ||
      some function call
      && cond3;
}
```

**Rationale and alternatives**


There are four considered semicolon styles:
1. On a new line without indentation.
  - This clearly marks a separation between attributes, however it is wasteful of space.
  ```nix
  attr3 =
    function call
      many
      arguments
  ;
  attr3 =
    let
      foo = "bar"
    in
    some statement
  ;
  ```
2. On a new line with one indentation level.
  - Just as wasteful on space as (1), but a bit less clear about signaling the end of the binding.
  ```nix
  inherit (pkgs)
    app1
    app2
    # …
    app42
    ;
  attr3 =
    function call
      many
      arguments
    ;
  ```
3. A mix of (1) and (2), where usually the semicolon is placed directly at the end of the binder.
   But with exceptions in which the semicolon is placed onto the following line instead in cases where the value is a multiline `if` expression or nested operator.
   These are the only syntax elements which may result in the semicolon being placed on a line with arbitrarily deep indentation.
   
   ```nix
   attr4 =
     if foo then
       "bar"
     else
       "baz"
   ;
   attr5 =
     let
       foo = false;
     in
     if foo then "bar" else "baz";
   attr7 =
     cond1
     || cond2
     ||
       some function call
       && cond3
   ;
   ```
