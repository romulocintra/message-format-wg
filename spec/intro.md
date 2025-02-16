# MessageFormat 2.0 Specification

## Table of Contents

1. [Introduction](intro.md)
   1. [Conformance](intro.md#conformance)
   1. [Terminology and Conventions](intro.md#terminology-and-conventions)
   1. [Stability Policy](intro.md#stability-policy)
1. [Syntax](syntax.md)
   1. [`message.abnf`](message.abnf)
1. [Formatting](formatting.md)
1. [Errors](errors.md)
1. [Default Functions](functions/README.md)
1. [`u:` Namespace](u-namespace.md)
1. [Interchange Data Model](data-model/README.md)
1. [Appendices](appendices.md)
   1. [Security Considerations](appendices.md#security-considerations)
   1. [Acknowledgements](appendices.md#acknowledgements)

## Introduction

One of the challenges in adapting software to work for
users with different languages and cultures is the need for **_<dfn>dynamic messages</dfn>_**.
Whenever a user interface needs to present data as part of a larger string,
that data needs to be formatted (and the message may need to be altered)
to make it culturally accepted and grammatically correct.

> For example, if your US English (`en-US`) interface has a message like:
>
> > Your item had 1,023 views on April 3, 2023
>
> You want the translated message to be appropriately formatted into French:
>
> > Votre article a eu 1 023 vues le 3 avril 2023
>
> Or Japanese:
>
> > あなたのアイテムは 2023 年 4 月 3 日に 1,023 回閲覧されました。

This specification defines the
data model, syntax, processing, and conformance requirements
for the next generation of _dynamic messages_.
It is intended for adoption by programming languages and APIs.
This will enable the integration of
existing internationalization APIs (such as the date and number formats shown above),
grammatical matching (such as plurals or genders),
as well as user-defined formats and message selectors.

The document is the successor to ICU MessageFormat,
henceforth called ICU MessageFormat 1.0.

### Conformance

Everything in this specification is normative except for:
sections marked as non-normative,
all authoring guidelines, diagrams, examples, and notes.

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL
NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED",
"MAY", and "OPTIONAL" in this document are to be interpreted as
described in BCP 14 \[[RFC2119](https://www.rfc-editor.org/rfc/rfc2119)\]
\[[RFC8174](https://www.rfc-editor.org/rfc/rfc8174)\] when, and only when, they
appear in all capitals, as shown here.

### Terminology and Conventions

A **_term_** looks like this when it is defined in this specification.

A reference to a _term_ looks like this.

> Examples are non-normative and styled like this.

> [!IMPORTANT]
> Text marked "Important" like this are normative.

> [!NOTE]
> Notes are non-normative.

### Stability Policy

Updates to this specification will not make any valid _message_ invalid.

Updates to this specification will not remove any syntax provided in this version.

Updates to this specification will not specify an _error_ for any _message_
that previously did not specify an _error_.

Updates to this specification will not specify the use of a _fallback value_ for any _message_
that previously did not specify a _fallback value_.

Updates to this specification will not change the syntactical meaning
of any syntax defined in this specification.

Updates to this specification will not remove any _default functions_.

Updates to this specification will not remove any _options_ or _option_ values
defined for _default functions_.

> [!NOTE]
> The foregoing policies are _not_ a guarantee that the results of formatting will never change.
> Even when this specification or its implementation do not change,
> the _function handlers_ for date formatting, number formatting and so on
> can change their results over time or behave differently due to local runtime
> differences in implementation or changes to locale data
> (such as due to the release of new CLDR versions).

Updates to this specification will only reserve, define, or require
_function_ _identifiers_ and _function_ _option_ _identifiers_
which satisfy either of the following two requirements:
- Includes no _namespace_,
  and has a _name_ consisting of characters in the ranges a-z, A-Z, and 0-9,
  and the characters U+002E FULL STOP `.`, U+002D HYPHEN-MINUS `-`, and U+005F LOW LINE `_`.
- Uses a _namespace_ consisting of a single character in the ranges a-z and A-Z.

All other _identifiers_ in these categories are reserved for the use of implementations or users.

> [!NOTE]
> Users defining custom _identifiers_ SHOULD include at least one character outside these ranges
> to ensure that they will be compatible with future versions of this specification.
> They SHOULD also use the _namespace_ feature to avoid collisions with other implementations.

Future versions of this specification will not introduce changes
to the data model that would result in a data model representation
based on this version being invalid.

> For example, existing interfaces or fields will not be removed.

> [!IMPORTANT]
> This stability policy allows any of the following, non-exhaustive list, of changes
> in future versions of this specification:
> - Future versions may define new syntax and structures
>   that would not be supported by this version of the specification.
> - Future versions may add additional structure or meaning to existing syntax.
> - Future versions may define new _keywords_.
> - Future versions may make previously invalid _messages_ valid.
> - Future versions may define additional _default functions_.
>   or may reserve the names of _functions_ for the purposes of interoperability.
> - Future versions may define additional _options_ to existing functions.
> - Future versions may define additional _option_ values for existing _options_.
> - Future versions may deprecate (but not remove) _keywords_, _functions_, _options_, or _option_ values.
> - Future versions of this specification may introduce changes
>   to the data model that would result in future data model representations
>   not being valid for implementations of this version of the data model.
>   - For example, a future version could introduce a new _keyword_,
>     whose data model representation would be a new interface
>     that is not recognized by this version's data model.

