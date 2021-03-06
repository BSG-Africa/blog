---
title: "Buzz-kills Chronicles - Part 1: CQRS"
author: brentontenner
date: 2016-09-16 14:00:00 +0200
---

Before we get to the topic at hand, CQRS, let me explain the title of this post. Throughout my adventures in the computer science and software development world, I have come across one deadly creature more than any other. It is deceptively tricky and in the wrong hands, I have seen it cause immense damage to software products, teams and development methodologies. This foul beast is none other than - (drum-roll) - the &quot;buzz word&quot;. In my decade as a software professional, the number of misused, misunderstood, &quot;flavour of the week&quot; terms that I have come across has been staggering – these are what I refer to as &quot;buzz words&quot;. Acronyms are a particularly nasty breed of this species since their biggest pro is also their biggest con - they are catchy and people like to use them since it makes them sound like they really know what they are talking about. They fit beautifully into the jargonauts vocabulary and help them feign being knowledgeable on subjects that they have actually just heard about and don&apos;t truly understand.<!--more-->

This brings me to the title, &quot;Buzz-kill Chronicles&quot;: this will be a series of posts each focused on a different buzz word. My aim is to debunk some of the myths that exist in the vast ocean of misinformation out there, saving you from having to do so yourself during your pursuit of enlightenment (queue epic quest music). In other words, I aim to kill off the fallacies around these concepts, sparing them the tragic fate of remaining abused, misused and misunderstood buzz words until the day that they eventually become irrelevant. I do not claim to be an expert on all of these topics, however, throughout my years in this industry, I have come to understand them at least well enough to refute most of the popular, yet incorrect, understandings of these concepts. Some such concepts include: &quot;Agile&quot;, &quot;Lean&quot;, &quot;Minimum Viable Product&quot;, &quot;Cross-functional teams including DevOps&quot; and &quot;Experimentation&quot;. For this particular post, however, I have chosen to focus on a concept that I was fortunate enough to learn a lot about when looking into designing a new architecture for a legacy application – Command and Query Responsibility Segregation (CQRS).

#### Definition and core benefit of CQRS

<center>
  <img title="Basic CQRS illustration" src="{{ site.baseurl }}/images/CQRS1.jpg"/>
</center>

<center>
  <strong>Image 1 (above): With CQRS, commands from clients travel one way to the command model. Queries are run against a separate data source optimized for presentation and delivered as user interface or reports (ref: Implementing DDD, Vaughn Vernon)</strong>
</center>

A good place to start is by looking at the definition of CQRS. In a nutshell, this concept refers to having two different models in your application that are kept very separate from one another: one for Commands (Write model) and one for Queries (Read model). It is really as simple as that. As we will discuss later, at its core it is not this extremely complex concept that involves things like Event Sourcing, eventual consistency, etc. Those are all implementation details and are relevant, but your CQRS implementation should not be driven by these other things, it need not include them. As is always the case, architecture is just a means to an end, not an end itself and you should always strive for the minimum architectural complexity that make sense for your application.

<center>
  <img title="Another basic CQRS illustration" src="{{ site.baseurl }}/images/CQRS2.jpg"/>
</center>

<center>
  <strong>Image 2 (above): Another depiction of CQRS in action (http://www.codeproject.com/Articles/555855/Introduction-to-CQRS)</strong>
</center>

You may be thinking &quot;why is this guy harping on about definitions?&quot; - but this is very important if you look at it from the following perspective: the benefits of CQRS are well documented, particularly that of creating two simple, fit for purpose models that are optimal for their very different purposes instead of one model that tries to fit both purposes and fails to an extent on both counts. This is the typical &quot;jack of all trades, master of none&quot; argument. One of the major benefits of this is performance. Look at reporting, for example: any report of average or greater complexity that requires the likes of paging / infinite scrolling, sorting, filtering, data to be collated from multiple sources (even just different tables in a relational database) for a large dataset are often cripplingly slow. Not only will this result in a poor user experience, but I have seen reports that when run in parallel by more than a one user, lock up the entire application resulting in it becoming unresponsive to all users. This is clearly a big issue and luckily for us CQRS solves it. This problem is generally caused because models and their underlying data stores (often persisted, e.g. in a relational database) are designed to achieve things like normal form. This is not a bad thing, however, its benefits are mainly realised when used as a write model and it often fails dismally as a read model. As an example: NoSQL data stores, like MongoDB, solve these read performance issues by creating flattened (non-normal form), dynamic schemas and by eliminating the main cause of performance issues when performing queries – joins. On the flip side, these types of data stores lose some of the benefits that one derives from normal forms on the write side, for example, data integrity achieved via foreign key constraints doesn&apos;t exist and data duplication is prevalent meaning that a simple change to one conceptually atomic piece of information may result in several (potentially complex) updates being required to keep all occurrences of the data in sync and consistent. If you have no master store without said duplication to refer back to, what if only a subset of the data changes were processed creating inconsistency? How would you get it back in sync?
By creating a separate read model that is flattened in such a way that it perfectly reflects the data that the report needs to display, performance will be massively improved since the report is already generated and persisted and even if thousands of users request it concurrently, they will simply get the results of a single query as opposed to having all the joins, data collation, etc. execute for every single request.

Let&apos;s get back to why the definitions are important: if you are experiencing such report-related performance issues for example, then you will likely do some research and find out quite quickly that CQRS might very well be the solution that you need. This is good, however, if you mistakenly conflate concepts like Event Sourcing, etc. into the core definition of CQRS, you will likely end up introducing unnecessary complexity just because you thought that these things must be part of any CQRS implementation. Even if CQRS makes sense in your situation, things like Event Sourcing may not. We will cover this point in more detail throughout this post, so let&apos;s move along and examine some of the common fallacies you might find out there regarding CQRS.

#### Fallacy 1: CQRS is complex no matter what

Let&apos;s take a look at some simple examples of CQRS: database views (materialised or otherwise) and Apache Lucene indexes. Database views are definitely within the realm of CQRS and anyone who has worked with relational databases would&apos;ve come across these at some point so I won&apos;t describe them in detail here. Suffice it to say, that it is widely accepted that these are not overly complex concepts to implement – they have been around for ages and are very simple to work with.

The second example is Lucene. For those of you who are not familiar with Lucene, here is the one sentence description: it is a data store that specialises in storing data in an easily full text search-able format. It will store tokenised data with stemmed versions of words, etc. – pretty standard full text searching requirements. In terms of complexity, I would argue that this is not complex at all. Having worked with it myself, I was able to go from not even knowing what it was to using it to meet my application&apos;s requirements entirely in this area in a matter of days (and this was years ago when I was first starting out). It is certainly far less complex than an alternative, bespoke solution that attempts to use a standard read model in normal form, for example, for full text searching. That would likely result in queries spanning many tables littered with nasty &quot;like&quot; comparisons and solutions for things like stemming that make both my head and stomach spin at the mere thought of their hideousness. No thanks – I&apos;ll stick with a read model data store technology that is designed specifically to meet full text searching needs.

Having said that, it is important to note that CQRS implementations can be (and often are) very complex. This is the case when using Event Sourcing and append-only databases, for example. These concepts are complex and a robust implementation thereof is not easy to create. But remember, as I have ranted about at least once already in this post, you do not have to use these concepts with CQRS and I would argue that more often than not, even when CQRS makes sense, things like Event Sourcing may not. The important point is that these things are not synonymous or tightly-coupled concepts, they are orthogonal.

#### Fallacy 2: CQRS is a general &quot;best practice&quot;

Mmmm, don&apos;t get me started on &quot;best practices&quot;. In my opinion the only &quot;best practice&quot; that one should follow without fail is to not follow &quot;best practices&quot; without fail :) Just because some practice has been labeled as such, you should never adhere to it without truly understanding its benefits and the specific contexts in which they are realised. That may sound obvious, but in my experience people prefer to just blindly follow these practices as if they were &quot;rules&quot; since it is easy to do so. But that&apos;s a discussion for another day, let&apos;s get back to CQRS.

If you subscribe to the mantra of keeping things (e.g. architecture, patterns used, etc.) as simple as possible while still meeting requirements, then you could never consider CQRS a general best practice. Many applications, especially those which are very CRUD (Create-Read-Update-Delete) based don&apos;t require a separate read and write model. It would be overkill and would introduce unnecessary complexity. Even for basic reporting a single model is usually fine. Where CQRS comes in is that it is a &quot;better practice&quot; than having a single convoluted model that is attempting to meet the minimum level of required read and write efficiency by making compromises against the one area in order to achieve additional efficiency in the other area. If you find yourself making such compromises / violations, you are probably heading down the wrong road and should consider introducing a CQRS based solution instead.

#### Fallacy 3: CQRS is all or nothing

To clarify: By &quot;all or nothing&quot; I mean that you either split your entire domain across your application as a whole into separate read and write models in order to use CQRS, or you cannot / should not use it at all. Nothing could be further from the truth. In an age where polyglot persistence is becoming accepted as &quot;the norm&quot;, this could never be the case. I am currently working on an application that makes use of polyglot persistence (relational database, NoSQL data store and Lucene indexes) and in my experience, CQRS has made sense within certain bounded-contexts / sub-domains but not others. For example, if a user is viewing information for an employee and he / she updates their name and the screen refreshes to show the update as well as some other information updated as a side-effect, then CQRS is not really necessary here. I would just write the updated data to the write model and then just read it from that same model. This is not a lot of data and performance will not be an issue either, these are just basic single-record CRUD operations. So keep it simple. However, for reports that show all employees with certain associated calculated values and other data linked to these employees but stored in disparate tables, etc. I would definitely use CQRS.

#### Fallacy 4: CQRS requires Event Sourcing

We have touched on this fallacy already a few times. The examples that I have given already disprove this. For that matter, even if you have two custom data stores / models, one for reading and one for writing, you still don&apos;t need Event Sourcing. Event Sourcing requires that you store every application event sequentially and this alone is your write model. This is often stored as an append-only database where there is no standard data store like a normal form relational database. To get the data you need, you have to &quot;replay&quot; events to generate state as of a given point in time. To make this process less time consuming, generally state snapshots are taken periodically and then events are just replayed from these milestones to generate current state. This has some major advantages, but it does introduce significant complexity and you will need to deal with issues, like concurrent updates, in more inventive ways than with a standard normal form relational database, for example.

You could quite easily just update your write model and your read model every time data is updated without generating an event of any type. If eventual consistency is acceptable, then you could update the write model immediately and the read model asynchronously via a scheduled task that runs an Extract-Transform-Load (ETL) routine. All of these approaches don&apos;t require events at all, let alone Event Sourcing.

What is a more common implementation, in my opinion, is to update the write model immediately and then broadcast a Domain Event that will be picked up and stored in an Event Store and processed from there to do the read model update. This will give you eventual consistency and be more efficient, but it will be far closer to full consistency than the &quot;X times a day&quot; ETL approach. However, it is more subject to error if events get &quot;lost&quot;. This is where a robust Event Store would be used. Note that an Event Store and Event Sourcing are not the same thing. With the Event Store we would simply &quot;log&quot; all Events that occurred, but it would not be used as the only write data store meaning we would still have a standard write model data store as well as opposed to just an append-only database.

#### Fallacy 5: CQRS requires an Event-Driven Architecture

I have already addressed this under fallacy 4.

#### Fallacy 6: CQRS requires multiple data stores and distributed transactions

You could quite easily have both your read and write models in the same data store, e.g. they could just be separate tables (or tables and views) in the same relational database schema. If you were to do this, then you would have no need for distributed transactions. For that matter, even if you used two different data stores, you should still try to avoid using distributed transactions through mechanisms that I have already touched on. For example, an Event Store. You would update your write model data store and Event Store in one transaction and later the read model data store would be updated in a completely separate transaction (or perhaps in no transaction at all depending on the nature of the data store).

#### Fallacy 7: CQRS always results in Eventual Consistency

You are probably familiar with the ACID and BASE acronyms already, but if not, here is a quick explanation:
ACID = Atomic, Consistent, Isolated, Durable. This is the case in standard relational databases. We are only interested in &quot;Consistent&quot; right now. This property will ensure that once a transaction completes, any data that was updated in permissible ways during the transaction will immediately reflect as being in its updated state in the database.
BASE = Basic Availability, Soft state, Eventual consistency. For this discussion we only care about &quot;Eventual Consistency&quot;. Unlike full / immediate consistency, eventual consistency does not guarantee that data changes will reflect immediately. It only guarantees that at some point in the future they will reflect, i.e. consistency will be achieved eventually, not immediately.

The examples given under fallacy 4 indicate that there is nothing stopping you from updating both your read and write models in the same transaction if you have them both in a relational database, for example. If they are in separate data stores, then distributed transactions are also an option (although I would stay far away from these). This may not be the most efficient way of doing things, dependent on the time it takes to update your read model, but it is perfectly acceptable from a CQRS perspective. Personally I would rather push for Eventual Consistency and update the read model asynchronously using an Event Store and Domain Events publishers / subscribers, but those are implementation options and don&apos;t speak towards the &quot;pure core&quot; of what CQRS is.

#### Fallacy 8: CQS is a synonym for CQRS

Command Query Separation (CQS) is very closely related to CQRS (the latter evolved from the former), but they are not exactly the same thing. CQS is a concept that predates CQRS. CQS is more about dividing an object / class / component&apos;s methods into very separate read and write categories. CQRS, on the other hand, refers to making the read / write separation at a higher-level, as described previously. So they are very similar concepts but at very different &quot;levels&quot; within your application.

#### Fallacy 9: ORMs should not be used when implementing CQRS

We won&apos;t discuss what Object-Relation-Mapping (ORM) is here, I will assume that you have a basic understanding of this already. If you do not recognize the term ORM, think of JPA (Java Persistence API), Hibernate and EclipseLink.

ORMs have a notoriously poor reputation when it comes to being used for reads. However, the real issue here is not the ORM itself, but rather that the models we create when we use ORMs are usually models that are geared far more to the write side of things. These would often be normal form relational databases, for example. We have already covered this issue and how CQRS solves it.

So there is no real reason not to use ORMs in a CQRS implementation. For that matter, you can use them for both your read and write models. Say for example that you decide to have a normal form relational database as your write model&apos;s data store and a NoSQL data store as your read model&apos;s data store. In this scenario you could create and map tables to your write model entity classes as you usually would with an ORM, such as Hibernate. On the read model / NoSQL side, you could use the Spring Data framework to map flattened representations of your data that are stored in MongoDB documents (suited for reads) to read model classes which might just be used directly as Data Transfer Objects (DTOs).

In summary, I see no reason why ORMs should not be used in a CQRS implementation since the underlying issues that often manifest when using them are actually not their &quot;fault&quot; directly.

#### Fallacy 10: CQRS is a new revelation that no one was ever using before

Under fallacy 1 I spoke about database views and Lucene as CQRS read model examples. Those have been around for ages. People have been using CQRS without even knowing it when using Lucene, database views and similar concepts. A similar argument can be made for what some people call &quot;projections&quot; and JPA (Java Persistence API) constructor queries. I won&apos;t describe these things in detail here, suffice it to say that the notion of separate, specialised read and write models was not born when the CQRS &quot;buzz word&quot; was coined.

#### A potential pragmatic CQRS implementation

I think that a really nice CQRS-based architecture / solution for projects that are quite CRUD-ish in nature but still have high performance reporting requirements is:

Write model data store: Normal form relational database with JPA / Hibernate on top.
Read model data store: MongoDB with Spring Data for MongoDB on top.
When updates occur, they are written to the write model data store. A Domain Event is placed in an Event Store in the same transaction and broad-casted as well.
On a separate thread, Event Listeners receive the broad-casted event and all read model components that require an update on said event are updated via their associated ETL routines.
Every evening all ETL routines are run updating read model data stores just to be safe.
Only reports are read from the separate read model data store (MongoDB). For all other reads a read model would be used as well but the underlying data store would still be the write model&apos;s data store. We would just use things like projections / constructor queries when querying this data store.

Some advantages to this sort of solution over full blown CQRS across the board with Event Sourcing, etc. is that it is still quite simple and you don&apos;t need to do a bunch of nasty hacks to try and get the user experience to be decent when the users perform data updates. What I mean by the latter is that in an eventual consistency environment like the one I have described, when the user updates data you have to employ tricks to make them think that there is full consistency. The ideas I have seen from some CQRS experts are quite horrible – displaying timestamps for when the data is consistent up until, just changing the data on the client and hoping that they don&apos;t refresh until the consistency lag is over otherwise the old version of the data will suddenly resurface, etc.

#### Conclusion

I hope that you have enjoyed reading this post and that it has shone some light on what I consider to be an extremely important topic. If you are currently feeling that you want to explore using CQRS in a minimalistic way, then I think that I have achieved my goal. CQRS is really great and solves a number of issues that I have faced over and over again with the architectures that I have had to use over my career. However, as is the case with everything that can do great good, it can do great evil as well. You really need to be acutely aware of accidental / unnecessary complexity when deciding on a CQRS solution since things can very easily get out of control if you start introducing the more complex related concepts into the fray (like Event Sourcing).

Thank you very much for taking the time to read this piece and please do experiment with CQRS, just tread carefully when doing so and always remember that minimal complexity is the goal.

#### References and further reading

1. [http://projects.spring.io/spring-data-mongodb/](http://projects.spring.io/spring-data-mongodb/)
2. [http://hibernate.org/orm/](http://hibernate.org/orm/)
3. [https://lucene.apache.org/](https://lucene.apache.org/)
4. [https://mariadb.org/eventually-consistent-databases-state-of-the-art/](https://mariadb.org/eventually-consistent-databases-state-of-the-art/)
5. [http://www.martinfowler.com/eaaDev/EventSourcing.html](http://www.martinfowler.com/eaaDev/EventSourcing.html)
6. [http://martinfowler.com/bliki/CQRS.html](http://martinfowler.com/bliki/CQRS.html)
7. [http://martinfowler.com/bliki/CommandQuerySeparation.html](http://martinfowler.com/bliki/CommandQuerySeparation.html)
8. [http://culttt.com/2014/11/12/domain-model-domain-driven-design/](http://culttt.com/2014/11/12/domain-model-domain-driven-design/)
9. [https://lostechies.com/jimmybogard/2012/08/22/busting-some-cqrs-myths/](https://lostechies.com/jimmybogard/2012/08/22/busting-some-cqrs-myths/)
10. [https://cqrs.wordpress.com/documents/cqrs-introduction/](https://cqrs.wordpress.com/documents/cqrs-introduction/)
11. [https://www.safaribooksonline.com/library/view/implementing-domain-driven-design/9780133039900/ch03lev2sec3.html#ch03lev2sec3](https://www.safaribooksonline.com/library/view/implementing-domain-driven-design/9780133039900/ch03lev2sec3.html#ch03lev2sec3)
12. [http://www.slideshare.net/heinodk/domain-event-the-hidden-gem-of-ddd](http://www.slideshare.net/heinodk/domain-event-the-hidden-gem-of-ddd)
13. [http://www.slideshare.net/dbellettini/cqrs-and-event-sourcing-with-mongodb-and-php](http://www.slideshare.net/dbellettini/cqrs-and-event-sourcing-with-mongodb-and-php)
14. [http://www.codeproject.com/Articles/555855/Introduction-to-CQRS](http://www.codeproject.com/Articles/555855/Introduction-to-CQRS)
15. [http://codebetter.com/gregyoung/2010/02/16/cqrs-task-based-uis-event-sourcing-agh/](http://codebetter.com/gregyoung/2010/02/16/cqrs-task-based-uis-event-sourcing-agh/)
