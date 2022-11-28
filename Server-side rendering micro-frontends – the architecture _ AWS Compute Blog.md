_This post is written by Luca Mezzalira, Principal Specialist Solutions Architect, Serverless._

Microservices are a common pattern for building distributed systems. As frontend developers have modified their approaches to build architectures at scale, many are building micro-frontends.

This blog series explores how to implement micro-frontends using a server-side rendering (SSR) approach with AWS services. This first article covers the architecture characteristics and building blocks for designing a successful micro-frontends architecture in the AWS Cloud.

What are micro-frontends?
-------------------------

Micro-frontends are the technical representation of a business subdomain. They allow independent teams to work in parallel, reducing external dependencies and increasing delivery throughput. They embody several microservices characteristics such as governance decentralization, design for failure, and evolutionary design.

The main difference between micro-frontends and components is related to the domain ownership present inside a micro-frontend. With components, the domain knowledge is usually delegated to its container, which knows how to use the component’s property based on the context. Owning the domain inside a micro-frontend enables the independence that you expect in a distributed system. This doesn’t mean that micro-frontends cannot communicate or share resources, but the mindset is different compared with components.

If you are using microservices today, you may benefit from micro-frontends for scaling your frontend applications. Before micro-frontends, scaling was based primarily on developers’ expertise. Micro-frontends allow you to modernize frontend applications iteratively like you would with microservices. Every user downloads only the code needed for accomplishing a specific task, increasing the performance and users experience of a web application.

Architecture characteristics
----------------------------

This blog series builds a product details page of an example ecommerce website using micro-frontends with serverless infrastructure.

[![](https://d2908q01vomqb2.cloudfront.net/1b6453892473a467d07372d45eb05abc2031647a/2022/11/04/micro-frontends1-893x1024.png)
](https://d2908q01vomqb2.cloudfront.net/1b6453892473a467d07372d45eb05abc2031647a/2022/11/04/micro-frontends1.png)

The page is composed of:

*   A template that includes a header. This could include more common parts but uses one in this example.
*   A notifications micro-frontend that is client-side rendered. The notifications system must react to user interactions, so cannot be server-side rendered with the rest of the page.
*   A product details micro-frontend.
*   A reviews micro-frontend.

Every micro-frontend is independent and can be developed by different teams working on the same project. This can reduce external dependencies and potential bugs across the entire application.

The key system characteristics of this project are:

1.  **Server-side rendering**: The system must be designed with a server-side rendering approach. This provides fast rendering of the page inside modern browsers and reduces the need of client-side JavaScript for rendering the page.
2.  **Framework agnostic**: The solution must work with a broad variety of JavaScript libraries available and not be bound or optimized to a specific framework.
3.  **Use optimizations best practices**: Optimization is a key feature for server-side rendering applications. Many industries rely on these characteristics for increasing sales. This example encapsulates [core web vitals](https://web.dev/vitals/) metrics, [progressive hydration](https://www.patterns.dev/posts/progressive-hydration/), and different levels of caches to speed up the response times of the webpages.
4.  **Team independence**: Every micro-frontend must be developed with minimum external dependencies. Constant coordination across teams can be a sign of [design-time coupling](https://www.infoq.com/presentations/microservices-design-time-coupling/#:~:text=Design%2Dtime%20coupling%20is%20the,are%20owned%20by%20another%20service.) that invalidates the purpose behind a distributed system.
5.  **Serverless infrastructure for frontend developers**: The serverless paradigm helps developers focus on the business logic instead of infrastructure, using a “pay for value” model, which helps to reduce costs. You can cache micro-frontend responses and reduce the traffic on the origin and the need to scale every part of the system in the same way.

High-level architecture design
------------------------------

This is the high-level design to incorporate these architectural characteristics:

[![](https://d2908q01vomqb2.cloudfront.net/1b6453892473a467d07372d45eb05abc2031647a/2022/11/04/micro-frontends2-1024x424.png)
](https://d2908q01vomqb2.cloudfront.net/1b6453892473a467d07372d45eb05abc2031647a/2022/11/04/micro-frontends2.png)

1.  The application entry point is a content delivery network ([CDN](https://aws.amazon.com/what-is/cdn/)) that is used for caching, performance, and security reasons.
2.  The server-side rendering approach requires a place to store all the static files to hydrate the JavaScript code in the browser and for styling components.
3.  Pages requests require a UI composer that retrieves the micro-frontends and stitches them together to provide the page consumed by a browser. It streams the final HTML page to the browser to enhance the [largest contentful paint (LCP)](https://web.dev/lcp/) metric from the core web vitals.
4.  Decouple micro-frontends from the UI composer relies on two mechanisms: A micro-frontends discovery that acts like [a service discovery](https://docs.aws.amazon.com/whitepapers/latest/microservices-on-aws/service-discovery.html) in a microservice architecture, and an HTML template per page that describes where to inject the micro-frontends inside a page. The templates can live in the same repository where the other static files are present.
5.  The notification micro-frontend reacts to user interactions, providing a notification when a user adds a product in the cart.
6.  The product details micro-frontend has highly cacheable data that doesn’t require many changes over time.
7.  The reviews micro-frontend must retrieve user reviews of a specific product.

The key element for avoiding design-time coupling in this architecture is the micro-frontends discovery. The main advantages of this approach are to provide discoverability to simplify multi-environments strategies, and also to reduce the blast radius thanks to using blue/green deployments or canary releases. This topic will be covered in depth in an upcoming post.

From high-level design into implementation
------------------------------------------

The framework-agnostic approach helps to enable control over system evolution. It achieves this by using HTML-over-the-wire, where every micro-frontend renders an HTML fragment and returns it to the UI composer.

When the UI composer gathers the HTML fragments, it composes the final page to render using [transclusion](https://en.wikipedia.org/wiki/Transclusion). Every page is represented by a specific template hosted in static files. The UI composer retrieves the template and then retrieves placeholder references in the template that can be replaced with the micro-frontend fragments.

This is the architecture used:

[![](https://d2908q01vomqb2.cloudfront.net/1b6453892473a467d07372d45eb05abc2031647a/2022/11/04/micro-frontends3-1024x843.png)
](https://d2908q01vomqb2.cloudfront.net/1b6453892473a467d07372d45eb05abc2031647a/2022/11/04/micro-frontends3.png)

1.  [Amazon CloudFront](https://aws.amazon.com/cloudfront/) provides a unique entry point to the application. The distribution has two origins: the first for static files and the second for the UI composer.
2.  Static files are hosted in an [Amazon S3](https://aws.amazon.com/s3/) bucket. They are consumed by the browser and the UI composer for HTML templates.
3.  The UI composer runs on a containers cluster in [AWS Fargate](https://aws.amazon.com/fargate/). Using a containerized solution allows you to use streaming capabilities and multithreading rendering if needed.
4.  [AWS Systems Manager Parameter Store](https://aws.amazon.com/systems-manager/) is used as a basic micro-frontends discovery system. This service is a key-value store used by the UI composer for retrieving the micro-frontends endpoints to consume.
5.  The notifications micro-frontend stores the optimized JavaScript bundle in the S3 bucket. This renders on the client since it must react to user interactions.
6.  The reviews micro-frontend is composed by an [AWS Lambda](https://aws.amazon.com/lambda/) function with the user reviews stored in [Amazon DynamoDB](https://aws.amazon.com/dynamodb/). It’s rendered fully server-side and it outputs an HTML fragment.
7.  The product details micro-frontend is a low-code micro-frontend using [AWS Step Function](https://aws.amazon.com/step-functions/)s. The Express Workflow can be invoked synchronously and contains the logic for rendering the HTML fragment and a caching layer. This increases performance due to the native integration with over 200 AWS services.

Using this approach, every team developing a micro-frontend is independent to build and evolve their business domain. The main touchpoints with other teams are related to the initial integrations and the communication mechanism between micro-frontends present in the same page. When these points are achieved, every team reduces external dependencies and can embrace the evolutionary nature of micro-frontends.

Conclusion
----------

This first post starts the journey into micro-frontends, a distributed architecture for frontend applications. The next post will explore the UI composer and micro-frontends discovery implementations.

If you are interested in learning more about micro-frontends, see the [micro-frontends decisions framework](https://increment.com/frontend/micro-frontends-in-context/), a mental model created for the initial complexity of approaching micro-frontends design. When used as a north star, the decisions framework simplifies the development of micro-frontends applications.

In the AWS reference architectures section, you can find a [complete diagram](https://d1.awsstatic.com/architecture-diagrams/ArchitectureDiagrams/server-side-rendering-micro-frontends-ra.pdf?did=wp_card&trk=wp_card) similar to the application described in this blog series with additional details.

For more serverless learning resources, visit [Serverless Land](https://serverlessland.com/).