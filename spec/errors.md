## Errors

Errors can occur during the processing of a _message_.
Some errors can be detected statically, 
such as those due to problems with _message_ syntax,
violations of requirements in the data model,
or requirements defined by a _function_.
Other errors might be detected during selection or formatting of a given _message_.
Where available, the use of validation tools is recommended,
as early detection of errors makes their correction easier.

### Error Handling

_Syntax Errors_ and _Data Model Errors_ apply to all message processors,
and MUST be emitted as soon as possible.
The other error categories are only emitted during formatting,
but it might be possible to detect them with validation tools.

During selection and formatting,
_expression_ handlers MUST only emit _Message Function Errors_.

Implementations do not have to check for or emit _Resolution Errors_
or _Message Function Errors_ in _expressions_ that are not otherwise used by the _message_,
such as _placeholders_ in unselected _patterns_
or _declarations_ that are never referenced during _formatting_.

When formatting a _message_ with one or more errors,
an implementation MUST provide a mechanism to discover and identify
at least one of the errors. 
The exact form of error signaling is implementation defined.
Some examples include throwing an exception,
returning an error code, 
or providing a function or method for enumerating any errors.

For all _valid_ _messages_,
an implementation MUST enable a user to get a formatted result.
The formatted result might include _fallback values_ 
such as when a _placeholder_'s _expression_ produced an error
during formatting.

The two above requirements MAY be fulfilled by a single formatting method,
or separately by more than one such method.

When a message contains more than one error,
or contains some error which leads to further errors,
an implementation which does not emit all of the errors
SHOULD prioritise _Syntax Errors_ and _Data Model Errors_ over others.

When an error occurs while resolving a _selector_
or calling MatchSelectorKeys with its resolved value,
the _selector_ MUST NOT match any _variant_ _key_ other than the catch-all `*`
and a _Bad Selector_ error MUST be emitted.

### Syntax Errors

**_<dfn>Syntax Errors</dfn>_** occur when the syntax representation of a message is not _well-formed_.

> Example invalid messages resulting in a _Syntax Error_:
>
> ```
> {{Missing end braces
> ```
>
> ```
> {{Missing one end brace}
> ```
>
> ```
> Unknown {{expression}}
> ```
>
> ```
> .local $var = {|no message body|}
> ```

### Data Model Errors

**_<dfn>Data Model Errors</dfn>_** occur when a message is not _valid_ due to
violating one of the semantic requirements on its structure.

#### Variant Key Mismatch

A **_<dfn>Variant Key Mismatch</dfn>_** occurs when the number of keys on a _variant_
does not equal the number of _selectors_.

> Example invalid messages resulting in a _Variant Key Mismatch_ error:
>
> ```
> .input {$one :ns:func}
> .match $one
> 1 2 {{Too many}}
> * {{Otherwise}}
> ```
>
> ```
> .input {$one :ns:func}
> .input {$two :ns:func}
> .match $one $two
> 1 2 {{Two keys}}
> * {{Missing a key}}
> * * {{Otherwise}}
> ```

#### Missing Fallback Variant

A **_<dfn>Missing Fallback Variant</dfn>_** error occurs when the message
does not include a _variant_ with only catch-all keys.

> Example invalid messages resulting in a _Missing Fallback Variant_ error:
>
> ```
> .input {$one :ns:func}
> .match $one
> 1 {{Value is one}}
> 2 {{Value is two}}
> ```
>
> ```
> .input {$one :ns:func}
> .input {$two :ns:func}
> .match $one $two
> 1 * {{First is one}}
> * 1 {{Second is one}}
> ```

#### Missing Selector Annotation

A **_<dfn>Missing Selector Annotation</dfn>_** error occurs when the _message_
contains a _selector_ that does not
directly or indirectly reference a _declaration_ with a _function_.

> Examples of invalid messages resulting in a _Missing Selector Annotation_ error:
>
> ```
> .match $one
> 1 {{Value is one}}
> * {{Value is not one}}
> ```
>
> ```
> .local $one = {|The one|}
> .match $one
> 1 {{Value is one}}
> * {{Value is not one}}
> ```
>
> ```
> .input {$one}
> .match $one
> 1 {{Value is one}}
> * {{Value is not one}}
> ```

#### Duplicate Declaration

A **_<dfn>Duplicate Declaration</dfn>_** error occurs when a _variable_ is declared more than once.
Note that an input _variable_ is implicitly declared when it is first used,
so explicitly declaring it after such use is also an error.

> Examples of invalid messages resulting in a _Duplicate Declaration_ error:
>
> ```
> .input {$var :number maximumFractionDigits=0}
> .input {$var :number minimumFractionDigits=0}
> {{Redeclaration of the same variable}}
>
> .local $var = {$ext :number maximumFractionDigits=0}
> .input {$var :number minimumFractionDigits=0}
> {{Redeclaration of a local variable}}
>
> .input {$var :number minimumFractionDigits=0}
> .local $var = {$ext :number maximumFractionDigits=0}
> {{Redeclaration of an input variable}}
>
> .input {$var :number minimumFractionDigits=$var2}
> .input {$var2 :number}
> {{Redeclaration of the implicit input variable $var2}}
>
> .local $var = {$ext :ns:func}
> .local $var = {$error}
> .local $var2 = {$var2 :ns:error}
> {{{$var} cannot be redefined. {$var2} cannot refer to itself}}
> ```

#### Duplicate Option Name

A **_<dfn>Duplicate Option Name</dfn>_** error occurs when the same _identifier_
appears on the left-hand side of more than one _option_ in the same _expression_.

> Examples of invalid messages resulting in a _Duplicate Option Name_ error:
>
> ```
> Value is {42 :number style=percent style=decimal}
> ```
>
> ```
> .local $foo = {horse :ns:func one=1 two=2 one=1}
> {{This is {$foo}}}
> ```

#### Duplicate Variant

A **_<dfn>Duplicate Variant</dfn>_** error occurs when the
same list of _keys_ is used for more than one _variant_.

> Examples of invalid messages resulting in a _Duplicate Variant_ error:
>
> ```
> .input {$var :string}
> .match $var
> * {{The first default}}
> * {{The second default}}
> ```
>
> ```
> .input {$x :string}
> .input {$y :string}
> .match $x $y
> *   foo   {{The first "foo" variant}}
> bar *     {{The "bar" variant}}
> *   |foo| {{The second "foo" variant}}
> *   *     {{The default variant}}
> ```

### Resolution Errors

**_<dfn>Resolution Errors</dfn>_** occur when the runtime value of a part of a message
cannot be determined.

#### Unresolved Variable

An **_<dfn>Unresolved Variable</dfn>_** error occurs when a variable reference cannot be resolved.

> For example, attempting to format either of the following messages
> would result in an _Unresolved Variable_ error if done within a context that
> does not provide for the variable reference `$var` to be successfully resolved:
>
> ```
> The value is {$var}.
> ```
>
> ```
> .input {$var :ns:func}
> .match $var
> 1 {{The value is one.}}
> * {{The value is not one.}}
> ```

#### Unknown Function

An **_<dfn>Unknown Function</dfn>_** error occurs when an _expression_ includes
a reference to a function which cannot be resolved.

> For example, attempting to format either of the following messages
> would result in an _Unknown Function_ error if done within a context that
> does not provide for the function `:ns:func` to be successfully resolved:
>
> ```
> The value is {horse :ns:func}.
> ```
>
> ```
> .local $horse = {|horse| :ns:func}
> .match $horse
> 1 {{The value is one.}}
> * {{The value is not one.}}
> ```

#### Bad Selector

A **_<dfn>Bad Selector</dfn>_** error occurs when a message includes a _selector_
with a _resolved value_ which does not support selection.

> For example, attempting to format this message
> would result in a _Bad Selector_ error:
>
> ```
> .local $day = {|2024-05-01| :date}
> .match $day
> * {{The due date is {$day}}}
> ```

### Message Function Errors

A **_<dfn>Message Function Error</dfn>_** is any error that occurs
when calling a _function handler_
or which depends on validation associated with a specific function.

Implementations SHOULD provide a way for _function handlers_ to emit 
(or cause to be emitted) any of the types of error defined in this section.
Implementations MAY also provide implementation-defined _Message Function Error_ types.

> For example, attempting to format any of the following messages
> might result in a _Message Function Error_ if done within a context that
>
> 1. Provides for the variable reference `$user` to resolve to
>    an object `{ name: 'Kat', id: 1234 }`,
> 2. Provides for the variable reference `$field` to resolve to
>    a string `'address'`, and
> 3. Uses a `:ns:get` message function which requires its argument to be an object and
>    an option `field` to be provided with a string value.
>
> The exact type of _Message Function Error_ is determined by the _function handler_.
>
> ```
> Hello, {horse :ns:get field=name}!
> ```
>
> ```
> Hello, {$user :ns:get}!
> ```
>
> ```
> .local $id = {$user :ns:get field=id}
> {{Hello, {$id :ns:get field=name}!}}
> ```
>
> ```
> Your {$field} is {$id :ns:get field=$field}
> ```

#### Bad Operand

A **_<dfn>Bad Operand</dfn>_** error is any error that occurs due to the content or format of the _operand_,
such as when the _operand_ provided to a _function_ during _function resolution_ does not match one of the
expected implementation-defined types for that function;
or in which a literal _operand_ value does not have the required format
and thus cannot be processed into one of the expected implementation-defined types
for that specific _function_.

> For example, the following _messages_ each produce a _Bad Operand_ error
> because the literal `|horse|` does not match the `number-literal` production,
> which is a requirement of the function `:number` for its operand:
>
> ```
> .local $horse = {|horse| :number}
> {{You have a {$horse}.}}
> ```
>
> ```
> .local $horse = {|horse| :number}
> .match $horse
> 1 {{The value is one.}}
> * {{The value is not one.}}
> ```

#### Bad Option

A **_<dfn>Bad Option</dfn>_** error is an error that occurs when there is
an implementation-defined error with an _option_ or its value.
These might include:
- A required _option_ is missing.
- Mutually exclusive _options_ are supplied.
- An _option_ value provided to a _function_ during _function resolution_
   does not match one of the implementation-defined types or values for that _function_;
   or in which the literal _option_ value does not have the required format
   and thus cannot be processed into one of the expected
   implementation-defined types for that specific _function_.

> For example, the following _message_ might produce a _Bad Option_ error
> because the literal `foo` does not match the production `digit-size-option`,
> which is a requirement of the function `:number` for its `minimumFractionDigits` _option_:
>
> ```
> The answer is {42 :number minimumFractionDigits=foo}.
> ```

#### Bad Variant Key

A **_<dfn>Bad Variant Key</dfn>_** error is an error that occurs when a _variant_ _key_
does not match the expected implementation-defined format.

> For example, the following _message_ produces a _Bad Variant Key_ error
> because `horse` is not a recognized plural category and
> does not match the `number-literal` production,
> which is a requirement of the `:number` function:
>
> ```
> .local $answer = {42 :number}
> .match $answer
> 1     {{The value is one.}}
> horse {{The value is a horse.}}
> *     {{The value is not one.}}
> ```

#### Unsupported Operation

A **_<dfn>Unsupported Operation</dfn>_** error is an implementation-specific error
that occurs when a given _option_, _option_ value, _operand_ value, or some combination
of these are incompatible or not supported by a given _function_ or its _function handler_.
