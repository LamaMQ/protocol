Welcome to RabbitMQ protocol !

RabbitMQ is a project of a new kind of message broker, designed for create plugin easily.

- [Protocol](./PROTOCOL.md)
- [Protocol Roadmap](./ROADMAP.md)

# Basic presentation
Like every broker, client publish & consume message.

When you publish a message, you target a topic.

To consume message, you need to first, register to a topic, you can use wildcard to target multiple topic.

Every message published matching your topic, will be pushed on your client, to be consumed.

## Middleware
The main difference between LamaMQ and Kafka or RabbitMQ, is the middleware concept.
A client can publish or consume message, but it could act as middleware too.
To act like a middleware, the client could register hook.
A hook could target message publishing, or topic registration or anything which happen,
on the server. 
When a hook is register every event matching the hook will be forwarded on the middleware.

```
// Classic broker ;
Client => Server (publish)
Client <= Server (ack publish)

// With middleware approach ;
Client =>  Server  | Middleware (publish)
Client  |  Server => Middleware (hook on publish)
Client  |  Server <= Middleware (ack hook)
Client <=  Server  | Middleware (ack publish)
```



