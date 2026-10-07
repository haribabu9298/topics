# Apache Kafka — Learn by Example

A beginner-friendly guide to Kafka concepts, taught through one running example: an
online food-ordering application. Source notes: [`kafka`](./kafka).

Every Kafka term is introduced only when the example needs it, so the vocabulary builds
up naturally instead of arriving as a list of definitions.

---

## Table of Contents

**Why Kafka Exists**
1. [Why Do We Need Kafka?](#1-why-do-we-need-kafka)
2. [Kafka as a Post Office](#2-kafka-as-a-post-office)

**Core Building Blocks**
3. [Producer](#3-producer)
4. [Consumer](#4-consumer)
5. [Topic](#5-topic)
6. [Partition](#6-partition)
7. [Why Do We Need Partitions?](#7-why-do-we-need-partitions)
8. [Offset](#8-offset)

**Consumers at Scale**
9. [Consumer Group](#9-consumer-group)
10. [The Important Consumer Group Rule](#10-the-important-consumer-group-rule)
11. [Why Multiple Consumer Groups?](#11-why-multiple-consumer-groups)
12. [Why Doesn't the Message Disappear?](#12-why-doesnt-the-message-disappear)
13. [Retention](#13-retention)

**Cluster and Durability**
14. [Broker](#14-broker)
15. [Kafka Cluster](#15-kafka-cluster)
16. [Partition + Broker](#16-partition--broker)
17. [Replication](#17-replication)
18. [Replication Factor](#18-replication-factor)
19. [Leader](#19-leader)
20. [Follower](#20-follower)
21. [ISR — In-Sync Replicas](#21-isr--in-sync-replicas)
22. [Rebalancing](#22-rebalancing)

**Ordering**
23. [Key](#23-key)
24. [Ordering](#24-ordering)

**Putting It Together**
25. [The Full Picture](#25-the-full-picture)
26. [The Entire Terminology in One Story](#26-the-entire-terminology-in-one-story)
27. [The 10 Terms to Remember First](#27-the-10-terms-to-remember-first)

**Cluster Coordination**
28. [What Did ZooKeeper Do?](#28-what-did-zookeeper-do)
29. [Why Did Kafka Need ZooKeeper?](#29-why-did-kafka-need-zookeeper)
30. [Modern Kafka Uses KRaft](#30-modern-kafka-uses-kraft)
31. [One Complete Example](#31-one-complete-example)

**Reference**
- [Glossary](#glossary)
- [Hands-On Next Steps](#hands-on-next-steps)

---

## 1. Why Do We Need Kafka?

Imagine you order a biryani from a food app.

```
You
 ↓
Food App
 ↓
Order Service
```

After the order is created, many things need to happen:

```
Order Created
    |
    +----> Payment Service
    |
    +----> Restaurant Service
    |
    +----> Delivery Service
    |
    +----> Notification Service
    |
    +----> Analytics Service
```

Without Kafka, the Order Service would directly call all of these services. That creates
**tight coupling** — every new downstream consumer means changing the Order Service.

With Kafka instead:

```
                Kafka
                  ↑
                  |
             Order Service
                  |
             "Order Created"
                  |
        +---------+---------+
        |         |         |
        ↓         ↓         ↓
     Payment  Restaurant  Delivery
                         |
                    Notification
```

The Order Service simply announces:

> "An order was created."

Kafka stores that event, and any interested application consumes it.

---

## 2. Kafka as a Post Office

This is probably the easiest analogy.

```
Person A
   |
   | sends letter
   ↓
Post Office
   |
   +----> Person B
   +----> Person C
   +----> Person D
```

Kafka works similarly:

```
Producer
   |
   | sends event
   ↓
 Kafka
   |
   +----> Consumer A
   +----> Consumer B
   +----> Consumer C
```

Now let's map every Kafka term onto the food-ordering example.

---

## 3. Producer

A **Producer** is the application that sends data to Kafka. Our Order Service is a
producer.

```
Order Service
      |
      | "Order #101 created"
      ↓
    Kafka
```

> **Producer = sends messages/events to Kafka**

Examples:

| Service | Role |
| --- | --- |
| Order Service | Producer |
| Payment Service | Producer |
| User Service | Producer |
| Inventory Service | Producer |

A service can be both a producer *and* a consumer.

---

## 4. Consumer

A **Consumer** reads messages from Kafka.

Suppose Order Service sends `Order #101 created`. Kafka stores it. Then:

```
Payment Service
      ↑
      |
    Kafka
```

The Payment Service reads the event.

> **Consumer = reads events from Kafka**

---

## 5. Topic

Imagine the post office has different sections:

```
Post Office
├── Letters
├── Packages
├── Registered Mail
└── International
```

Kafka has something similar, called a **Topic**. For our application:

```
orders
payments
users
delivery
notifications
```

The Order Service sends events into the `orders` topic:

```
orders topic
├── Order #101 created
├── Order #102 created
├── Order #103 created
└── Order #104 created
```

> **Topic = a named place/category where related events are stored** — a particular
> stream of messages.

---

## 6. Partition

This is the part beginners usually find confusing.

Suppose the `orders` topic becomes extremely busy — millions of orders are coming in.
Instead of one giant log:

```
orders
├── Order 1
├── Order 2
├── Order 3
├── ...
└── 1 billion orders
```

Kafka splits the topic into multiple **partitions**:

```
orders topic
   |
   +-------- Partition 0
   |
   +-------- Partition 1
   |
   +-------- Partition 2
```

Each partition is an ordered log of events:

| Partition 0 | Partition 1 | Partition 2 |
| --- | --- | --- |
| Order 1 | Order 2 | Order 3 |
| Order 4 | Order 5 | Order 6 |
| Order 7 | Order 8 | Order 9 |
| Order 10 | Order 11 | Order 12 |

> **Topic = logical stream**
> **Partition = the actual ordered pieces of that stream**

---

## 7. Why Do We Need Partitions?

Because of **parallelism**.

Imagine one cashier at a supermarket:

```
Customers
   |
   ↓
Cashier          ← very slow
```

Now:

```
Customers
   |
   +---- Cashier 1
   +---- Cashier 2
   +---- Cashier 3
   +---- Cashier 4     ← much faster
```

Kafka does the same thing:

```
orders topic

Partition 0 ───── Consumer
Partition 1 ───── Consumer
Partition 2 ───── Consumer
```

Multiple partitions allow work to happen in parallel.

---

## 8. Offset

Take Partition 0:

```
Partition 0
├── Order A
├── Order B
├── Order C
├── Order D
└── Order E
```

Kafka assigns a number to each event:

```
Partition 0
Offset 0 → Order A
Offset 1 → Order B
Offset 2 → Order C
Offset 3 → Order D
Offset 4 → Order E
```

That number is the **offset** — the position of a message inside a partition.

> **Important:** an offset is unique only *within* a partition.

```
Partition 0             Partition 1

Offset 0 → A            Offset 0 → X
Offset 1 → B            Offset 1 → Y
Offset 2 → C            Offset 2 → Z
```

Both partitions can have offset 0.

---

## 9. Consumer Group

Suppose you run three instances of your Payment Service:

```
Payment Service #1
Payment Service #2
Payment Service #3
```

They all need to process orders, so you put them into one **consumer group**:

```
Consumer Group: payment-service

         +------ Payment-1
         |
Kafka ---+------ Payment-2
         |
         +------ Payment-3
```

Kafka distributes partitions between them:

```
orders topic          Assignment
├── Partition 0  →    Payment-1
├── Partition 1  →    Payment-2
└── Partition 2  →    Payment-3
```

They work in parallel.

---

## 10. The Important Consumer Group Rule

Suppose you have **3 partitions** and **5 consumers**. You might expect:

```
P0 → C1
P1 → C2
P2 → C3
P? → C4
P? → C5
```

But there are only three partitions, so:

```
P0 → C1
P1 → C2
P2 → C3

C4 → idle
C5 → idle
```

> **Rule:** one partition can be assigned to only one consumer within a consumer group.
>
> Therefore: **maximum active consumers in a consumer group = number of partitions.**

---

## 11. Why Multiple Consumer Groups?

Suppose the same order needs to be processed by the Payment, Inventory and Notification
services. Create three consumer groups:

```
orders topic
     |
     +---- payment-group
     |
     +---- inventory-group
     |
     +---- notification-group
```

All three groups receive the same events:

```
Order #101
     |
     +------ Payment Service
     |
     +------ Inventory Service
     |
     +------ Notification Service
```

This is one of Kafka's most powerful concepts: **each consumer group gets its own
independent view of the topic.**

---

## 12. Why Doesn't the Message Disappear?

Imagine:

```
orders topic
Offset 0 → Order 101
Offset 1 → Order 102
Offset 2 → Order 103
```

The Payment Service reads orders 101–103. You might think Kafka deletes them. **It does
not.** Kafka keeps records according to its retention policy (for example, 7 days).

```
Kafka
 |
 +--- Order 101
 +--- Order 102
 +--- Order 103
       |
       | Payment Service reads
       |
       | Inventory Service reads
       |
       | Analytics reads
```

The same event can be read by different consumer groups.

---

## 13. Retention

**Retention = how long Kafka keeps data.**

```
retention.ms = 7 days
```

Means Kafka retains records for approximately that period before they become eligible
for deletion.

```
Topic: orders

Monday       Order 1
Tuesday      Order 2
Wednesday    Order 3
...
```

After the retention period, older records can be deleted.

> **Important:** Kafka retention is independent of whether a consumer has read the
> message.

---

## 14. Broker

Each Kafka server is called a **Broker**.

```
Kafka Cluster
├── Broker 1
├── Broker 2
└── Broker 3
```

> **Broker = Kafka server**

---

## 15. Kafka Cluster

A collection of brokers is a **Kafka cluster**.

```
Kafka Cluster
      |
      +---- Broker 1
      |
      +---- Broker 2
      |
      +---- Broker 3
```

Why multiple brokers?

- High availability
- Fault tolerance
- Scalability
- Distributed storage

---

## 16. Partition + Broker

Here's where things become interesting. Given the `orders` topic with P0, P1, P2, Kafka
distributes them across brokers:

```
Broker 1          Broker 2          Broker 3
   |                 |                 |
   +--- P0           +--- P1           +--- P2
```

Now the workload is distributed.

---

## 17. Replication

But what happens if Broker 1 dies and P0 only exists there?

```
Broker 1  ❌ crashes
   |
   +--- P0        →  P0 unavailable
```

Not good. So Kafka creates **replicas**:

```
Broker 1          Broker 2             Broker 3
   |                 |                    |
   +--- P0           +--- P0 replica      +--- P0 replica
```

This is **replication**.

---

## 18. Replication Factor

Suppose you configure:

```
replication.factor = 3
```

That means each partition has three replicas:

```
P0
Broker 1 → Leader
Broker 2 → Replica
Broker 3 → Replica
```

> **Replication Factor = number of copies of each partition**

---

## 19. Leader

For each partition, one replica acts as the **leader**.

```
Partition 0
Broker 1 → Leader
Broker 2 → Follower
Broker 3 → Follower
```

The leader handles requests for that partition; followers replicate the data.

If Broker 1 dies:

```
Broker 1 ❌

Broker 2 → becomes Leader
Broker 3 → Follower
```

Kafka can continue operating, assuming the cluster has enough healthy replicas.

---

## 20. Follower

A **follower** replica copies data from the leader.

```
Leader
   |
   | replicate
   ↓
Follower
   |
   | replicate
   ↓
Follower
```

> **Leader = primary replica**
> **Follower = replica that follows the leader**

---

## 21. ISR — In-Sync Replicas

You will often see the term **ISR**, which means **In-Sync Replicas**.

```
P0
Broker 1 → Leader
Broker 2 → Follower
Broker 3 → Follower
```

If everyone is caught up:

```
ISR = Broker 1, Broker 2, Broker 3
```

But suppose Broker 3 becomes slow:

```
Broker 1 → Leader   ✅
Broker 2 → Follower ✅
Broker 3 → Follower ❌ lagging
```

Then Kafka may have:

```
ISR = Broker 1, Broker 2
```

This is important for Kafka's durability and failover behaviour.

---

## 22. Rebalancing

Back to consumer groups. Given three partitions and three consumers:

```
P0 → C1
P1 → C2
P2 → C3
```

Now C2 crashes:

```
C1 ✅
C2 ❌
C3 ✅
```

Kafka redistributes the partitions — maybe:

```
P0 → C1
P1 → C3
P2 → C3
```

This redistribution is called a **rebalance**. The same thing happens when a consumer
*joins* (e.g. `C4 joins`) — Kafka redistributes partitions again.

---

## 23. Key

Suppose you have:

```
Order #100 → customer A
Order #101 → customer B
Order #102 → customer A
```

Kafka records can have a **key**:

```
Key   = customer-A
Value = Order #100
```

Kafka uses the key to determine which partition receives the record:

```
customer-A
     |
     ↓
partition calculation
     |
     ↓
Partition 1
```

If another event has the same key, it lands on the same partition:

```
customer-A
     |
     ↓
Partition 1
```

This is useful when you need **ordering for a particular entity**.

---

## 24. Ordering

Within a partition, order is preserved:

```
Partition 0
Offset 0 → OrderCreated
Offset 1 → PaymentCompleted
Offset 2 → OrderShipped
```

But across partitions:

```
Partition 0:        Partition 1:
Order A             Order B
Order C             Order D
```

Kafka does **not** give you one global ordering between P0 and P1.

> **Remember:** ordering is guaranteed *within a partition*, not across the entire topic.

---

## 25. The Full Picture

Now let's build the food-ordering example completely. You order biryani:

```
                    YOU
                     |
                     ↓
                Food App
                     |
                     ↓
               Order Service
                  Producer
                     |
                     | OrderCreated
                     ↓
              Kafka Cluster
                     |
             Topic: orders
                     |
          +----------+----------+
          |          |          |
         P0         P1         P2
          |          |          |
          +----------+----------+
                     |
        +------------+-------------+
        |            |             |
        ↓            ↓             ↓
   Payment Group  Inventory     Notification
                  Group           Group
```

Kafka itself might look like:

```
Kafka Cluster

Broker 1              Broker 2              Broker 3
  |                     |                     |
  +--- P0 Leader        +--- P1 Leader        +--- P2 Leader
  +--- P1 Replica       +--- P2 Replica       +--- P0 Replica
```

Now you can see how all the terminology connects.

---

## 26. The Entire Terminology in One Story

**"The Order Service produces an event to the orders topic."**

```
Order Service
     |
     | PRODUCER
     ↓
orders TOPIC
```

**"The topic has three partitions."**

```
orders
 ├── P0
 ├── P1
 └── P2
```

**"The consumer group has three consumers."**

```
Consumer Group
 ├── C1
 ├── C2
 └── C3
```

Kafka might assign:

```
P0 → C1
P1 → C2
P2 → C3
```

**"The consumer committed offset 500."**

The consumer has recorded its processing position around offset 500 for that partition.

**"Replication factor is 3."**

```
P0 → Broker 1
      Broker 2
      Broker 3
```

There are three replicas.

**"Broker 2 is the leader for P0."**

```
P0
Broker 2 → Leader
Broker 1 → Replica
Broker 3 → Replica
```

**"The consumer group rebalanced."**

Kafka redistributed partitions among the consumers because the group membership or
assignment changed.

---

## 27. The 10 Terms to Remember First

Don't try to memorise 30 Kafka terms today. Start with these:

```
1. Kafka
      ↓
2. Broker
      ↓
3. Topic
      ↓
4. Partition
      ↓
5. Offset
      ↓
6. Producer
      ↓
7. Consumer
      ↓
8. Consumer Group
      ↓
9. Replication
      ↓
10. Leader / Follower
```

And remember this one picture:

```
                     KAFKA CLUSTER
                          |
            +-------------+-------------+
            |             |             |
         Broker 1      Broker 2      Broker 3
            |             |             |
            +-------------+-------------+
                          |
                     orders TOPIC
                          |
             +------------+------------+
             |            |            |
            P0           P1           P2
             |            |            |
          offsets      offsets      offsets
             |            |            |
             +------------+------------+
                          |
                    CONSUMER GROUP
                    /      |      \
                   C1      C2      C3
```

If you understand this diagram, you already understand the core architecture of Kafka.

---

## 28. What Did ZooKeeper Do?

Older Kafka architectures looked like:

```
                  ZooKeeper
                      |
          manages Kafka metadata
                      |
       +--------------+--------------+
       |              |              |
    Broker 1       Broker 2       Broker 3
       |
       +---- telemetry topic
       +---- logs topic
       +---- events topic
```

ZooKeeper helped Kafka coordinate/manage things such as:

- Broker membership
- Controller election
- Cluster metadata
- Partition leadership information
- Configuration / coordination

> **Kafka** = stores and moves your events
> **ZooKeeper** = historically helped Kafka coordinate/manage the cluster

### Simple analogy

Think about a cricket stadium.

```
Players
   |
   | play the game
   ↓
Cricket Match
```

The players are like Kafka brokers handling data. Then there is an organiser/referee
coordinating things:

```
Organizer
   |
   +---- who is playing
   +---- who is captain
   +---- coordination
```

Historically, ZooKeeper played some of that coordination role for Kafka. It was **not**
where the actual "game data" was stored.

---

## 29. Why Did Kafka Need ZooKeeper?

Kafka is distributed. With three brokers, Kafka needs to know things like:

> "Which broker is currently the leader for Partition 0?"

```
Partition 0
Broker 1 → Leader
Broker 2 → Replica
Broker 3 → Replica
```

If Broker 1 dies:

```
Broker 1 ❌

Partition 0
Broker 2 → new Leader
Broker 3 → Replica
```

Historically, ZooKeeper participated in this cluster coordination.

### The old architecture

You will often see this diagram in older Kafka tutorials:

```
              ZooKeeper
                  |
        +---------+---------+
        |         |         |
        ↓         ↓         ↓
     Kafka     Kafka     Kafka
    Broker 1  Broker 2  Broker 3
        |
        +---- Topics
        |
        +---- Partitions
        |
        +---- Telemetry
```

The important distinction:

```
ZooKeeper
    |
    | cluster coordination
    ↓
Kafka brokers
    |
    | actual events
    ↓
Telemetry / logs / business events
```

---

## 30. Modern Kafka Uses KRaft

This is very important if you are learning Kafka now. Modern Kafka uses **KRaft**
(**Kafka Raft**) instead of relying on ZooKeeper.

| | Old Kafka | Modern Kafka |
| --- | --- | --- |
| Data path | Producer → Kafka Brokers | Producer → Kafka Brokers |
| Coordination | ↕ ZooKeeper | ↕ KRaft Controllers |

### What is KRaft?

KRaft allows Kafka to manage its own metadata and coordination using Kafka's own
controller quorum, based on the Raft consensus protocol.

```
             KRaft Controller Quorum

             Controller 1
                  |
        +---------+---------+
        |                   |
 Controller 2          Controller 3

                  |
                  ↓

          Kafka Broker Cluster
       +----------+----------+
       |          |          |
    Broker 1   Broker 2   Broker 3
```

So Kafka no longer needs ZooKeeper for cluster metadata management in a KRaft
deployment.

### The easiest way to remember it

Think about three different layers:

```
Your application data        Telemetry, Logs, Orders, Payments, Events
            ⬇️
Kafka                        Topics, Partitions, Offsets, Replication, Consumers
            ⬇️
Cluster management           Historically: ZooKeeper   →   Modern: KRaft
```

---

## 31. One Complete Example

Suppose Blue Origin has an engine monitoring system.

```
Engine
 |
 | temperature
 | pressure
 | vibration
 ↓
Telemetry Producer
 |
 ↓
Kafka
 |
 +--- engine-telemetry topic
       |
       +--- Partition 0
       +--- Partition 1
       +--- Partition 2
 |
 +----------------------+
 |                      |
 ↓                      ↓
Anomaly Detection    Data Analytics
```

In old Kafka:

```
                  ZooKeeper
                      |
                      | cluster coordination
                      ↓
Telemetry → Kafka Brokers → Consumers
```

In modern Kafka:

```
             KRaft Controllers
                    |
                    | cluster metadata/coordination
                    ↓
Telemetry → Kafka Brokers → Consumers
```

> The telemetry itself never needed to go through ZooKeeper.

---

## Glossary

| Term | Meaning |
| --- | --- |
| **Kafka** | Distributed event streaming platform that stores and moves events |
| **Broker** | A single Kafka server |
| **Cluster** | A collection of brokers |
| **Topic** | A named category/stream where related events are stored |
| **Partition** | An ordered log; a topic is split into one or more partitions |
| **Offset** | Position of a message within a partition (unique per partition) |
| **Producer** | Application that writes events to Kafka |
| **Consumer** | Application that reads events from Kafka |
| **Consumer Group** | Set of consumers that share partitions of a topic |
| **Rebalance** | Redistribution of partitions when group membership changes |
| **Key** | Field used to decide which partition a record goes to |
| **Replication Factor** | Number of copies of each partition |
| **Leader** | Replica that handles reads/writes for a partition |
| **Follower** | Replica that copies data from the leader |
| **ISR** | In-Sync Replicas — replicas caught up with the leader |
| **Retention** | How long Kafka keeps records before they can be deleted |
| **ZooKeeper** | Legacy external coordination service for Kafka metadata |
| **KRaft** | Kafka's built-in Raft-based metadata/coordination mode |

---

## Hands-On Next Steps

The fastest way to make these terms stick is to run Kafka locally with Docker or
Kubernetes and work through this sequence:

1. Create a topic
2. Create 3 partitions
3. Produce messages
4. Consume them
5. Inspect offsets
6. Create a consumer group
7. Scale consumers
8. Kill a consumer and watch a rebalance
9. Kill a broker and observe leader failover

That will teach you far more than reading definitions.
