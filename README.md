## Indent of Doom

## What is this?
Indent of doom is a library which allows you to adjust your major mode's
indentation rules. This library provides a small EDSL to customize your rules.

Why would you want this?

For example let's say you're working in a team and you have a certain indentation
standard to follow. There might be a chance that the Major mode's indentation
does not comply with these standards. In this case you'd have to adjust
your indentation manually whenever working on your team's project to
stay consistent. A good example is Erlang mode which is explained in the
usage section.

## Usage

### DSL Functions

#### Targets
* iod/prev    `Previous line`
* iod/current `Current line`
* iod/next    `Next line`

#### Target functions
* iod/ends-on `Takes 1 or more strings. Returns true of the target line ends with one of the arguments`
* iod/starts-with `Takes 1 or more strings. Returns true of the target line starts with one of the arguments`
* iod/contains ``
* iod/indent `Takes optionally 1 integer argument. Get the indent level of the target line in space + (argument * tab-width)`
* iod/indent-char `Takes no arguments. Get the first character of the target line.`

### Using the DSL
Let's say as an example that we are writing a list in Erlang code:

```
MyLongVariable = [
                  1,
                  2,
                  3,
                  4
                  5
                 ].
```

This is how Erlang mode would normally indent this list. However I don't find this practical.
Using indent-of-doom we can /monkey patch/ this.
Inside your emacs config file add this:
```
(setq iod--rules '(
   (erlang-mode . (
       ((iod/current 'iod/starts-with "]") (iod/prev-tab -1))
       ((iod/prev    'iod/ends-on "[")     (iod/prev-tab 1))
       ((iod/prev    'iod/ends-on ",")     (iod/prev-tab))
   ))
))
```

- **line** 1
    - Set the iod--rules variable
- **line** 2
    - Add rules for erlang-mode
- **line** 3
    - If the current line starts with the character "]" then
the current line should have the same indentation as the
previous line __MINUS__ 1 tab-width.
- **line** 4
    - If the previous line ends with the character "[" then
the current line should have the same indentation as the
previous line __PLUS__ 1 tab-width.
- **line** 5
    - If the previous line ends with the character "," then
the current line should have the same indentation as the
previous line.

Now if we enable indent-of-doom-mode inside our Erlang file it will now indent as follows:

```
MyLongVariable = [
    1,
    2,
    3,
    4
    5
].
```

Another example:

```
AnonFunctionOfDoomAndDestruction = fun(X) ->
                                           where_am_i
                                   end.
```
Adding these rules:

```
(setq iod--rules '(
    (erlang-mode . (
        ((and (iod/current 'iod/starts-with "end")
         (iod/prev 'iod/ends-on ") ->"))      (iod/prev-tab))
        ((iod/current 'iod/starts-with "end") (iod/prev-tab -1))
        ((iod/prev 'iod/ends-on ") ->")       (iod/prev-tab 1))
        ((iod/current 'iod/starts-with "]")   (iod/prev-tab -1))
        ((iod/prev 'iod/ends-on "[")          (iod/prev-tab 1))
        ((iod/prev 'iod/ends-on ",")          (iod/prev-tab))
   ))
))
```
Resulting in:

```
AnonFunctionOfDoomAndDestruction = fun(X) ->
    where_am_i
end.
```

The first case however, is very common among many languages, not just Erlang. Let's make it work for all languages instead.

```
(setq iod--rules '(
    (all . (
        ((iod/current 'iod/starts-with "]")   (iod/prev-tab -1))
        ((iod/prev 'iod/ends-on "[")          (iod/prev-tab 1))
        ((iod/prev 'iod/ends-on ",")          (iod/prev-tab))
   ))
   (erlang-mode . (
        ((and (iod/current 'iod/starts-with "end")
         (iod/prev 'iod/ends-on ") ->"))      (iod/prev-tab))
        ((iod/current 'iod/starts-with "end") (iod/prev-tab -1))
        ((iod/prev 'iod/ends-on ") ->")       (iod/prev-tab 1))
   ))
))
```
Now the fun rule is only applied to Erlang mode, and the list rules are applied to all modes.

Some lists however are written with curly braces instead of square brackets, let's fix that.

```
(setq iod--rules '(
    (all . (
        ((iod/current 'iod/starts-with "]" "}") (iod/prev-tab -1))
        ((iod/prev 'iod/ends-on "[" "[")        (iod/prev-tab 1))
        ((iod/prev 'iod/ends-on ",")            (iod/prev-tab))
   ))
   (erlang-mode . (
        ((and (iod/current 'iod/starts-with "end")
         (iod/prev 'iod/ends-on ") ->"))      (iod/prev-tab))
        ((iod/current 'iod/starts-with "end") (iod/prev-tab -1))
        ((iod/prev 'iod/ends-on ") ->")       (iod/prev-tab 1))
   ))
))
```
EDSL:
** Targets
* iod/prev
* iod/next
* iod/current

** Target addon
* iod/starts-with
* iod/ends-on
* iod/contains

** Indentors
* iod/next-tab
* iod/current-tab
* iod/prev-char
* iod/next-char
* iod/current-char
* iod/ends-on
