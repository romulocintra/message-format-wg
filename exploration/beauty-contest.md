# Syntax Variation Beauty Contest

As discussed in the 2023-09-25 teleconference, we need to choose a syntax to proceed with.
This page hosts various options for considering in the 2023-10-02 call.

## Shorthand description of some aspects of the options

Please dispute factual errors in this. The table uses multiple `-` when an option requires more than one of something. `+` is not weighted.

| Option | Description                                                    | Doesn’t Nest {} | Doesn’t Need More Escapes | Doesn’t Require Quoted Pattern | Allows Unquoted Literal Operands | Counted {} works | Adds no sigils |
| :----- | :------------------------------------------------------------- | :-------------- | :------------------------ | :----------------------------- | :------------------------------- | :--------------- | :------------- |
| 0      | Current syntax, starts in code mode                            | -               | +                         | --                             | +                                | -                | +              |
| 1      | Invert for text mode                                           | -               | +                         | +                              | -                                | +                | +              |
| 1a     | Invert for text mode, distinguish statements from placeholders | -               | +                         | +                              | +                                | +                | -              |
| 2      | Text first, but always code mode after code                    | -               | +                         | -                              | -                                | +                | +              |
| 3      | Use sigils for code mode                                       | +               | -                         | +                              | +                                | +                | --             |
| 3a     | Use sigils for code mode, use {/} for keys                     | +               | -                         | +                              | +                                | +                | -              |
| 4      | Reducing keywords                                              | +               | ---                       | +                              | +                                | +                | ---            |
| 5      | Use blocks for declarations and body                           | --              | -                         | +                              | -                                | -                | -              |
| 5a     | Use blocks for declarations but sigils for selectors           | +               | --                        | +                              | -                                | -                | -              |
| 6      | Use statements                                                 | --              | ---                       | +                              | +                                | +                | -              |

## 0. Current

This is the current syntax,
including the changes introduced in [#488](https://github.com/unicode-org/message-format-wg/pull/488).
The list of messages in this section also serves as the basis for all other examples.

```
{Hello world!}

{Hello {$user}}

input {$var :function option=value}
{Hello {$var}}

input {$var :function option=value}
local $foo = {$bar :function option=value}
{Hello {$var}, you have a {$foo}}

match {$foo} {$bar}
when foo bar {Hello {$foo} you have a {$var}}
when * * {{$foo} hello you have a {$var}}

match {$foo :function option=value} {$bar :function option=value}
when a b {  {$foo} is {$bar}  }
when x y {  {$foo} is {$bar}  }
when * * {  {$foo} is {$bar}  }

input {$var :function option=value} local $foo = {$bar :function option=value}{Hello {$var}, you have a {$foo}}

match {$foo} {$bar} when foo bar {Hello {$foo} you have a {$var}} when * * {{$foo} hello you have a {$var}}

match {$foo :function option=value} {$bar :function option=value}when a b {  {$foo} is {$bar}  }when x y {  {$foo} is {$bar}  }when * * {  {$foo} is {$bar}  }
```

## 1. Invert for Text Mode

Consumes exterior whitespace.

Requires dropping unquoted literal operands from the syntax.

```
Hello world!

Hello {$user}

{input $var :function option=value}
Hello {$var}

{input $var :function option=value}
{local $foo = $bar :function option=value}
Hello {$var}, you have a {$foo}

{match {$foo} {$bar}}
{when foo bar} Hello {$foo} you have a {$var}
{when * *} {$foo} hello you have a {$var}

{match {$foo :function option=value} {$bar :function option=value}}
{when a b} {{  {$foo} is {$bar}  }}
{when x y} {{  {$foo} is {$bar}  }}
{when * *} {|  |}{$foo} is {$bar}{|  |}

{input $var :function option=value}{local $foo = $bar :function option=value}Hello {$var}, you have a {$foo}

{match {$foo} {$bar}}{when foo bar} Hello {$foo} you have a {$var}{when * *} {$foo} hello you have a {$var}

{match {$foo :function option=value}{$bar :function option=value}}{when a b} {{  {$foo} is {$bar}  }}{when x y} {{  {$foo} is {$bar}  }}{when * *} {|  |}{$foo} is {$bar}{|  |}
```

## 1a. Invert for Text Mode, distinguish statements from placeholders

Same as 1, but non-placeholder statements use a `#` prefix.
This allows us to keep unquoted literal syntax.

```
Hello world!

Hello {$user}

{#input $var :function option=value}
Hello {$var}

{#input $var :function option=value}
{#local $foo = $bar :function option=value}
Hello {$var}, you have a {$foo}

{#match {$foo} {$bar}}
{#when foo bar} Hello {$foo} you have a {$var}
{#when * *} {$foo} hello you have a {$var}

{#match {$foo :function option=value} {$bar :function option=value}}
{#when a b} {{  {$foo} is {$bar}  }}
{#when x y} {{  {$foo} is {$bar}  }}
{#when * *} {|  |}{$foo} is {$bar}{|  |}

{#input $var :function option=value} {#local $foo = $bar :function option=value} Hello {$var}, you have a {$foo}

{#match {$foo} {$bar}} {#when foo bar} Hello {$foo} you have a {$var} {#when * *} {$foo} hello you have a {$var}

{#match {$foo :function option=value}{$bar :function option=value}} {#when a b} {{  {$foo} is {$bar}  }} {#when x y} {{  {$foo} is {$bar}  }} {#when * *} {|  |}{$foo} is {$bar}{|  |}
```

## 2. Text First, but Always Code After Code-Mode

This is @mihnita's proposal, mentioned in the 2023-10-02 call.
Start in text mode, but stay in code mode once you get there.
Adapted @eemeli's start-in-text-mode syntax for these examples.

Requires dropping unquoted literal operands from the syntax.

```
Hello world!

Hello {$user}

{input $var :function option=value}
{{Hello {$var}}}

{input $var :function option=value}
{local $foo = $bar :function option=value}
{{Hello {$var}, you have a {$foo}}}

{match {$foo} {$bar}}
{when foo bar} {{Hello {$foo} you have a {$var}}}
{when * *} {{{$foo} hello you have a {$var}}}

{match {$foo :function option=value} {$bar :function option=value}}
{when a b} {{  {$foo} is {$bar}  }}
{when x y} {{  {$foo} is {$bar}  }}

{input $var :function option=value}{local $foo = $bar :function option=value}{{Hello {$var}, you have a {$foo}}}

{match {$foo} {$bar}}{when foo bar} {{Hello {$foo} you have a {$var}}}{when * *}{{{$foo} hello you have a {$var}}}

{match {$foo :function option=value}{$bar :function option=value}}{when a b} {  {$foo} is {$bar}  }{when x y} {  {$foo} is {$bar}  }
```

## 3. Use sigils for code mode

Try to redues the use of `{`/`}` to just expressions and placeholders instead of the three
uses we have now (the other use is for patterns). This requires escaping whitespace or using
a placeholder for it.
See [#487](https://github.com/unicode-org/message-format-wg/pull/487) for a discussion of whitespace options.

The sigil `#` was chosen because `#define` type constructs are fairly common.
Introduces `[`/`]` for keys.

Requires escaping `#` when used as a pattern character.

```
#input {$var :function option=value}
Hello {$var}

#input {$var :function option=value}
#local $foo = {$bar :function option=value}
Hello {$var}, you have a {$foo}

#match {$foo} {$bar}
#when[foo bar] Hello {$foo} you have a {$var}
#when[  *   *] {$foo} hello you have a {$var}

#match {$foo :function option=value} {$bar :function option=value}
#when [a b] {{  {$foo} is {$bar}  }}
#when [x y] {||}  {$foo} is {$bar}  {||}
#when [* *] {|  |}{$foo} is {$bar}{|  |}

#input {$var :function option=value}#local $foo = {$bar :function option=value}Hello {$var}, you have a {$foo}

#match {$foo} {$bar}#when[foo bar] Hello {$foo} you have a {$var}#when[* *] {$foo} hello you have a {$var}

#match {$foo :function option=value} {$bar :function option=value}#when [a b] {{  {$foo} is {$bar}  }} #when [x y] {||}  {$foo} is {$bar}  {||}#when [* *] {|  |}{$foo} is {$bar}{|  |}
```

## 3a. Use sigils for code mode, use `{`/`}` for keys

Similar to 3, but uses braces instead of `[`/`]` square brackets for keys, reducing variation and
the need for additional pattern escapes.
See Slack thread.

Requires `#` to be escaped in unquoted patterns.

```
#input {$var :function option=value}
Hello {$var}

#input {$var :function option=value}
#local $foo = {$bar :function option=value}
Hello {$var}, you have a {$foo}

#match {$foo} {$bar}
#when{foo bar} Hello {$foo} you have a {$var}
#when{  *   *} {$foo} hello you have a {$var}

#match {$foo :function option=value} {$bar :function option=value}
#when {a b} {{  {$foo} is {$bar}  }}
#when {x y} {||}  {$foo} is {$bar}  {||}
#when {* *} {|  |}{$foo} is {$bar}{|  |}

#input {$var :function option=value}#local $foo = {$bar :function option=value}Hello {$var}, you have a {$foo}

#match {$foo} {$bar}#when{foo bar} Hello {$foo} you have a {$var}#when{* *} {$foo} hello you have a {$var}

#match {$foo :function option=value} {$bar :function option=value}#when{a b}{{  {$foo} is {$bar}  }} #when{x y}{||}  {$foo} is {$bar}  {||}#when{* *}{|  |}{$foo} is {$bar}{|  |}
```

## 4. Reducing keywords

Avoids keywords in favor of sigil based parsing.
The theory here is that the syntactic sugar of `match` and `when` are nice the first time you use them
but the benefit is lost or reduced after that.

Uses double or paired sigils to reduce the need for escaping common characters (such as `?`)

Requires escaping `#` in a pattern.
Requires escaping `??` at the start of a pattern.
Requires escaping `::[` in a pattern.

```
#{$var :function option=value}
Hello {$var}

#{$var :function option=value}
#$foo = {$bar :function option=value}
Hello {$var}, you have a {$foo}

?? {$foo} {$bar}
::[ foo bar] Hello {$foo} you have a {$var}
::[ *     *] {$foo} hello you have a {$var}

?? {$foo :function option=value} {$bar :function option=value}
::[a b] {{  {$foo} is {$bar}  }}
::[x y] {{  {$foo} is {$bar}  }}
::[* *] {|  |}{$foo} is {$bar}{|  |}

#{$var :function option=value}#$foo = {$bar :function option=value}Hello {$var}, you have a {$foo}

??{$foo}{$bar}::[foo bar] Hello {$foo} you have a {$var}::[* *] {$foo} hello you have a {$var}

??{$foo :function option=value}{$bar :function option=value}::[a b] {  {$foo} is {$bar}  }::[x y] {  {$foo} is {$bar}  }::[* *] {|  |}{$foo} is {$bar}{|  |}
```

## 5. Use "blocks" for declarations and body

Use code-mode "blocks" to introduce code. _(This section has an additional example)_

The theory here is that a "declarations block" would be a natural add-on and it doesn't
require additional typing for each declaration.

Note too that this syntax could be extended to allow other types of blocks,
such as comments or different types of statement.

Requires escaping `[` when used as a pattern character in an unquoted pattern.

```
{#
  input {$var :function option=value}
}
Hello {$var}

// might be more natural as:
{#input {$var :function option=value}}
Hello {$var}

{#
   input {$var :function option=value}
   local $foo = {$bar :function option=value}
}
Hello {$var}, you have a {$foo}

{#
  match {$foo} {$bar}
  [ foo bar] Hello {$foo} you have a {$var}
  [ *     *] {$foo} hello you have a {$var}
}

{#
   input $foo :function option=value
}{
  match {$foo :function option=value} {$bar :function option=value}
  [a b] {{  {$foo} is {$bar}  }}
  [x y] {{  {$foo} is {$bar}  }}
  [* *] {|  |}{$foo} is {$bar}{|  |}
}

{#input {$var :function option=value}}Hello {$var}

{#input {$var :function option=value} local $foo = {$bar :function option=value}}Hello {$var}, you have a {$foo}

{#match {$foo} {$bar}[foo bar] Hello {$foo} you have a {$var}[* *] {$foo} hello you have a {$var}}

{#input {$foo :function option=value}match {$foo :function option=value} {$bar :function option=value}[a b] {{  {$foo} is {$bar}  }}[x y] {{  {$foo} is {$bar}  }}[* *] {|  |}{$foo} is {$bar}{|  |}}
```

## 5a. Declaration block but sigils for selectors

Start in text mode.
Enclose declarations in a block.
Use a sigil for `match` selector.

Requires escaping `[` when used as a pattern character,
and the sequence `#match` when used at the start of a pattern.

```
{# input $var :function option=value}
Hello {$var}

{# input $var :function option=value
   local $foo = {$bar :function option=value}
}
Hello {$var}, you have a {$foo}

#match {$foo :function option=value} {$bar :function option=value}
[a b] {$foo} is {$bar}
[x y] {$foo} is {$bar}
[* *] {$foo} is {$bar}

{#input $foo :function option=value
local $baz = {|some annotation|}}
#match {$foo} {$bar :function}
[a b] {$foo} is {$bar}
[x y] {$foo} is {$bar}
[* *] {$foo} is {$bar}
```

## 6. Use "statements"

Many languages delimit statements using a terminator, such as `;`.
In some languages, the terminator is the newline and lines can be extended using an escape like `\`.
Here we use `;` as a terminator.
Note that the closing `;` on `match` might be optional.

Defining a statement syntax might allow us to introduce different types of selectors in future versions,
such as looping constructs.

Requires escaping the word `when` and the character `;` in a pattern,
and the character `#` when used at the start of a pattern.

```
#input {$var :function option=value};
Hello {$var}

#input {$var :function option=value};
#local $foo = {$bar :function option=value};
Hello {$var}, you have a {$foo}

#match {$foo} {$bar}
when [ foo bar] Hello {$foo} you have a {$var}
when [ *     *] {$foo} hello you have a {$var}
;

#match {$foo :function option=value} {$bar :function option=value}
   when [a b] {  {$foo} is {$bar}  }
   when [x y] {  {$foo} is {$bar}  }
   when [* *] {|  |}{$foo} is {$bar}{|  |}
;

#input {$var :function option=value};#local $foo = {$bar :function option=value};Hello {$var}, you have a {$foo}

#match {$foo}{$bar}when [foo bar] Hello {$foo} you have a {$var}when [* *] {$foo} hello you have a {$var};

#match {$foo :function option=value}{$bar :function option=value}when [a b] {  {$foo} is {$bar}  }when[x y] {  {$foo} is {$bar}  }when[* *] {|  |}{$foo} is {$bar}{|  |};

```
