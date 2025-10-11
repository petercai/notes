### Key Takeaways

*   We can define different categories of design patterns, including OOP design patterns, organizational patterns, and so on.
*   A number of design patterns are specifically suited to the serverless paradigm.
*   The Pipes and Filters design pattern can be used to decouple a serveless system into  simple functional units interconnected in a chain.
*   The AWS EventBridge, Simple Queue Service, and Lambda layer are useful abstractions to build event-based serverless systems.  
     

In the software architecture as well as in the application design landscape, design patterns are among the fundamental building blocks. The concept of design patterns was introduced by Christopher Alexander in the late 1970s \[The Timeless Way of Building, 1979; A Pattern Language—Towns, Buildings, Construction, 1977\]:

> Each pattern describes a problem which occurs over and over again in our environment, and then describes the core of the solution to that problem, in such a way that you can use this solution a million times over, without ever doing it the same way twice - Alexander et al.

Later, this concept was adopted by the software community, which led to the creation of different kinds of design patterns applied to the software design space. 

Object-oriented design patterns are an abstract tool to design code-level building blocks following the OOP approach. The book [Design Patterns Elements of Reusable Object-Oriented Software](https://www.journaldev.com/31902/gangs-of-four-gof-design-patterns) by Erich Gamma, Richard Helm, Ralph Johnson, and John Vlissides ([Gangs of Four - GoF](https://en.wikipedia.org/wiki/Design_Patterns)) is a sort of guidebook to the Object-Oriented Design space for developers. The book was first published in the year 1994 and since then design patterns are an integral part of software design.

Design patterns apply to organizations, too. A large organization is indeed like a machine of massive volume. It has lots of gears, pipes, filters, motors, etc. In our digital era where we are trying to digitize the human brain, digitizing the enterprise machinery is nothing extraordinary. Digitizing a single part or section of an enterprise is not enough. In fact, to maneuver an enterprise it is necessary to integrate all of its different parts. Enterprise and solution architects are trying to solve everyday integration scenarios using patterns. The process is truly agile. Every day, from every corner of the world, thinkers are solving problems and inventing new kinds of enterprise integration patterns. I would like to mention here two masterminds in the field, [Martin Fowler](https://martinfowler.com/) and [Gregor Hohpe](https://www.enterpriseintegrationpatterns.com/index.html). 

Top management is constantly chasing new technology trends, with new variations of digital products debuting every day. Business people are on the toe to get maximum benefits from this digital ocean, hence there is a need to modernize legacy systems, a.k.a digital transformation. In this field, too, researchers like [Ian Cartwright](https://www.linkedin.com/in/ian-cartwright-282952/), [Rob Horn](https://www.linkedin.com/in/rob-horn/), [James Lewis](https://bovon.org/) proposed a number of patterns based on their years-long migration experience in their recent article [Patterns of Legacy Displacement](https://martinfowler.com/articles/patterns-legacy-displacement/).

During this limber age, agility is key to success. Resiliency, continuous delivery, faster time to market, efficient development, etc., are some of the forces that drive the [transition to microservices](https://www.cio.com/article/3201193/7-reasons-to-switch-to-microservices-and-5-reasons-you-might-not-succeed.html). At the same time, though, not all scenarios are appropriate for microservice. To help understand where this borderline lies, Chris Richardson, author of [Microservices patterns](https://microservices.io/patterns/microservices.html), proposed plenty of microservice patterns for different use cases.

There are more pattern categories than those I mentioned above. Indeed, there is abundant literature concerning patterns for enterprise system [architecture](https://martinfowler.com/eaaCatalog/) and [software](https://martinfowler.com/articles/enterprisePatterns.html). This means, architects need to choose wisely how to satisfy their requirements.

Enter the serverless world
--------------------------

So far, we have discussed different genres of patterns for different kinds of requirements and architectures, but we left one important case out, serverless systems. Within the current technological spectrum, serverless is one of the most significant and vibrant approaches, especially in the IaaS and Cloud landscape. 

Serverless platforms can be categorized into two broad categories, Function as a Service (FaaS) and Backend as a Service (BaaS). The FaaS paradigm allows customers to build, deploy, run, and manage their applications without managing the underlying infrastructure. BaaS, instead, provides online services to handle specific tasks over the cloud, like authentication, storage management, notification, messaging etc.

All serverless computation-oriented services come under the FaaS category (e.g. [AWS Lambda](https://aws.amazon.com/lambda/), [Google Cloud Function](https://cloud.google.com/functions), [Google Run](https://cloud.google.com/run), [Apache OpenWhisk](https://openwhisk.apache.org/)), while the rest of serverless services can be categorized as BaaS, like serverless storage ([AWS DynamoDB](https://aws.amazon.com/dynamodb/), [AWS S3](https://aws.amazon.com/s3/), [Google Cloud Storage](https://cloud.google.com/storage)), serverless workflow ([AWS Step Function](https://aws.amazon.com/step-functions)), serverless messaging ([AWS SNS](https://aws.amazon.com/sns/), [AWS SQS](https://aws.amazon.com/sqs/), [Google PubSub](https://cloud.google.com/pubsub/docs/overview)), so on and so forth. 

The term serverless is quite attractive, but it can be a bit misleading. Can any service really exist without a server? Behind all the serverless components cloud providers are offering, there lies a simple magic: for all of them, there is a server behind the curtain. Cloud providers are responsible for managing scalability (auto-scaling), invocability, concurrency, networking, etc. for physical and/or virtual servers, and also to provide an interface for end-users to configure them, including things like custom runtimes, environment variables, version, security vault, concurrency, read/write capacity, etc.

If we focus on implementing an architecture using a serverless approach, then some basic, high-level questions may come up. 

*   What would be the preferred architectural style to use to design a system using serverless building blocks? 
*   Will our application be purely serverless or shall we go with an hybrid approach? 
*   What are the use cases where we should  go with a serverless approach? 
*   Are there any reusable architectural building blocks or patterns to implement serverless applications?  

In the rest of this article, I will try to elaborate an answer to the fourth question. 

### Serverless patterns

The serverless paradigm is comparatively new in the technology landscape and it is rapidly evolving. Its different facets, including mechanisms, applicability, use cases, usage patterns, implementation pattern, etc. are changing at each new step. Not only that: as cloud vendors are inventing new serverless offerings, the same serverless patterns can be implemented in multiple ways with varying prices and performance. Across the world, software engineers are thinking in distinct ways from distinct viewpoints. As a consequence, there is no common approach to building serverless systems, as of now. 

At the [API Days Australia conference](https://thenewstack.io/serverless-architecture-five-design-patterns/), AWS solution architect [Cassandra Bonner](https://www.linkedin.com/in/cassandralbonner/?utm_source=thenewstack&utm_medium=website&utm_campaign=platform) presented five major usage patterns for Lambda serverless services. She defined these [five patterns](https://thenewstack.io/serverless-architecture-five-design-patterns/) from the requirement perspective as:

1.  Event-driven data processing.
2.  Web applications.
3.  Mobile and Internet-of-Things applications.
4.  Application ecosystems.
5.  Event workflows.

[Peter Sbarski](https://www.linkedin.com/in/petersbarski/) listed five patterns to solve common design problems in a serverless architecture in his book “[Serverless Architectures on AWS](https://www.manning.com/books/serverless-architectures-on-aws)”. Those are:

1.  Command
2.  Messaging
3.  Priority queue
4.  Fan-out
5.  Pipes and filters

These patterns are not exclusive to serverless architectures. In fact, they are a subset of patterns for distributed systems, such as for example the [65 messaging patterns](https://www.enterpriseintegrationpatterns.com/patterns/messaging/toc.html) organized by Gregor Hohpe and Bobby Woolf, which represent the most extensive collection of such patterns. 

![](articles/design-patterns-for-serverless-systems/en/resources/11-1638176138910.jpeg)

My intention with this article is to implement the Pipes and Filters pattern in the AWS cloud environment following a serverless approach. I will try to discuss some of the alternative implementations and their respective advantages and disadvantages. Reusability is one specific aspect I will consider during the implementation. 

### The Pipes and Filters pattern in a serverless architecture

In agile programming, as well as in a microservice-friendly environment, the general approach to designing and coding has changed from the monolith era. Instead of stuffing all the logic into a single functional unit, agile and microservice developers prefer more granular services or tasks obeying the [single responsibility principle](https://en.wikipedia.org/wiki/Single-responsibility_principle#:~:text=The%20single%2Dresponsibility%20principle%20(SRP,it%20should%20encapsulate%20that%20part.&text=Hence%2C%20each%20module%20should%20be%20responsible%20for%20each%20role.) (SRP). Keeping this in mind, developers can decompose complex functionality into a series of separately manageable tasks. Each task gets some input from the client, executes its specific responsibility consuming that input, and generates some output, which is then transferred to the next task. Following this principle, multiple tasks constitute a chain of tasks. Each task transforms input data into the required output, which is an input for the next task. These transformers are traditionally known as filters and the connector to pass data from one filter to another is known as a pipe. 

A very common usage of the pipes and filter pattern is the following: when a client request arrives at the server, the request payload must go through a filtering and authentication process. While the request is being served, new traffic may come in, and the system has to perform some common tasks like decrypting, authenticating, validating, and removing duplicate messages or events from the request payload before executing the business logic.

Another scenario can be a process to add a product to the cart in an e-commerce app. In this case, the chain of tasks could include the following tasks:  check availability, calculate the price, add a discount, update the cart total, etc. For each of these steps we can write a filter and then join them all using a pipe.

The easiest way to implement this pattern is by using lambda functions. As we know there are two ways to invoke AWS services, synchronously or asynchronously. In the synchronous case, lambda runs the function and  waits until the invoker lambda receives the response from the invoked lambda, whereas in the asynchronous case, it does not wait.  AWS supports callback methods and future objects to receive the response asynchronously. Here the role of a pipe will be played by internal network connections. 

![](articles/design-patterns-for-serverless-systems/en/resources/12-1638176138910.jpeg)

In this direct lambda-to-lambda invocation, both in the synchronous and the asynchronous case, there is a chance of throttling. When the request in-flow is faster than the function’s scale-up capacity, and the function is at its maximum concurrency level (by default 1000), or if the lambda instance count reaches the configured reserved concurrency limit, any additional requests will fail with a throttling error (429 status code). To handle this, we need some intermediate storage between two lambda invocations to temporarily store requests that cannot be processed immediately and some retry mechanism for throttled messages that, once a lambda instance becomes available, picks up the message and starts processing it. 

We can achieve this by using AWS [Simple Queue Service (SQS)](https://aws.amazon.com/sqs/), as shown in the figure below. Each lambda filter processes an event and pushes it onto the queue. In this design, the Lambda can poll multiple events from SQS and process them as a batch, as well, which can improve performance and reduce costs.

![](articles/design-patterns-for-serverless-systems/en/resources/13-1638176138910.jpeg)

This approach can also reduce the  risk of throttling but not avoid it completely. There are few configurable parameters using which we can balance the throttling. Additionally, we can implement a [Dead Letter Queue](https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/sqs-dead-letter-queues.html) (DLQ) for a lambda to address throttled events/messages, and prevent those messages from getting lost . [Lessons learned from combining SQS and Lambda in a data project](https://data.solita.fi/lessons-learned-from-combining-sqs-and-lambda-in-a-data-project/) is a good read to understand the key parameters to address this problem.

In the next  section, I will try to build a generic reusable solution to implement the Pipes and Filters design pattern using another promising AWS component for serverless event processing, the [Amazon EventBridge](https://aws.amazon.com/eventbridge/). 

### Implementing the Pipes and Filters pattern in a serverless architecture

> Amazon EventBridge is a serverless event bus that makes it easier to build event-driven applications at scale using events generated from your applications, integrated Software-as-a-Service (SaaS) applications, and AWS services.

Before reviewing how it works, we need to understand some basic terminology used in relation to AWS EventBridge. 

The [event bus](https://aws.amazon.com/eventbridge/features/) is one of the key components of EventBridge. The event bus receives events/messages from various sources and matches them against a set of defined rules. EventBridge has a default event bus, but users can create their own event bus as well. For this POC I have created an event bus named “pipe”.

![](articles/design-patterns-for-serverless-systems/en/resources/14-1638176138910.jpeg)

[Rules](https://docs.aws.amazon.com/eventbridge/latest/userguide/eb-rules.html) must be associated with a specific event bus. For this POC, I have created three rules for three different filters, as shown in the following picture.  

![](articles/design-patterns-for-serverless-systems/en/resources/15-1638176138910.jpeg)

[Event pattern](https://docs.aws.amazon.com/eventbridge/latest/userguide/eb-event-patterns.html) and [Target](https://docs.aws.amazon.com/eventbridge/latest/userguide/eb-targets.html) are two very basic configurations for each rule. 

Event pattern is a condition. It has the same structure as the events it matches. If the incoming event has a matching pattern, then the rule gets activated and invokes the target (destination) by passing the incoming event. A target is a resource or endpoint that EventBridge sends an event to. One can set multiple targets for a particular pattern.

In our case, I set the lambda name as `detail.target` inside the pattern and once the lambda name matches, the target lambda gets triggered. 

![](articles/design-patterns-for-serverless-systems/en/resources/16-1638176138910.jpeg)

Note: `detail.target` is a json field. Target is a configurable endpoint/destination of the event.  
 

![](https://lh6.googleusercontent.com/w5nMMqpr2Qy83mCEQWMshn1xACqm6YjfSCbdMFmO4SgHFPxBDqoJDDJBDLKKeDNF_3hYj033ywBsFHdQoXfnPVibFK2s4NbH6WRUsGl8NbBn1k-AXi3lnfsVwm9wTepUjxjI2Vlg)

The different steps that can be executed during the event flow are listed below: 

1.  The source generates an event (which must follow the pattern defined by the event source generator and the event bridge rule creator)
    

To test the implementation I have used the following event:

![](articles/design-patterns-for-serverless-systems/en/resources/18-1638176138910.jpeg)

2.  For the specific `detail.target` value of the test event, a rule gets matched and executed. In our case, this causes the event/message to be routed to the target lambda associated with the rule, `filter1_lambda`. 
    
3.  The target lambda completes its task and replaces the event target (`detail.target`) with the next lambda from the `detail.filterlist` json list, i.e. `filter2_lambda`. 
    
4.  The target lambda then invokes the utility function `next_filter()` of the [lambda layer](https://docs.aws.amazon.com/lambda/latest/dg/configuration-layers.html). 
    
5.  `next_filter()` function is responsible to build the final event and put it into the event bridge.
    
6.  Based on the new target value (i.e. `filter2_lambda`) another rule is matched to invoke a separate filter lambda.
    

```
{
  “Detail”: {
    “target”: [“filter2_lambda”]
  }
} 
```

7.  After completing all the tasks, the end filter simply sends the message to the next non-filter destination. In this POC,  the end filter is `filter3_lambda`. Instead of calling the `next_filter` function, this lambda could invoke some DynamoDb API to save data into the DynamoDb table.
    

As we have shown, leveraging the pattern-matching routing facility of EventBridge, one can implement the Pipes and Filters pattern with a single event bus whereby each stage of the chain is free to start processing the next event even if its successor is still busy with the previous one, thus improving the overall efficiency.

![](articles/design-patterns-for-serverless-systems/en/resources/19-1638176138910.jpeg)

As the above picture illustrates, the event initially traverses to `filter1_lambda` because the client event’s `detail.target` matches `filter-rule1` event pattern whose target is `filter1_lambda`. After completing, `filter1_lambda` sets the event’s `detail.target` to the next lambda i.e., `filter2_lambda` and sends back the modified event to the event bus mypipe. Due to the fact that `detail.target`’s value is `filter2_lambda`, `filter-rule2` is triggered and so on and so forth. Using this recursive process, all the filters get executed. The last one can invoke some other resource instead of `next_filter()` utility layer. 

In the above implementation, one of the important tasks common to every lambda is modifying the event target `(detail.target)` with the next lambda from the `filterlist`. To accomplish this, we used the lambda layer.

The lambda layer is a lambda feature which helps developers to extract common functionalities or libraries from the lambda code and put them into a layer. This layer can be used as an utility block, and actual lambda code can execute on top of the layer. Lambda can reuse common functionalities and/or libraries from the layer as required. As per [AWS documentation](https://aws.amazon.com/blogs/compute/using-lambda-layers-to-simplify-your-development-process/) 

> A Lambda layer is an archive containing additional code, such as libraries, dependencies, or even custom runtimes.

For this POC, I wrote a utility layer which exports the `next_filter` function. Lambda filters use this function to deduce the next filter name from the filterlist. The relevant code snippet is given in the appendix at the end of this article.

![](articles/design-patterns-for-serverless-systems/en/resources/110-1638176138910.jpeg)

![](articles/design-patterns-for-serverless-systems/en/resources/111-1638176138910.jpeg)

The entire POC code along with AWS Cloud Development Kit (CDK) infrastructure code can be found in its [github](https://github.com/tridibbolar/serverless-enterprise-designpattern.git) repository. 

Summary
-------

Patterns are one of the most useful and effective tools in the area of software design. To address any common design problem in a standard way we can apply a suitable design pattern. Patterns are just like a design plugin. Serverless is a rapidly growing area in the technological landscape, with all cloud vendors launching new managed serverless services on a regular basis. So it is difficult to decide an appropriate serverless managed service stack. In this article I have discussed different implementation approaches of one such design pattern in a serverless fashion using different AWS managed serverless services.      

Appendix
--------

`next_filter` code snippet:

```
module.exports.next_filter = (async function (event) {
   var i = event.detail.filterlist.indexOf(event.detail.target);
   if (event.detail.filterlist.length === i + 1) {
       return null;
   } else {
       event.detail.target = event.detail.filterlist[i + 1];
       var finalEvent = {
           "Source": event.source,
           "EventBusName": "mypipe",
           "DetailType": event["detail-type"],
           "Time": new Date(),
           "Detail": JSON.stringify(event.detail, null, 2)
       }

       var Entries = [];
       Entries.push(finalEvent);
       var entry = { "Entries": Entries };
       var result = await eventbridge.putEvents(entry).promise();
       return result;
   }
}); 
```

References
----------

*   [Lambda Sqs Scaling](https://aws.amazon.com/premiumsupport/knowledge-center/lambda-sqs-scaling/)
*   [Sqs Short and Long Polling](https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/sqs-short-and-long-polling.html#sqs-long-polling)
*   [Throttling](https://docs.aws.amazon.com/lambda/latest/operatorguide/throttling.html)
*   [Lessons Learned From Combining Sqs and Lambda in a Data Projects](https://data.solita.fi/lessons-learned-from-combining-sqs-and-lambda-in-a-data-project/)

About the Author
----------------

**![](articles/design-patterns-for-serverless-systems/en/resources/1Tridib-Bolar-1638177945871.jpg)
Tridib Bolar** is based in Kolkata, India where he works as a Cloud Solution Architect for an IT firm. He has been working in the programming landscape for over 18 years. Mostly engaged with the AWS platform, he is exploring GCP as a side project as well. In addition to being an admirer of the cloud serverless paradigm, he is also an enthusiast of IoT technology.