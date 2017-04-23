![eventStream](eventStreamBanner.png "The Event Stream banner")

# Event Stream

Event Stream is a file format specification. It is meant as a standard for storing events streams, and can be used for transmitting events between electronic devices. The specification includes a versioning system, in order to allow massive changes while keeping backward compatibility. The recommanded extension for Event Stream files is *.es*. For a byte `b`, `b[0]` denotes the LSB (least significant bit) and `b[7]` denotes the MSB (most significant bit).

# Binary file structure

Every Event Stream file starts with a header of 15 bytes:

| Position      | Content                                                                                      |
|:-------------:|:--------------------------------------------------------------------------------------------:|
| Bytes 0 to 11 | `0x45 0x76 0x65 0x6e 0x74 0x20 0x53 0x74 0x72 0x65 0x61 0x6d` (_Event Stream_ ascii-encoded) |
| Byte 12       | Major version number                                                                         |
| Byte 13       | Minor version number                                                                         |

Bytes from 14 to the end are version dependent. The content description for each version is given below.

## Version 1.0

The file can represent three types of streams: ATIS events, Asynchronous & Modular Display events and color events. The type is stored in byte 14, which completes the header:

| Byte 14 | Stream type                           |
|:-------:|:-------------------------------------:|
| `0x00`  | DVS events                            |
| `0x01`  | ATIS events                           |
| `0x02`  | Asynchronous & Modular Display events |
| `0x03`  | Color events                          |
| `0x04`  | Generic events                        |

### DVS events

Each byte from the 15th to the end can be any of _byte 0_, _byte 1_, _byte 2_, _reset_ and _overflow_. The possible order of these bytes is given by the state machine:

![dvsStateMachine](dvsStateMachine.png "DVS state machine")

The bytes encode the following data:

| Byte name  | LSB            | Bit 1          | Bit 2          | Bit 3          | Bit 4          | Bit 5         | Bit 6         | MSB           |
|:----------:|:--------------:|:--------------:|:--------------:|:--------------:|:--------------:|:-------------:|:-------------:|:-------------:|
| _byte 0_   | `timestamp[0]` | `timestamp[1]` | `timestamp[2]` | `timestamp[3]` | `x[0]`         | `x[1]`        | `x[2]`        | `x[3]`        |
| _byte 1_   | `x[4]`         | `x[5]`         | `x[6]`         | `x[7]`         | `x[8]`         | `x[9]`        | `y[0]`        | `y[1]`        |
| _byte 2_   | `y[2]`         | `y[3]`         | `y[4]`         | `y[5]`         | `y[6]`         | `y[7]`        | `y[8]`        | `isIncrease`  |
| _reset_    | `1`            | `1`            | `1`            | `1`            | `0`            | `0`           | `0`           | `0`           |
| _overflow_ | `1`            | `1`            | `1`            | `1`            | `overflow[0]`  | `overflow[1]` | `overflow[2]` | `overflow[3]` |

_reset_ is a special event inserted when deemed necessary to correct state machine errors resulting from bit errors. _reset_ events are always sent three-by-three, to make sure that at least the third _reset_ is read while in _idle_ state.

_timestamp_ encodes the time elapsed since the previous event in microseconds, and cannot be `0b1111`. If this time is equal or larger than `0b1111` microseconds, one or serveral _overflow_ events are inserted before the event. The actual time elapsed since the last event can be computed as the current event's timestamp plus `0b1111` microseconds multiplied by the number represented by `overflow[0]`, `overflow[1]`, `overflow[2]`, `overflow[3]` for each _overflow_ event.

### ATIS events

Each byte from the 15th to the end can be any of _byte 0_, _byte 1_, _byte 2_, _reset_ and _overflow_. The possible order of these bytes is given by the state machine:

![atisStateMachine](atisStateMachine.png "ATIS state machine")

The bytes encode the following data:

| Byte name  | LSB            | Bit 1          | Bit 2          | Bit 3          | Bit 4          | Bit 5         | Bit 6                 | MSB           |
|:----------:|:--------------:|:--------------:|:--------------:|:--------------:|:--------------:|:-------------:|:---------------------:|:-------------:|
| _byte 0_   | `timestamp[0]` | `timestamp[1]` | `timestamp[2]` | `timestamp[3]` | `timestamp[4]` | `x[0]`        | `x[1]`                | `x[2]`        |
| _byte 1_   | `x[3]`         | `x[4]`         | `x[5]`         | `x[6]`         | `x[7]`         | `x[8]`        | `y[0]`                | `y[1]`        |
| _byte 2_   | `y[2]`         | `y[3]`         | `y[4]`         | `y[5]`         | `y[6]`         | `y[7]`        | `isThresholdCrossing` | `polarity`    |
| _reset_    | `1`            | `1`            | `1`            | `1`            | `1`            | `0`           | `0`                   | `0`           |
| _overflow_ | `1`            | `1`            | `1`            | `1`            | `1`            | `overflow[0]` | `overflow[1]`         | `overflow[2]` |

_reset_ is a special event inserted when deemed necessary to correct state machine errors resulting from bit errors. _reset_ events are always sent three-by-three, to make sure that at least the third _reset_ is read while in _idle_ state.

_timestamp_ encodes the time elapsed since the previous event in microseconds, and cannot be `0b11111`. If this time is equal or larger than `0b11111` microseconds, one or serveral _overflow_ events are inserted before the event. The actual time elapsed since the last event can be computed as the current event's timestamp plus `0b11111` microseconds multiplied by the number represented by `overflow[0]`, `overflow[1]`, `overflow[2]` for each _overflow_ event.

### Asynchronous & Modular Display events

Each byte from the 15th to the end can be any of _byte 0_, _byte 1_, _byte 2_, _reset_ and _overflow_. The possible order of these bytes is given by the state machine:

![atisStateMachine](atisStateMachine.png "ATIS State machine")

The bytes encode the following data:

| Byte name  | LSB            | Bit 1          | Bit 2          | Bit 3          | Bit 4          | Bit 5         | Bit 6         | MSB           |
|:----------:|:--------------:|:--------------:|:--------------:|:--------------:|:--------------:|:-------------:|:-------------:|:-------------:|
| _byte 0_   | `timestamp[0]` | `timestamp[1]` | `timestamp[2]` | `timestamp[3]` | `timestamp[4]` | `x[0]`        | `x[1]`        | `x[2]`        |
| _byte 1_   | `intensity[0]` | `intensity[1]` | `intensity[2]` | `intensity[3]` | `intensity[4]` | `y[0]`        | `y[1]`        | `y[2]`        |
| _byte 2_   | `address[0]`   | `address[1]`   | `address[2]`   | `address[3]`   | `address[4]`   | `address[5]`  | `address[6]`  | `address[7]`  |
| _reset_    | `1`            | `1`            | `1`            | `1`            | `1`            | `0`           | `0`           | `0`           |
| _overflow_ | `1`            | `1`            | `1`            | `1`            | `1`            | `overflow[0]` | `overflow[1]` | `overflow[2]` |

_reset_ is a special event inserted when deemed necessary to correct state machine errors resulting from bit errors. _reset_ events are always sent three-by-three, to make sure that at least the third _reset_ is read while in _idle_ state.

_timestamp_ encodes the time elapsed since the previous event in microseconds, and cannot be `0b11111`. If this time is equal or larger than `0b11111` microseconds, one or serveral _overflow_ events are inserted before the event. The actual time elapsed since the last event can be computed as the current event's timestamp plus `0b11111` microseconds multiplied by the number represented by `overflow[0]`, `overflow[1]`, `overflow[2]` for each _overflow_ event.

### Color events

Each byte from 15 to the end can be any of _byte 0_, _byte 1_, _byte 2_, _byte 3_, _byte 4_, _byte 5_, _reset_ and _overflow_. The possible order of these bytes is given by the state machine:

![colorStateMachine](colorStateMachine.png "Color state machine")

The bytes encode the following data:

| Byte name  | LSB            | Bit 1          | Bit 2          | Bit 3          | Bit 4          | Bit 5          | Bit 6          | MSB    |
|:----------:|:--------------:|:--------------:|:--------------:|:--------------:|:--------------:|:--------------:|:--------------:|:------:|
| _byte 0_   | `timestamp[0]` | `timestamp[1]` | `timestamp[2]` | `timestamp[3]` | `timestamp[4]` | `timestamp[5]` | `timestamp[6]` | `x[0]` |
| _byte 1_   | `x[1]`         | `x[2]`         | `x[3]`         | `x[4]`         | `x[5]`         | `x[6]`         | `x[7]`         | `x[8]` |
| _byte 2_   | `y[0]`         | `y[1]`         | `y[2]`         | `y[3]`         | `y[4]`         | `y[5]`         | `y[6]`         | `y[7]` |
| _byte 3_   | `r[0]`         | `r[1]`         | `r[2]`         | `r[3]`         | `r[4]`         | `r[5]`         | `r[6]`         | `r[7]` |
| _byte 4_   | `g[0]`         | `g[1]`         | `g[2]`         | `g[3]`         | `g[4]`         | `g[5]`         | `g[6]`         | `g[7]` |
| _byte 5_   | `b[0]`         | `b[1]`         | `b[2]`         | `b[3]`         | `b[4]`         | `b[5]`         | `b[6]`         | `b[7]` |
| _reset_    | `1`            | `1`            | `1`            | `1`            | `1`            | `1`            | `1`            | `0`    |
| _overflow_ | `1`            | `1`            | `1`            | `1`            | `1`            | `1`            | `1`            | `1`    |

_reset_ is a special event inserted when deemed necessary to correct state machine errors resulting from bit errors. _reset_ events are always sent six-by-six, to make sure that at least the sixth _reset_ is read while in _idle_ state.

_timestamp_ encodes the time elapsed since the previous event in microseconds, and cannot be `0b1111111`. If this time is equal or larger than `0b1111111` microseconds, one or serveral _overflow_ events are inserted before the event. The actual time elapsed since the last event can be computed as the current event's timestamp plus `0b1111111` microseconds multiplied by the number of _overflow_ events.

### Generic events

Each byte from 15 to the end can be any of _byte 0_, _byte 1_, _byte 2_, _byte 3_, _byte 4_, _byte 5_, _byte 6_, _byte 7_, _byte 8_, _reset_ and _overflow_. The possible order of these bytes is given by the state machine:

![genericStateMachine](genericStateMachine.png "Generic state machine")

The bytes encode the following data:

| Byte name  | LSB            | Bit 1          | Bit 2          | Bit 3          | Bit 4          | Bit 5          | Bit 6          | MSB        |
|:----------:|:--------------:|:--------------:|:--------------:|:--------------:|:--------------:|:--------------:|:--------------:|:----------:|
| _byte 0_   | `timestamp[0]` | `timestamp[1]` | `timestamp[2]` | `timestamp[3]` | `timestamp[4]` | `timestamp[5]` | `timestamp[6]` | `extraBit` |
| _byte 1_   | `data[0]`      | `data[1]`      | `data[2]`      | `data[3]`      | `data[4]`      | `data[5]`      | `data[6]`      | `data[7]`  |
| _byte 2_   | `data[8]`      | `data[9]`      | `data[10]`     | `data[11]`     | `data[12]`     | `data[13]`     | `data[14]`     | `data[15]` |
| _byte 3_   | `data[16]`     | `data[17]`     | `data[18]`     | `data[19]`     | `data[20]`     | `data[21]`     | `data[22]`     | `data[23]` |
| _byte 4_   | `data[24]`     | `data[25]`     | `data[26]`     | `data[27]`     | `data[28]`     | `data[29]`     | `data[30]`     | `data[31]` |
| _byte 5_   | `data[32]`     | `data[33]`     | `data[34]`     | `data[35]`     | `data[36]`     | `data[37]`     | `data[38]`     | `data[39]` |
| _byte 6_   | `data[40]`     | `data[41]`     | `data[42]`     | `data[43]`     | `data[44]`     | `data[45]`     | `data[46]`     | `data[47]` |
| _byte 7_   | `data[48]`     | `data[49]`     | `data[50]`     | `data[51]`     | `data[52]`     | `data[53]`     | `data[54]`     | `data[55]` |
| _byte 8_   | `data[56]`     | `data[57]`     | `data[58]`     | `data[59]`     | `data[60]`     | `data[61]`     | `data[62]`     | `data[63]` |
| _reset_    | `1`            | `1`            | `1`            | `1`            | `1`            | `1`            | `1`            | `0`        |
| _overflow_ | `1`            | `1`            | `1`            | `1`            | `1`            | `1`            | `1`            | `1`        |

_reset_ is a special event inserted when deemed necessary to correct state machine errors resulting from bit errors. _reset_ events are always sent six-by-six, to make sure that at least the sixth _reset_ is read while in _idle_ state.

_timestamp_ encodes the time elapsed since the previous event in microseconds, and cannot be `0b1111111`. If this time is equal or larger than `0b1111111` microseconds, one or serveral _overflow_ events are inserted before the event. The actual time elapsed since the last event can be computed as the current event's timestamp plus `0b1111111` microseconds multiplied by the number of _overflow_ events.

# License

See the [LICENSE](LICENSE.txt) file for license rights and limitations (GNU GPLv3).
