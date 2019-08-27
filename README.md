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
  * Type compatible with [Concise Binary Encoding (CBE)](https://github.com/kstenerud/concise-binary-encoding/cbe-specification.md)



Supported Types
---------------


### Scalar Types

| Type          | Description                                           |
| ------------- | ----------------------------------------------------- |
| Boolean       | True or false                                         |
| Integer       | Signed integers                                       |
| Float         | Floating point, interpreted as IEEE 754 binary        |
| Decimal       | Floating point, interpreted as IEEE 754 decimal       |


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


### Other Types

| Type          | Description                                           |
| ------------- | ----------------------------------------------------- |
| Nil           | Denotes the absence of data                           |
| Comment       | A UTF-8 encoded comment string                        |



Specification
-------------

 * [Concise Text Encoding](cte-specification.md)



Implementations
---------------

* [C implementation](reference-implementation)



License
-------

Copyright (c) 2018 Karl Stenerud. All rights reserved.

Specifications released under Creative Commons Attribution 4.0 International Public License.
Reference implementation released under MIT License.
