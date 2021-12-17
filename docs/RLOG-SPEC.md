# Byte Format for Robot Logs (.rlog)

## Log Revisions

The first byte represents the log format revision. The decoding device should always check whether it supports the specified revision before continuing. Below is a list of possible revisions:

* 0x00 = Invalid file. This is the first byte of logs produced before a revision number was included.
* 0x01 = Current revision. The log follows the specification described below.

## Message Types

The next byte represents the which of 3 message types are being used:

* 0x00 = Timestamp (start of a new cycle)
* 0x01 = Key (defines string value for a key ID)
* 0x02 = Field (value of a single field)

## Timestamp

Following the byte for message type, a timestamp message consists of a single double (8 bytes) representing the timestamp in seconds. This message marks the start of a new cycle and the timestamp should be associated with subsequent fields.

## Key

Each human readable string key (e.g. "/DriveTrain/LeftPositionRadians") is represented by a key ID to reduce the space required to encode the data from each cycle. Key IDs are shorts (2 bytes) which count up from 0. Each key message contains the following information:

1. Key ID (short, 2 bytes)
2. Length of string key (short, 2 bytes)
3. String key (UTF-8 encoded)

## Field

Field messages represent a change to a single value. If a key is not provided for a given cycle, the robot code should set its type to "null". A field's type can change over the course of a log, though each cycle can only contain a single value for each key (e.g. a field cannot have both an integer and boolean value at the same time). The structure of these messages begins with the following information:

1. Key ID (short, 2 bytes)
2. Value type (1 byte)

The possible value types are listed below along with the format of the value. Null (0x00) does not include more information.

### Boolean (0x01)

3. Field value (0x00 or 0x01, 1 byte)

### BooleanArray (0x02)

3. Length of array (short, 2 bytes)
4. Contents of array (series of 0x00 or 0x01, 1 byte each)

### Integer (0x03)

3. Field value (integer, 4 bytes)

### IntegerArray (0x04)

3. Length of array (short, 2 bytes)
4. Contents of array (integers, 4 bytes each)

### Double (0x05)

3. Field value (double, 8 bytes)

### DoubleArray (0x06)

3. Length of array (short, 2 bytes)
4. Contents of array (doubles, 8 bytes each)

### String (0x07)

3. Length of string (short, 2 bytes)
4. Field value (UTF-8 encoded)

### StringArray (0x08)

3. Length of array (short, 2 bytes)
4. Contents of array (same format as single string)

### Byte (0x09)

3. Field value (1 byte)

### ByteArray(0x0A)

3. Length of array (short, 2 bytes)
4. Contents of array (series of bytes)

## Networking

When sending log data over a network, the same format is used. For efficiency, the server device can encode the same data for every client after the first cycle. When a new client connects, the server should send:

1. The log format revision (described above).
2. Definitions of each pre-existing key ID.
3. Every value from the most recent cycle, regardless of whether any changes occurred.

This information allows the new device to "catch up" and decode the log the same as any older client.