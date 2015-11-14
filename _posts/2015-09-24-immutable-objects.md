---
title: Immutable Objects in Java
author: Rowan Titlestad
---

#### Problems with mutable objects

A mutable object’s state can change whereas an immutable object must always remain in the same state in which it was created. Why would you want to make an object immutable?
<!--more-->
Consider the following method:

```java
public Date dayAfter(Date date) { ... }
```

which is called like so:

```java
final Date today = new Date();
final Date tomorrow = dayAfter(today);
```

You would expect `today` to hold today’s date and `tomorrow` to be one day later. However, there’s nothing preventing `dayAfter` from being implemented as follows:

```java
public Date dayAfter(Date date) {
    Calendar calendar = Calendar.getInstance();
    calendar.setTime(date);
    calendar.add(Calendar.DAY_OF_YEAR, 1);
    date.setTime(calendar.getTimeInMillis());
    return date;
}
```

As you can see, the method is updating the existing object and returning it. This means that both `today` and `tomorrow` are now pointing at the same `Date` object which has tomorrow’s date as its value! The fact that `today` is declared `final` does nothing to prevent the object itself from being changed.

We could avoid this by always remembering to pass a copy of `today` to `dayAfter()` but we could easily forget to do so, especially in a large codebase. If `Date` was immutable then `dayAfter()` would have to return a new `Date` instance and our existing `Date` object would be safe.

**Note:** Java 8 does, in fact, have an immutable date class called `LocalDate` and if you’re using an older version of Java then `LocalDate` is available in the Joda-Time library.

Here is another example of mutable objects opening the door for trouble:

```java
Set<Date> dates = new HashSet<>();
Date theDate = new Date();

dates.add(theDate);
boolean containsTheDate = dates.contains(theDate);
// true

theDate.setTime(0);
boolean stillContainsTheDate = dates.contains(theDate);
// false!
```

How could changing the value of `theDate` cause it to disappear from the `HashSet`? It is, in fact, still there but the `contains()` method can no longer find it. When we add `theDate` to the `Set` its hash code is used to store it in a particular “bucket” of a hash table. When we call `setTime()` the hash code changes because the `Date` class uses the time in its implementation of the `hashCode()` method. When we now call `contains()` the new hash code is used to look up the appropriate “bucket” in the hash table. Since the hash code is now different, the wrong bucket will be looked up and it will seem like `theDate` has vanished.

Other problems with mutable objects include:
* They cannot be safely shared between threads (lack of thread-safety)
* If a method that updates an object throws an exception before it is finished then the object could be left in a partially updated state (lack of failure atomicity)

#### Implementing an immutable object

So how does one write a Java class for an immutable object? The simplest way is to declare all fields on the object `final` and to ensure that all of these fields themselves point to immutable objects. If any field points to a mutable object then it has to be declared `private` and any accessor methods have to return a "defensive copy" of the object. Here's an example:

```java
public class Article {

    private final String title;
    private final Date publishedOn;

    public Article(String title, Date publishedOn) {
       this.title = title;
       this.publishedOn = publishedOn;
    }

    public String title() {
       return title;
    }

    public Date publishedOn() {
       return new Date(publishedOn.getTime());
    }
}
```

Note that we’re protecting `publishedOn` by creating a new `Date` object with the same value. If the caller of `publishedOn()` later decides to alter the `Date` object we returned then the value stored in the `publishedOn` field will remain unchanged.

#### Working with immutable objects

Say we have an instance of `Article` and the user needs to correct its `title`. We can’t update the existing object so we have to create a new `Article` with a different `title` but the same `publishedOn` date.

```java
Article article = new Article("mutables", new Date());
Article updatedArticle = new Article("immutables", article.publishedOn());
```

If we needed to do this in more than one place then we could define a method on `Article`:

```java
public Article withTitle(String title) {
    return new Article(title, this.publishedOn);
}
```

and then change the code to:

```java
Article article = new Article("mutables", new Date());
Article updatedArticle = article.withTitle("immutables");
```

If `Article` had many fields then we could end up with a lot of these "with" methods, each one passing every other field to the constructor. For example:

```java
public Article withTitle(String title) {
   return new Article(title, publishedOn, author, rating, publisher,
                      price, ageRestriction, genre, reviews);
}
```

An alternative would be to create a separate `ArticleBuilder` class:

```java
public class ArticleBuilder {

    private String title;
    private Date publishedOn;

    public ArticleBuilder() {
    }

    public ArticleBuilder(Article article) {
       title = article.title();
       publishedOn = article.publishedOn();
    }

    public ArticleBuilder title(String title) {
       this.title = title;
       return this;
    }

    public ArticleBuilder publishedOn(Date publishedOn) {
       this.publishedOn = publishedOn;
       return this;
    }

    public Article build() {
       return new Article(title, publishedOn);
    }
}
```

and use it like so:

```java
Article article = new ArticleBuilder()
    .title("mutables")
    .publishedOn(new Date())
    .build();

Article updatedArticle = new ArticleBuilder(article)
    .title("immutables")
    .build();
```

The `Article.withTitle()` method could also make use of the builder:

```java
public Article withTitle(String title) {
    return new ArticleBuilder(this).title(title).build();
}
```

Note that the builder itself is mutable so it should only be used to create a new `Article` instance and then be discarded.

#### But I hate writing all that boilerplate!

An alternative to writing the code by hand is to use something like Immutables 2.0 (http://immutables.org) to do it for you. Here’s all we need to define our `Article` (note that an abstract class will work too):

```java
@Value.Immutable
public interface Article {
    @Value.Parameter
    String title();
    @Value.Parameter
    Date publishedOn();
}
```

If you’re using Maven then include the dependency for the annotation processor (other build tools should also support annotation processors):

```xml
<dependency>
  <groupId>org.immutables</groupId>
  <artifactId>value</artifactId>
  <version>2.0.21</version>
  <scope>provided</scope>
</dependency>
```

Now when you build, an `ImmutableArticle` class will be generated and can be used like so:

```java
Article article = ImmutableArticle.of("immutables", new Date());
```

It even generates "with" methods:

```java
ImmutableArticle article = ImmutableArticle.of("mutables", new Date());
ImmutableArticle updatedArticle = article.withTitle("immutables");
```

and a builder class:

```java
Article article = ImmutableArticle.builder()
    .title("immutables")
    .publishedOn(new Date())
    .build();
```

If you import your Maven project into IntelliJ then it will also generate the immutable class every time you either “Make” the project or “Compile” the `Article` interface. If, however, you need to manually set up the annotation processor in your IDE then follow these instructions: http://immutables.github.io/apt.html

#### References:
* https://docs.oracle.com/javase/tutorial/essential/concurrency/immutable.html
* http://www.yegor256.com/2014/06/09/objects-should-be-immutable.html
* http://www.javapractices.com/topic/TopicAction.do?Id=29
* http://immutables.github.io/
