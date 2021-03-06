# AMQP 0.9.1 sample

The sample uses RabbitMQ implementation of AMQP 0.9.1 through the EasyNetQ library.

## Navigating the solution

The solution consists of these projects:

* `DeferralService` A custom service for scheduling future message deliveries. A `FuturePublish` feature exists in RabbitMQ, but that is vendor-specific extension to the AMQP definition
* `EventProducer` A sample event producer (in the Galactic Empire Bounded Context), publishing an `Events.Greeting` message every second
* `EventConsumer` A samle event consumer (in the Rebel Alliance Bounded Context), listening for any `Events.Greeting` messages from the Galactic Empire BC
* `Messaging.Support` Extensions and wrapping on top of the EasyNetQ library, so the concrete implementation doesn't leak into the code using it

## Bounded Contexts

The solution consists of two bounded contexts: 
* The producing Bounded Context `Galactic Empire BC`
* The consuming Bounded Context `Rebel Alliance BC`

## Published Langugage

`Galactic Empire BC`publishes the event `Events.Greeting` with this content: 

```JSON
{
  "sender": "",
  "gratulation": ""
}
```
All messages are wrapped in a platform `Envelope` looking like this:

```JSON
{
  "messageId":"",
  "correlationId":"",
  "sourceBC":"",
  "sequence": 0,
  "payload": {}
}
```

The `payload` field contains the published language entity. The typename of the message is extrapolated from the naming convention below. 

This envelope is used in the platform infrastructure for routing and metadata exchange

## Topics (Exchanges)

All messages are published to topics following this naming convention:

`Ecosystem.<Source Bounded Context>.<Published Language Entity>`

So e.g. the message entity `Events.Greeting` from the `GalacticEmpireBC` will be published to topic `Ecosystem.GalacticEmpireBC.Events.Greeting`.

Interested consumers should declare a private queue in the format `<Source Bounded Context>.<Published Language Entity>:<Bounded Context>.<SubscriberId>`.

A consumer `RebelAllianceBC` listening for the `Events.Greeting` message from `GalacticEmpireBC` will therefore declare a private queue `GalacticEmpireBC.Events.Greeting:RebelAlliance.EventConsumerService1-myhost1`

The sample supports a competing consumer pattern by sharing the consumer queue (same name) among multiple consumers.

## Message schema validation

All messages are described in a JSON schema and placed in the `/schemas/bounded-context/` folder. This enables us to validate message integrity when sending and when receiving a message, if needed.

The message JSON schema references the `/schemas/platform/envelope.json` schema for metadata fields and applies individual constraints to the `payload` field only.

The sample currently validates JSON schema on consumption and drops messages if they are not compliant with the defined schema annotated with the `JsonSchemaAttribute`. Dropped messages will go to the dead letter queue.

This might not be exactly the way you'd want to do schema validation in your solution, as it is quite strictly implemented in this library. If, how and when to validate schemas should be up to the individual consumer, or it should be a feature embedded in the AMQP broker instead through a schema repository.
