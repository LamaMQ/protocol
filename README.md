Welcome to the **`LamaMQ`** protocol !

**`LamaMQ`**  is a project introducing a new kind of message broker, designed for eased plugin creation.

- [Protocol](./PROTOCOL.md)
- [Protocol Roadmap](./ROADMAP.md)

# Basic presentation

- Like every broker, clients publish & consume messages.
- When you publish a message, you target a topic.
- To consume message, you need to, first, register to a topic (you can use wildcard to target multiple topics).
- Every message published matching your topic will be pushed on your client, to be consumed.

## Middleware
The main difference between **`LamaMQ`** and **`Kafka`** or **`RabbitMQ`** is is the concept of middlewares.
A client can publish or consume message, but it could also act as a middleware.

To act like a middleware, the client has to register one (or more) hook(s).
A hook can be apply onto any of:
- message publishing
- topic registration 
- or anything else happenning on the server.

Once a hook is registered, every matching event will be forwarded to the middleware.

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



