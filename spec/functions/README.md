## Default Functions

### Table of Contents

1. [String Value Selection and Formatting](string.md)
   1. [`:string`](string.md#the-string-function)
1. [Numeric Value Selection and Formatting](number.md)
   1. [`:number`](number.md#the-number-function)
   1. [`:integer`](number.md#the-integer-function)
   1. [`:math`](number.md#the-math-function)
   1. [`:currency`](number.md#the-currency-function)
   1. [`:unit`](number.md#the-unit-function)
1. [Date and Time Value Formatting](datetime.md)
   1. [`:datetime`](datetime.md#the-datetime-function)
   1. [`:date`](datetime.md#the-date-function)
   1. [`:time`](datetime.md#the-time-function)

This section defines the **_<dfn>default functions</dfn>_**
which are REQUIRED for conformance with this specification,
along with _default functions_ that SHOULD be implemented to support
additional functionality.

> [!IMPORTANT]
> _Functions_ that are not marked **Draft** are **Stable** and subject to
> the provisions of the [stability policy](../intro.md#stability-policy).
>
> _Functions_ and _options_ marked as **Draft** are not stable.
> Their name, _operands_, and _options_/_values_, and other requirements
> might change or be removed before being declared **Stable** in a future release.
>
> Some **Draft** _functions_ or _options_ are also marked as **Proposed**.
> These are additions to the specification that have not yet reached Technical Preview.

To **_<dfn>accept</dfn>_** a function means that an implementation MUST NOT
emit an _Unknown Function_ error for that _function_'s _identifier_.
To _accept_ an _option_ means that a _function handler_ MUST NOT
emit a _Bad Option_ error for that _option_'s _identifier_ when used with the _function_
it is defined for
and MUST NOT emit a _Bad Option_ error for any of the _option_ values
defined for that _option_.
Accepting a _function_ or its _options_ does not mean that a particular output is produced.
Implementations MAY emit an _Unsupported Operation_ error for _options_
or _option_ values that they cannot support.

_Functions_ can define _options_. 
An _option_ can be REQUIRED or RECOMMENDED.

Implementations MUST _accept_ each REQUIRED _default function_ and
MUST _accept_ all _options_ defined as REQUIRED for those _functions_.

Implementations SHOULD _accept_ each RECOMMENDED _default function_.
For each such _function_, the implementation MUST accept all _options_
listed as REQUIRED for that _function_.

Implementations SHOULD _accept_ _options_ that are marked as RECOMMENDED.

Implementations MAY _accept_ _functions_ not defined in this specification.
In addition, implementations SHOULD provide mechanisms for users to
register and use user-defined _functions_ and their associated _function handlers_.
Functions not defined by any version of this specification SHOULD use 
an implementation-defined or user-defined _namespace_.

Implementations MAY implement additional _options_ not defined
by any version of this specification for _default functions_.
Such _options_ MUST use an implementation-specific _namespace_.

Implementations MAY _accept_, for _options_ defined in this specification,
_option_ values which are not defined in this specification.
However, such values might become defined with a different meaning in the future,
including with a different, incompatible name
or using an incompatible value space.
Supporting implementation-specific _option_ values for _default functions_ is NOT RECOMMENDED.

Implementations MAY _accept_, for _operands_ or _options_ defined in this specification,
values with implementation-defined types.
Such values can be useful to users in cases where local usage and support exists
(including cases in which details vary from those defined by Unicode and CLDR).

> For example:
> - Implementations are encouraged to _accept_ some native representation
>   for currency amounts as the _operand_ in the _function_ `:currency`.
> - A Java implementation might _accept_ a `java.time.chrono.Chronology` object
>   as a value for the _date/time override option_ `calendar`
> - ICU4J's implementation might _accept_ a `com.ibm.icu.text.NumberingSystem` object
>   instead of using a [Unicode Numbering System Identifier](https://unicode.org/reports/tr35/tr35.html#UnicodeNumberSystemIdentifier)
>   for the option `numberingSystem` in _functions_ such as `:number` or `:integer`.

Future versions of this specification MAY define additional _options_ and _option_ values,
subject to the rules in the [Stability Policy](#stability-policy),
for _functions_ found in this specification.
As implementations are permitted to ignore _options_ that they do not support,
it is possible to write _messages_ using _options_ not defined below
which currently format with no error, but which could produce errors
when formatted with a later edition of this specification.
Therefore, using _options_ not explicitly defined here is NOT RECOMMENDED.
