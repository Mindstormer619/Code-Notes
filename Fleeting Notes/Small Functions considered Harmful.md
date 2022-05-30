# Small Functions considered Harmful
> Created: 2022-05-03 21:00

The book (Clean Code) goes on to lay blame on the _length_ of the function as its most grievous offense, stating that:

> Not only is it (the function) long, but it’s got duplicated code, lots of odd strings, and many strange and inobvious data types and APIs. Do you understand the function after three minutes of study? Probably not. There’s too much going on in there at too many different levels of abstraction. There are strange strings and odd function calls mixed in with doubly nested if statements controlled by flags.

The bit where this gets murky is when this “one thing” needs to be defined. The “one thing” can be anything from a simple return statement to a conditional expression to a piece of mathematical computation to a network call. As it so happens, many a time this “one thing” means a single level abstraction of some (often business) logic.

For instance, in a web application, a CRUD operation like “create user” can be “one thing”. Typically, at the very least, creating a user entails creating a record in the database (and handling any concomitant errors). Additionally, creating a user might also require sending them a welcome email. Furthermore, one might also want to trigger a custom event to a message broker like Kafka to feed this event into various other systems.

----

## References
1. https://copyconstruct.medium.com/small-functions-considered-harmful-91035d316c29