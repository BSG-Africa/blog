---
title: Flyway Database Migration Basic Tutorial
author: marvinmkhabela
---
Whether one chooses to acknowledge it or not, data has become a very central part of the applications that we develop today. The use of relational databases is more often enlisted when such data needs to be persisted but in our ever-changing world like the one we live in, the way we store our data might need to change. This requires some migration scheme of sorts in the case of relational databases.

This post aims to introduce the reader to the basics of the Flyway database migration tool. One could imagine having a hibernate application which obviously makes use of a database. Suppose you need to change one of your entity objects for whatever reason you might deem valid, let us say… it no longer makes sense to keep all your data in one table for performance reasons or maybe there is a need for a new column to your table. Hibernate can handle some changes but it is less convenient when it comes to migrations. Flyway, on the other hand is crafted to overcome that, the Flyway dependency is available on the maven dependency repository. <!--more-->

#### How it works :
To use Flyway in your application, you need to define your Flyway properties in your application.properties file. Here, you have the option of configuring your data source in this file also but I have in my limited experience with the technology found that it is not such a great idea to do so, rather do so in your java configuration setup because, it becomes easier for you to configure different data sources for your test and production databases

*Application.properties snippet*

{% highlight java %}
# Flyway properties 
flyway.baseline-on-migrate=true 
flyway.baseline-version=1 
{% endhighlight %}

#### Here is what you need to do :

You need to create the directory {project}/main/resources/db/migration to keep your database migration scrips. The names of this scripts need to comply to a certain format, all scripts must begin with the prefix "V", which is the default Flyway prefix, though this can be changed via the flyway.prefix property in the application.properties file. The version number then follows this prefix, these can be natural numbers or positive real numbers. Flyway also uses another special character which is the version separator, which is defaulted to a double underscore( "__") this can also be changed via the application.properties file. Flyway will ignore everything between this character and the .sql suffix, which can also be changed.

These migration scripts comprise of database queries to perform the necessary database operations that you desire. A valid script name under the default settings would be as so,

*V1__Create_table.sql which might look something like this:*

{% highlight sql %} 
CREATE TABLE IF NOT EXISTS sample_table(
	key 		INT PRIMARY KEY NOT NULL, 
	value 	VARCHAR(255) 
); 
{% endhighlight %}

Something that you should be very careful of is setting the baseline-on-migrate property on Flyway, this should only be done on an initial Flyway run. This properly overrides one of Flyway's "safety checks" which ensures that only 'safe' migrations are executed. On setting this property to true, which is otherwise set to false by default, Flyway allows migrations onto a non-empty and "unversioned" schema, unversioned because, Flyway creates a table named schema_version on its initial run and maintains such a table on subsequent runs. You will soon realize how this works because once you manually alter the schema or any of its constituents, Flyway will fail to validate the integrity of your database schema, which would require you to drop your schema_version or perhaps your whole schema if your migration scripts do not offer the flexibility of a fresh database or have checksums for existence of tables.

Another very important fact to note is that Flyway will not allow you to set the baseline-on-migrate property on the fly inside your java code, this has to be set in the application.properties file because of the potential risks it introduces. Flyway allows you to set a baseline version which indicates the current state of your database, this number (decimal or natural) specifies the migration version script to start running from. One should note that despite setting a baseline version higher than a certain script version, Flyway will still validate all the scripts in the db/migration folder.

Flyway also allows one the option to clear the schema on error, i.e. drop and recreate the target schema when it encounters a versioning related error. This neat and possibly very destructive feature can be activated by setting the clean-on-error property to true, needless to say, one should extra care when setting this property to true. Another such property available on Flyway is the clean property which drops and recreates the schema before doing any migrations, this is activated by setting the clean property in your application.properties file. The two clean features can only be set in that file because of the potential damage they can cause.

*Example of Flyway java configuration service using the Spring framework:*

{% highlight java %} 
@Service
public class DatabaseMigrationService
{
	private DataSource dataSource;

	@Autowired
	public DatabaseMigrationService(DataSource dataSource)
	{
	    this.dataSource = dataSource;
	}

	@PostConstruct
	public void migrateDatabase()
	{
    		Flyway flyway = new Flyway();
    		flyway.setDataSource(dataSource);
    		flyway.migrate();
	}
} 
{% endhighlight %}
One should also decide when to run the Flyway migrations, it is generally a good idea to do so as soon as the datasource been is constructed, I have mainly favoured the PostConstruct annotation (This will run migrations every time the application starts up) in my previous usages of Flyway. To initialize Flyway in your java code one needs to create a new Flyway object, then set Flyway properties on the fly (if applicable), and finally call the Flyway migrate method. To ensure that Flyway runs without any snags one should set the ddl.auto property on their hibernate properties to "validate", otherwise Flyway will throw an exception upon initialization of the migrate method.

To perform tests with Flyway one needs to setup a new test data source unless if, of course you wish to use the same database for testing, although I cannot think of a logical reason why one might need to do that. Setting a separate data source for your tests will require you to have scripts that initialize a blank database since the most commonly used testing databases are in-memory databases.

Another maven dependency by the name flyway-test-extensions needs to be installed to run Flyway database migrations in your integration tests. One needs to annotate each test with the annotation, FlywayTest, there are two ways in which one can do this; annotate each method in your test with the FlywayTest annotation or annotate the whole class. When a whole class is annotated, Flyway will only create the database once and all the methods within the class will share the same database instance which tends to be the faster Otherwise  if each method is annotated, each method will have its own database instance created by Flyway and will not share with all the other tests in the class. Please note that, this is easier to do with in memory databases such as Hyper SQL and the Apache Derby databases.

*A skeleton of a Spring JUnit integration test utilizing the Flyway migration tool,* 

{% highlight java%}
@Transactional
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration( classes = {/* Your Java configuration classes here */},
	loader = AnnotationConfigContextLoader.class)
@ActiveProfiles("test")
@FlywayTest

public class sampleIT
{
	@Autowired
	private SessionFactory sessionFactory;
	@Autowired
	private SampleDAO sampleDAO;

	@Test
	public void testName()
	{
		// Your test goes here
	}
} 
{% endhighlight %}

*Maven dependencies for both Flyway and Flyway Test:*

 {% highlight java%}
 org.flywaydb flyway-core 4.0 
 org.flywaydb.flyway-test-extensions
 flyway-spring4-test 4.0 test
{% endhighlight %}

In conclusion, Flyway is a very neat tool that aids the development environment immensely, especially one where there are a lot of developers working on the same code base. Flyway serves as an automated synchronization tool for all the developers' local databases and the production database. When used appropriately, this can greatly increase productivity and ensure that the production environment runs like a Swiss watch regardless of changes to the relations in the database.
