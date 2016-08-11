---
title: "Thymeleaf - Working with nested objects "
author: kapeshifk
date: 2016-08-19 16:00:00 +0200
---

I recently joined BSG and on day one I was tasked to work on a graph theory assignment project that required usage of spring framework, hibernate and web-based technology. Without prior experience of the Spring Framework, the challenge was to get my spring application to boot up with my data source, persistence, and web configurations. After difficulties playing with the configurations to get it started, I stumbled upon [spring-boot](http://projects.spring.io/spring-boot/), the answer to all Spring Framework non-adopters, myself included coming from a Java Enterprise Edition (JEE, or the always known J2EE) background. 

Setting up a Spring application can be painful before even getting to the logic implementation. 
The spring-boot project takes away the pain to get Spring up and running as it is designed to simplify the bootstrapping and development of a new Spring application. Although it may look like a developer is giving away control in letting spring boot automate the configurations 
of dependencies elements found in the POM file, one can still disable autoconfiguration of modules as needed and opt for custom implementations. 

This blog will however not cover spring-boot itself, it has been shown that 140 characters can be enough to get a web application up and running with [spring-boot](http://www.slideshare.net/andypiper/andy-p-boot). I will cover my experience of working with nested objects entities in hibernate using thymeleaf. 

Consider a graph that is represented by vertices (locations) and edges (routes), then overlay the routes with traffics to solve the good old shortest path problem. Representing vertices, edges and traffics with POJO classes can be quite straight forward, but I wanted to take advantage of Hibernate capability which facilitates the mapping of collections and associations between entity classes, thus allowing the POJO classes to be persisted to the database with their parent-child relationships.

###Hibernate models
**Vertex:**
```
@Id
@Column
private String id;

@OneToMany(mappedBy = "source", cascade = CascadeType.ALL, orphanRemoval = true)
private List<Edge> sourceEdges = new ArrayList<>();

@OneToMany(mappedBy = "destination", cascade = CascadeType.ALL, orphanRemoval = true)
private List<Edge> destinationEdges = new ArrayList<>();
```

**Edge:**
```
@Id
@Column
private Long id;

@ManyToOne
private Vertex source;

@ManyToOne
private Vertex destination;

@OneToOne(mappedBy = "route", cascade = CascadeType.ALL, orphanRemoval = true, fetch = FetchType.LAZY)
private Traffic traffic;
```

###Thymeleaf.

> Written with [StackEdit](https://stackedit.io/).
