I recently joined BSG and on day one I was tasked to work on a graph theory assignment project that required usage of Spring Framework, Hibernate and web-based technology. Without prior experience of the Spring Framework, the challenge was to get my Spring application up and running with the datasource, persistence, and web configurations. After difficulties playing with the configurations, I stumbled upon [Spring Boot](http://projects.spring.io/spring-boot/), the answer to all Spring non-adopters, myself included coming from a Java Enterprise Edition (JEE, or the always known J2EE) background. 

Setting up a Spring application can be painful before even getting to the logic implementation. No wonder the [love and hate](https://spring.io/blog/2015/11/29/how-not-to-hate-spring-in-2016) relationship.
The Spring Boot project takes away the pain to get Spring up and running as it is designed to simplify the bootstrapping and development of a new Spring application. Although it may look like a developer is giving away control in letting Spring Boot automate the configurations of the POM file dependencies, one can still disable autoconfiguration of a module as needed and opt for a custom implementation. 

This blog will however not cover Spring Boot itself, it has been reported that 140 characters can be enough to get a [web application up and running with Spring Boot](http://www.slideshare.net/andypiper/andy-p-boot). I will cover my experience of working with Hibernate entities using Thymeleaf. The dependency below is all I needed to include Thymeleaf in my Spring Boot application and included all my HTML pages in the templates folder under resources.

```
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-thymeleaf</artifactId>
</dependency>
``` 

Considering a graph that is represented by vertices (locations) and edges (routes) that are overlaid by traffics to solve the good old shortest path problem. Representing vertices, edges and traffics with POJO classes can be quite straight forward, but I wanted to take advantage of Hibernate capability which facilitates the mapping of collections and associations between entity classes, thus allowing the POJO classes to be persisted to the database with their parent-child relationships.

####Hibernate models
The design of the models below is to have an object Vertex holding all edges connected to it, subsequently, the edges holding all traffic objects related to them. This relationship allows easy handling of CRUD operations for a small project like this. This means deleting a vertex should also delete the edges and traffics associated with it; Deleting an edge should only delete the traffic associated with it and not the vertex; Lastly, deleting the traffic object should only remove any association to the edge and delete that object. The models are as follows.

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
**Traffic:**
```
@Id
@Column
private Long id;

@OneToOne(fetch = FetchType.LAZY)
@JoinColumn(name = "edge_id")
private Edge route;
```
#### Here comes Thymeleaf

I previously worked with JSP and Facelets, so when it came to the choice of what templating technologies I was going to use with Spring MVC, JSPs were in my mind but I found out that JSPs should be avoided if possible when working with Spring Boot as there are several [known limitations](http://docs.spring.io/spring-boot/docs/current/reference/html/boot-features-developing-web-applications.html#boot-features-jsp-limitations). It was time to find another template that has full integration with Spring Boot. There are several Java template engines available, but one of them comes with out-of-the-box support for Spring MVC: [Thymeleaf](http://www.thymeleaf.org/).

Thymeleaf is an XML/XHTML/HTML5 Java template engine that can work in both standalone and web (Servlet-based) environments. In web applications, Thymeleaf aims to be a complete substitute for JSPs and implements the concept of natural templates, meaning that the template file itself is an HTML file that can be rendered by any Web browser. This is made possible by Thymeleaf being an attribute-based template engine, in the sense that all the language syntax relies heavily on HTML tag attributes. The template file is a full HTML file with some special attribute prefixed by the Thymeleaf namespace **th**. A web browser will simply ignore these additional attributes whereas the template engine will process them.

The biggest challenge I faced when working with thymeleaf was to find a way of working directly with Hibernate models on the HTML page without using Data Transfer Object (DTO) handling query and response objects or even complex DTOs.

Looking at a simple Spring MVC controller below that retrieves a list of Java objects from the back-end service and put the list in the model:

```
@RequestMapping(value = "/edges", method = RequestMethod.GET)
    public String listEdges(Model model) {
        List allEdges = entityManagerService.getAllEdges();
        model.addAttribute("edges", allEdges);
        return "edges";
    }
```
Displaying the list of edges an HTML page with Thymeleaf can be done with a table that iterates through the list using th:each attribute:
```
<table>
	<tr>
		<th>Id</th>
		<th>Origin</th>
		<th>Destination</th>
	</tr>
	<tr th:each="edge : ${edges}">
		<td th:text="${edge.id}">Id</td>
		<td th:text="${edge.source}">Planet Origin</td>
		<td th:text="${edge.destination}">Planet Destination</td>
	</tr>
</table>
```

The problem with the above implementation is that the source and destination properties are Java Vertex objects and Thymeleaf will render them as string with the generated HTML data in plan simple object references instead of their actual values (`co.za.package.Vertex@#####`). To solve this problem I had to use Spring Converters and Formatters. Converter components are used for converting one type to another type. Spring already supports built-in converters for the commonly used types and the framework is extensible enough for writing custom converters as well. On the other end, a Formatter is both a Printer and a Parser for an object type and comes into the picture to format the data according to the display where it is rendered.

In this case, I needed to create custom Formatters for Vertex, Edge and Traffic objects to define the behavior of parsing and printing (displaying) the domain objects. This is done by overriding the parse() and print() methods.

```
public class EdgeFormatter implements Formatter<Edge> {

    @Autowired
    EdgeDao edgeDao;

    @Override
    public String print(Edge object, Locale locale) {
        return String.valueOf(object.getId());
    }

    @Override
    public Edge parse(String id, Locale locale) throws ParseException {
        return edgeDao.selectUnique(id);
    }
}
```

The first parameter passed to the print() method is the object itself and the second parameter is the Locale object. It is up to the implementation to consider or to ignore the Locale parameter. In the parse() method, the id is passed as a string which will then be used to query the actual user-defined object, i.e. **Edge**. 
Note that the parse() method also accepts a Locale object as the second parameter. Again, It is up to the implementation whether to consider or to ignore the ‘Locale’ parameter. In my implementation, I ignored the Locale parameter.

The next step is to register the Formatter classes to a Conversation Service as follows: 

```
@Bean(name = "conversionService")
    public FormattingConversionService conversionService() {
        FormattingConversionServiceFactoryBean bean = new FormattingConversionServiceFactoryBean();
        bean.setRegisterDefaultFormatters(false);
        bean.setFormatters(buildFormatters());
        return bean.getObject();
    }

    private Set<Formatter> buildFormatters() {
        Set<Formatter> converters = new HashSet<>();
        converters.add(new VertexFormatter());
        converters.add(new EdgeFormatter());
        converters.add(new TrafficFormatter());
        return converters;
    }
```

Finally, to use the user-defined object on the HTML page, the **id** field linked in the Formatter should be used in the table to lookup nested objects: 

```
<table>
	<tr>
		<th>Id</th>
		<th>Origin</th>
		<th>Destination</th>
	</tr>
	<tr th:each="edge : ${edges}">
		<td th:text="${edge.id}">Id</td>
		<td th:text="${edge.source.id}">Planet Origin</td>
		<td th:text="${edge.destination.id}">Planet Destination</td>
	</tr>
</table>
```

> **Note:**

> - `th:text="${edge.source.id}"` points to the source Vertex.
> - `th:text="${edge.destination.id}"` points to the destination Vertex.

Another good example is to use this same logic on a select component by having Java objects in the dropdown elements: 

```
<select id="paths" th:field="*{helper.sourceVertex}">
	<option th:each="v : ${vertices}"
			th:value="${v.id}"
			th:text="${v.name}+' ('+${v.id}+')'">
	</option>
</select>
```
> **Note:**

> - `th:field="*{helper.sourceVertex}"` represents the selected nested-object.
> - `th:each="v : ${vertices}"`: the iterating attribute to list all vertices.
> - `th:value="${v.id}"` is the selected value that uses the id field of Vertex which will then be converted to its Java object in the parse() method from the back-end.
> - `th:text="${v.name}+' ('+${v.id}+')'"` is what is shown to the front-end. This can be anything meaningful to the user.

We can even go further and access properties of nested objects: If we have a list of edges, for example, we can show additional information of the source and destination Vertex properties like the name/description.

```
<select id="route" th:field="*{helper.selectedEdge}">
	<option th:each="e : ${edges}"
		th:value="${e.id}"
		th:text="${e.source.name}+' - '+${e.destination.name}">
	</option>
</select>
```
The above select component elements can then be handled in the controller on form submission where the selected edge should contain all it related details (e.g. Vertex objects)

```
@RequestMapping(value = "save_traffic", method = RequestMethod.POST)
    public String saveTraffic(Traffic traffic, @ModelAttribute PathHelper helper) {
	    traffic.setRoute(helper.getSelectedEdge());
	    entityManagerService.saveTraffic(traffic);
        return "redirect:/traffic/" + traffic.getId();
}
```

#### Conclusion

Thymeleaf provides first-class support for the Spring Framework and supports nested objects manipulation on the HTML page. By using Thymeleaf in your Spring application, you get all the features of this templating engine: full HTML templating language, modular feature sets called dialects, fast rendering powered by the template caching, form rendering and binding, powerful syntax, flexibility, and good integration with popular web technologies.

> Written with [StackEdit](https://stackedit.io/).