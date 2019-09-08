Concise Text Encoding
=====================

A general purpose, human readable representation of semi-structured hierarchical data.


Concise Text Encoding (CTE) is a general purpose, human and machine readable, compact representation of semi-structured hierarchical data.

CTE is non-cycic and hierarchical like XML and JSON, and supports the most common data types natively. CTE is type compatible with [Concise Binary Encoding (CBE)](https://github.com/kstenerud/concise-binary-encoding/blob/master/cbe-specification.md), but is a text format for human readability.


Features
--------

  * General purpose encoding for a large number of applications
  * Supports the most common data types
  * Supports hierarchical data structuring
  * Human readable format
  * Minimal complexity
  * Type compatible with [Concise Binary Encoding (CBE)](https://github.com/kstenerud/concise-binary-encoding/blob/master/cbe-specification.md)



Contents
--------

* [Structure](#structure)
  - [Maximum Depth](#maximum-depth)
* [Scalar Types](#scalar-types)
  - [Boolean](#boolean)
  - [Integer](#integer)
  - [Floating Point](#floating-point)
    - [Binary Floating Point](#binary-floating-point)
    - [Decimal Floating Point](#decinal-floating-point)
    - [Floating Point Rules](#floating-point-rules)
    - [Infinity and Not a Number](#infinity-and-not-a-number)
  - [Numeric Whitespace](#numeric-whitespace)
* [Temporal Types](#temporal-types)
  - [Time Zones](#time-zones)
    - [Area/Location](#arealocation)
    - [Global Coordinates](#global-coordinates)
  - [Date](#date)
  - [Time](#time)
  - [Timestamp](#timestamp)
* [Array Types](#array-types)
  - [Bytes](#bytes)
  - [String](#string)
  - [URI](#uri)
* [Container Types](#container-types)
  - [List](#list)
  - [Unordered Map](#unordered-map)
  - [Ordered Map](#ordered-map)
* [Other Types](#other-types)
  - [Nil](#nil)
* [Comments](#comments)
* [Letter Case](#letter-case)
* [Whitespace](#whitespace)
* [Invalid Encodings](#invalid-encodings)
* [File Format](#file-format)
* [Version History](#version-history)
* [License](#license)



Structure
---------


A CTE document is a UTF-8 encoded (with no byte order mark) text document consisting of a single, top-level object of any type. To store multiple values in a CTE document, use a container as the top-level object and store other objects within that container.

Whitespace is used to separate elements in a container. In maps, the key and value portions of a key-value pair are separated by an equals character `=` and possible whitespace. The key-value pairs themselves are separated by whitespace.

Example:

    {
        # A comment
        "a list"        = [1 2 "a string"]
        "unordered map" = {2="two" 3=3000 1="one"}
        "ordered map"   = <1="one" 2="two" 3=3000>
        "boolean"       = true
        "binary int"    = -0b10001011
        "octal int"     = 0o644
        "regular int"   = -10000000
        "hex int"       = 0xfffe0001
        "float"         = 14.125
        "decimal"       = -d1.02e+40
        "time"          = 2019.7.1-18:04:00/Z
        "nil"           = nil
        "bytes"         = h/10 ff 38 9a dd 00 4f 4f 91/
        "url"           = :https://example.com/
        1               = "Keys don't have to be strings"
    }

The top-level object can also be a more simple type, such as:

    "A single string object"

or:

    30.09

### Maximum Depth

Since nested objects (in containers such as maps and lists) are possible, it is necessary to impose an arbitrary depth limit to insure interoperability between implementations. For the purposes of this spec, that limit is 1000 levels of nesting from the top level container to the most nested object (inclusive), unless both sending and receiving parties agree to a different max depth.



Scalar Types
------------

### Boolean

Supports the values `true` and `false`.

Example:

    true
    false


### Integer

Represents signed integers. Negative values are prefixed with a dash `-` as a sign character. Values must be written in lower case.

Integers can be specified in base 2, 8, 10, or 16. Bases other than 10 must be prefixed:

| Base | Name        | Digits           | Prefix |
| ---- | ----------- | ---------------- | ------ |
|   2  | Binary      | 01               | 0b     |
|   8  | Octal       | 01234567         | 0o     |
|  10  | Decimal     | 0123456789       |        |
|  16  | Hexadecimal | 0123456789abcdef | 0h     |

Examples:

| Notation   | Notational Base | Base-10 Equivalent |
| ---------- | --------------- | ------------------ |
| -0b1100    |               2 |                -12 |
| 0o755      |               8 |                493 |
| 900000     |              10 |             900000 |
| 0xdeadbeef |              16 |         3735928559 |


### Floating Point

A floating point number is composed of a whole part and a fractional part, separated by a dot `.`, with an optional exponential portion. Negative values are prefixed with a dash `-`.

    1.0
    -98.413

The exponential portion is denoted by the lowercase character `e`, followed by the signed size of the base-10 exponent. Values with exponential fields must be normalized.

    6.411e+9 = 6411000000
    6.411e-9 = 0.000000006411

The maximum number of significant digits, if not defined a-priori for participating parties, is as many digits as ieee754 allows for the data type.

Numbers must be written in lower case.

There are two types of floating point numbers supported: binary and decimal.


#### Binary Floating Point

Represents ieee754 binary floating point values.

Examples:

    1.25e+7
    -9.00001

Binary floating point values are more common, and are supported by almost all hardware, but suffer from precision loss in the fractional portion due to the values being represented internally as powers of 2. Converting base-10 fractions to base-2 causes loss of precision because not all base-10 fractions can be accurately represented as sums of base-2 fractions within the constraints of the ieee754 spec. For example, `90.1` in decimal would be somewhere around `90.0999` in binary.

Even though binary floating point values lose accuracy in certain cases, it is important to retain fidelity when transmitting such data.


#### Decimal Floating Point

Represents ieee754 decimal floating point values. These are primarily used in financial and other applications where binary rounding errors are unacceptable.

Decimal floating point values are differentiated from binary floating point values by prefixing with `d`.

Examples:

    d12.99
    -d100.04
    d2.58411e-50

Decimal floating point values don't lose precision because they are internally represented as powers of 10, but they have slightly less range, and aren't supported in hardware on many platforms (yet).


#### Floating Point Rules

**There must be at least one digit on each side of the dot character:**

| Invalid    | Valid   | Notes                                                |
| ---------- | ------- | ---------------------------------------------------- |
| -1.        | -1.0    | Or use integer value -1                              |
| .1         | 0.1     |                                                      |
| .218901e+2 | 21.8901 | Or 2.18901e+1                                        |
| -0         | -0.0    | Special case: -0 cannot be represented as an integer |

**If there is an exponential portion, there must also be a dot:**

| Invalid    | Valid    |
| ---------- | -------- |
| 1e+100     | 1.0e+100 |

**Exponential notation must be normalized (one and only one digit to the left of the dot, non-zero):**

| Invalid    | Valid      |
| ---------- | ---------- |
| 22e+50     | 2.2e+51    |
| 508.44e+10 | 5.0844e+12 |
| -1000e+5   | -1.0e+8    |
| 65.0e-20   | 6.5e-21    |
| 0.5e+10    | 5.0e+11    |

**There must be one and only one dot character:**

* `10.4.5` is not a floating point number.
* `10` is interpreted as an integer.


#### Infinity and Not a Number

The following are special floating point values:

 * `inf`: Infinity
 * `-inf`: Negative Infinity
 * `nan`: Not a Number (non-signaling)

Although ieee754 allows many different NaN values (signaling or non-signaling + payload), all NaN values for the purpose of this spec are considered equal and non-signaling, and must be treated as equivalent when used as map keys (meaning you cannot have multiple NAN keys in a map).

A decoder is free to decode infinity and NaN as binary or decimal floating point values.


### Numeric Whitespace

The `_` character may be used as "numeric whitespace" when encoding numeric values.

All numeric values of any type (with the exception of +- "nan" and "inf") may contain any amount of whitespace at any point after the first character and before the last character. Numeric whitespace characters must be ignored while decoding numeric values.

Examples:

    1_000_000 = 1000000
    -_7_._4_e_+_100 = -7.4e+100
    -d_4_10.___2_2 = -d410.22

Invalid:

    _100
    _-d6.33
    50_
    n_an
    -_inf



Temporal Types
--------------

The temporal types deal with time.


### Notes

#### Field Width

Fields other than year may be pre-padded with zeros (`0`) up to their maximum allowed digits for aesthetic reasons if desired. Minutes and seconds must always be padded to 2 digits.

Field values must never be abbreviated (the year 19 refers to 19 AD, not 2019).

#### The Year Field

The year field may be any number of digits, and may be positive (representing AD dates) or negative (representing BC dates). Negative (BC) years are prefixed with a dash character (`-`). The year must always be written in full, and may not be abbreviated.

Note: The Anno Domini system has no zero year (there is no 0 BC or 0 AD), and so the year values `0` and `-0` are invalid. Many date systems internally use the value 0 to represent 1 BC and offset all BC dates by 1 for mathematical continuity, but in interchange formats it's preferable to avoid exposing potentially confusing internal details.

#### Date Structure

All dates are structured according to the Gregorian calendar, containing a month and day-of-month.

#### Dates prior to 1582

Dates prior to the introduction of the Gregorian calendar in 1582 must be written according to the proleptic Gregorian calendar.


### Time Zones

A time zone refers to the political designation of a location having a specific time offset from UTC during a particular time period. Time zones are in a constant state of flux, and can change at any time for many reasons. There are two ways to denote a time zone: by area/location, and by global coordinates.

If the time zone is unspecified, it is assumed to be `Zero` (UTC).


#### Area/Location

The area/location method is the more human-readable of the two, but may not be precise enough for certain applications. Time zones are partitioned into areas containing locations, and are written in the form `Area/Location`. These areas and locations are specified in the [IANA time zone database](https://www.iana.org/time-zones).

##### Abbreviated Areas

Since there are only a limited number of areas in the database, the following abbreviations may be used in the area portion of the time zone:

| Area         | Abbreviation |
| ------------ | ------------ |
| `Africa`     | `F`          |
| `America`    | `M`          |
| `Antarctica` | `N`          |
| `Arctic`     | `R`          |
| `Asia`       | `S`          |
| `Atlantic`   | `T`          |
| `Australia`  | `U`          |
| `Etc`        | `C`          |
| `Europe`     | `E`          |
| `Indian`     | `I`          |
| `Pacific`    | `P`          |

##### Special Areas

The following special "areas" may also be used. They do not contain a location component.

| Area    | Abbreviation | Meaning            |
| ------- | ------------ | ------------------ |
| `Zero`  | `Z`          | Alias to `Etc/UTC` |
| `Local` | `L`          | "Local" time zone, meaning that the accompanying time value is to be interpreted as if in the time zone of the observer. |

##### Examples

* `E/Paris`
* `America/Vancouver`
* `Etc/UTC` == `Zero` == `Z`
* `L`


#### Global Coordinates

The global coordinates method uses the global position to hundredths of degrees, giving a resolution of around 1km at the equator. Locations are written as longitude and latitude, separated by a slash character (`/`). Negative values are prefixed with a dash character (`-`), and the period character (`.`) is used as a decimal separator.

This method has the advantage of being unambiguous, which can be useful for areas that are in an inconsistent political state at a particular time. The disadvantage is that it's not easily decodable by humans.

##### Examples

* `51.60/11.11`
* `-13.53/-172.37`


### Date

Refers to a particular date without specifying a time during that day.

A date is made up of the following fields, separated by a period character (`.`):

| Field | Mandatory | Min Value | Max Value | Max Digits |
| ----- | --------- | --------- | --------- | ---------- |
| Year  |     Y     |         * |         * |          * |
| Month |     Y     |         1 |        12 |          2 |
| Day   |     Y     |         1 |        31 |          2 |

#### Examples

* August 5, 2019: `2019.8.5`
* March 30, 5081: `5081.03.30`
* December 21, 300 BC (proleptic Gregorian): `-300.12.21`



### Time

Represents a time of day without specifying a particular date.

A time is made up of the following fields:

| Field        | Mandatory | Separator | Min Value | Max Value | Max Digits |
| ------------ | --------- | --------- | --------- | --------- | ---------- |
| Hour         |     Y     |           |         0 |        24 |          2 |
| Minute       |     Y     |    `:`    |         0 |        59 |          2 |
| Second       |     Y     |    `:`    |         0 |        60 |          2 |
| Subseconds   |     N     |    `.`    |         0 | 999999999 |          9 |
| Time Zone    |     N     |    `/`    |         - |         - |          - |

#### Notes

* Hours are always written according to the 24h clock (21:00, not 9:00 PM).
* Minutes and seconds must always be padded to 2 digits.
* Since there is no date component, time zone data must be interpreted as if it were "today", and so the time may not remain constant should the political situation at that time zone change at a later date.
* If the time zone is unspecified, it is assumed to be `Zero` (UTC).

#### Examples

* `9:04:21` (9:04:21 UTC)
* `23:59:59.999999999` (23:59:59 and 999999999 nanoseconds UTC)
* `12:05:50.102/Z` (12:05:50 and 102 milliseconds UTC)
* `4:00:00/Asia/Tokyo` (4:00:00 Tokyo time)
* `17:41:03/-13.54/-172.36` (17:41:03 Samoa time)
* `9:00:00/L` (9:00:00 local time)


### Timestamp

A timestamp combines a date and a time, separated by a dash character (`-`).

#### Examples

* `2019.01.23-14:08:51.941245`: January 23, 2019, at 14:08:51 and 941245 microseconds, UTC
* `5192.11.01-03:00:00/48.86/2.36`: November 1st, 5192, at 3:00:00, at whatever is in the place of Paris at that time.



Array Types
-----------

### Bytes

An array of octets. This data type should only be used as a last resort if the other data types cannot represent the data you need. To reduce cross-platform confusion, multibyte data types stored within the binary blob should be represented in little endian order whenever possible.

Byte array data is enclosed in forward slashes `/`, and is prefixed by the encoding type. The encoded contents may contain whitespace at any point.

Encoding Types:

Supported encoding types are [hex](https://github.com/kstenerud/safe-encoding/blob/master/safe16-specification.md), [safe64](https://github.com/kstenerud/safe-encoding/blob/master/safe64-specification.md), and [safe85](https://github.com/kstenerud/safe-encoding/blob/master/safe85-specification.md). You can choose your encoding type based on your size constraints and desired features.

| Type   | Prefix | Bloat | Features                               |
| ------ | ------ | ----- | -------------------------------------- |
| Hex    |    h   | 2.0   | Human readable, fast encoding/decoding |
| Safe64 |    6   | 1.33  | Fast encoding/decoding                 |
| Safe85 |    8   | 1.25  | Smallest size                          |

#### Examples

    h/39 12 82 e1 81 39 d9 8b 39 4c 63 9d 04 8c/

    h/1 f 4 8 ae 4 56 3/ # looks terrible, but is valid

    8/8F2{*RVCLI8LDzZ!3e/

    8/CmsAT9+UpvN!1v=H_SgpMm@mDHDFy(I[~!{I@2
    yx1MU*1I[u!)NL20.1LOvFN-+cu1M_VMH_)d)HD=
    T)I6F~3Ml=.;JP_@>Ln!H$N-xV.1MUpTNKoD71L(
    nBIZop{LR-.0Nh}Y.1ML**I>@ziISc.1OfbXN/


### String

An array of UTF-8 encoded bytes, without a byte order mark (BOM). Strings must be enclosed in double-quotes `"`, and cannot contain values or sequences that evaluate to NUL.

The following escape sequences are allowed inside a string's contents, and must be in lower case:

| Sequence            | Interpretation                  |
| ------------------- | ------------------------------- |
| `\\`                | literal backslash               |
| `\"`                | double quote                    |
| `\r`                | carriage return                 |
| `\n`                | linefeed                        |
| `\t`                | horizontal tab                  |
| `\x01` - `\xff`     | one octet, hexadecimal notation |
| `\u0001` - `\uffff` | unicode character               |

Strings must always resolve to complete, valid unicode sequences when fully decoded (after evaluating backslash sequences).

#### Example

    "A string\twith\ttabs\nand\nnewlines"


### URI

Uniform Resource Identifier, structured in accordance with [RFC 3986](https://tools.ietf.org/html/rfc3986). URIs are prefixed with a colon character (`:`) and end with a whitespace character.

Examples:

    :https://john.doe@www.example.com:123/forum/questions/?tag=networking&order=newest#top

    :mailto:John.Doe@example.com

    :urn:oasis:names:specification:docbook:dtd:xml:4.1.2

    :https://example.com/percent-encoding/?double-quote=%22



Container Types
---------------

### List

A sequential list of objects. Lists can contain any mix of any type, including other containers.

A list begins with an opening square bracket `[`, whitespace separated contents, and finally a closing bracket `]`.

Note: While this spec allows mixed types in lists, not all languages do. Use mixed types with caution.

Example:

    [1 "two" 3.1 {} nil]


### Unordered Map

A unordered map associates objects (keys) with other objects (values), storing the key-value pairs in no particular order. Implementations must not rely on any incidental ordering. Keys may be any mix of scalar or array types. A key must not be a container type or the `nil` type. Values may be any mix of any type, including other containers. All keys in a map must resolve to a unique value, even across data types. For example, the following keys would clash:

 * 2000
 * 2000.0
 * 2000.0d

Map entries are split into key-value pairs using the equals `=` character and optional whitespace. Key-value pairs are separated from each other using whitespace. A key without a paired value is not allowed in a map.

A map begins with an opening curly brace `{`, whitespace separated key-value pairs, and finally a closing brace `}`.

Note: While this spec allows mixed types in maps, not all languages do. Use mixed types with caution.

Example:

    {
        1 = "alpha"
        2 = "beta"
        "a map" = {one = 1}
    }


### Ordered Map

An ordered map is the same as an unordered map, except that the ordering of the key-value pairs is explicit, and must be preserved.

Ordered maps use the opening parenthesis `(` and closing parenthesis `)` instead of curly braces `{` and `}`.



Other Types
-----------

### Nil

Denotes the absence of data. Some languages implement this as the `null` value.

Note: Use nil judiciously and sparingly, as some languages may have restrictions on how and if it may be used.

Example:

    nil



Comments
--------

Comments may be placed before or after any object. Any number of comments may occur in a row. A parser is free to preserve or discard comments.

Comment contents must contain only complete and valid UTF-8 sequences. Escape sequences in comments are not interpreted (they are passed through verbatim).

A comment begins with a `#` character, followed by an optional space (U+0020) which is discarded if present. If multiple spaces follow the `#`, only the first is discarded. Everything else up to (but not including) the next carriage return (U+000D) or newline (U+000A), including whitespace, is preserved as-is.


### Illustration (using underscore `_` to represent space U+0020)

| CTE Document    | Comment Value |
| --------------- | ------------- |
| `#`             | (empty)       |
| `#_`            | (empty)       |
| `#__`           | `_`           |
| `#A_comment`    | `A_comment`   |
| `#_A_comment`   | `A_comment`   |
| `#__A_comment_` | `_A_comment_` |


### Character Restrictions

The following characters are disallowed:

 * Control characters (Unicode u+0000 to u+001f, u+007f to u+009f) with the exception of:
   - Horizontal Tab (u+0009)
 * Line breaking characters (such as u+2028, u+2029).
 * Byte order mark.

The following characters are allowed in comments if they aren't in the above disallowed list:

 * UTF-8 printable characters
 * UTF-8 whitespace characters


### Example

    # Comment before top level object
    {
        # Comment before the "name" object.
        # And another comment.
        "name" = "Joe Average" # Comment before the "email" object.
        "email" = # Comment before the "joe@average.org" object.
        "joe@average.org"
        "numbers" # Comment after numbers
        =
        #
        # Comment before some binary data (but not inside it)
        h"01 02 03 04 05 06 07 08 09 0a"
    }
    # Comments at the
    # end of the document.



Letter Case
-----------

A CTE document must be entirely in lower case, with the following exceptions:

 * String and comment contents: `"A string may contain UPPER CASE. Escape sequences must be lower case: \x3d"`
 * [Time zones](#time-zone) are case sensitive, and contain uppercase characters.
 * Safe85 encoding makes use of uppercase characters in its code table.

Everything else, including hexadecimal digits, exponents and escape sequences, must be lower case.



Whitespace
----------

Whitespace characters outside of strings and comments are ignored by a CTE parser. Any number of whitespace characters may occur in a sequence.


### Valid Whitespace Characters

While there are many characters classified as "whitespace" within the Unicode set, only the following are valid whitespace characters in a CTE document:

| Code Point | Name            |
| ---------- | --------------- |
| U+0009     | horizontal tab  |
| U+000A     | line feed       |
| U+000D     | carriage return |
| U+0020     | space           |


### Whitespace **may** occur:

 * Before an object (including at the beginning of a document)
 * After an object (including at the end of a document)
 * Between array/container openings & closings: `[`, `]`, `{`, `}`, `(`, `)`, `/`
 * Between encoding characters in a byte array.

Examples:

 * `[   1     2      3 ]` is equivalent to `[1 2 3]`
 * `h/ 01 02 03   0 4 /` is equivalent to `h/01020304/`
 * `{ 1="one" 2 = "two" 3= "three" 4 ="four"}` is equivalent to `{1="one" 2="two" 3="three" 4="four"}`


### Whitespace **must** occur:

 * Between values in a list: `[12"one-two or twelve?"]` is invalid
 * Between key-value pairs in a map: `{1="one"2="two"3="three"4="four"}` is invalid.


### Whitespace **must not** occur:

 * Between a byte array encoding type and the opening double-quote: `h "` is invalid.
 * Splitting a time value: `2018.07.01-10 :53:22.001481/Z` is invalid.
 * Splitting a numeric value: `3f h`, `9.41 d`, `3 000`, `9.3 e+3`, `- 1.0` are invalid.
 * Splitting special values: `t rue`, `ni l`, `i nf`, `n a n` are invalid.


### Whitespace is interpreted literally (not ignored) within a string or comment:

    "This string has spaces
    and a newline, which are all preserved."

    # Comment whitepsace      is preserved.



Invalid Encodings
-----------------

Invalid encodings must not be used, as they may cause problems or even API violations in certain languages. A parser must halt processing when invalid data is detected.

 * Numeric values must be representable in their respective binary formats (integer, binary float, decimal float).
 * Times must be valid. For example: 2000.2.30, while technically encodable, is not allowed.
 * Map keys must not be container types or the `nil` type.
 * Maps must not contain duplicate keys (this includes mathematically equivalent numeric keys).
 * Upper case text is not allowed, except as described in section [Letter Case](#letter-case).
 * Whitespace must only occur as described in section [Whitespace](#whitespace).
 * Incomplete or invalid UTF-8 sequences are not allowed.



File Format
-----------

A CTE file is simply a file containing a single CTE document. Recall that a CTE document consists of a single top-level object, and that you can store multiple objects by making the top level object a container. CTE files should be named using the extension `cte`.

For example: File `mydata.cte`

    # This is an example CTE document.
    # Remember: Any number of comments can appear in a row.
    {
        # Here are some mapped values...
        "first" = 1
        "second" = 2
    }



Version History
---------------

July 22, 2018: Preview Version 1



License
-------

Copyright (c) 2018 Karl Stenerud. All rights reserved.

Distributed under the Creative Commons Attribution License: https://creativecommons.org/licenses/by/4.0/legalcode
License deed: https://creativecommons.org/licenses/by/4.0/
