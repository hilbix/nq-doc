# Coding style

## DRY, KISS, KTBT and ETR

- DRY: Don't repeat yourself
- KISS: Keep it stupid simple
- KTBT: Keep together what belongs together
- ETR: Easy to read

This means:

- No long routines.  Routines spanning more than 1 screen page (where a screen is 60x120 at least) shall be avoided.
- Short explanative and nonredundant comments.  Things like `a++ // increment 1` are a no go.  Likewise non-obvious things must be commented
- Avoid bloat.  If there is more than one good way to write it, write it the way it is best to understand.  For others.
- Try to keep parantheses or braces, which belong together, either on the same line or on the same column.  Except for certain idioms.

Certain idioms which do not follow the rule are:

- `(function(args){ ... })` with chained calles - to avoid right drift on those constructs, `})` should be on the same indentation level as where the constructor started.
- `try {` and `} catch {` and `} finally {`.  Those idioms keep the braces in their line.  They are always on separate lines except if it fits nicely.
- `fncall(args)` while `if (`.  This is true for all operators, except functional operators.
- `a+1` is OK, but in more complex formulas use spaces to make them better readable.
- `var` definitions on a single line please.  If they span multiple lines, start a new `var` declaration as soon as possible.

To be continnued if needed.

