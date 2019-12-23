# Introduction
LamaMQ protocol V.1
- [Types](#types)
- [Errors](#errors)
- [Handshake](#handshake)
- [Messages](#messages)

# Types
All types related in the message protocol are explained here ;

## Basic

Name |  Description
-----|---------------
BOOLEAN | Signed 8bits integer (0 => False / 1 => TRUE)
UINT8 | Unsigned 8 bits
UINT16 | Unsigned 16 bits
UINT32 | Unsigned 32 bits
INT32 | Signed 32 bits integer
ARRAY | UINT16 defining the size of the array (array copy the following type)
STRING | UINT32 defining the size of the string, following string.
BUFFER | UINT32 defining the size of the buffer (in byte) following the buffer.

## Structures

Name | Description
-------|--------------
[Message header](#structure-message-header) | Message header is used to communicate between a client a server.
[Message](#structure-message) | A message is the basic data transmitted on the server.

### Structure - Message header
Message header is used to communicate between a client a server.
```
Message header{
  Message id => INT32
  Message type => UINT8
}
```
Field | Description
------|-------------
Message id | Contain an id, the id need to be uniq at a given time, only between the client and the server. The server could use same id at a given to communicate with another client. And in one way, the server and the client can use both the same id at a given time (for standard message). The message id is used to match the answer with the request.
Message type | A number to know which [message](#messages) the client or the server have received.

### Structure - Message
A message is the basic data transmited on the server.
```
Message{
  created at => UINT32
  routing key => STRING
  headers => ARRAY (header)
  header{
    key => STRING
    value => STRING
  }
  payload type => UINT8
    - [payload type = 0] => payload_empty
    - [payload type = 1] => payload_string
    - [payload type = 2] => payload_buffer

  payload_empty{}

  payload_string{
    payload => STRING
  }

  payload_buffer{
    payload => BUFFER
  }
}
```
Field | Description
------|-------------
created at | A timestamp set by the message emitter to track when the message was created
routing key | The routing key used to know where the message need to be forwarded
headers | You can set multiple headers, headers could be processed by the message receiver or a middleware.
header.key | The key of the header (like a variable name).
header.value | The value of the header (like a variable value).
payload type | Define the content of the message.
payload_empty | If payload type=0, the message contains nothing more.
payload_string | If payload type=1, the message contains a string. Could be use to share data in JSON, YAML, XML...
payload_buffer | If payload type=2, The message contains a buffer.

# Errors
Error code help create easy exception matching the case of the errors.

Error code  | Error Name | Description 
-------------|--------------|-------------------------
 1   | Server Exception | Unknown error from the server.
 2   | Bad request | The request make do not follow expected payload.
 3 | Middleware timeout | The middleware timeout during the request processing.
 4 | Middleware error | The middleware response with an error.

# Handshake
When a client connect to a Lama node. It need to init a handshake, the sequence of messages does not evolve.

## Sequence
- Client => Server : Authentication
- Server => Client : Supported protocol version
- Client => Server : Choose a protocol version to use

## Authentication

Client call the server, for authentication.
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
max_version_major_supported | Max version supported (major.minor.patch) of the protocol
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
0 | [Basic ACK](#basic-ack) | Standard answer for many message
1 | [Push message](#push-message) | Push a message on a given server
2 | [Forward message](#forward-message) | Transmit a message from the server to a client or another server
32 | [Topic subscribe](#topic-subscribe) | Register a client to a specific topic
33 | [Topic subscription ACK](#topic-subscription-ack) | ACK to a topic subscription
34 | [Topic unsubscribe](#topic-unsubscribe) | Unregister to a topic
64 | [Middleware register](#middleware-register) | Register a new middleware
65 | [Middleware registration ack](#middleware-registration-ack) | ACK to a middleware subscription
66 | [Middleware unregister](#middleware-unregister) | Remove a middleware

## Basic ACK

```
message header => message header (with message type = 0)
error code => INT8
error message => STRING
```
Field | Description
------|-------------
[message header](#structure-message-header) | The message header with, message type = 0
error code | If error code = 0, no error, the message was processed successfuly. [Error code will match the errors listed here.](#errors)
error message | Optional message if error code > 0

## Push message

Push a message on a server
```
message header => message header (with message type = 1)
mesage => message
```
Field | Description
------|-------------
[message header](#structure-message-header) | The message header with, message type = 1
[message](#structure-message) | The message content

Server will answer with [Basic ack](#basic-ack) when message will be processed.

## Forward message
Forward a message on a client or a server
```
message header => message header (with message type = 2)
mesage => message

```
Field | Description
------|-------------
[message header](#structure-message-header) | The message header with, message type = 2
[message](#structure-message) | The message content

Client will answer with [Basic ack](#basic-ack) when message will be processed.
## Topic subscribe
Subscribe to a topic. All message mathing the topic will be forwarded on the client
```
message header => message header (with message type = 32)
routing key => STRING
```
Field | Description
------|-------------
[message header](#structure-message-header) | The message header with, message type = 32
routing key | The routing key used to know when message need to be routed to this subscription

The server response with [a topic subscribtion ACK](#topic-subscription-ack).

## Topic subscription ack
```
message header => message header (with message type = 33)
subscription id => INT32
```
Field | Description
------|-------------
[message header](#structure-message-header) | The message header with, message type = 33
subscription id | A unique id, use to refer this subscription (used during message forwarding).

## Topic unsubscribe
Unsubscribe to a topic.
```
message header => message header (with message type = 34)

```
Field | Description
------|-------------
[message header](#structure-message-header) | The message header with, message type = 34
subscription id | The unique id used to refer this subscription.

## Middleware register
Call for register a new hook on the server.
```
message header => message header (with message type = 64)
hook name => STRING
hook priority => INT32
hook timeout => UINT32
hook strategy => UINT8
hook type => UINT8
  - [hook type = 0] => hook_client_connect
  - [hook type = 1] => hook_client_disconnect
  - [hook type = 16] => hook_message_pushed
  - [hook type = 32] => hook_topic_subscribed
  - [hook type = 33] => hook_topic_unsubscribed
  - [hook type = 64] => hook_middleware_registred
  - [hook type = 65] => hook_middleware_unregistred
```
Field | Description
------|-------------
[message header](#structure-message-header) | The message header with, message type = 64.
hook name | The name of the hook (if middleware already register with the same name, does not create hook again).
hook priority | Order of execution of the hook on the server, if multiple hook at the same point.
hook strategy | Hook strategy. See [Hook strategy](#hook-strategy).
hook type | Different type of hook are available. See [Hook type](#hook-types)

The server will answer with [a middleware registration ack](#middleware-registration-ack) depending on the hook type provided.

### Hook strategy
Middleware register hook with multiple different behavior.
- If we want no ACK (hook strategy = 0), the server does not wait for any ACK from the middleware, the timeout is not used.
- If we want an ACK(hook strategy = 1), the server wait for an ACK from the middleware, without ACK after the timeout, an error will be raised to stop the process.
- If we want an ACK, but it's not a big deal, if we don't get one (hook strategy = 2), the server wait for an ACK from the middleware, without ACK after the timeout, no error will be raised and the middleware will be ignored.

### Hook types
#### Hook - Client connect
Hook triggered when a client connect to the server.
```
// Middleware registration
hook_client_connect{
  on socket => BOOLEAN
  on authentication => BOOLEAN
  on version choose => BOOLEAN
}
```
Field | Description
------|-------------
[Middleware registration](#middleware-register) | Message payload of a middleware registration with hook type = 0
on socket | Trigger the hook when the connection is etablished.
on Authentication | Trigger the hook when the client auth.
on version choose | Trigger when the client choose the version of the protocol to use.

At least one field need to be at true.

#### Hook - Client disconnect
Hook when a client disconnect from the server.
```
// Middleware registration
hook_client_disconnect{}
```
Field | Description
------|-------------
[Middleware registration](#middleware-register) | Message payload of a middleware registration with hook type = 1

No payload required.

#### Hook - Message pushed
Register a hook on message pushed. Every message pushed could be intercepted or just some of them, depending of the routing condition.

```
// Middleware registration
hook_message_pushed{
  routing conditions => ARRAY<routing condition>
    routing condition{
      UINT8 routing type
      - [routing type = 0] => routing_topic_match
      - [routing type = 4] => routing_header_match
      - [routing type = 5] => routing_header_exists
      
      routing_topic_match{
        routing topic => STRING
      }
      
      routing_header_match{
        header => STRING
        value => STRING
      }
      
      routing_header_exists{
        header => STRING
      }
    }
}
```
Field | Description
------|-------------
[Middleware registration](#middleware-register) | Message payload of a middleware registration with hook type = 16
routing conditions| Array of reason to push the message on the middleware (act like an AND between conditions).
routing type | Define the type of conditions
routing topic  | A routing type to listen to, with wildcare authorized (like a topic subscription).
header | Name of a message header
value | Value of a message header

#### Hook - Topic subscribed
```
// Middleware registration
hook_topic_subscribed{}
```
Field | Description
------|-------------
[Middleware registration](#middleware-register) | Message payload of a middleware registration with hook type = 32

#### Hook - Topic unsubscribed
```
// Middleware registration
hook_topic_unsubscribed{}
```
Field | Description
------|-------------
[Middleware registration](#middleware-register) | Message payload of a middleware registration with hook type = 33

#### Hook - Middleware registred
```
// Middleware registration
hook_middleware_registred{}
```
Field | Description
------|-------------
[Middleware registration](#middleware-register) | Message payload of a middleware registration with hook type = 64

#### Hook - Middleware unregistred
```
// Middleware registration
hook_middleware_unregistred{}
```
Field | Description
------|-------------
[Middleware registration](#middleware-register) | Message payload of a middleware registration with hook type = 65

## Middleware registration ack
```
message header => message header (with message type = 65)

```
Field | Description
------|-------------
[message header](#structure-message-header) | The message header with, message type = 65

## Middleware unregister
```
message header => message header (with message type = 66)

```
Field | Description
------|-------------
[message header](#structure-message-header) | The message header with, message type = 66
