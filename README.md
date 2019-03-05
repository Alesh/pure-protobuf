# `pure-protobuf`

Python implementation of [Protocol Buffers](http://code.google.com/apis/protocolbuffers/docs/encoding.html) data types.

[![Build Status](https://travis-ci.org/eigenein/protobuf.svg?branch=master)](https://travis-ci.org/eigenein/protobuf)
[![PyPI - Downloads](https://img.shields.io/pypi/dm/pure-protobuf.svg)](https://pypi.org/project/pure-protobuf/)
[![PyPI – Version](https://img.shields.io/pypi/v/pure-protobuf.svg)](https://pypi.org/project/pure-protobuf/#history)
[![PyPI – Python](https://img.shields.io/pypi/pyversions/pure-protobuf.svg)](https://pypi.org/project/pure-protobuf/#files)
![License](https://img.shields.io/pypi/l/pure-protobuf.svg)

## Usage

Assume you have the following definition:

```proto
message Test2 {
  string b = 2;
}
```
    
This is how you can create a message and get it serialized:

```python
from __future__ import print_function

from StringIO import StringIO

from pure_protobuf import MessageType, Unicode

# Create the type instance and add the field.
type_ = MessageType()
type_.add_field(2, 'b', Unicode)

message = type_()
message.b = 'testing'

# Dump into a string.
print(message.dumps())

# Dump into a file-like object.
fp = StringIO()
message.dump(fp)

# Load from a string.
assert type_.loads(message.dumps()) == message

# Load from a file-like object.
fp.seek(0)
assert type_.load(fp) == message
```

### Sample 2. Required field

To add a missing field you should pass an additional `flags` parameter to `add_field` like this:

```python
from pure_protobuf.protobuf import Flags, MessageType, Unicode

type_ = MessageType()
type_.add_field(2, 'b', Unicode, flags=Flags.REQUIRED)

message = type_()
message.b = 'hello, world'

assert type_.dumps(message)
```
    
If you'll not fill in a required field, then `ValueError` will be raised during serialization.

### Sample 3. Repeated field

```python
from pure_protobuf.protobuf import Flags, MessageType, UVarint

type_ = MessageType()
type_.add_field(1, 'b', UVarint, flags=Flags.REPEATED)

message = type_()
message.b = (1, 2, 3)

assert type_.dumps(message)
```
    
Value of a repeated field can be any iterable object. The loaded value will always be `list`.

### Sample 4. Packed repeated field

```python
from pure_protobuf.protobuf import Flags, MessageType, UVarint

type_ = MessageType()
type_.add_field(4, 'd', UVarint, flags=Flags.PACKED_REPEATED)

message = type_()
message.d = (3, 270, 86942)

assert type_.dumps(message)
```
    
### Sample 5. Embedded messages

```proto
message Test1 {
  int32 a = 1;
}

message Test3 {
  required Test1 c = 3;
}
```
    
To create an embedded field, wrap inner type with `EmbeddedMessage`:

```python
from pure_protobuf import EmbeddedMessage, MessageType, UVarint

inner_type = MessageType()
inner_type.add_field(1, 'a', UVarint)
outer_type = MessageType()
outer_type.add_field(3, 'c', EmbeddedMessage(inner_type))

message = outer_type()
message.c = inner_type()
message.c.a = 150

assert outer_type.dumps(message)
```
    
## Data types

There are the following data types supported for now:

    UVarint             # Unsigned integer.
    Varint              # Signed integer.
    Bool                # Boolean.
    Fixed64             # 8-byte string.
    UInt64              # C++'s 64-bit `unsigned long long`
    Int64               # C++'s 64-bit `long long`
    Float64             # C++'s `double`.
    Fixed32             # 4-byte string.
    UInt32              # C++'s 32-bit `unsigned int`.
    Int32               # C++'s 32-bit `int`.
    Float32             # C++'s `float`.
    Bytes               # Pure bytes string.
    Unicode             # Unicode string.

## Some techniques

### Streaming messages

The Protocol Buffers format is not self-delimiting. But you can wrap you message type in `EmbeddedMessage` class and write or read it sequentially.

The other option is to use `protobuf.EofWrapper` that has a `limit` parameter in its constructor. The `EofWrapper` raises `EOFError` when the specified number of bytes is read.

### `add_field` chaining

`add_field` return the message type itself, thus you can do so:

```python
from pure_protobuf import EmbeddedMessage, MessageType, UVarint

MessageType().add_field(1, 'a', EmbeddedMessage(MessageType().add_field(1, 'a', UVarint)))
```
