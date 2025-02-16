## Appendices

### Security Considerations

MessageFormat 2.0 _patterns_ are meant to allow a _message_ to include any string value
which users might normally wish to use in their environment.
Programming languages and other environments vary in what characters are permitted
to appear in a valid string.
In many cases, certain types of characters, such as invisible control characters,
require escaping by these host formats.
In other cases, strings are not permitted to contain certain characters at all.
Since _messages_ are subject to the restrictions and limitations of their 
host environments, their serializations and resource formats,
that might be sufficient to prevent most problems.
However, MessageFormat itself does not supply such a restriction.

MessageFormat _messages_ permit nearly all Unicode code points
to appear in _literals_, including the text portions of a _pattern_.
This means that it can be possible for a _message_ to contain invisible characters
(such as bidirectional controls, ASCII control characters in the range U+0000 to U+001F,
or characters that might be interpreted as escapes or syntax in the host format)
that abnormally affect the display of the _message_
when viewed as source code, or in resource formats or translation tools,
but do not generate errors from MessageFormat parsers or processing APIs.

Bidirectional text containing right-to-left characters (such as used for Arabic or Hebrew) 
also poses a potential source of confusion for users. 
Since MessageFormat 2.0's syntax makes use of 
keywords and symbols that are left-to-right or consist of neutral characters 
(including characters subject to mirroring under the Unicode Bidirectional Algorithm), 
it is possible to create messages that,
when displayed in source code, or in resource formats or translation tools, 
have a misleading appearance or are difficult to parse visually.

For more information, see \[[UTS#55](https://unicode.org/reports/tr55/)\] 
<cite>Unicode Source Code Handling</cite>.

MessageFormat 2.0 implementations might allow end-users to install
_selectors_, _functions_, or _markup_ from third-party sources.
Such functionality can be a vector for various exploits,
including buffer overflow, code injection, user tracking,
fingerprinting, and other types of bad behavior.
Any installed code needs to be appropriately sandboxed.
In addition, end-users need to be aware of the risks involved.

### Acknowledgements

Special thanks to the following people for their contributions to making MessageFormat v2.
The following people contributed to our github repo and are listed in order by contribution size:

Addison Phillips, 
Eemeli Aro, 
Romulo Cintra, 
Stanisław Małolepszy, 
Tim Chevalier, 
Elango Cheran, 
Richard Gibson, 
Mihai Niță, 
Mark Davis, 
Steven R. Loomis, 
Shane F. Carr, 
Matt Radbourne, 
Caleb Maclennan, 
David Filip, 
Daniel Minor, 
Christopher Dieringer,
Bruno Haible,
Danny Gleckler,
George Rhoten, 
Ujjwal Sharma, 
Daniel Ehrenberg, 
Markus Scherer, 
Zibi Braniecki, 
Lionel Rowe,
Luca Casonato,
and Rafael Xavier de Souza.

Addison Phillips was chair of the working group from January 2023.
Prior to 2023, the group was governed by a chair group, consisting of
Romulo Cintra,
Elango Cheran,
Mihai Niță,
David Filip,
Nicolas Bouvrette,
Stanisław Małolepszy,
Rafael Xavier de Souza,
Addison Phillips,
and Daniel Minor.
Romulo Cintra chaired the chair group.
