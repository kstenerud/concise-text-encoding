Concise Text Encoding
=====================

Data communications have become needlessly bloated, wasting bandwidth and power serializing, deserializing, and transmitting data in text formats. The open, text based encodings like SGML, HTML, XML, and JSON got us past the obscure, proprietary formats of the 90s. But now their size, codec complexity, and limited type support are becoming liabilities. The available binary formats also suffer from lack of types or are unnecessarily complex, and can't be read or edited by humans (a major benefit of text formats).

The age of plentiful bandwidth and processing power is coming to an end. Energy efficiency in software and data transmission is fast becoming a serious cost, battery, heat dissipation, and environmental issue. We must meet these challenges with data formats that are versatile, compact, and computationally simple to process when on the critical path, while also retaining human readability and editability.

Concise Encoding is the next step in the evolution of ad-hoc hierarchical data formats, aiming to support 80% of data use cases in a power and bandwidth friendly way:

 * Completely redesigned from the ground up to balance human readability, encoded size, and codec complexity.
 * Split into two formats: [binary-based CBE](https://github.com/kstenerud/concise-binary-encoding) and [text-based CTE](https://github.com/kstenerud/concise-text-encoding).
 * 1:1 type compatibility between formats, allowing transparent conversion between CBE and CTE. You can use the more efficient binary format for data interchange and storage, and convert to/from text only when a human needs to be involved.
 * Fixed-length multi-byte data fields are encoded in little endian byte order to avoid extra processing in the most popular hardware.
 * CBE and CTE are fully specified, eliminating ambiguities and covering edge cases.
 * Documents and specifications are versioned to support future expansion.
 * Support for metadata and comments.
 * Support for references to other parts of the document or to other documents.
 * Support for the most commonly used data types.



Request for Comments
--------------------

This prototype specification is now [open to comments from the public](https://github.com/kstenerud/concise-encoding/blob/master/request-for-comments.md). Please join the discussion!



Example
-------

    v1
    // _ct is the creation time, in this case referring to the document
    (_ct = 2019.9.1-22:14:01)
    {
        // A comment
        /* A multiline
           comment */
        (metadata_about_a_list = "something interesting about a_list")
        a_list          = [1 2 "a string"]
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



Supported Types
---------------

CTE aims to natively support 80% of data use cases. To this end, it provides the following fundamental data types:


### Numeric Types

| Type          | Description                                                |
| ------------- | ---------------------------------------------------------- |
| Boolean       | True or false                                              |
| Integer       | Positive or negative integer of arbitrary size             |
| Decimal Float | Decimal exponent based floating point of arbitrary size    |
| Binary Float  | IEEE754 compatible binary floating point                   |


### Temporal Types

| Type          | Description                                                |
| ------------- | ---------------------------------------------------------- |
| Date          | Date with unlimited year range                             |
| Time          | Time with time zone and precision to the nanosecond        |
| Timestamp     | Combined date and time                                     |


### Array Types

Array types refer to arrays of octets, encoded in specific ways.

| Type          | Description                                                |
| ------------- | ---------------------------------------------------------- |
| Bytes         | Array of binary data                                       |
| String        | Array of UTF-8 encoded characters                          |
| URI           | Universal Resource Identifier                              |


### Container Types

Containers can hold other objects, including other containers. Contents can be of any type, including mixed types. Map keys can be of any type except containers, nil, and NaN.

| Type          | Description                                                |
| ------------- | ---------------------------------------------------------- |
| List          | Ordered collection of objects                              |
| Map           | Ordered mapping of key objects to value objects            |


### Metadata Types

Metadata types describe other data.

| Type          | Description                                                |
| ------------- | ---------------------------------------------------------- |
| Metadata Map  | A map containing metadata about the object that follows it |
| Comment       | A UTF-8 encoded comment string                             |


### Other Types

| Type          | Description                                                |
| ------------- | ---------------------------------------------------------- |
| Nil           | Denotes the absence of data                                |
| Reference     | References another object in the document or externally    |



Specification
-------------

 * [Concise Text Encoding](cte-specification.md)



Implementations
---------------

TODO:

 * C implementation
 * [Go implementation](https://github.com/kstenerud/go-cte)



License
-------

Copyright (C) Karl Stenerud. All rights reserved.

Specifications released under [Creative Commons Attribution 4.0 International Public License](LICENSE.md).
