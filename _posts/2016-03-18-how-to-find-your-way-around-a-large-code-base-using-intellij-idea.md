---
layout: post
title:  "How to find your way around a large code base using IntelliJ IDEA"
author: nieldewet
date:   2016-03-18 15:00:00 +0200
---

Joining an existing software project normally means there is a large code base
to familiarise yourself with. As soon as you pick up the first task, you will
find yourself deep in unfamiliar territory and will inevitably spend a lot of
time blindly stumbling from one side of the project directory to the other.
Here are a few tips to help to illuminate your path.  <!--more-->

#### A ubiquitous language

The best solution to any team or process problem is usually a non-technical
one. Enforcing a [ubiquitous
language](https://en.wikipedia.org/wiki/Domain-driven_design#Building_blocks)
to ensure your business users and developers all use the same terms for the
same things will enable you to find an entity in your system by simply looking
for the class that defines it by name. Add to that a consistent naming scheme
for your repositories, factories and services, and you will easily find what
you're looking for.

#### Know where to start

Start exploring the project by noting the different modules and high level
packages. From there, investigate which entities exist in the system. If the
project makes use of an annotation-driven configuration, look for usages of the
[spring
stereotypes](https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/stereotype/package-summary.html),
like __@Controller__ and __@Repository__, to familiarise yourself with some of
the most important components in the system. Once you know about the different
services, repositories, factories and controllers that make up the system you
should read through a few acceptance tests relevant to the changes you trying
to make to understand how the system should behave. Using a debugger to step
through an acceptance test will enable you to understand how everything fits
together.

#### Know your tools

Modern IDE's provide several options for navigating around your system. These
are the features and shortcuts that I find most useful when using [IntelliJ
IDEA](https://www.jetbrains.com/idea/). 

- `Ctrl + N` "Go to class." The mainstay of navigation. Use this to explore
  class names containing the term you are looking for. For example, this could
  show you the _Person_ and _PersonRepository_ classes by entering "Person"
  into the dialog. To see all classes with "Repository" in its name, simply
  enter "Repository".
- `Ctrl + Shift + N` "Go to file." Use this if you are looking for something
  that is not a class, for example the _create_person.sql_ file.
- `Alt + F7` "Find usages." Once you've found an example of the term you were
  looking for you can explore all the places where that symbol is used.
- `Ctrl + Alt + H` "Call hierarchy." You can also explore how and where a
  method is called, and where the caller is called, and so on. This will help
  you to understand what role it plays in the bigger picture.
- `Ctrl + Shift + F` "Find in path." There is nothing quite like a full text
  search to find what you are looking for by looking for any occurrence of a
  string. The dialog offers many ways to narrow down the search, like by
  applying a file name filter.

I hope these tips will help you next time you delve into a new code base.

> Written with [StackEdit](https://stackedit.io/).
