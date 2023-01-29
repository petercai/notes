# What Is a Data Mesh Architecture?
In this article, you'll learn more about data mesh architectures and why they're relevant in the modern business environment, as well as why they're important for the next generation of data platforms.

Table of Contents

Over the past decade, organizations have become increasingly dependent on the storage of large amounts of data, creating pipelines to channel the data and extract value from it. This has also increased the need for effective data management and consumption architecture. A recent concept designed to help organizations achieve that goal is the data mesh.

Technology consultant Zhamak Dehghani [first coined the term data mesh](https://martinfowler.com/articles/data-monolith-to-mesh.html) to describe a data platform model based on distributed software architecture and a domain-oriented design. "Domain-oriented" here is derived from the [theory of domain-driven design](https://www.domainlanguage.com/ddd/) (DDD), in which the structuring and language of your code match your corresponding business domain.

All of this simply means that a data mesh architecture supports distributed data users in their specific business branch or domain. In this model, data is a product and each business domain in an enterprise is empowered to handle its own data pipelines.

In this article, you'll learn more about data mesh architectures and why they're relevant in the modern business environment, as well as why they're important for the next generation of data platforms.

Why Do You Need Data Mesh Architecture?
---------------------------------------

The current landscape of data is divided into two planes: operational and analytics data. Operational data is stored in operational databases like [SingleStoreDB](https://www.singlestore.com/), and serves the needs of the business in real time. Analytical data is aggregated over time to be queried for business intelligence. A data mesh recognizes the differences between these data planes and the nature of their architecture, and it seeks to create value from analytical data at scale.

Up until recently, many companies have relied on a [monolithic data infrastructure](https://towardsdatascience.com/data-management-architectures-monolithic-data-architectures-and-distributed-data-mesh-63743794966c#:~:text=A%20monolithic%20data%20architecture%20is%20a%20framework%20where%20application%20data,a%20single%2C%20centralized%20data%20store.) that communicates directly with business intelligence (BI) tools and executes all data transformations for all use cases. These singular data infrastructures — data lakes or warehouses — are responsible for data ingestion, storage, transformation and data output. Such centralized models are effective for organizations that operate with a smaller number of business domains and data consumption needs.

With bigger enterprises, though, there are a wider range of data sources and domains and a more diverse set of customers. The monolithic data approach is less effective in these situations due to a number of shortcomings:

*   An overburdened data pipeline: The traditional model of data architecture ensures your data lake can ingest, store, cleanse, enrich, transform and serve your data. But as more data becomes available, the data system's ability to consume and blend the data into one platform gradually diminishes.
    
*   Isolated and over-specialized teams: Building and managing these data platforms can cause different teams to be heavily siloed and highly specialized. The teams managing the upstream data sources, the engineers building the extract, transform, load (ETL) pipeline and the data consumers or decision makers downstream are usually separated organizationally as well as in their knowledge areas, often without overall business knowledge.
    
*   Slower delivery of output with new customers or data sources: As organizations experiment with innovative agendas to grow their consumer base, new data use cases emerge to meet the demand. This leads to an increase in the organizations' data processing requirements as well as different transformation needs. As a result, organizations will deal with longer response times in their attempt to deliver value to new consumers.
    

To illustrate, consider a media streaming platform that launches operations by providing video media services like YouTube to its consumers. To expand its market, the company adds song streaming services, podcasts and live broadcasts. These new services or business domains mean that new features, such as the number of viewers on a live broadcast or the play rate of a song, need to be included on the platform.

To achieve this, operational data must be ingested, cleansed and newly transformed in the existing ETL pipeline, which will require changing multiple components of the pipeline. This same process will be repeated for every new feature or business domain the platform adds, slowing down the expansion process.

Data mesh architecture represents a paradigm shift away from the traditional approach of building data platforms. At its core, a data mesh is made up of distributed networks of data processing nodes that link application domains to one another.

Similarly to how microservices are a shift away from monolithic applications for modern software engineering, data meshes can be seen as microservices for data platforms. Although the concept is relatively new, it promises a solution to the above drawbacks of monolithic data platforms by allowing a higher degree of flexibility and data autonomy. It creates a reproducible way of managing data sources in an organization's ecosystem, which then allows data consumers — or end users — to quickly and efficiently access their required data.

A data mesh architecture offers multiple benefits to your business:

*   Decentralized data ownership and skills: A data mesh enables the decentralization of your data architecture into separate domains in which the data is never duplicated. This means that the ownership of data and skills required in each domain are also distributed and decentralized, so your organization can democratize data while ensuring that the IT and business requirements in each domain are met.
    
*   Single version of the truth: The long-term organizational goal of achieving a [single version of the truth](https://en.wikipedia.org/wiki/Single_version_of_the_truth) is achievable with a data mesh. Since required data for each business domain is channeled directly to the domain from the source, you can eliminate duplication of data and control any gaps between the data source and data consumers. Each business unit is responsible for providing the consumers and overall business with the truths of their domain as source data sets.
    
*   Distributed security: The highly distributed data architecture that a data mesh creates also requires a distributed security architecture. Identity and access management activities in your data system can either be hosted in-house or outsourced to a third-party platform. Either way, your data will be secure under several layers of encryption and different levels of access will be distributed to users in your business domain.
    

The data mesh concept will continue to evolve as more enterprises plan to store distributed data in silos in order to efficiently generate insights and provide business value. In order to further understand this architecture, you should learn more about its guiding principles.

Principles of Data Mesh Architecture
------------------------------------

There are four basic principles of the data mesh model.

### Domain Ownership

Modern software architecture has been influenced by domain-driven design, as delineated by Eric Evans in his [book of the same title](https://domainlanguage.com/ddd/). The idea is that the microservices architecture of software systems is decomposed and structured into distributed services to mirror the capabilities of business functions or domains. These domains can then be independently owned and managed by domain-specific teams. This is the idea behind domain ownership in data mesh architecture.

Ownership of data is federated in a data mesh. These data owners — engineers and scientists — are held accountable for the availability of their domain data as a service and ensure interoperability with other domains in different locations. Every domain is also responsible for its own ETL pipeline, tasked with data ingestion, aggregation and cleaning.

Once a domain receives data, it carries out specific transformation tasks, and the resulting data is leveraged for the business functions of the domain. One key benefit of implementing this domain ownership of data in a data mesh is that it reduces the number of steps and necessary handoffs between the team producing data and the location where the data is consumed.

Continuing with the media streaming platform example used earlier, if the platform adopts the distributed data architecture in a data mesh when adding a new business domain, that domain can own, process and serve its own data sets in a suitable format for specific use cases downstream. A powerful distributed database like [SingleStoreDB](https://www.singlestore.com/) can serve as storage for data sets in each domain.

### Data as a Product

This simply means that data is handled as a product. As software applications have product owners, so does domain data. Data product owners will ensure data is delivered at the right quality, speed and format. They must have a deep understanding of the domain data transformation requirements and must ensure data meets the users' standards.

Data products are also developed in each domain alongside other functional features. The data product can include cleaning and transformation pipelines, the data set and so on. Teams in each domain that understand the source of the data and the downstream use case are directly responsible for creating and managing data resources. This helps to reduce the friction generated by the traditional approach of aggregating data into a central location and applying the same data model over and over for every use case.

Think of the guiding principles and processes of software development and apply them to data. Concepts like versioning, documentation and encryption are also adopted for data products. Zhamak Dehghani sums up the [major qualities of domain data products](https://martinfowler.com/articles/data-monolith-to-mesh.html#DataAndProductThinkingConvergence:~:text=assets%20come%20handy.-,Domain%20data%20as%20a%20product,-Over%20the%20last), for example, as being discoverable, interoperable, addressable, trustworthy and secure.

### Self-Service Platform

The data mesh model was developed to give teams complete control of their data. Teams are empowered to extract value independently, without the help of a centralized data specialist. The technical complexities of the data platform are abstracted over by automation in self-serve data architecture.

The key benefit of a self-serve data platform is that it lowers the [lead time to create a new product](https://martinfowler.com/articles/data-monolith-to-mesh.html#:~:text='lead%20time%20to%20create%20a%20new%20data%20product') as a result of the built-in automation.

### Federated Governance

Even though data and its domain products are available and accessible everywhere, [data governance](https://www.talend.com/resources/what-is-data-governance/) is still restricted where the data is produced. The objective of having federated governance in a data mesh model is to create an ecosystem that strictly adheres to the rules and regulations of the domain, organization and industry.

In the previous data management models, data is moved from operational databases like SingleStoreDB and stored in a central data lake where cleansing, quality control and encryption take place (centralized data governance). This is decentralized with a data mesh. Each business domain has its own quality control and data governance principles, which must comply with the global business standard before the data becomes a product.

Conclusion
----------

The data mesh architecture model is still a relatively new concept, but it offers multiple benefits to organizations. Shifting data responsibilities to domain teams empowers those teams to work more efficiently and with a greater level of knowledge, while ensuring that the data they use won't be duplicated elsewhere in the organization.

Keep in mind that this shift to decentralize existing data architectures must be done gradually and methodically. Domain engineers and data product owners will need time to gain a deep understanding of the domain data, tools and use cases. The goal is to think in terms of how a data mesh architecture can best serve your team.