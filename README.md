# TheForkReplica
A simple "The Fork" microservice system replica.

For now it's composed of three microservices:
- [restaurant-catalog-service](https://github.com/Danver97/restaurant-catalog-service)
- [reservation-service](https://github.com/Danver97/reservation-service)
- [user-service](https://github.com/Danver97/user-service)

The business logic of every service is quite simple, the main focus of the project was on the infrastructure. In particular a big attention was dedicated on the data persistence layer. 

I chose an Event Sourcing approach: since the data schema is not completely clear at this stage, I opted for a way to retain any possible piece of information and Event Sourcing seemed the best option to avoid loosing any informations. Another important thing was the loose coupling that event sourcing permits.

This choice brought me to use CQRS: since the data data schema wasn't clear, using CQRS made possible to use different reporting databases for every kind of data representation which better suited the most hit queries. If a new representation is needed, replaying events and the creation of a new reporting database seemed to me another big advantage.

In order to mantain business logic separated by everything else the Onion pattern was followed.

The final step is getting everything running on AWS.

![AWS Microservice Architecture](https://github.com/Danver97/TheForkReplica/blob/master/Microservice%20Architecture.svg)
