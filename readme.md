### a. What is *amqp*?

**AMQP** (Advanced Message Queuing Protocol) is an open standard application layer protocol for message-oriented middleware. It sets rules for how messages are formatted, sent, and received between systems. AMQP allows different applications, which may be written in various programming languages and run on different platforms, to communicate reliably through a message broker like RabbitMQ. It includes features such as message queuing, routing (both point-to-point and publish-subscribe), reliability, and security.

### b. What does `guest:guest@localhost:5672` mean?

In the connection string `amqp://guest:guest@localhost:5672`:

- **`guest` (first)** — This is the **username** used to log in to the RabbitMQ message broker. `guest` is the default username set up with a new RabbitMQ installation.
- **`guest` (second)** — This is the **password** for the username above. Just like the username, `guest` is the default password provided by RabbitMQ.
- **`localhost:5672`** — `localhost` refers to the local machine (the RabbitMQ server runs on the same machine as the application), and **`5672`** is the default port number where RabbitMQ listens for AMQP connections.