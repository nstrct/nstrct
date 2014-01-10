# nstrct

**a multi-purpose binary protocol for instruction interchange**

Interchange formats like json or xml are great to keep data visible, but due to their parse and pack complexity they aren't used in embedded applications. There are alternatives like msgpack or Google's protocol buffer, which allow a more binary representation of data, but these protcols are still heavy and developers tend to rather implement their own 'simple' binary protocols instead of porting or using the big ones. 

The protcol **nstrct** is designed to be an alternative in those situations where a tiny and versatile protocol is needed. The main transportable unit in nstrct are **instructions** which carry a dynamic amount of data in the form of **arguments**. Each instruction is identified by a code between 0-65535 which can be used by two communicating applications to identify the blueprint of the message. Arguments support all standard types of integers(boolean, int8-64, uint8-64, float32/64), strings, and arrays of integers or strings. There is no support for hashes by design to keep the pack and unpacking functions as small as possible. [Check the main repository for the binary layout of the instructions](http://github.com/nstrct/nstrct).

**Start using nstrct by getting the library for your favorite programming language:**

* [C/C++ (nstrct-c)](http://github.com/nstrct/nstrct-c)
* [Ruby (nstrct-ruby)](http://github.com/nstrct/nstrct-ruby)
* [Obj-C (nstrct-objc)](http://github.com/nstrct/nstrct-objc)
* [Java (nstrct-c)](http://github.com/nstrct/nstrct-java)

_This software has been open sourced by [ElectricFeel Mobility Systems Gmbh](http://electricfeel.com) and is further maintained by [Joël Gähwiler](http://github.com/256dpi)._

## Binary Layout

Check the following tables and details to understand the binary structure of the protocol. Keep in mind that network byte order (BigEndian) is used by all implementations and libraries.

### Frame

The default frame secures transport with a start and end byte plus the crc16 checksum of the payload.

The payload can be up to 64 KB.

    +------+-----------------+~~~~~~~~~+-------------+------+
    | 0x55 | payload_size 2B | payload | checksum 2B | 0xAA |
    +------+-----------------+~~~~~~~~~+-------------+------+

* **payload_size**: the payload in amount of bytes
* **payload**: the payload
* **checkusm**: crc16 checksum of the payload

### Instruction

An instruction can have up to 255 arguments.

    +-------------+---------------------+---------------------------+~~~~~~~~~~~~~+
    | code uint16 | num_arguments uint8 | num_array_elements uint16 | [arguments] |
    +-------------+---------------------+---------------------------+~~~~~~~~~~~~~+

* **code**: the instructions code
* **num\_arguments**: the total of arguments following
* **num\_array\_elements**: the total number of array elements in all arguments following
* **arguments**: specified amount of arguments after each other

### Argument

    +------------+~~~~~~~+
    | type uint8 | value |
    +------------+~~~~~~~+

* **type**: the type of the value following
* **value**: the value

**Argument Types:**

    Dec  Type     Length
    --------------------
    01   boolean  1B
    10   int8     1B
    11   int16    2B
    12   int32    4B
    13   int64    8B
    14   uint8    1B
    15   uint16   2B
    16   uint32   4B
    17   uint64   8B
    20   float32  4B
    21   float64  8B
    31   string
    32   array

### String

A string can be up to 255 characters.

    +--------------+~~~~~~+
    | length uint8 | data |
    +--------------+~~~~~~+

* **length**: the strings length
* **data**: the string itself

### Array

An array can have up to 255 elements.

    +---------------------+--------------------+~~~~~~~~+
    | elements_type uint8 | num_elements uint8 | values |
    +---------------------+--------------------+~~~~~~~~+

* **element\_type:** the type of the following elements
* **num\_elements:** the total number of following elements
* **values:** all the elements values after each other
