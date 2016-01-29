# Microservice architecture is not a software development approach

Microservices are often presented in contrast of monolithic applications which are frequently associated to a Big Ball of Mud. If your application is a BBoM don't expect microservices will help. They are not a turnkey solution to good design and modularity.

I think that it is really important to understand that microservices are just an architecture style. It's nothing more than Service Oriented Architecture with a couple of opiniated patterns, principles and practices. These tools give guidelines in order to create distributed systems. The main goal of this kind of architecture is to enable different teams to work autonomously on different part of a system and thus improve the overall productivity. When well applied microservices are an investment that will allow you to quickly evolve and make choices independently from other teams.

Microservices won't help you to improve the quality of your code. This architecture style is not a solution to better understand what your application do and how. It doesn't tell you how to organise the code to make it less messy or less buggy.

If you ended up with a monolithic BBoM application and you haven't learn from your mistakes you will probably still end up with some micro-BBoM. Moreover I think that the sum of each micro-BBoM is probably greater than the monolithic BBoM. 
A solution relying on bad designed and implemented microservices is worst than a bad monolithic application.

Building, maintaining and monitoring distributed systems is a lot of challenges. **The additional complexitiy of distributed system architectures plus the complexity of your application's domain could make the overall system a bigger mess**.

In order to get the advantages of the microservice architecture in your projects I think you must have the certain level of maturity. 

Here are some examples of additional complexities introduced by distributed architectures.

## Domain knowledge and design

The Sam Newman's definition of a microservice is:
> *"Small autonomous services that work together around business domain"* - Sam Newman 

Domain Driven Design is a good approach in order to organise microservice according to the domain.

At the beginning the domain model used to solve domain problems will probably evolve a lot in same time you learn the domain. Your will probably want to refactor the model toward deeper insights.

The risk of falling too quickly in the microservice envy is to have the domain model distributed accross different systems. This situation could make it harder and/or slower to try new domain models or improve the existing ones. For instance:

* you may want to split some parts but your microservice is a dependence of another and if you change something you will break the API and thus compromise all of systems that depend on yours;
* you discovered that some concepts distributed accross several micro services are tightly related and it could be better to group them. Unfortunatly these micro services are owned by another team so you have to be synchronised with them. 

The risk of applying microservice architecture too early in the application life is to make experiments harder. So you may end up with a couple of microservices that don't really do the right job or do the job right.

It is better to start applying microservices when you already have a good knowledge of your domain and your models are mature enough.

## Development practices and skills

### Release

A major goal of the microservice architecture style is the capability of each small service to evolve autonomously. Each service has its own release planning and versionning.

An application only exists to solve its end users problems. When composed of distributed systems an application's version is equal to the set of all versions of all microservices involved.

If your microservices pass the acceptance tests with some versions of its dependencies would it be still ok in the production environment where the versions of your dependenances are not the same? The key thing is how in a microservices environment you deal with services dependencies, their versionning and their release.

The worst case is to solve this problem by creating a monolithic-distributed application where all of your microservices are release together and push togerther in production. It is totally against the microservice phylosophy.

Probably some practices like agile acceptance testing, consumer-driven contract and continous delivery could solve (at least partially) the problem.

However in order to apply these practices you have to have the culture of automation.

### Evolving APIs

Microservices communicates together through APIs that hides implementation details. The problems is when your have to evolve the API and introduce changes that could not co-exist with the existing. You have to create a new version of the API that is not backward compatible with the previous one.

In a microservice environment you don't control the consumers of your API. You can't force all of your clients to use your new API. You must still support the old one until all services that depends on you updates their code.

To solve this problem the API versionning comes in the game. But once you have a new version you have to deprecate the old one, communicate to other teams and do the needed stuff in order to one day remove the old endpoints. This also add complexities and rigor in practices.

### Documentation

A service only exists in order to be called. To be called you have to provide to your consumers a high quality documentation. Your must also keep it up to date as your code API evolve.

The risk of a bad documentation is the wrong use of your API which could cause buggy client code and compromise the overall system.

### Monitoring and Debugging

In a monolithic application monitoring is the application himself. Debugging step by step is easy. In a distributed environment it could be a lot more tricky.

Tracing and correlation accross system.

### Failure

How do you avoid the propagation of a failure all over your system?

## Conclusion

Working with microservices is really hard due to their complexity. The set of potential problems related to distributed systems contains more points than the small list I mentionned in this article.

Maybe microservices are a solution for you but maybe not. Falling too early in the microservice envy could move you away from the first purpose of your application: solving business problems. 

Microservice architecture have a lot of advantages in terms of deployment and release management. But it is only true if you apply it well. Applying it well requires skills and practices.

It could be harder to deal with problems issued by distributed systems than dealing with monolithic application problems. It might not be worth.

My advice is that we should first design a monolithic application and try to apply a good and modular design and then move step by step to a microservices architecture... only if needed. If you have designed a modular monolithic application it should be easy to evolve to a distributed application.

Approach like Domain Driven Design really helps you to deal the complexity of your application and create modular design. Principles like Clean Code gives you some practices in order to improve your code quality. These tools can really give your keys to create better software. Not the microservice architecture.


## REMAINING


The productivity could fall. The bug fixing in a distributed environement may have a slower convergence time.

However 

Moreover it is really important to understand what you need. Modularity doesn't necesserally means microservces.

-- and its main advantages are more in the DevOps, release management and multiple teams work.

If your application is a big mess the microservices architecture is not a solution. Micro services architecture have of course a lot of advantages:

* Distributed and autonomous teams
* Code ownership
* Deployment
* Experimentations

In fact as Sam newwman said in the following talks microservices gives your choices.

But its applicabilty (to get these advantages, to view these advantages in your projets) requires maturity.

### Continous Integration

### Testing

and confidence in the problem domain solving. and You have to have a maturity in the design of your application. There is nothing in the microservice principles that defines how to split your application in distrubuted system and the size of each. Some people claims that a micro services should be rewroted in two week. I really think that it is a stupid measure. It is really easy to write really bad code in two weeks.

You have to have maturity in your knowledges and practices. Buggy distributed system is a killer for you productivity.

Having autnomous systems communicating though interfaces and hiding implementation details allow your teams to innovate and experiment. But you can experiment as much as you want but the risk is to break the overall.

If you are working on a very importent micro services which is a central part of your systems your have and your want to experiment something.


-- The more your system is ditributed the more it brings technology complexity. Plus the complexity of your domain could make a bigger mess than before.

It won't helps you to tackle the complexity of your application.

It just brings some solutions in terms of deployment, release management.

All pieces of the distrubuted system are independent and can be release and push in production independtly. It makes your release planning more flexible. It gives you more possibilities.

Experimentation without impact on the overall system (new languages).

It won't help us to better communicates or better share the knowledge. What it could help you is the domain driven design.

There is nothing in ms that defines the size of a micro services. How do you from your use cases, form the user point of view, how do you split the system in terms in micro services. There is nothing that provides the answer oob. 

Some people says that should be rewroter in two weeks. => bullshit.
You can write shit in two weeks and it does not helps you to have a good architecture.

Another point is that why create micro services if the only path to your micro services. YAGNI!

The most important is to have modularity. Even all is packaged together you can have a modularity. Only when the release planning must differ for the several parts of the application or mutiples teams will works on different part you can think about ms.

Ms of MS = stupid.

MS alone is nothing.

Micro services are just a service oriented architecture with some best practices:

* loosly coupled
* cohesion

Remove intelligence in your transport layer. Reduce the impact of a change.

-- In a microservice environment all of your distributed systems is a dependance or depends on another. These microservices work together in order to solve business problems. These small services are small and autonmous. They can have an separated release plannings and deployed independently from each other.

-- As developper we all create bugs (pas intentionnelement). That is a part of our job. Of course our job is also to fix them but it is possible that a bug is not discovered by the actual test suites. So you could have a bug in production or in user acceptance environement. 

Solution?


## You ain't gonna need it