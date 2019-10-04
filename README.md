Concise Text Encoding
=====================

General purpose, compact representations of semi-structured hierarchical data.

Alternative To:

* JSON
* XML



Features
--------

  * General purpose encoding for a large number of applications
  * Supports the most common data types
  * Supports hierarchical data structuring
  * Human readable format
  * Minimal complexity
  * Type compatible with [Concise Binary Encoding (CBE)](https://github.com/kstenerud/concise-binary-encoding/blob/master/cbe-specification.md)



Example
-------

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



Supported Types
---------------


### Numeric Types

| Type          | Description                                           |
| ------------- | ----------------------------------------------------- |
| Boolean       | True or false                                         |
| Integer       | Signed two's complement integer                       |
| Decimal Float | Compressed decimal floating point                     |
| Binary Float  | IEEE 754 binary floating point                        |


### Temporal Types

| Type          | Description                                           |
| ------------- | ----------------------------------------------------- |
| Date          | Date, with unlimited year range                       |
| Time          | Time, with precision to the nanosecond, and time zone |
| Timestamp     | Combined date and time                                |


### Array Types
 
| Type          | Description                                           |
| ------------- | ----------------------------------------------------- |
| Bytes         | Array of binary data                                  |
| String        | Array of UTF-8 encoded bytes                          |
| URI           | Universal Resource Identifier                         |


### Container Types

Containers can hold any combination of types, including other containers.

| Type          | Description                                           |
| ------------- | ----------------------------------------------------- |
| List          | A list may containin any types, even mixed            |
| Unordered Map | Scalar or array types for keys, any types for values  |
| Ordered Map   | key-value pairs are explicitly ordered                |


### Metadata Types

| Type          | Description                                           |
| ------------- | ----------------------------------------------------- |
| Metadata Map  | Metadata about an object                              |
| Comment       | A UTF-8 encoded comment string                        |


### Other Types

| Type          | Description                                           |
| ------------- | ----------------------------------------------------- |
| Nil           | Denotes the absence of data                           |



Specification
-------------

 * [Concise Text Encoding](cte-specification.md)



Implementations
---------------

TODO:

* [C implementation](reference-implementation)



License
-------

Copyright Karl Stenerud. All rights reserved.

Specifications released under [Creative Commons Attribution 4.0 International Public License](LICENSE.md).
