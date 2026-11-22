# Packet Format
Bytes are sent as most significant bit first.
Packets are sent as most significant byte first (big endian).

# Packet Structure
1 Byte Command and Request
13 Bytes Data 
2 Byte CRC

When referring to a specific bit in this page we say byte:bit. 15:7 is the most significant bit of the most significant byte (for our 16 byte packets). We can also refer to an entire byte at once with just its index. 0 is the least significant byte for example. Ranges are depicted with -> and are inclusive on both ends.

Any bits or bytes not used should be packed with 0's.

# Data Types
String is UTF8 zero filled string.
Float is :
 - 1 Byte = Open Compute Project OFP8 E4M3
 - 2 Byte = IEEE 754 binary16
 - 4 Byte = IEEE 754 binary32

When given size options, floats, ints and uints are only valid for 1, 2, or 4 byte options. 

Only strings are valid for max length options. 

Enums are only valid for 0 -> 60 options.

# Commands
0 - Send Control Signals
1 - Send Logged Values
2 - Request/Send Logged Value Names
3 - Bind Request/Accept
4 - Setting Info
5 - Setting Enum Name
6 - Apply Setting
7:15 - I feel like i could use these values as well but idk what for...
    Maybe i could do the ranging thing, that could be cool


# Command and Request Byte
Command is what this packet is
Request is desired response
Request is not enforced, other responses are allowed

The receiver should almost always do as requested. There are no known reasons for responding differently to that requested.

The transmitter can do things other than requested, primarily that would be requesting or applying settings when the receiver requests a control signal.

| Position | Name | Description |
| ---- | ---- | ---- |
| 15:7 -> 15:4 | Command | |
| 15:3 -> 15:0 | Request | |



# CRC
To calculate the crc, append the UID to the Command/Request and Data bytes, then use CRC16 IBM 3740 (Also known as CRC16 CCITT FALSE) as described in https://reveng.sourceforge.io/crc-catalogue/all.htm


# Send Control Signals (0)
6x 12 bit analog channels (9 Bytes)
1x Button byte

| Position | Name | Description |
| ---- | ---- | ---- |
| 14:7 -> 13:4 | Left X | |
| 13:3 -> 12:0 | Left Y | |
| 11:7 -> 10:4 | Left Trigger | |
| 10:3 -> 9:0 | Right X | |
| 8:7 -> 7:4 | Right Y | |
| 7:3 -> 6:0 | Right Trigger | |
| 5:1 | Left Bumper | |
| 5:0 | Right Bumper | |




# Send Logged Values (1)

| Position | Name | Description |
| ---- | ---- | ---- |
| 14:7 -> 14:6 | Logged Value Number of Bytes  | n = 00:1, 01:2, 10:4, 11:12 |
| 14:5 -> 14:0 | Logged Value Index |  |
| 13 -> (14 - n) | Logged Value | |

Number of Bytes, Index and Value sets can be repeated to fill 13 bytes total



# Request/Send Logged Value Names (2)

| Position | Name | Description |
| ---- | ---- | ---- |
| 14:7 -> 14:6 | Logged Value Type  | 00:uint, 01:int, 10:float, 11:string |
| 14:5 -> 14:0 | Logged Value Index | |
| 13 -> 3 | Logged Value Name | Request for name if all 0's |

When the transmitter requests the name of a logged value, use type as 00, fill name with 0's.

Logged Value Names can't be all 0's.

# Bind Request/Accept (3)
If not in bind mode, do not respond to a bind request.

If no telemetry is requested, a bind accept is still required, but no normal responses should be given.

A bind request packet uses a UID of 0 for CRC purposes. Bind accepts should use the correct UID for CRC.

| Position | Name | Description |
| ---- | ---- | ---- |
| 14 | Protocol Version | As uint. Version described in this document is 1 |
| 13 -> 10 | Unique Id | As 4 byte unsigned int |
| 9:1 | Telemetry yes/no | Yes = 1, No = 0 |
| 9:0 | Accept/Request | Accept = 1, Request = 0 |


# Setting Info (4)
Request/provide info about settings

The transmitter should always use 00 for Setting Value Type and Setting Number of Bytes.

| Position | Name | Description |
| ---- | ---- | ---- |
| 14:7 -> 14:6 | Setting Number of Bytes  | n = 00:1, 01:2, 10:4, 11:11 |
| 14:5 -> 14:0 | Setting Index | 63 is reserved |
| 13:7 -> 13:6 | Setting Value Type | 00:enum, 01:int, 10:float, 11:string |
| 13:5 -> 13:0 | Descriptor | Number of Values if enum, 0 otherwise. 61, 62 and 63 are reserved.
| 12 -> (13 - n) | Setting Info |

### Requesting number of available settings. 
Setting Index = 63 and Descriptor = 63
Use 00 for Number of Bytes.

#### Number of available settings
Setting Index = 63 and Descriptor < 63
Descriptor is number of available settings (0 to 62).

### Request name of a specific Setting
If Setting Index != 63 and descriptor = 63 and Setting Info is 0's

### Name of a specific Setting (and enum count)
If Setting Index != 63 and descriptor < 61 giving info about a given setting

Setting Info is Name, all 11 possible bytes.

If type is an enum: Descriptor is number of enum values

### Current Value of a Specific Setting Request
If Setting Index != 63 and descriptor = 62 and Setting Info is 0's

### Current Value of a Specific Setting
If Setting Index != 63 and descriptor = 61

Setting Info is Current Value, sized based on Setting Number of Bytes.


# Setting Enum Name (5)
Request/provide the name of a specific enum value

| Position | Name | Description |
| ---- | ---- | ---- |
| 14:5 -> 14:0 | Setting Index | 63 is reserved |
| 13:5 -> 13:0 | Enum Index | 61, 62 and 63 are reserved |
| 12 -> 2 | Enum Name | All 0s = Request for Name |

# Apply Setting (6)

| Position | Name | Description |
| ---- | ---- | ---- |
| 14:7 -> 14:6 | Setting Number of Bytes  | n = 00:1, 01:2, 10:4, 11:11 |
| 14:5 -> 14:0 | Setting Index | 63 is reserved |
| 13 -> (14 - n) | Setting Value | |



Number of Bytes, Index and Value sets can be repeated to fill 13 bytes total.
