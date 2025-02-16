## Date and Time Value Formatting

This subsection describes the _functions_ and _options_ for date/time formatting.

> [!IMPORTANT]
> The _functions_ in this section have a status of **Draft**.
> They are proposed for inclusion in a future release and are not Stable.

> [!NOTE]
> Selection based on date/time types is not required by this release of MessageFormat.
> Use care when defining implementation-specific _selectors_ based on date/time types.
> The types of queries found in implementations such as `java.time.TemporalAccessor`
> are complex and user expectations might be inconsistent with good I18N practices.

### The `:datetime` function

The function `:datetime` is used to format date/time values, including
the ability to compose user-specified combinations of fields.

If no options are specified, this function defaults to the following:

- `{$d :datetime}` is the same as `{$d :datetime dateStyle=medium timeStyle=short}`

> [!NOTE]
> The default formatting behavior of `:datetime` is inconsistent with `Intl.DateTimeFormat`
> in JavaScript and with `{d,date}` in ICU MessageFormat 1.0.
> This is because, unlike those implementations, `:datetime` is distinct from `:date` and `:time`.

#### Operands

The _operand_ of the `:datetime` function is either
an implementation-defined date/time type
or a _date/time literal value_, as defined in [Date and Time Operand](#date-and-time-operands).
All other _operand_ values produce a _Bad Operand_ error.

#### Options

The `:datetime` function can use either the appropriate _style options_
or can use a collection of _field options_ (but not both) to control the formatted
output.
_Date/time override options_ can be combined with either _style options_ or _field options_.

If both _style options_ and _field options_ are specified,
a _Bad Option_ error is emitted
and a _fallback value_ used as the _resolved value_ of the _expression_.

If the _operand_ of the _expression_ is an implementation-defined date/time type,
it can include _style options_, _field options_, or other option values.
These are included in the resolved option values of the _expression_,
with _options_ on the _expression_ taking priority over any option values of the _operand_.

> [!NOTE]
> The names of _options_ and their _values_ were derived from the
> [options](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Intl/DateTimeFormat/resolvedOptions#description)
> in JavaScript's `Intl.DateTimeFormat`.

##### Style Options

The function `:datetime` has these _style options_.

- `dateStyle`
  - `full`
  - `long`
  - `medium`
  - `short`
- `timeStyle`
  - `full`
  - `long`
  - `medium`
  - `short`

##### Field Options

_Field options_ describe which fields to include in the formatted output
and what format to use for that field.

> [!NOTE] > _Field options_ do not have default values because they are only to be used
> to compose the formatter.

The function `:datetime` has the following _field options_:

- `weekday`
  - `long`
  - `short`
  - `narrow`
- `era`
  - `long`
  - `short`
  - `narrow`
- `year`
  - `numeric`
  - `2-digit`
- `month`
  - `numeric`
  - `2-digit`
  - `long`
  - `short`
  - `narrow`
- `day`
  - `numeric`
  - `2-digit`
- `hour`
  - `numeric`
  - `2-digit`
- `minute`
  - `numeric`
  - `2-digit`
- `second`
  - `numeric`
  - `2-digit`
- `fractionalSecondDigits`
  - `1`
  - `2`
  - `3`
- `timeZoneName`
  - `long`
  - `short`
  - `shortOffset`
  - `longOffset`
  - `shortGeneric`
  - `longGeneric`

#### Resolved Value

The _resolved value_ of an _expression_ with a `:datetime` _function_
contains an implementation-defined date/time value
of the _operand_ of the annotated _expression_,
together with the resolved options values.

### The `:date` function

The function `:date` is used to format the date portion of date/time values.

If no options are specified, this function defaults to the following:

- `{$d :date}` is the same as `{$d :date style=medium}`

#### Operands

The _operand_ of the `:date` function is either
an implementation-defined date/time type
or a _date/time literal value_, as defined in [Date and Time Operand](#date-and-time-operands).
All other _operand_ values produce a _Bad Operand_ error.

#### Options

The function `:date` has these _options_:

- `style`
  - `full`
  - `long`
  - `medium` (default)
  - `short`
- _Date/time override options_

If the _operand_ of the _expression_ is an implementation-defined date/time type,
it can include other option values.
Any _operand_ option values matching the `:datetime` _style options_ or _field options_ are ignored,
as is any `style` option.

#### Resolved Value

The _resolved value_ of an _expression_ with a `:date` _function_
is implementation-defined.

An implementation MAY emit a _Bad Operand_ or _Bad Option_ error (as appropriate)
when a _variable_ annotated directly or indirectly by a `:date` _annotation_
is used as an _operand_ or an _option_ value.

### The `:time` function

The function `:time` is used to format the time portion of date/time values.

If no options are specified, this function defaults to the following:

- `{$t :time}` is the same as `{$t :time style=short}`

#### Operands

The _operand_ of the `:time` function is either
an implementation-defined date/time type
or a _date/time literal value_, as defined in [Date and Time Operand](#date-and-time-operands).
All other _operand_ values produce a _Bad Operand_ error.

#### Options

The function `:time` has these _options_:

- `style`
  - `full`
  - `long`
  - `medium`
  - `short` (default)
- _Date/time override options_

If the _operand_ of the _expression_ is an implementation-defined date/time type,
it can include other option values.
Any _operand_ option values matching the `:datetime` _style options_ or _field options_ are ignored,
as is any `style` option.

#### Resolved Value

The _resolved value_ of an _expression_ with a `:time` _function_
is implementation-defined.

An implementation MAY emit a _Bad Operand_ or _Bad Option_ error (as appropriate)
when a _variable_ annotated directly or indirectly by a `:time` _annotation_
is used as an _operand_ or an _option_ value.

### Date and Time Operands

The _operand_ of a date/time function is either
an implementation-defined date/time type
or a _date/time literal value_, as defined below.
All other _operand_ values produce a _Bad Operand_ error.

A **_<dfn>date/time literal value</dfn>_** is a non-empty string consisting of an ISO 8601 date,
or an ISO 8601 datetime optionally followed by a timezone offset.
As implementations differ slightly in their parsing of such strings,
ISO 8601 date and datetime values not matching the following regular expression MAY also be supported.
Furthermore, matching this regular expression does not guarantee validity,
given the variable number of days in each month.

```regexp
(?!0000)[0-9]{4}-(0[1-9]|1[0-2])-(0[1-9]|[12][0-9]|3[01])(T([01][0-9]|2[0-3]):[0-5][0-9]:[0-5][0-9](\.[0-9]{1,3})?(Z|[+-]((0[0-9]|1[0-3]):[0-5][0-9]|14:00))?)?
```

When the time is not present, implementations SHOULD use `00:00:00` as the time.
When the offset is not present, implementations SHOULD use a floating time type
(such as Java's `java.time.LocalDateTime`) to represent the time value.
For more information, see [Working with Timezones](https://w3c.github.io/timezone).

> [!IMPORTANT]
> The [ABNF](/spec/message.abnf) and [syntax](/spec/syntax.md) of MF2
> do not formally define date/time literals.
> This means that a _message_ can be syntactically valid but produce
> a _Bad Operand_ error at runtime.

> [!NOTE]
> String values passed as variables in the _formatting context_'s
> _input mapping_ can be formatted as date/time values as long as their
> contents are date/time literals.
>
> For example, if the value of the variable `now` were the string
> `2024-02-06T16:40:00Z`, it would behave identically to the local
> variable in this example:
>
> ```
> .local $example = {|2024-02-06T16:40:00Z| :datetime}
> {{{$now :datetime} == {$example}}}
> ```

> [!NOTE]
> True time zone support in serializations is expected to coincide with the adoption
> of Temporal in JavaScript.
> The form of these serializations is known and is a de facto standard.
> Support for these extensions is expected to be required in the post-tech preview.
> See: https://datatracker.ietf.org/doc/draft-ietf-sedate-datetime-extended/

### Date and Time Override Options

**_<dfn>Date/time override options</dfn>_** are _options_ that allow an _expression_ to
override values set by the current locale,
or provided by the _formatting context_ (such as the default time zone),
or embedded in an implementation-defined date/time _operand_ value.

> [!NOTE]
> These _options_ do not have default values because they are only to be used
> as overrides for locale-and-value dependent implementation-defined defaults.

The following _option_ and its values are REQUIRED to be available on
the functions `:datetime` and `:time`:

- `hour12`
  - `true`
  - `false`

The following _options_ and their values are RECOMMENDED to be available on
the functions `:datetime`, `:date`, and `:time`.

- `calendar`
  - valid [Unicode Calendar Identifier](https://unicode.org/reports/tr35/tr35.html#UnicodeCalendarIdentifier)
- `numberingSystem`
  - valid [Unicode Number System Identifier](https://unicode.org/reports/tr35/tr35.html#UnicodeNumberSystemIdentifier)

> [!IMPORTANT]
> The following _option_ and its values are **Proposed** for
> inclusion in a future release of this specification but have not yet been
> finalized.
> If accepted, implementations could be REQUIRED to make this _option_
> available in the functions `:datetime`, `:date`, and `:time`.

- `timeZone`
  - A valid time zone identifier
    (see [TZDB](https://www.iana.org/time-zones)
    and [LDML](https://www.unicode.org/reports/tr35/tr35-dates.html#Time_Zone_Names)
    for information on identifiers)
  - `local`
  - `UTC`

> [!NOTE]
> The value `local` permits a _message_ to convert a date/time value
> into a [floating](https://www.w3.org/TR/timezone/#floating) time value
> (sometimes called a _plain_ or _local_ time value) by removing
> the association with a specific time zone.
