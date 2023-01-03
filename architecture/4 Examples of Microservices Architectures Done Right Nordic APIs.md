# 4 Examples of Microservices Architectures Done Right | Nordic APIs |
Microservices are everywhere in today’s increasingly virtual, decentralized world. [85% of organizations](https://www.statista.com/statistics/1236823/microservices-usage-per-organization-size/) with 5,000 or more employees use microservices in their organization in some capacity as of 2021. Even more tellingly, 0% report having no intention of adopting microservices in their company.

Clearly, microservices are here to stay, meaning more and more businesses will be adopting microservices in the coming months and years. This is good news, as microservices are capable of so much. This popularity comes with its own risks, though. Some businesses that integrate microservices into their existing workflow will need help figuring out what to do with them.

It’s well worth learning how to set up and navigate microservices. According to IBM’s [Microservices in the Enterprise, 2021 report](https://www.ibm.com/downloads/cas/OQG4AJAM), the majority of companies currently using microservices report improved customer productivity, customer satisfaction, and getting products to market faster. They also open up about some of their issues with microservices, though.

52% of companies using microservices report having issues with the complexity of learning microservices. 54% report having problems finding employees with relevant experience, and 49% experience trouble deciding which services to migrate to microservices.

So while there is nearly endless opportunity for what microservices can bring to your business, there’s also a need to see microservices in action to get a better sense of what good microservice architecture looks like. To help you make the most of microservices in your enterprise, we’ve pulled together some examples of microservice architecture done right to give you some ideas of current best practices.

Amazon
------

In the early 2000s, Amazon was set up as a [monolithic application](https://nordicapis.com/whats-the-difference-between-monolith-and-microservices/). This led to tight dependencies that needed to be navigated whenever a developer wanted to scale or upgrade Amazon’s system.

Rob Brigham, a senior product developer for Amazon in the early 2000s, accounts to [Newstack.io](https://thenewstack.io/led-amazon-microservices-architecture/) his experience in these early days:

> “If you go back to 2001 the [Amazon.com](http://amazon.com/) retail website was a large architectural monolith. Now, don’t get me wrong. It was architected in multiple tiers, and those tiers had many components in them … But they’re all very tightly coupled together, where they behaved like one big monolith. Now, a lot of startups, and even projects inside of big companies, start out this way … But over time, as that project matures, as you add more developers on it, as it grows and the code base gets larger and the architecture gets more complex, that monolith is going to add overhead into your process, and that software development lifecycle is going to begin to slow down.”

This cautionary tale is also an excellent example of how an enterprise can successfully adopt microservice architecture. As Brigham notes, nearly every organization starts out as a monolith. It requires a paradigm shift to get into the microservice mindset.

To overcome these obstacles, Amazon started by breaking down its site into [small chunks of code](https://nordicapis.com/the-bezos-api-mandate-amazons-manifesto-for-externalization/) that fulfill a single purpose. These code snippets were then wrapped up into a simple web interface. Examples include the ‘Buy’ button or the tax calculator.

Each of these functions was then assigned to its own team. These small pieces were then reconnected to the larger system, using web service APIs for communication. This allowed Amazon developers to create a robust ecosystem of [loosely coupled](https://nordicapis.com/how-to-design-loosely-coupled-microservices/) web services.

Amazon’s innovations in the early 2000s laid the groundwork for today’s microservices. It also made Amazon’s technical infrastructure possible, making them one of the most valuable companies on Earth.

Netflix
-------

One of the world’s largest streaming sites is also largely responsible for pioneering today’s microservice architecture. Netflix simply couldn’t keep up with the demand when it started its movie streaming service in 2007. High demand for orders even managed to shut down their legacy DVD-shipping service for three days in 2008.

Prior to this failure, Netflix was storing all of its data in one massive database. Relying on vertically scaled single points of failure, such as a relational database, made the system far too fragile. Instead, they needed to transition to distributed systems in the cloud.

Netflix started transitioning to its current microservice-based architecture in 2009. They began by transitioning all non-customer-facing code and services to AWS Cloud.

This transition made Netflix more scalable and helped avoid the service outages they were incurring, allowing them to become the global juggernaut they are today. By 2015, Netflix’s API was receiving [two billion API edge case requests](https://smartbear.com/blog/develop/why-you-cant-talk-about-microservices-without-ment/) each day across over 500 microservices. This increased to 700 microservices by 2017.

Transitioning to microservices also helped Netflix to reduce costs, allowing them to keep costs down to become the household name they are today.

Uber
----

Uber would not be able to exist if not for microservices. Although very brief, their monolithic structure resulted in insurmountable hurdles to their growth. Without microservices, the ride-sharing app couldn’t fix bugs when they occurred, develop or launch new services, or transition to a global marketplace. Their monolithic structure was prohibitively complex, requiring developers to have extensive experience working with the system to make even simple changes.

Before the transition, passengers would have to connect to Uber’s monolithic system through a [REST API](https://nordicapis.com/what-is-the-difference-between-apis-and-microservices/). This system contained three adapters for simple functions: billing, payment, and text messaging. There was also a MySQL database contained in the monolith.

[![](https://nordicapis.com/wp-content/uploads/designmicroservicesuber.png)
](https://nordicapis.com/4-examples-of-microservices-architectures-done-right/designmicroservicesuber/)

Here’s an illustration of Uber’s original monolithic architecture from DZone. [Image source](https://dzone.com/articles/microservice-architecture-learn-build-and-deploy-a).

To remedy this situation, Uber developers broke individual functions down into microservices like passenger management or trip management. They then connected with the microservices via an API gateway.

[![](https://nordicapis.com/wp-content/uploads/designmicroservicesubermicroservices.png)
](https://nordicapis.com/4-examples-of-microservices-architectures-done-right/designmicroservicesubermicroservices/)

Here’s a diagram of Uber’s revised microservices-based system. [Image source](https://dzone.com/articles/microservice-architecture-learn-build-and-deploy-a).

At Uber, transitioning to microservices architecture let individual development teams focus on specific features. This resulted in new features being implemented in far less time while simultaneously increasing quality and ease of management. It also gave Uber the ability to work on individual components without breaking the rest of the system, making the network more resilient.

Uber’s transition to microservices introduced new challenges, however. The lack of existing standards when they began transitioning to microservices quickly resulted in their microservice architecture getting [out of control](https://nordicapis.com/tips-for-right-sizing-microservices/).

Susan Fowler, a reliability engineer for Uber, [discussed the problem](https://www.theserverside.com/feature/How-microservices-patterns-helped-Uber-systems-perform-better) as well as its eventual solution:

> “We have thousands of microservices at Uber. Some are old and some are not used anymore and that became a problem as well. A lot of work has to be put into making sure you cut those out and do a lot of deprecating and decommissioning.”

The implementation of global standards allowed their microservices to become more loosely decoupled, making their network far less brittle. Previously, changing one microservice often required changing the rest of their microservices as a result.

To create global standards, Uber developers started by creating metrics for microservices availability. These include fault tolerance, documentation, performance, reliability, stability, and scalability. They then devised ways to measure these factors, allowing them to assess what was working and what wasn’t. It gave developers an actionable blueprint to follow and made the process far less abstract and more complete. All in all, Uber’s transition services as a valuable advocacy for microservices’ viability.

[![](https://nordicapis.com/wp-content/uploads/designmicroservicesuber3.png)
](https://nordicapis.com/4-examples-of-microservices-architectures-done-right/designmicroservicesuber3/)

Here’s a representation of Uber’s microservices network as of 2019. [Image source](https://twitter.com/msuriar/status/1110244877424578560).

Etsy
----

High-speed internet has dramatically driven up user expectations for app and website performance. It’s no longer enough for a website to be available — it needs to be _fast_.

Etsy, the popular marketplace app, started having issues when its performance began to slip due to increased demand. Etsy’s development team set their goal at 1000 milliseconds per transaction. They quickly realized this was only possible by using concurrent API calls. The problem was their existing PHP-based system couldn’t accommodate concurrent API calls.

This meant they needed to update their existing system while still making it familiar to both their existing users as well as their developers.

They started by implementing a two-level API system using meta-endpoints similar to Netflix. Using this system, the customer-facing website and app create a custom view based on these meta-endpoints. The meta-endpoints then call the atomic component endpoints.

As a result, the atomic component endpoints are the only resources communicating with the actual database. This improved the website and app performance, but the lack of support for concurrent processing was still acting as a bottleneck.

Finally, Etsy developers adopted [cURL](https://nordicapis.com/understanding-the-hidden-powers-of-curl/) to allow concurrent HTTP requests. Finally, they created their own custom [libcurl patch](https://gist.github.com/mdg/fde0fedafb0de42ae411) and custom monitoring tools. They also created a suite of API tools to make developers’ lives easier and to increase the speed of adopting the two-tier API system.

Ever since implementing this microservice architecture in 2016, Etsy’s developers have been able to continually update their products, make use of concurrent processing, and more easily scale their products.

[![](https://nordicapis.com/wp-content/uploads/designmicroservicesetsy.png)
](https://nordicapis.com/4-examples-of-microservices-architectures-done-right/designmicroservicesetsy/)

A slide illustrating Etsy’s two-tier microservice system. [Image source](https://www.infoq.com/presentations/etsy-api/).

Final Thoughts on Microservices Architecture Done Right
-------------------------------------------------------

When appropriately implemented, microservices boost an organization’s scalability, reliability, and performance. It makes for a more pleasant experience for users and developers alike.

When microservices architecture is done right, it sets an organization up to be ready for anything. Each of these four examples — Amazon, Netflix, Uber, and Etsy — went on to become some of the most powerful corporations the Earth has ever seen. None of it would have been possible without microservices.