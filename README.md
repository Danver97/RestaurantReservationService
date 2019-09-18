# RestaurantReservationService
A microservice system that handles restaurant reservations from users to restaurant owners.

For now it's composed of three microservices:
- [restaurant-catalog-service](https://github.com/Danver97/restaurant-catalog-service) **DEPLOYED**
- [reservation-service](https://github.com/Danver97/reservation-service) **DEPLOYED**
- [user-service](https://github.com/Danver97/user-service) **WORKING**

The main focus of the project was on the infrastructure. In particular a big attention was dedicated on the scalabilty of the system.
Every microservice uses a library developed for [event sourcing](https://github.com/Danver97/eventSourcing), which implements Event Store and Event Broker interfaces.

## The Design

![Architecture](https://user-images.githubusercontent.com/28715404/65134055-ffa0dc00-da03-11e9-8bec-3b7d0d6fef64.png)

I chose an Event Sourcing + CQRS approach.  
Every microservice is loosely coupled with others via asynchronous message-based communication.  
Every microservice writes to a Event Store database and reads from "projections".  
A *Projection* is a denormalized view of a stream of events which allows to get data in the form is needed. A *Projection* can be every type of database. For now I sticked to MongoDb, but my design allows any database required.  
In order to create and maintain the projection a *Denormalizer* is required. A *Denormalizer* is a component which handles events and use them to update the projection database.  
To keep the system tolerant to temporary downtime of the single components, queues or *Event Brokers* are used. Every components that needs to process events (or messages) has its own dedicated queue. This allows to replay events as required for every single component.  
For example you can start a new projection without restarting the others, or you can start a new microservice which needs to process past events from other microservices.

## On AWS

Every microservice is a ECS service that writes on its dedicated table on DynamoDb. Each microservice can write only to its own table.  Every microservice uses a library which handles the writes like if DynamoDb is an *Event Store*.  
DynamoDb Streams events trigger a Lambda function that pushes event to the microservice SNS Topic.  
DynamoDb + DynamoDb Streams + Lambda function + SNS Topic is what I defined as *Event Store*.
The SNS Topic sends events to every SQS queue is subscribed to it. Every component that reads from the queue uses a library that handles the messages like it was the original event wrote on the *Event Store* and helps handling any additional envelope added to it.
For easier maintenance every *Denormalizer* is a Lamda function that process SQS queue messages.


[comment]: <![AWS Microservice Architecture](https://github.com/Danver97/TheForkReplica/blob/master/Microservice%20Architecture.svg)>
