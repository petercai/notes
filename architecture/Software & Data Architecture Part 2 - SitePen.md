# Software & Data Architecture: Part 2 - SitePen
Why are data architecture concepts valuable?
--------------------------------------------

In [the first part of this article](https://www.sitepen.com/blog/software-data-architecture-part-1), we learned about fundamental data architecture and modeling concepts that provide valuable insights into your system. While the visual nature of these artifacts is helpful in its own right, what other benefits can teams gain from analyzing and documenting these concepts?

The implementation teams benefit the most – technology and product in particular – but adjacent support groups and the system users also benefit. Short of having a dedicated Data Architect role within your team to manage everything described here, keeping everyone mindful of these concepts and promoting good documentation discipline will help at every stage of product delivery. Current project contributors, maintainers, users, and others to follow throughout a system’s lifetime can find value in this information.

### Visibility and Discoverability

Misunderstandings and confusion are inevitable when every contributor builds up a different system knowledge base in their minds, complete with their own gaps, flaws, and biases. Centralized data architecture documentation accessible by everyone helps reduce these problems by promoting a shared understanding of the system. Specific subsets of this information are only relevant for engineers, but a substantial amount will help all people involved in a product – including its users.

System documentation is an increasingly scarce commodity. Developers are often guilty of favoring working code over “irrelevant, out-of-date documentation.” Business stakeholders are encouraged to see visible window dressing without considering everything it takes to deliver functional and robust software over time. Short-term profit hunting makes things worse.

Documenting the data architecture of a system helps identify areas of possible compartmentalization. These could be wholly isolated subsystems or related areas separated through well-defined boundaries. Having visibility of the subsystem boundaries and rates of change allows for more representative team structuring and resource allocation.

When retroactively documenting an existing system, the domain model boundaries can help identify where teams can decompose a monolithic design into smaller units for easier maintenance. Teams and domain specialists can parallelize across independent subsystems. Each could be a candidate for separate implementations with distinct maintenance and release cadences that better reflect the rates of functional change within each area.

### Consistency

Beyond the consistency gained from accurate product documentation accessible to the whole team, data architecture provides the basis for a data dictionary or term glossary. System concepts can be listed and explained, and by doing so, terminology can be standardized. Having a single term to describe a product concept reduces confusion and allows everyone to build on a shared understanding when discussing aspects of the system. Newcomers can refer to the dictionary when looking at data architecture diagrams to quickly ramp up to the team’s established wisdom. Users can benefit from the glossary as part of their product onboarding to learn the system – or the concept definitions should be self-evident if the user experience design within the product itself is achieving its goals.

Standardized concept terminology helps in other areas. The extra visibility also reduces pockets of tribal knowledge around important concepts, minimizing “low bus factor” areas where a team member leaves with their sole understanding of critical product areas. It also facilitates deriving related concepts from existing entities. For example, when assigning security identifiers to resources. A clearly-defined term glossary and a security resource naming scheme mean unambiguous resource locators are easily derivable.

Looking at the [Super Corp information security example](https://www.sitepen.com/blog/software-data-architecture-part-1#information-security) from the first part of this article, a resource naming scheme of “urn:mycorp:<domain>:<concept>” would result in an unambiguous locator of “urn:mycorp:txn:approval” for an “approval (noun)” record within the “txn (singular, abridged)” domain. This shared terminology and naming scheme would dissuade other teams from creating a locator of, for example, “urn:mycorp:transactions:approve” to represent the same security resource – possibly without even realizing the duplication.

### Budget & Planning

Understanding a system’s data and how it changes provides planning insights for how the team structure should change over time. A single small team may be sufficient when starting a project, but as the system grows, the team will struggle to develop new features and maintain the existing platform. Similarly, understanding how to subdivide components and responsibilities is vital if a system was first developed as a monolith but grows to a level where it would benefit from decomposition. Ranking all subcomponents by their relative priority and scale within a large platform ecosystem facilitates such targeted resource division. 

Low-priority areas that deal with small amounts of data or provide non-essential application functionality will require less investment. Support and maintenance for these areas could be centralized to a core support team (possibly in a low-cost location) or shared rotationally between all groups, as each component would rarely justify an entire dedicated team. Components serving higher-value product areas can benefit from a higher budget allocation. Dedicated teams can be allocated to these components, allowing for rapid turnaround of new functionality and immediate resolution of potential issues.

### Product Lifecycle Management

More accurate project planning allows for better consideration of an application’s entire lifecycle, particularly around major version upgrades and associated activities. The documentation process can include data model versioning as part of a more comprehensive application or API platform versioning strategy. Documentation can be versioned and released alongside the product to detail specific data model information related to each product version.

Documentation for subsequent versions can include migration strategies for deprecated or breaking changes from earlier versions. Accurate migration information is key to successful integration upgrades when moving to newer versions or rolling back to a previous version when encountering errors.

This information is also critical to ETL projects looking to migrate data between systems. Migrations between different data model versions apply at multiple granularities. Internal delivery cycles may require low-level migration information between a handful of data domains and platform components – internal teams communicating their changes with each other.

Internal changes roll up into documentation for higher-level version changes, which external system integrators require when moving between major component or product versions. Such external integrations could be with a different division within the same organization or an external third-party customer. Good data architecture documentation is a critical piece of third-party integrations – in this sense, the system’s data model is a significant aspect of what an external integrator purchases, so it’s vital that they understand all relevant concepts and how best to integrate with them.

A system’s data lifecycle closely matches the system’s overall lifecycle. Businesses are keen to beat the competition and get new products in front of users but rarely consider what happens to a system beyond simple growth. Feature development may continue for a time, but will eventually settle, after which a system likely enters a maintenance-only state. Maintainers may fix issues, but not much development happens beyond that. Finally, longer term, a system will be sunset.

Insights into a system’s data lifecycle allow for sufficient and accurate resource allocation and planning throughout all these periods – including what happens to the data when it gets decommissioned or migrated to a successor product.

This still seems like a lot of work
-----------------------------------

There’s no denying that documenting a system’s data architecture takes time and effort, more so when maintaining it over time. However, most teams value this information – whether they realize it or not. They inherently use it when communicating many aspects of product design and implementation – on a whiteboard, a scribble on a piece of paper, or even a simple diagram in a marketing slide deck.

The key is getting these details out of the minds of individual stakeholders into more centralized and consistent formats.  Some minor extra work to formalize and publish data architecture documentation will reap many future rewards. Information is best when verified, published, shared, versioned, and backed up for the future benefit of all. 

Documenting the data model can be done up-front when designing a new system and updated throughout its lifetime or retroactively when analyzing an existing system. It’s up for debate whether there is any real difference in the amount of effort between approaches – but starting an analysis process on an established platform from nothing just to try to meet an immediate planning deadline is a recipe for disaster.

### Keep documentation lightweight

Data modeling should not be an ivory-tower exercise. Instead, it should evolve alongside regular feature development iterations. Creating documentation for it to be outdated immediately is wasteful and dissuades participants from viewing the process or its output as valuable. Using the system to describe itself helps when hashing out product requirements at the start of a project and where the data model remains in flux. 

Establishing standalone data documentation that shows higher-level, more abstract perspectives can help existing and new team members understand systems as they grow over time. Keep longer-term documentation in mind when designing a system beyond directly writing code.

For example, when conducting a whiteboard design session to hash out component layouts and data flows for a new requirement, take a picture of the outcome and spend a few minutes translating it into a neater diagram using the documentation tool of your choice. Several great online tools are freely available these days – no need for expensive graphing software. Collate the new diagrams alongside others within the project’s documentation repository, so they are not lost in a camera roll or the depths of an inbox.

### Automate

To ease the burden of keeping documentation up-to-date, look to derive and automate as much of it as possible. Modern tooling means the bulk of the data documentation can be auto-generated from the system, whether generating ERDs off the current database schema or generating a component graph from the dependency lists within each module’s project descriptor. Rates of change tables could be auto-generated as a dashboard within a system’s monitoring software.

Versioning requirements should also be kept in mind. Ideally, documentation can live alongside the system’s codebase to benefit from delivery pipeline automation – version control, toolchains, release management, packaging, and distribution. Branching off an old version of the system’s codebase to fix a critical security issue may require similar updates to the documentation set that is relevant to that specific version. The challenge with maintaining documentation similar to code is making sure it remains accessible to non-technical people, so everyone still benefits.

### Use documentation to initiate and drive further change

If your project does not have a documentation repository (even though a simple shared folder could work), promote the data architecture information in prominent positions, such as the readme file in a project’s source repository. Encourage team members to keep the diagrams up to date and always use them as the basis for further design conversations to maintain awareness and increase prominence across the team.

The more team members realize the importance of data documentation by directly benefiting from it, the more likely they will maintain it in the future. The benefits of data documentation are primarily visual – it gives new perspectives into a system that are not readily apparent when looking at raw source code. Seeing a system visually on paper or a whiteboard sparks new conversations and ideas, as non-technical team members can join the ideation design process.

### Keep documentation accurate

Data concepts should be documented and made available to the entire team as the “central source of truth.”, but not become an ivory tower that is out of touch with the system implementation. Ideally, through shared ownership and appreciation of the value provided by the documentation, everyone in the project team would contribute to maintaining its accuracy.

As a system grows, it may also help to work at different documentation granularities and delegate responsibilities and ownership. High-level abstract diagrams describing the full system are quickly updated and maintained by technical leads or architects for consistency. Implementation teams could maintain detailed low-level documentation for the domains and components they are responsible for. Component owners can use their slice of the overall documentation when interacting with other teams in designing new feature interactions and data flows. Everyone will be on the same page as they all work within the same information framework.

Without diligence in maintaining documentation, there is a divergence risk between what a system “should” be doing versus what the code means it’s actually doing. Reaching this state is often the death knell for documentation. Nothing is more frustrating to end users and engineers alike than trying to understand a system through its documentation, only to realize the documentation is outdated and does not reflect reality. People discovering this state tend to immediately distrust the documentation and rely on other means of figuring out how the system works, losing a considerable amount of potential value when the vast majority of documented information remains accurate. The documentation could even be correct in the first place, and a divergence highlights an implementation bug that needs fixing!

Keeping documentation as the entry point for understanding a system should mean that people continue to see value in it. Anyone on the team should feel empowered to contribute updates when finding inaccuracies. Small amounts of effort over time can maintain accuracy and usefulness to delivery teams and the product end users, as well as support teams that bridge the two. 

Summary
-------

The broad nature of Information or Data Architecture can dissuade many people from applying the concepts to their projects, especially when simplicity and agility are favored.

However, delivery teams may not appreciate that they undertake many data architecture activities as part of their regular feature delivery work. With a small amount of extra effort to structure these activities and record and publish the output, teams can realize many benefits both in their daily product delivery responsibilities and throughout the overall lifetime of their product.