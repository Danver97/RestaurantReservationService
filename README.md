# RestaurantReservationService
A microservice system that handles restaurant reservations from users to restaurant owners.  
A small demo **Flutter client** has been developed [here](https://github.com/Danver97/RistoCustomer).

For now it's composed of three microservices:
- [restaurant-catalog-service](https://github.com/Danver97/restaurant-catalog-service) **DEPLOYED**
- [reservation-service](https://github.com/Danver97/reservation-service) **DEPLOYED**
- [auth-service](https://github.com/Danver97/auth-service) **WORKING**

The main focus of the project was on the infrastructure. In particular a big attention was dedicated on the scalabilty of the system.
Every microservice uses a library developed for [event sourcing](https://github.com/Danver97/eventSourcing), which implements Event Store and Event Broker interfaces.

For more info on the architecture see the following articles:
- [Event Sourcing + CQRS: from theory to AWS — part 1](https://medium.com/@chri.pae/event-sourcing-cqrs-from-theory-to-aws-part-1-cb5134a035d5?source=friends_link&sk=aaf1d0d5ddb4ab875cc6532d75c39fde)
- [Event Sourcing + CQRS: from theory to AWS — part 2](https://medium.com/@chri.pae/event-sourcing-cqrs-from-theory-to-aws-part-2-50f126014d91?source=friends_link&sk=71ecc5c07e775018d56332c3c8abf2e5)
- [Event Sourcing + CQRS: from theory to AWS — part 3](https://medium.com/@chri.pae/event-sourcing-cqrs-from-theory-to-aws-part-3-87172efa3971?source=friends_link&sk=17acf8fbdc7b77630420c285c0f12881)

Here's a summary of it:

## The Design

![Architecture](https://user-images.githubusercontent.com/28715404/65134055-ffa0dc00-da03-11e9-8bec-3b7d0d6fef64.png)

I chose an Event Sourcing + CQRS approach.  
Every microservice is loosely coupled with others via asynchronous message-based communication. Every microservice has its own dedicated queue or *Event Broker* (more on this later).

Every microservice writes to a Event Store database and reads from *Projections*.  
A *Projection* is a denormalized view of a stream of events which allows to get data in the form is needed. A *Projection* can be every type of database. For now I sticked to MongoDb, but my design allows any database required.

In order to create and maintain the projection a *Denormalizer* is required. A *Denormalizer* is a component which handles events and use them to update the projection database.

To keep the system tolerant to temporary downtime of the single components, queues or *Event Brokers* are used. Every components that needs to process events (or messages) has its own dedicated queue. This allows to replay events as required for every single component.  
For example you can start a new projection without restarting the others, or you can start a new microservice which needs to process past events from other microservices.

## On AWS

Every microservice is a ECS service that writes on its dedicated table on DynamoDb. Each microservice can write only to its own table.  Every microservice uses a library which handles the writes like if DynamoDb is an *Event Store*.  
DynamoDb Streams events trigger a Lambda function that pushes event to the microservice SNS Topic.  
DynamoDb + DynamoDb Streams + Lambda function + SNS Topic is what I defined previously as *Event Store*.

The SNS Topic sends events to every SQS queue is subscribed to it. Every component that reads from the queue uses a library that handles the messages like it was the original event wrote on the *Event Store* and helps handling any additional envelope added to it.

For easier maintenance every *Denormalizer* is a Lamda function that process SQS queue messages.

[comment]: <![AWS Microservice Architecture](https://github.com/Danver97/TheForkReplica/blob/master/Microservice%20Architecture.svg)>
