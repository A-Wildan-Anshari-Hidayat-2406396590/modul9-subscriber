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

### Simulating Multiple Subscribers

![Multiple Subscribers Performance](assets/rabbitmq-drop.png)

**Why is the spike reduced much quicker than before?**
When we run multiple subscribers (e.g., 3 subscriber terminals) at the same time, they all connect to the exact same `user_created` queue. RabbitMQ automatically distributes the incoming messages evenly among all active subscribers using the **Competing Consumers pattern**. Because the workload is split three ways, the 3 subscribers process the messages in parallel, clearing the backlog exactly three times faster than a single slow subscriber could on its own.

**What could be improved in the publisher and subscriber code?**
Looking at the codebase, there are a few significant improvements that should be made:
1. **CPU Usage (Subscriber):** The subscriber ends with an empty infinite loop (`loop {}`) just to keep the program alive. This forces the CPU core to spin at 100% usage needlessly. A better approach would be to use a proper thread blocking mechanism (like an empty `mpsc::channel` receiver or `std::thread::park()`) to put the main thread to sleep while the background listener handles messages.
2. **Hardcoded Credentials:** Both the publisher and subscriber hardcode the AMQP connection string (`amqp://guest:guest@localhost:5672`). This is a bad security practice. The connection string and credentials should be extracted into environment variables using a crate like `dotenv`.
3. **Asynchronous Publishing (Publisher):** The publisher sends messages synchronously. If the network to RabbitMQ is slow, the whole program blocks. Using asynchronous publishing via `tokio` would allow it to fire off events much faster.