# RabbitMQ DIRECT Exchange Node.js Lab

## Prepare project

### Install dependencies

```bash
yarn install
```

### Run RabbitMQ container (docker is required)

Rabbit on 5672 port and management on 8081.

Management user and password are "**guest**" without quotes.

```bash
docker run -d -v ${PWD}/rabbit-db:/var/lib/rabbitmq --hostname yt-rabbit -p 5672:5672 -p 8081:15672 --name yt-rabbit rabbitmq:3-management
```

## Same Exchange and Routing Key, different queue

All subscribers receive all messages.

### Subscribers (Run commands in different terminals)

```bash
PATTERN=A QUEUE=first EXCHANGE=my-direct node subscriber/direct-exchang
e.js

PATTERN=A QUEUE=second EXCHANGE=my-direct node subscriber/direct-exchang
e.js
```

### Publishers

```bash
ROUTING_KEY=A EXCHANGE=my-direct node publisher/topic-exchange.js
```

## Same Exchange and Routing Key and queue

All subscribers receive messages using round-robin.

### Subscribers (Run commands in different terminals)

```bash
PATTERN=A QUEUE=first EXCHANGE=my-direct node subscriber/topic-exchange.js

PATTERN=A QUEUE=first EXCHANGE=my-direct node subscriber/topic-exchange.js
```

### Publishers

```bash
ROUTING_KEY=A EXCHANGE=my-direct node publisher/topic-exchange.js
```

## Different Exchange, same Routing Key and queue

All subscribers receive messages using round-robin.

**This happens because queue is bound to both exchanges.

### Subscribers (Run commands in different terminals)

```bash
PATTERN=A QUEUE=second EXCHANGE=my-direct-2 node subscriber/topic-exchange.js

PATTERN=A QUEUE=second EXCHANGE=my-direct node subscriber/topic-exchange.js
```

### Publishers

```bash
ROUTING_KEY=A EXCHANGE=my-direct node publisher/topic-exchange.js
```

## Keep messages after RabbitMQ restart
### Durability and persistence
It is necessary to set exchange and queue as **durable** and messages
as **persistent** when they are published.
#### Add exchange as "durable"

```js
channel.assertExchange(exchangeName, exchangeType, {
    durable: true
})
```

#### Add queue as "durable"

```js
await channel.assertQueue(queue, {
    durable: true
})
```

#### Add publish "persistent" option as true (default is false)

```js
const sent = channel.publish(
    exchangeName,
    routingKey,
    Buffer.from(JSON.stringify(message)),
    {
        persistent: true
    }
)
```
