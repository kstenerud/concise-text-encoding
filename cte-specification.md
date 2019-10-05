Concise Text Encoding
=====================

Concise Text Encoding (CTE) is a general purpose, human and machine readable, compact representation of semi-structured hierarchical data.

CTE is non-cycic and hierarchical like XML and JSON, and supports the most common data types natively. CTE is type compatible with [Concise Binary Encoding (CBE)](https://github.com/kstenerud/concise-binary-encoding/blob/master/cbe-specification.md), but is a text format for human readability.



TODO
----

- Reference implementation doesn't match the updated spec.



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
  - [Version Specifier](#version-specifier)
  - [Maximum Depth](#maximum-depth)
  - [Maximum Length](#maximum-length)
  - [Line Endings](#line-endings)
* [Numeric Types](#numeric-types)
  - [Boolean](#boolean)
  - [Integer](#integer)
  - [Floating Point](#floating-point)
    - [Base-10 Exponential Notation](#base-10-exponential-notation)
    - [Base-16 Exponential Notation](#base-16-exponential-notation)
    - [Floating Point Rules](#floating-point-rules)
    - [Infinity and Not a Number](#infinity-and-not-a-number)
  - [Numeric Whitespace](#numeric-whitespace)
* [Temporal Types](#temporal-types)
  - [Temporal Type Notes](#temporal-type-notes)
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
* [Metadata Types](#metadata-types)
  - [Metadata Association](#metadata-association)
  - [Metadata Map](#metadata-map)
    - [Name Clashes](#name-clashes)
    - [Predefines Keys](#predefined-keys)
  - [Comment](#comment)
* [Other Types](#other-types)
  - [Nil](#nil)
* [Letter Case](#letter-case)
* [Whitespace](#whitespace)
* [Invalid Encodings](#invalid-encodings)
* [Version History](#version-history)
* [License](#license)



Structure
---------


A CTE document is a UTF-8 encoded (with no byte order mark) text document consisting of a version specifier followed by a single, top-level object of any type. To store multiple values in a CTE document, use a container as the top-level object and store other objects within that container.

Whitespace is used to separate elements in a container. In maps, the key and value portions of a key-value pair are separated by an equals character `=` and possible whitespace. The key-value pairs themselves are separated by whitespace.

#### Example

    v1
    // _ct is the creation time, in this case referring to the document
    (_ct = 2019.9.1-22:14:01)
    {
        // A comment
        /* A multiline
           comment */
        (metadata_about_a_list = "something interesting about 'a list'")
        "a list"        = [1 2 "a string"]
        "unordered map" = {2=two 3=3000 1=one}
        "ordered map"   = <1=one 2.5="two and a half" 3=3000>
        boolean         = true
        "binary int"    = -0b10001011
        "octal int"     = 0o644
        "regular int"   = -10000000
        "hex int"       = 0xfffe0001
        float           = 14.125
        time            = 2019.7.1-18:04:00/Z
        // nil must be quoted when representing the string "nil"
        "nil"           = nil
        bytes           = h"10 ff 38 9a dd 00 4f 4f 91"
        url             = u"https://example.com/"
        email           = u"mailto:me@somewhere.com"
        1               = "Keys don't have to be strings"
    }

The top-level object can also be a simple type, for example:

    v1 "A single string object"

or:

    v1 30.09


### Version Specifier

All CTE documents must begin with a version specifier.

The version specifier is the lowercase letter `v` followed immediately by a number representing the version of this specification that the document adheres to (there is no whitespace between the `v` and the number). The version specifier must be followed by whitespace to separate it from the rest of the document.

#### Examples

    v1
    {
        a = 1
    }

Or:

    v1 "this is a string"


### Maximum Depth

Since nested objects (in containers such as maps and lists) are possible, it is necessary to impose an arbitrary depth limit to insure interoperability between implementations. For the purposes of this spec, that limit is 1000 levels of nesting from the top level container to the most nested object (inclusive), unless both sending and receiving parties agree to a different max depth.


### Maximum Length

Maximum lengths (max list length, max map length, max array length, max total objects in document, max byte length, etc) are implementation defined, and should be negotiated beforehand. A decoder is free to discard documents that threaten to exceed its resources.


### Line Endings

Line endings may be encoded as LF only (u+0009) or as CR/LF (u+000d u+0009) to maintain compatibility with editors on various popular platforms. However, for data transmission, the canonical format is LF only. Decoders should accept CR/LF as input, but encoders should only output LF when the destination is not a file.



Numeric Types
-------------

### Boolean

Supports the values `true` and `false`.

You may also use the aliases `t` and `f`.

#### Example

    true
    false


### Integer

Integer values may be positive or negative, and may be represented in various bases. Negative values are prefixed with a dash `-` as a sign character. Values must be written in lower case.

Integers may be specified in base 2, 8, 10, or 16. Bases other than 10 must be prefixed:

| Base | Name        | Digits           | Prefix | Example      | Decimal Equivalent |
| ---- | ----------- | ---------------- | ------ | ------------ | ------------------ |
|   2  | Binary      | 01               | 0b     | `-0b1100`    | -12                |
|   8  | Octal       | 01234567         | 0o     | `0o755`      | 493                |
|  10  | Decimal     | 0123456789       |        | `900000`     | 900000             |
|  16  | Hexadecimal | 0123456789abcdef | 0h     | `0xdeadbeef` | 3735928559         |


### Floating Point

A floating point number is composed of a whole part and a fractional part, separated by a dot `.`, with an optional exponential portion. Negative values are prefixed with a dash `-`. If no exponential portion is present, the floating point value is assumed to be in base-10.

    1.0
    -98.413

#### Base-10 Exponential Notation

The exponential portion of a base-10 number is denoted by the lowercase character `e`, followed by the signed size of the base-10 exponent. Values must be normalized (only one digit to the left of the decimal point).

* `6.411e+9` = 6411000000
* `6.411e-9` = 0.000000006411

There is no maximum number of significant digits or exponent digits, but care must be taken to ensure that the receiving end will be able to store the value. 64-bit ieee754 floating point values, for example, can store up to 16 significant digits.

#### Base-16 Exponential Notation

Base-16 floating point numbers allow 100% accurate representation of ieee754 binary floating point values. They begin with `0x`, and the exponential portion is denoted by the lowercase character `p`. The exponential portion itself is a base-10 number representing the power-of-2 to multiply the significand by. Values must be normalized.

* `1.3dep42` = 1.3de (base 16) x 2 ^ 42

Base-16 notation should only be used to support legacy systems that can't handle decimal rounded values. Decimal floating point values tend to be smaller, and also avoid the false precision of binary floating point values. [More info](https://github.com/kstenerud/compact-float/blob/master/compact-float-specification.md#how-much-precision-do-you-need)

#### Floating Point Rules

**There must be one (and only one) dot character:**

| Value          | Notes              |
| -------------- | ------------------ |
| `1`            | Integer, not float |
| `1.0`          | Float              |
| `500000000000` | Integer, not float |
| `5.0e+11`      | Float              |
| `5e+11`        | Invalid            |
| `10.4.5`       | Invalid            |

**There must be at least one digit on each side of the dot character:**

| Invalid      | Valid     | Notes                                                |
| ------------ | --------- | ---------------------------------------------------- |
| `-1.`        | `-1.0`    | Or use integer value -1                              |
| `.1`         | `0.1`     |                                                      |
| `.218901e+2` | `21.8901` | Or `2.18901e+1`                                      |
| `-0`         | `-0.0`    | Special case: -0 cannot be represented as an integer |

**Values with exponential notation must be normalized (one digit, non-zero, to the left of the dot):**

| Invalid      | Valid        |
| ------------ | ------------ |
| `22e+50`     | `2.2e+51`    |
| `508.44e+10` | `5.0844e+12` |
| `-1000e+5`   | `-1.0e+8`    |
| `65.0e-20`   | `6.5e-21`    |
| `0.5e+10`    | `5.0e+11`    |


#### Infinity and Not a Number

The following are special floating point values:

 * `inf`: Infinity
 * `-inf`: Negative Infinity
 * `nan`: Not a Number (quiet)
 * `snan`: Not a Number (signaling)


### Numeric Whitespace

The `_` character may be used as "numeric whitespace" when encoding numeric values.

Rules:

* Numeric values of any type may contain any amount of whitespace at any point after the first digit and before the last digit.
* Special named values `nan`, `snan`, and `inf` must not contain whitespace.

| Value       | Valid Whitespace     | Invalid Whitespace | Notes                                         |
| ----------- | -------------------- | ------------------ | --------------------------------------------- |
| `1000000`   | `1_000_000`          | `_1_000_000`       | `_1_000_000` would be interpreted as a string |
| `-7.4e+100` | `-7_._4__e_+___100`  | `-_7.4e+100`       |                                               |
| `nan`       | `nan`                | `n_an`             | `n_an` would be interpreted as a string       |
| `-inf`      | `-inf`               | `-inf_`            |                                               |

Numeric whitespace characters must be ignored when decoding numeric values.



Temporal Types
--------------

The temporal types deal with time.


### Temporal Type Notes

#### Field Width

Fields other than year may be pre-padded with zeros (`0`) up to their maximum allowed digits for aesthetic reasons if desired. Minutes and seconds must always be padded to 2 digits.

Field values must never be abbreviated (the year 19 refers to 19 AD, not 2019).

#### The Year Field

The year field may be any number of digits, and may be positive (representing AD dates) or negative (representing BC dates). Negative (BC) years are prefixed with a dash character (`-`). The year must always be written in full, and may not be abbreviated.

Note: The Anno Domini system has no zero year (there is no 0 BC or 0 AD), and so the year values `0` and `-0` are invalid. Although many date systems internally use the value 0 to represent 1 BC and offset all BC dates by 1 for mathematical continuity, it's preferable in interchange formats to avoid potential confusion from such tricks.

#### Date Structure

All dates are structured according to the Gregorian calendar, containing a month and day-of-month.

#### Dates prior to 1582

Dates prior to the introduction of the Gregorian calendar in 1582 must be written according to the proleptic Gregorian calendar.


### Time Zones

A time zone refers to the political designation of a location having a specific time offset from UTC during a particular time period. Time zones are in a constant state of flux, and can change at any time for many reasons. There are two ways to denote a time zone: by area/location, and by global coordinates.

If the time zone is unspecified, it is assumed to be `Zero` (UTC).


#### Area/Location

The area/location method is the more human-readable of the two, but may not be precise enough for certain applications. Time zones are partitioned into areas containing locations, and are written in the form `Area/Location`. These areas and locations are specified in the [IANA time zone database](https://www.iana.org/time-zones). Area/Location timezones are case-sensitive because they tend to be implemented that way on most platforms.

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

* `2019.8.5`: August 5, 2019
* `5081.03.30`: March 30, 5081
* `-300.12.21`: December 21, 300 BC (proleptic Gregorian)



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
* Since there is no date component, time zone data must be interpreted as if it were "today", and so the time may not remain constant should the political situation at that time zone change at a later date (except when using `Etc/GMT+1`, etc).
* If the time zone is unspecified, it is assumed to be `Zero` (UTC).

#### Examples

* `9:04:21`: 9:04:21 UTC
* `23:59:59.999999999`: 23:59:59 and 999999999 nanoseconds UTC
* `12:05:50.102/Z`: 12:05:50 and 102 milliseconds UTC
* `4:00:00/Asia/Tokyo`: 4:00:00 Tokyo time
* `17:41:03/-13.54/-172.36`: 17:41:03 Samoa time
* `9:00:00/L`: 9:00:00 local time


### Timestamp

A timestamp combines a date and a time, separated by a dash character (`-`).

#### Examples

* `2019.01.23-14:08:51.941245`: January 23, 2019, at 14:08:51 and 941245 microseconds, UTC
* `5192.11.01-03:00:00/48.86/2.36`: November 1st, 5192, at 3:00:00, at whatever is in the place of Paris at that time.



Array Types
-----------

An "array" for the purposes of this spec represents a contiguous sequence of octets.


### Bytes

An array of octets. This data type should only be used as a last resort if the other data types cannot represent the data you need. To reduce cross-platform confusion, multibyte data types stored within the binary blob should be represented in little endian order whenever possible.

Byte array data is enclosed in double quotes `"`, and is prefixed by the encoding type. The encoded contents may contain whitespace at any point, but there must be no whitespace between the encoding type and the opening double-quote.

Encoding Types:

Supported encoding types are [hex](https://github.com/kstenerud/safe-encoding/blob/master/safe16-specification.md), [safe64](https://github.com/kstenerud/safe-encoding/blob/master/safe64-specification.md), and [safe85](https://github.com/kstenerud/safe-encoding/blob/master/safe85-specification.md). You can choose your encoding type based on your size constraints and desired features.

| Type   | Prefix | Bloat | Features                                |
| ------ | ------ | ----- | --------------------------------------- |
| Hex    |    h   | 2.0   | Human readable, fast encoding/decoding  |
| Safe64 |    6   | 1.33  | Fast encoding/decoding                  |
| Safe85 |    8   | 1.25  | Smallest size, slower encoding/decoding |

#### Examples

    h"39 12 82 e1 81 39 d9 8b 39 4c 63 9d 04 8c"

    h"1 f 4 8 ae 4 56 3" // looks terrible, but is valid

    8"8F2{*RVCLI8LDzZ!3e"

    8"CmsAT9+UpvN!1v=H_SgpMm@mDHDFy(I[~!{I@2
    yx1MU*1I[u!)NL20.1LOvFN-+cu1M_VMH_)d)HD=
    T)I6F~3Ml=.;JP_@>Ln!H$N-xV.1MUpTNKoD71L(
    nBIZop{LR-.0Nh}Y.1ML**I>@ziISc.1OfbXN"


### String

An array of UTF-8 encoded bytes, without a byte order mark (BOM). Strings must not contain NUL (u+0000) or escape sequences that evaluate to NUL.

Strings must be enclosed in double-quotes `"`. Literal double quotes and backslashes `\` within the string must be escaped.

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

Strings must always resolve to complete, valid unicode characters when fully decoded (after evaluating escape sequences).


#### Line Breaks and Whitespace

Care should be taken to ensure that CTE documents can be easily edited in a text editor. For this reason, it is preferable to encode line breaking characters and certain whitespace characters (such as TAB) using escape sequences rather than the bare character coding.

Note: While carriage return (u+000d) is technically allowed in strings, line endings should be converted to linefeed (u+0009) whenever possible to maximize compatibiity between systems.

#### Unquoted String

Normally, strings must be enclosed in double-quotes `"`, but this rule may be relaxed if:

* The string does not contain characters from u+0000 to u+007f - except for lowercase a-z, uppercase A-Z, and underscore (`_`).
* The string does not start with a numeral `[0-9]`.
* The string does not clash with existing CTE keywords such as `nil`, `inf`, `nan`, `snan`, `true`, `false`, `t`, `f`, etc.
* The string does not contain escape sequences or whitespace or line breaks.

Care must be taken that the string values you use are visible and editable in text editors.

#### Example

    "A string\twith\ttabs\nand\nnewlines"

    a_bare_string


### URI

Uniform Resource Identifier, structured in accordance with [RFC 3986](https://tools.ietf.org/html/rfc3986). URIs are prefixed with a lowercase `u` and enclosed in double-quotes `"`.

Note: Percent-encoding sequences within URIs are NOT interpreted; they are passed through as-is.

#### Examples

    u"https://john.doe@www.example.com:123/forum/questions/?tag=networking&order=newest#top"

    u"mailto:John.Doe@example.com"

    u"urn:oasis:names:specification:docbook:dtd:xml:4.1.2"

    u"https://example.com/percent-encoding/?double-quote=%22"



Container Types
---------------

### List

A sequential list of objects. Lists can contain any mix of any type, including other containers.

A list begins with an opening square bracket `[`, whitespace separated contents, and finally a closing bracket `]`.

Note: While this spec allows mixed types in lists, not all languages do. Use mixed types with caution.

#### Example

    [1 two 3.1 {} nil]


### Unordered Map

A unordered map associates objects (keys) with other objects (values), storing the key-value pairs in no particular order. Implementations must not rely on any incidental ordering. Keys may be any mix of scalar or array types. A key must not be a container type, the `nil` type, or any value that evaluates to NaN (not-a-number). Values may be any mix of any type, including other containers.

All keys in a map must resolve to a unique value, even across data types. For example, the following keys would clash:

 * `2000`
 * `2000.0`

Map entries are split into key-value pairs using the equals `=` character and optional whitespace. Key-value pairs are separated from each other using whitespace. A key without a paired value is not allowed in a map.

A map begins with an opening curly brace `{`, whitespace separated key-value pairs, and finally a closing brace `}`.

Note: While this spec allows mixed types in maps, not all languages do. Use mixed types with caution. A decoder may abort processing or ignore key-value pairs of mixed key types if the implementation language doesn't support it.

#### Example

    {
        1 = alpha
        2 = beta
        "a map" = {one = 1}
    }


### Ordered Map

An ordered map works the same as an unordered map, except that the order of the key-value pairs in the map is significant, and must be preserved.

Ordered maps use angle brackets `<` and `>` instead of curly braces `{` and `}`.

#### Example

    <
        first = "This is the first item"
        new_second = "This is the second item"
        old_second = "This used to be the second item"
    >



Metadata Types
--------------

Metadata is data about the data. It describes things about whatever data follows it in a document, which may or may not necessarily be of interest to a consumer of the data. For this reason, decoders are free to ignore and discard metadata if they so choose. Senders and receivers should negotiate beforehand how to react to metadata.


### Metadata Association

Metadata objects are pseudo-objects that can be placed anywhere a real object can be placed, but do not count as objects themselves. Instead, metadata is associated with the object that follows it. For example:

    (map) ("a key") (metadata) ("a value") (end)

In this case, the metadata refers to the value `"a value"`, but the actual data for purposes of decoding the map is `(map) ("a key") ("a value") (end)`.

    (map) ("a key") (metadata) (end)

This map is invalid, because it resolves to `(map) ("a key") (end)`, with no value associated with the key (the metadata doesn't count).

Metadata can also refer to other metadata, for example:

    (map) (metadata-1) (metadata) ("a key") ("a value") (end)

In this case, `(metadata)` refers to the string `"a key"`, and `(metadata-1)` refers to `(metadata)`. The actual map is `(map) ("a key") ("a value") (end)`.

#### Exception: Comments

The metadata association rules do not apply to [comments](#comment). Comments stand entirely on their own, and do not officially refer to anything, nor can any other metadata refer to a comment (i.e. comments are invisible to other metadata).


### Metadata Map

A metadata map is an ordered map containing metadata about the object that follows the map. A metadata map may contain anything that an ordered map can, but all string keys that begin with an underscore (`_`) are reserved, and must not be used except in the ways defined by this specification.

Metadata map contents are enclosed within parenthesis: `(` and `)`

#### Name Clashes

There are various metadata standards in use today (https://en.wikipedia.org/wiki/Metadata_standard). Care should be taken to ensure that your chosen metadata system doesn't clash with other established naming schemes. In general, international standards tend to use URI style identifiers, so the risk is usually minimal when applying string and numeric keys. However, a little foresight and research goes a long way towards avoiding pain down the line!

#### Predefines Keys

This standard specifies predefined keys for the most common metadata in ad-hoc data structures. Specifying these keys maximizes the chances of disparate systems understanding one other, and avoids a lot of duplication.

The following predefined metadata keys must be used for the specified type of information (decoders must accept both regular and short key versions):

| Regular Key          | Short Key | Type          | Contents          |
| -------------------- | --------- | ------------- | ----------------- |
| `_creation_time`     | `_ct`     | Timestamp     | Creation time     |
| `_modification_time` | `_mt`     | Timestamp     | Modification time |
| `_access_time`       | `_at`     | Timestamp     | Last access time  |
| `_tags`              | `_t`      | List          | Set of tags       |
| `_attributes`        | `_a`      | Unordered Map | Attributes        |

All other metadata keys beginning with `_` are reserved for future expansion, and must not be used.

Note: Metadata must not be placed before the [version specifier](#version-specifier).

#### Example

    v1
    // Metadata for the entire document
    (
        _ct = 2017.01.14-15:22:41/Z
        _mt = 2019.08.17-12:44:31/Z
        _at = 2019.09.14-09:55:00/Z
    )
    {
        records = [
            // Metadata for "ABC Corp" record
            (
                _ct = 2019.05.14-10:22:55/Z
                _t = ["longtime client" "big purchases"]
            )
            {
                client = "ABC Corp"
                amount = 10499.28
                due = 2020.05.14
            }
            // Metadata for "XYZ Corp" record
            ( _ct = 2019.02.30-09:00:01/Z  _mt = 2019.08.17-12:44:31/Z )
            {
                client = "XYZ Corp"
                amount = 3994.01
                due = 2020.08.30
            }
        ]
    }


### Comment

Comments are user-defined string metadata equivalent to comments in a source code document. Comments do not officially refer to other objects, although conventionally they tend to refer to what follows in the document, be it a single object, a series of objects, a distant object, or they might even be entirely standalone. This is similar to how source code comments are used.

Comment contents must contain only complete and valid UTF-8 sequences. Escape sequences in comments are not interpreted (they are passed through verbatim).

Comments may be written in single-line or multi-line form. The single-line form starts with a double slash `//` and ends at a newline. The multi-line form starts with the sequence `/*` and ends with the sequence `*/`. They operate similarly to how comments operate in C-like languages. Nested multiline comments are not allowed.

Note: Comments must not be placed before the [version specifier](#version-specifier).

#### Comment Character Restrictions

The following characters are explicitly allowed:

 * Horizontal Tab (u+0009)
 * Linefeed (u+000a)
 * Carriage Return (u+000d)

The following characters are disallowed if they aren't in the above allowed section:

 * Control characters (such as u+0000 to u+001f, u+007f to u+009f).
 * Line breaking characters (such as u+2028, u+2029).
 * Byte order mark.

The following characters are allowed if they aren't in the above disallowed section:

 * UTF-8 printable characters
 * UTF-8 whitespace characters

#### Example

    v1
    // Comment before top level object
    {
        // Comment before the "name" object.
        // And another comment.
        "name" = "Joe Average" // Comment after the "Joe Average" object.
        "email" = // Comment after the "email" key.
        /* Multiline comment with nested single line comment inside
        u"mailto:joe@average.org" // Comment after email
        */
        u"mailto:someone@somewhere.com"
        "data" // Comment after data
        =
        //
        // Comment before some binary data (but not inside it)
        h"01 02 03 04 05 06 07 08 09 0a"
    }
    // Comments at the
    // end of the document.



Other Types
-----------

### Nil

Denotes the absence of data. Some languages implement this as the `null` value.

Note: Use nil judiciously and sparingly, as some languages may have restrictions on how and if it may be used.

#### Example

    nil



Letter Case
-----------

A CTE document must be entirely in lower case, with the following exceptions:

 * String and comment contents: `"A string may contain UPPER CASE. Escape sequences must be lower case: \x3d"`
 * [Time zones](#time-zones) are case sensitive, and contain uppercase characters.
 * Safe85 and Safe64 encodings make use of uppercase characters in their code tables.

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
 * Between array/container openings & closings: `[`, `]`, `{`, `}`, `<`, `>`, `(`, `)`
 * Between encoding characters in a byte array, and array delimiters.

Examples:

 * `[   1     2      3 ]` is equivalent to `[1 2 3]`
 * `h" 01 02 03   0 4 "` is equivalent to `h"01020304"`
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

    // Comment whitepsace      is preserved.



Invalid Encodings
-----------------

Invalid encodings must not be used, as they may cause problems or even API violations in certain languages. A parser must halt processing when invalid data is detected.

 * Numeric values must be representable in their respective binary formats (integer, binary float, decimal float).
 * Times must be valid. For example: 2000.2.30, while technically encodable, is not allowed.
 * Containers must be properly terminated. Extra container endings (`}`, `]`, etc) are invalid.
 * All map keys must have corresponding values. A key with a missing value is invalid.
 * Map keys must not be container types, the `nil` type, or values the resolve to NaN (not-a-number).
 * Maps must not contain duplicate keys. This includes numeric keys of different types that resolve to the same value.
 * All UTF-8 sequences must evaluate to complete, valid characters.
 * Metadata keys beginning with `_` must not be used, except for those listed in this specifiction.
 * Nested multiline comments are not allowed.
 * Upper case text is not allowed, except as described in section [Letter Case](#letter-case).
 * Whitespace must only occur as described in section [Whitespace](#whitespace).



Version History
---------------

July 22, 2018: Preview Version 1



License
-------

Copyright (c) 2018 Karl Stenerud. All rights reserved.

Distributed under the Creative Commons Attribution License: https://creativecommons.org/licenses/by/4.0/legalcode
License deed: https://creativecommons.org/licenses/by/4.0/
