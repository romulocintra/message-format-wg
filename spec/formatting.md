# WIP DRAFT MessageFormat 2.0 Formatting Behaviour

## Introduction

This document defines the behaviour of a MessageFormat 2.0 implementation
when formatting a message for display in a user interface, or for some later processing.

The document is part of the MessageFormat 2.0 specification,
the successor to ICU MessageFormat, henceforth called ICU MessageFormat 1.0.

## Variable Resolution

To resolve the value of a Variable,
its Name is used to identify either a local variable,
or a variable defined elsewhere.
If a local variable and an externally defined one use the same name,
the local variable takes precedence.

It is an error for a local variable definition to
refer to a local variable that's defined after it in the message.

## Pattern Selection

When formatting a message with one or more _selectors_,
the _pattern_ of one of the _variants_ must be selected for formatting.

When a message has a single _selector_,
an implementation-defined method compares each key to the _selector_
and determines which of the keys match, and in what order of preference.
A catch-all key will always match, but is always the least preferred choice.
During selection, the _variant_ with the best-matching key is selected.

In a message with more than one _selector_,
each _variant_ also has a corresponding number of keys.
These correspond to _selectors_ by position.
The same implementation-defined method as above is used to compare
the corresponding key of each _variant_ to its _selector_,
to determine which of the keys match, and in what order of preference.
In order to select a single _variant_,
the full list of _variants_ will need to be filtered and sorted.
First, each _variant_ with a key that does not match its _selector_ is left out.
Then, the remaining _variants_ are sorted lexicographically by key preference,
with earlier _selectors_ having higher priority than later ones.
Finally, the highest-sorted _variant_ is selected.

This selection method is defined in more detail below.
An implementation MAY use any pattern selection method,
as long as its observable behaviour matches the results of the method defined here.

### Resolve Selectors

First, resolve the values of each _selector_:

1. Let `res` be a new empty list of resolved values that support selection.
1. For each _expression_ `exp` of the message's _selectors_,
   1. Let `rv` be the resolved value of `exp`.
   1. If selection is supported for `rv`:
      1. Append `rv` as the last element of the list `res`.
   1. Else:
      1. Let `nomatch` be a resolved value for which selection always fails.
      1. Append `nomatch` as the last element of the list `res`.
      1. Emit a Selection Error.

The shape of the resolved values is determined by each implementation,
along with the manner of determining their support for selection.

### Resolve Preferences

Next, using `res`, resolve the preferential order for all message keys:

1. Let `pref` be a new empty list of lists of strings.
1. For each index `i` in `res`:
   1. Let `keys` be a new empty list of strings.
   1. For each _variant_ `var` of the message:
      1. Let `key` be the `var` key at position `i`.
      1. If `key` is not the catch-all key `'*'`:
         1. Let `ks` be the decoded value of `key`.
         1. Append `ks` as the last element of the list `keys`.
   1. Let `rv` be the resolved value at index `i` of `res`.
   1. Let `matches` be the result of calling the method MatchSelectorKeys(`rv`, `keys`)
   1. Append `matches` as the last element of the list `pref`.

The method MatchSelectorKeys is determined by the implementation.
It takes as arguments a resolved _selector_ value `rv` and a list of string keys `keys`,
and returns a list of string keys in preferential order.
The returned list must only contain unique elements of the input list `keys`,
and it may be empty.
The most preferred key is first, with the rest sorted by decreasing preference.

### Filter Variants

Then, using the preferential key orders `pref`,
filter the list of _variants_ to the ones that match with some preference:

1. Let `vars` be a new empty list of _variants_.
1. For each _variant_ `var` of the message:
   1. For each index `i` in `pref`:
      1. Let `key` be the `var` key at position `i`.
      1. If `key` is the catch-all key `'*'`:
         1. Continue the inner loop on `pref`.
      1. Let `ks` be the decoded value of `key`.
      1. Let `matches` be the list of strings at index `i` of `pref`.
      1. If `matches` includes `ks`:
         1. Continue the inner loop on `pref`.
      1. Else:
         1. Continue the outer loop on message _variants_.
   1. Append `var` as the last element of the list `vars`.

### Sort Variants

Finally, sort the list of variants `vars` and select the _pattern_:

1. Let `sortable` be a new empty list of (integer, _variant_) tuples.
1. For each _variant_ `var` of `vars`:
   1. Let `tuple` be a new tuple (-1, `var`).
   1. Append `tuple` as the last element of the list `sortable`.
1. Let `len` be the integer count of items in `pref`.
1. Let `i` be `len` - 1.
1. While `i` >= 0:
   1. Let `matches` be the list of strings at index `i` of `pref`.
   1. Let `minpref` be the integer count of items in `matches`.
   1. For each tuple `tuple` of `sortable`:
      1. Let `matchpref` be an integer with the value `minpref`.
      1. Let `key` be the `tuple` _variant_ key at position `i`.
      1. If `key` is not the catch-all key `'*'`:
         1. Let `ks` be the decoded value of `key`.
         1. Let `matchpref` be the integer position of `ks` in `matches`.
      1. Set the `tuple` integer value as `matchpref`.
   1. Call the method SortVariants(`sortable`).
   1. Set `i` to be `i` - 1.
1. Let `var` be the _variant_ element of the first element of `sortable`.
1. Select the _pattern_ of `var`.

The method SortVariants is determined by the implementation.
It takes as an argument a `sortable` list of (integer, _variant_) tuples,
which it modifies in place using some stable sorting algorithm.
The method does not return anything.
The list is sorted according to the tuple's first integer element,
such that a lower number is sorted before a higher one,
and entries that have the same number retain their order.

### Examples

Presuming a minimal implementation which only supports string values
and matches keys by using string comparison,
and a formatting context in which
the variable reference `$foo` resolves to the string `'foo'` and
the variable reference `$bar` resolves to the string `'bar'`,
pattern selection proceeds as follows for this message:

```
match {$foo} {$bar}
when bar bar {All bar}
when foo foo {All foo}
when * * {Otherwise}
```

1. For the first selector:<br>
   The value of the selector is resolved to be `'foo'`.<br>
   The available keys « `'bar'`, `'foo'` » are compared to `'foo'`,<br>
   resulting in a list « `'foo'` » of matching keys.

2. For the second selector:<br>
   The value of the selector is resolved to be `'bar'`.<br>
   The available keys « `'bar'`, `'foo'` » are compared to `'bar'`,<br>
   resulting in a list « `'bar'` » of matching keys.

3. Creating the list `vars` of variants matching all keys:<br>
   The first variant `bar bar` is discarded as its first key does not match the first selector.<br>
   The second variant `foo foo` is discarded as its second key does not match the second selector.<br>
   The catch-all keys of the third variant `* *` always match, and this is added to `vars`,<br>
   resulting in a list « `* *` » of variants.

4. As the list `vars` only has one entry, it does not need to be sorted.<br>
   The pattern `{Otherwise}` of the third variant is selected.

Alternatively, with the same implementation and formatting context,
pattern selection would proceed as follows for this message:

```
match {$foo} {$bar}
when * bar {Any and bar}
when foo * {Foo and any}
when foo bar {Foo and bar}
when * * {Otherwise}
```

1. For the first selector:<br>
   The value of the selector is resolved to be `'foo'`.<br>
   The available keys « `'foo'` » are compared to `'foo'`,<br>
   resulting in a list « `'foo'` » of matching keys.

2. For the second selector:<br>
   The value of the selector is resolved to be `'bar'`.<br>
   The available keys « `'bar'` » are compared to `'bar'`,<br>
   resulting in a list « `'bar'` » of matching keys.

3. Creating the list `vars` of variants matching all keys:<br>
   The keys of all variants either match each selector exactly, or via the catch-all key,<br>
   resulting in a list « `* bar`, `foo *`, `foo bar`, `* *` » of variants.

4. Sorting the variants:<br>
   The list `sortable` is first set with the variants in their source order
   and scores determined by the second selector:<br>
   « ( 0, `* bar` ), ( 1, `foo *` ), ( 0, `foo bar` ), ( 1, `* *` ) »<br>
   This is then sorted as:<br>
   « ( 0, `* bar` ), ( 0, `foo bar` ), ( 1, `foo *` ), ( 1, `* *` ) ».<br>
   To sort according to the first selector, the scores are updated to:<br>
   « ( 1, `* bar` ), ( 0, `foo bar` ), ( 0, `foo *` ), ( 1, `* *` ) ».<br>
   This is then sorted as:<br>
   « ( 0, `foo bar` ), ( 0, `foo *` ), ( 1, `* bar` ), ( 1, `* *` ) ».<br>

5. The pattern `{Foo and bar}` of the most preferred `foo bar` variant is selected.

## Error Handling

Errors in messages and their formatting may occur and be detected
at multiple different stages of their processing.
Where available,
the use of validation tools is recommended,
as early detection of errors makes their correction easier.

During the formatting of a message,
various errors may be encountered.
These are divided into the following categories:

- **Syntax errors** occur when the syntax representation of a message is not well-formed.

  Example invalid messages resulting in a Syntax error:

  ```
  {Missing end brace
  ```

  ```
  {Unknown {#placeholder#}}
  ```

  ```
  let $var = {|no message body|}
  ```

- **Data Model errors** occur when a message is invalid due to
  violating one of the semantic requirements on its structure:

  - **Variant Key Mismatch errors** occur when the number of keys on a Variant
    does not equal the number of Selectors.

    Example invalid messages resulting in a Variant Key Mismatch error:

    ```
    match {$one}
    when 1 2 {Too many}
    when * {Otherwise}
    ```

    ```
    match {$one} {$two}
    when 1 2 {Two keys}
    when * {Missing a key}
    when * * {Otherwise}
    ```

  - **Missing Fallback Variant errors** occur when the message
    does not include a Variant with only catch-all keys.

    Example invalid messages resulting in a Missing Fallback Variant error:

    ```
    match {$one}
    when 1 {Value is one}
    when 2 {Value is two}
    ```

    ```
    match {$one} {$two}
    when 1 * {First is one}
    when * 1 {Second is one}
    ```

- **Resolution errors** occur when the runtime value of a part of a message
  cannot be determined.

  - **Unresolved Variable errors** occur when a variable reference cannot be resolved.

    For example, attempting to format either of the following messages
    must result in an Unresolved Variable error if done within a context that
    does not provide for the variable reference `$var` to be successfully resolved:

    ```
    {The value is {$var}.}
    ```

    ```
    match {$var}
    when 1 {The value is one.}
    when * {The value is not one.}
    ```

  - **Unknown Function errors** occur when an Expression includes
    a reference to a function which cannot be resolved.

    For example, attempting to format either of the following messages
    must result in an Unknown Function error if done within a context that
    does not provide for the function `:func` to be successfully resolved:

    ```
    {The value is {|horse| :func}.}
    ```

    ```
    match {|horse| :func}
    when 1 {The value is one.}
    when * {The value is not one.}
    ```

- **Selection errors** occur when message selection fails.

  - **Selector errors** are failures in the matching of a key to a specific selector.

    For example, attempting to format either of the following messages
    might result in a Selector error if done within a context that
    uses a `:plural` selector function which requires its input to be numeric:

    ```
    match {|horse| :plural}
    when 1 {The value is one.}
    when * {The value is not one.}
    ```

    ```
    let $sel = {|horse| :plural}
    match {$sel}
    when 1 {The value is one.}
    when * {The value is not one.}
    ```

- **Formatting errors** occur during the formatting of a resolved value,
  for example when encountering a value with an unsupported type
  or an internally inconsistent set of options.

  For example, attempting to format any of the following messages
  might result in a Formatting error if done within a context that

  1. provides for the variable reference `$user` to resolve to
     an object `{ name: 'Kat', id: 1234 }`,
  2. provides for the variable reference `$field` to resolve to
     a string `'address'`, and
  3. uses a `:get` formatting function which requires its argument to be an object and
     an option `field` to be provided with a string value,

  ```
  {Hello, {|horse| :get field=name}!}
  ```

  ```
  {Hello, {$user :get}!}
  ```

  ```
  let $id = {$user :get field=id}
  {Hello, {$id :get field=name}!}
  ```

  ```
  {Your {$field} is {$id :get field=$field}}
  ```

Syntax and Data Model errors must be emitted as soon as possible.

During selection, an Expression handler must only emit Resolution and Selection errors.
During formatting, an Expression handler must only emit Resolution and Formatting errors.

In all cases, when encountering an error,
a message formatter must provide some representation of the message.
An informative error or errors must also be separately provided.
When a message contains more than one error,
or contains some error which leads to further errors,
an implementation which does not emit all of the errors
should prioritise Syntax and Data Model errors over others.

When an error occurs in the resolution of an Expression or Markup Option,
the Expression or Markup in question is processed as if the option were not defined.
This may allow for the fallback handling described below to be avoided,
though an error must still be emitted.

When an error occurs within a Selector,
the selector must not match any VariantKey other than the catch-all `*`
and a Resolution or Selector error is emitted.

## Fallback String Representations

The formatted string representation of a message with a Syntax or Data Model error
is the concatenation of U+007B LEFT CURLY BRACKET `{`,
a fallback string,
and U+007D RIGHT CURLY BRACKET `}`.
If a fallback string is not defined,
the U+FFFD REPLACEMENT CHARACTER `�` character is used,
resulting in the string `{�}`.

When an error occurs in a Placeholder that is being formatted,
the fallback string representation of the Placeholder
always starts with U+007B LEFT CURLY BRACKET `{`
and ends with U+007D RIGHT CURLY BRACKET `}`.
Between the brackets, the following contents are used:

- Expression with Literal Operand: U+007C VERTICAL LINE `|`
  followed by the value of the Literal,
  and then by U+007C VERTICAL LINE `|`

  Examples: `{|horse|}`, `{|42|}`

- Expression with Variable Operand: U+0024 DOLLAR SIGN `$`
  followed by the Variable Name of the Operand

  Example: `{$user}`

- Expression with no Operand: U+003A COLON `:` followed by the Expression Name

  Example: `{:platform}`

- Markup start: U+002B PLUS SIGN `+` followed by the MarkupStart Name

  Example: `{+tag}`

- Markup end: U+002D HYPHEN-MINUS `-` followed by the MarkupEnd Name

  Example: `{-tag}`

- Otherwise: The U+FFFD REPLACEMENT CHARACTER `�` character

  Example: `{�}`

Option names and values are not included in the fallback string representations.

When an error occurs in an Expression with a Variable Operand
and the Variable refers to a local variable Declaration,
the fallback string is formatted based on the Expression of the Declaration,
rather than the Expression of the Placeholder.

For example, attempting to format either of the following messages within a context that
does not provide for the function `:func` to be successfully resolved:

```
let $var = {|horse| :func}
{The value is {$var}.}
```

```
let $var = {|horse|}
{The value is {$var :func}.}
```

would result in both cases with this formatted string representation:

```
The value is {|horse|}.
```
