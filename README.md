# Introduction
LamaMQ protocol V.1
- [Types](#types)
- [Errors](#errors)
- [Handshake](#handshake)
- [Messages](#messages)

# Types
All types related in the message protocol are explained here ;

Name |  Description
-----|---------------
UINT8 | Unsigned 8 bits
UINT16 | Unsigned 16 bits
UINT32 | Unsigned 32 bits
ARRAY | UINT16 defining the size of the array (array copy the following type)
STRING | UINT32 defining the size of the string, following string.
BUFFER | UINT32 defining the size of the buffer (in byte) following the buffer.

# Errors
Error code help create easy exception matching the case of the errors.

Error code  | Error Name | Description 
-------------|--------------|-------------------------
 1   | Server Exception | Unknown error from the server 
 2   | Bad request | The request make do not follow expected payload 

# Handshake
When a client connect to a Lama node. It need to init a handshake, the sequence of messages does not evolve.

## Sequence
- Client => Server : Authentification
- Server => Client : Supported protocol version
- Client => Server : Choose a protocol version to use

## Authentification

Client call the server, for authentification.
```
auth_method => UINT8
identity_name => STRING
identity_key => [BUFFER]
```

Field | Description
------|--------------
auth_method | Version of the auth method (always 0 currently).
identity_name | The name of the client
identity_key | Optional buffer containing a key for identifily uniquely a client

## Supported protocol version
Server answer (if success) with the protocol version supported

```
max_version_major_supported=UINT8
max_version_minor_supported=UNIT8
min_version_major_supported=UINT8
min_version_minor_supported=UINT8
```

Field | Description
-------|-----------
max_version_major_supported | Max version supported (major.minor.patch) of the protocol
max_version_minor_suppored | Max version supported (major.minor.patch) of the protocol
min_version_major_supported | Min version supported (major.minor.patch) of the protocol
min_version_minor_supported | Min version supported (major.minor.patch) of the protocol

## Choose the protocol version to use
Client answer
```
version_used_major=UINT8
version_used_minor=UINT8
```


# Messages

 Message Code | Message Name | Description 
-------------|---------------|--------------
1 | [Push message](#push-message) | Push a message on a given server or client
2 | 

## Push message

Push a message on a server
```
message id => UINT32
message type => UINT8
created at => UINT32
routing key => STRING
headers => ARRAY (header)
header{
  key => STRING
  value => STRING
}
payload type => UINT8
  payload_empty (payload_type = 0)
  payload_string (payload_type = 1)
  payload_buffer (payload_type = 2)

payload_empty{}

payload_string{
  payload => STRING
}

payload_buffer{
  payload => BUFFER
}

```

Server answer with
```
message id => UINT32
error => UINT8 (if error=0 ; no error, message ACK)
[error_message => STRING] (only error > 0)
```
