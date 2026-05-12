### a. What is *amqp*?

**AMQP** (Advanced Message Queuing Protocol) is an open standard application layer protocol for message-oriented middleware. It sets rules for how messages are formatted, sent, and received between systems. AMQP allows different applications, which may be written in various programming languages and run on different platforms, to communicate reliably through a message broker like RabbitMQ. It includes features such as message queuing, routing (both point-to-point and publish-subscribe), reliability, and security.

### b. What does `guest:guest@localhost:5672` mean?

In the connection string `amqp://guest:guest@localhost:5672`:

- **`guest` (first)** — This is the **username** used to log in to the RabbitMQ message broker. `guest` is the default username set up with a new RabbitMQ installation.
- **`guest` (second)** — This is the **password** for the username above. Just like the username, `guest` is the default password provided by RabbitMQ.
- **`localhost:5672`** — `localhost` refers to the local machine (the RabbitMQ server runs on the same machine as the application), and **`5672`** is the default port number where RabbitMQ listens for AMQP connections.

### Simulating a Slow Subscriber

![Slow Subscriber Performance](assets/rabbitmq-performance2.png)

**Why does the queue stay at 0 on my machine?**
In the tutorial, the queued messages spike to 20. However, on my machine, the total number of queued messages in the first chart stays completely flat at 0. This happens because the newer version of the `crosstown_bus` AMQP library uses **Aggressive Prefetching**. When the slow subscriber connects, it instantly pulls all 20 messages from the RabbitMQ queue into its own internal RAM buffer and automatically acknowledges them. As a result, RabbitMQ considers the queue empty (0 messages) immediately, while the subscriber is secretly holding those 20 messages in memory and printing them out slowly (1 message per second). Even though the first chart shows 0 queued, the second chart (Message Rates) successfully proves that the messages were published in a huge spike and are being slowly delivered/acknowledged over time.