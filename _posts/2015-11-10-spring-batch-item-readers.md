---
title: Challenges with built-in item readers in Spring Batch
---

Challenges with built-in item readers in Spring Batch
===================

Lately I have done a lot of work with [spring-batch](http://projects.spring.io/spring-batch/). The learning curve is quite steep because there is a lot concepts and principles to master before you can start using it effectively. By now I've read every page of documentation and a number of blogs. Once you get going you think you know what you are doing, but there are still a lot of pitfalls that can trip you up. Here are three.

<!--more-->

Any paging item reader
-------------

When using a [paging item reader](http://docs.spring.io/spring-batch/trunk/reference/html/readersAndWriters.html#pagingItemReaders) and performing processing on the returned items, beware that you could be changing the results of your query between chunks, which means that page 2 might contain different records before processing page 1 and after processing page 1, resulting either in certain records not being returned by your item reader, or others being returned more than once. You can think of this almost like the fail-fast behaviour of iterators in Java which throw an exception when the underlining collection is changed.

Any cursor item reader
-------------

[Cursor-based item readers](http://docs.spring.io/spring-batch/trunk/reference/html/readersAndWriters.html#cursorBasedItemReaders) solve the problem of changing result sets, but have other [problems](http://stackoverflow.com/q/33043411/297331). Spring Batch by default performs a commit after every chunk, which closes the cursor. The framework solves this problem by creating a new connection that does not participate in a transaction. However, if your job is deployed in an application server, the step will fail at the start of the next chunk, because the *"Result set already closed"*. This is because connections supplied by the container always participate in a transaction. To solve this you have to set up another, non-transactional data source.

Hibernate paging item reader
-------------

When I was confident that my job would not change the underlying result set, I used the [HibernatePagingItemReader](http://docs.spring.io/spring-batch/apidocs/org/springframework/batch/item/database/HibernatePagingItemReader.html) supplied by Spring Batch. I've read before that people advise against using this class, but it seemed most appropriate for the task. In the end it caused a lot of pain, because sometimes when processing large amounts of data some items were duplicated, and others omitted. The number of duplicated items was always exactly the same as the number of omitted items, and there were no skips or retries. The only way of detecting the problem was to manually check the data. At the moment I still can't explain why this happens, and why it only happens occasionally, but I have replaced the item reader with my own, and did more work during the processing phase to work around the problem.

Conclusion
-------------

Spring Batch is a powerful framework for performing a batch operation on data from a database. It is important to be aware of the pitfalls and to test thoroughly.
