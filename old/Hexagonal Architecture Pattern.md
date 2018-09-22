# Hexagonal Architecture Pattern

A pattern is a generic scheme that describes a solution to a problem in a specific context.

## Reusability

Application frameworks are great! They allow us to concentrate our efforts on the business
logic of our application. By providing a set of reusable components they avoid us
to reinvent the wheel and have stronger foundations.

A very common kind of application framework is for instance all an MVC framework.

It is common to have the business logic inside the "C". The controller has the
responsabilities to:
- receive a user request
- act as coordinator between the several steps of the use case beeing called
 - execute some application logic
   - Authorisation
 - Retrieve data from a datastore
 - execute some pure business logic
 - Call web services
 - Store data to a datastore
- format the response

In that kind of application each request to a controller could be a call to a use
case.

With application that constantly evovle
