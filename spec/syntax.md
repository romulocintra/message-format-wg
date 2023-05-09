# DRAFT MessageFormat 2.0 Syntax

## Table of Contents

1. [Introduction](#introduction)
   1. [Design Goals](#design-goals)
   1. [Design Restrictions](#design-restrictions)
1. [Overview & Examples](#overview--examples)
   1. [Messages](#messages)
   1. [Expressions](#expression)
   1. [Formatting Functions](#formatting-functions)
   1. [Selection](#selection)
   1. [Local Variables](#local-variables)
   1. [Complex Messages](#complex-messages)
1. [Productions](#productions)
   1. [Message](#message)
   1. [Variable Declarations](#variable-declarations)
   1. [Selectors](#selectors)
   1. [Variants](#variants)
   1. [Patterns](#patterns)
   1. [Expressions](#expressions)
       1. [Reserved Sequences](#reserved)
1. [Tokens](#tokens)
   1. [Keywords](#keywords)
   1. [Text and Literals](#text-and-literals)
   1. [Names](#names)
   1. [Escape Sequences](#escape-sequences)
   1. [Whitespace](#whitespace)
1. [Complete ABNF](#complete-abnf)

### Introduction

This section defines the formal grammar describing the syntax of a single message.

### Design Goals

_This section is non-normative._

The design goals of the syntax specification are as follows:

1. The syntax should leverage the familiarity with ICU MessageFormat 1.0
   in order to lower the barrier to entry and increase the chance of adoption.
   At the same time,
   the syntax should fix the [pain points of ICU MessageFormat 1.0](../docs/why_mf_next.md).

   - _Non-Goal_: Be backwards-compatible with the ICU MessageFormat 1.0 syntax.

1. The syntax inside translatable content should be easy to understand for humans.
   This includes making it clear which parts of the message body _are_ translatable content,
   which parts inside it are placeholders for expressions,
   as well as making the selection logic predictable and easy to reason about.

   - _Non-Goal_: Make the syntax intuitive enough for non-technical translators to hand-edit.
     Instead, we assume that most translators will work with MessageFormat 2.0
     by means of GUI tooling, CAT workbenches etc.

1. The syntax surrounding translatable content should be easy to write and edit
   for developers, localization engineers, and easy to parse by machines.

1. The syntax should make a single message easily embeddable inside many container formats:
   `.properties`, YAML, XML, inlined as string literals in programming languages, etc.
   This includes a future _MessageResource_ specification.

   - _Non-Goal_: Support unnecessary escape sequences, which would theirselves require
     additional escaping when embedded. Instead, we tolerate direct use of nearly all
     characters (including line breaks, control characters, etc.) and rely upon escaping
     in those outer formats to aid human comprehension (e.g., depending upon container
     format, a U+000A LINE FEED might be represented as `\n`, `\012`, `\x0A`, `\u000A`,
     `\U0000000A`, `&#xA;`, `&NewLine;`, `%0A`, `<LF>`, or something else entirely).

### Design Restrictions

_This section is non-normative._

The syntax specification takes into account the following design restrictions:

1. Whitespace outside the translatable content should be insignificant.
   It should be possible to define a message entirely on a single line with no ambiguity,
   as well as to format it over multiple lines for clarity.

1. The syntax should define as few special characters and sigils as possible.
   Note that this necessitates extra care when presenting messages for human consumption,
   because they may contain invisible characters such as U+200B ZERO WIDTH SPACE,
   control characters such as U+0000 NULL and U+0009 TAB, permanently reserved noncharacters
   (U+FDD0 through U+FDEF and U+<i>n</i>FFFE and U+<i>n</i>FFFF where <i>n</i> is 0x0 through 0x10),
   private-use code points (U+E000 through U+F8FF, U+F0000 through U+FFFFD, and
   U+100000 through U+10FFFD), unassigned code points, and other potentially confusing content.

## Overview & Examples

_This section is non-normative._

### Messages

All messages, including simple ones, are enclosed in `{…}` delimiters:

    {Hello, world!}

The same message defined in a `.properties` file:

```properties
app.greetings.hello = {Hello, world!}
```

The same message defined inline in JavaScript:

```js
let hello = new MessageFormat('{Hello, world!}')
hello.format()
```

### Expression

An _expression_ represents a part of a message that will be determined
during the message's formatting.

An _expression_ always uses `{…}` delimiters.
An _expression_ can appear as a local variable value, as a _selector_, and within a _pattern_.

A simple _expression_ is a bare variable name:

    {Hello, {$userName}!}

### Formatting Functions

A _function_ is named functionality, possibly with _options_, that format,
process, or operate on a _variable_.

For example, a _message_ with an interpolated `$date` _variable_ formatted with the `:datetime` _function_:

    {Today is {$date :datetime weekday=long}.}

A _message_ with an interpolated `$userName` _variable_ formatted with
the custom `:person` _function_ capable of
declension (using either a fixed dictionary, algorithmic declension, ML, etc.):

    {Hello, {$userName :person case=vocative}!}

A _message_ with an interpolated `$userObj` _variable_ formatted with
the custom `:person` _function_ capable of
plucking the first name from the object representing a person:

    {Hello, {$userObj :person firstName=long}!}

Functions use one of the following prefix sigils:

- `:` for standalone content
- `+` for starting or opening elements
- `-` for ending or closing elements

A message with two markup-like _functions_, `button` and `link`,
which the runtime can use to construct a document tree structure for a UI framework:

    {{+button}Submit{-button} or {+link}cancel{-link}.}

An opening element MAY be present in a message without a corresponding closing element,
and vice versa.

### Selection

A _selector_ selects a specific _pattern_ from a list of available _patterns_
in a _message_ based on the value of its _expression_.
A message can have multiple selectors.

A message with a single _selector_:

    match {$count :number}
    when 1 {You have one notification.}
    when * {You have {$count} notifications.}

A message with a single _selector_ which is an invocation of
a custom function `:platform`, formatted on a single line:

    match {:platform} when windows {Settings} when * {Preferences}

A message with a single _selector_ and a custom `:hasCase` function
which allows the message to query for presence of grammatical cases required for each variant:

    match {$userName :hasCase}
    when vocative {Hello, {$userName :person case=vocative}!}
    when accusative {Please welcome {$userName :person case=accusative}!}
    when * {Hello!}

A message with 2 _selectors_:

    match {$photoCount :number} {$userGender :equals}
    when 1 masculine {{$userName} added a new photo to his album.}
    when 1 feminine {{$userName} added a new photo to her album.}
    when 1 * {{$userName} added a new photo to their album.}
    when * masculine {{$userName} added {$photoCount} photos to his album.}
    when * feminine {{$userName} added {$photoCount} photos to her album.}
    when * * {{$userName} added {$photoCount} photos to their album.}

### Local Variables

A _message_ can define local variables,
such as might be needed for transforming input
or providing additional data to an _expression_.
Local variables appear in a _declaration_,
which defines the value of a named local variable.

A _message_ containing a _declaration_ defining a local variable `$whom` which is then used twice inside the pattern:

    let $whom = {$monster :noun case=accusative}
    {You see {$quality :adjective article=indefinite accord=$whom} {$whom}!}

A message defining two local variables:
`$itemAcc` and `$countInt`, and using `$countInt` as a selector:

    let $countInt = {$count :number maximumFractionDigits=0}
    let $itemAcc = {$item :noun count=$count case=accusative}
    match {$countInt}
    when one {You bought {$color :adjective article=indefinite accord=$itemAcc} {$itemAcc}.}
    when * {You bought {$countInt} {$color :adjective accord=$itemAcc} {$itemAcc}.}

### Complex Messages

The various features can be used to produce arbitrarily complex messages by combining
_declarations_, _selectors_, _functions_, and more.

A complex message with 2 _selectors_ and 3 local variable _declarations_:

    let $hostName = {$host :person firstName=long}
    let $guestName = {$guest :person firstName=long}
    let $guestsOther = {$guestCount :number offset=1}

    match {$host :gender} {$guestOther :number}

    when female 0 {{$hostName} does not give a party.}
    when female 1 {{$hostName} invites {$guestName} to her party.}
    when female 2 {{$hostName} invites {$guestName} and one other person to her party.}
    when female * {{$hostName} invites {$guestName} and {$guestsOther} other people to her party.}

    when male 0 {{$hostName} does not give a party.}
    when male 1 {{$hostName} invites {$guestName} to his party.}
    when male 2 {{$hostName} invites {$guestName} and one other person to his party.}
    when male * {{$hostName} invites {$guestName} and {$guestsOther} other people to his party.}

    when * 0 {{$hostName} does not give a party.}
    when * 1 {{$hostName} invites {$guestName} to their party.}
    when * 2 {{$hostName} invites {$guestName} and one other person to their party.}
    when * * {{$hostName} invites {$guestName} and {$guestsOther} other people to their party.}

## Productions

The specification defines the following grammar productions.

A message satisfying all rules of the grammar is considered _well-formed_.

Furthermore, a well-formed message is considered _valid_
if it meets additional semantic requirements about its structure, defined below.

### Message

A **_message_** is a (possibly empty) list of _declarations_ followed by either a single _pattern_,
or a `match` statement followed by one or more _variants_ which represent the translatable body of the message.

A _message_ MUST be delimited with `{` at the start, and `}` at the end. Whitespace MAY
appear outside the delimiters; such whitespace is ignored. No other content is permitted
outside the delimiters.

```abnf
message = [s] *(declaration [s]) body [s]
body = pattern
     / (selectors 1*([s] variant))
```

### Variable Declarations

A **_declaration_** is an expression binding a variable identifier
within the scope of the message to the value of an expression.
This local variable can then be used in other expressions within the same message.

```abnf
declaration = let s variable [s] "=" [s] expression
```

### Selectors

A `match` statement contains one or more **_selectors_**
which will be used to choose one of the _variants_ during formatting.

```abnf
selectors = match 1*([s] expression)
```

Examples:

```
match {$count :plural}
when 1 {One apple}
when * {{$count} apples}
```

```
let $frac = {$count: number minFractionDigits=2}
match {$frac}
when 1 {One apple}
when * {{$frac} apples}
```

### Variants

A **_variant_** is a keyed _pattern_.
The keys are used to match against the _selectors_ defined in the `match` statement.
The key `*` is a "catch-all" key, matching all selector values.

```abnf
variant = when 1*(s key) [s] pattern
key = nmtoken / literal / "*"
```

A _well-formed_ message is considered _valid_ if the following requirements are satisfied:

- The number of keys on each _variant_ MUST be equal to the number of _selectors_.
- At least one _variant's_ keys MUST all be equal to the catch-all key (`*`).

### Patterns

A **_pattern_** is a sequence of translatable elements.
Patterns MUST be delimited with `{` at the start, and `}` at the end.
This serves 3 purposes:

- The message can be unambiguously embeddable in various container formats
  regardless of the container's whitespace trimming rules.
  E.g. in Java `.properties` files,
  `hello = {Hello}` will unambiguously define the `Hello` message without the space in front of it.
- The message can be conveniently embeddable in various programming languages
  without the need to escape characters commonly related to strings, e.g. `"` and `'`.
  Such need might still occur when a single or double quote is
  used in the translatable content.
- The syntax needs to make it as clear as possible which parts of the message body
  are translatable and which ones are part of the formatting logic definition.

```abnf
pattern = "{" *(text / expression) "}"
```

Examples:

```
{Hello, world!}
```

Whitespace within a _pattern_ is meaningful and MUST be preserved.

### Expressions

_Expressions_ ***must*** start with a _literal_, a _variable_, or an _annotation_. An _expression_ ***must not*** be empty.

A _literal_ or _variable_ ***may*** be optionally followed by an _annotation_. 

An _annotation_ consists of a _function_ and its named _options_, or consists of a _reserved_ sequence.

_Functions_ do not accept any positional arguments other than the _literal_ or _variable_ in front of them.

```abnf
expression = "{" [s] (((literal / variable) [s annotation]) / annotation) [s] "}"
annotation = (function *(s option)) / reserved
option = name [s] "=" [s] (literal / nmtoken / variable)
```

Expression examples:

```
{|1.23|}
```

```
{|1.23| :number maxFractionDigits=1}
```

```
{|Thu Jan 01 1970 14:37:00 GMT+0100 (CET)| :datetime weekday=long}
```

```
{$when :datetime month=2-digit}
```

```
{:message id=some_other_message}
```

```
{+ssml.emphasis level=strong}
```

Message examples:

```
{This is {+b}bold{-b}.}
```

```
{{+h1 name=above-and-beyond}Above And Beyond{-h1}}
```

#### Reserved

_Reserved_ sequences start with a reserved character and are intended for future standardization. 
A reserved sequence can be empty or contain arbitrary text. 
A reserved sequence does not include any trailing whitespace.
While a reserved sequence is technically "well-formed", unrecognized reserved sequences have no meaning and might result in errors during formatting.

## Tokens

The grammar defines the following tokens for the purpose of the lexical analysis.

### Keywords

The following three keywords are reserved: `let`, `match`, and `when`.

```abnf
; reserved keywords are always lowercase
let   = %x6C.65.74        ; "let"
match = %x6D.61.74.63.68  ; "match"
when  = %x77.68.65.6E     ; "when"
```

### Text and Literals

_Text_ is the translatable content of a _pattern_, and _Literal_ is used for matching
variants and providing input to expressions.
Any Unicode code point is allowed in either, with the exception of
the relevant delimiters (`{` and `}` for Text, `|` for Literal),
`\` (which starts an escape sequence), and
surrogate code points U+D800 through U+DFFF (which cannot be encoded into UTF-8).

All code points are preserved.

```abnf
text = 1*(text-char / text-escape)
text-char = %x0-5B         ; omit \
          / %x5D-7A        ; omit {
          / %x7C           ; omit }
          / %x7E-D7FF      ; omit surrogates
          / %xE000-10FFFF
```

```abnf
literal = "|" *(literal-char / literal-escape) "|"
literal-char = %x0-5B         ; omit \
             / %x5D-7B        ; omit |
             / %x7D-D7FF      ; omit surrogates
             / %xE000-10FFFF
```

### Names

The _name_ token is used for variable names (prefixed with `$`),
function names (prefixed with `:`, `+` or `-`),
as well as option names.
A name MUST NOT start with an ASCII digit and certain basic combining characters.
Otherwise, the set of characters allowed in names is large.

The _nmtoken_ token doesn't have _name_'s restriction on the first character
and is used as variant keys and option values.

_Note:_ _nmtoken_ is intentionally defined to be the same as XML's [Nmtoken](https://www.w3.org/TR/xml/#NT-Nmtoken)
in order to increase the interoperability with data defined in XML.
In particular, the grammatical data [specified in LDML](https://unicode.org/reports/tr35/tr35-general.html#Grammatical_Features)
and [defined in CLDR](https://unicode-org.github.io/cldr-staging/charts/latest/grammar/index.html)
uses Nmtokens.

```abnf
variable = "$" name
function = (":" / "+" / "-") name
```

```abnf
name    = name-start *name-char ; based on https://www.w3.org/TR/xml/#NT-Name, but cannot start with U+003A COLON ":"
nmtoken = 1*name-char           ; equal to https://www.w3.org/TR/xml/#NT-Nmtoken
name-start = ALPHA / "_"
           / %xC0-D6 / %xD8-F6 / %xF8-2FF
           / %x370-37D / %x37F-1FFF / %x200C-200D
           / %x2070-218F / %x2C00-2FEF / %x3001-D7FF
           / %xF900-FDCF / %xFDF0-FFFD / %x10000-EFFFF
name-char = name-start / DIGIT / "-" / "." / ":"
          / %xB7 / %x0300-036F / %x203F-2040
```

### Escape Sequences

Escape sequences are introduced by the backslash character (`\`) and allow the appearance of lexically meaningful characters in the body of `text`, `literal`, or `reserved` sequences respectively:

```abnf
text-escape    = backslash ( backslash / "{" / "}" )
literal-escape = backslash ( backslash / "|" )
reserve-escape = backslash ( backslash / "{" / "|" / "}" )
backslash      = %x5C ; U+005C REVERSE SOLIDUS "\"
```

### Whitespace

**_Whitespace_** is defined as tab, carriage return, line feed, or the space character.

Inside _patterns_,
whitespace is part of the translatable content and is recorded and stored verbatim.
Whitespace is not significant outside translatable text, except where required by the syntax.


```abnf
s = 1*( SP / HTAB / CR / LF )
```

## Complete ABNF

The grammar is formally defined in [`message.abnf`](./message.abnf)
using the ABNF notation,
as specified by [RFC 5234](https://datatracker.ietf.org/doc/html/rfc5234).
