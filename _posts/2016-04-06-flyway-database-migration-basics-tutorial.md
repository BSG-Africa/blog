---
title: Flyway Database Migration Basic Tutorial
author: marvinmkhabela
---

Whether one chooses to acknowledge it or not, data has become a very central part of the applications that we develop in modern days. The use of databases is more often enlisted when such data needs to be persisted but in an ever-changing world like the one we live in,
the way we store our data might need to change, this requires some migration scheme of sorts in the case of databases.

This post is to introduce the reader to the basics of the flyway database migration tool, imagine you have a hibernate application which obviously makes use of  a database, suppose you need to change one of your entity objects for whatever reason you might deem valid, let us
say, it no longer makes sense to keep all your data in one table for performance reasons or another one could be that you have decided to add in a new column to your.
<!--more-->
Hibernate can handle some changes but it is less convenient when it comes to migrations, flyway is crafted to overcome that, the flyway dependency is available maven dependency repository.

#### How it works:

To use flyway in your application, you need to define your flyway properties in your application.properties file, you have the option of configuring your DataSource in this file also but I have in my limited experience with the technology found that it is not such a great idea to do so,

{% highlight java %}
# Flyway properties
flyway.baseline-on-migrate=true
flyway.baseline-version=1
{% endhighlight %}

  rather do so in your java configuration setup because, it becomes easier for you to configure different DataSources for your test and production databases, more to follow on flyway tests.
}
}

You need to create the directory {project}/main/resources/db/migration to keep your database migration scrips, the names of this scripts need to comply to a certain format, all scripts must begin with the prefix "V", which is the default flyway prefix, though this can be changed via the flyway.prefix property in the application.properties file, the version number then follows this prefix, these can be natural numbers or positive real numbers. Flyway also uses another special character which is the version separator, which is defaulted to a double underscore, "__", this can also be changed via the application.properties file, flyway will ignore everything between this character and the .sql suffix, which can also be changed too.

These migration scripts comprise of database queries to perform the necessary database operations that you desire, a valid script name under the default settings would be as so, "V1__Create_table.sql"
which might look something like this,

{% highlight sql %}
CREATE TABLE IF NOT EXISTS sample_table(
	key 	INT PRIMARY KEY NOT NULL,
	value	VARCHAR(255)
);
{% endhighlight %}

One thing that one should be very careful of is setting the baseline-on-migrate property on flyway, this should only be done on an initial flyway run, this propery overrides one flyway's "safety checks" which ensures that only 'safe' migrations are executed, on setting this property to true which is otherwise set to false by default, flyway allows migrations onto a non-empty and "unversioned" schema, unversioned because, flyway creates a table named "schema_version" on its initial run and maintains such a table on subsequent runs, you will soon
realize how this works because once you manually alter the schema or any of its constituents, flyway will fail to validate the integrity of your database schema, which would require you to drop your schema_version or perhaps your whole schema if your migration scripts do not
offer the flexibility of a fresh database or have checksums for existence of tables.

One very important fact to note is that flyway will not allow you to set the baseline-on-migrate property on the fly inside your java code, this has to be set in the application.properties file because of the potential risks it introduces. Flyway allows you to set a baseline version which indicates the current state of your database, this number (decimal or natural) specifies the migration version script to start running from, one should note that despite setting a baseline version higher than a certain script version, flyway will still validate all the scripts in the
db/migration folder.

Flyway also allows one the option to clear the schema on error and also just the option to wipe the database on initialization, by setting the clean-on-error and the clean properties to true, only in the application.properties file, it will not work when done in Java code.

Example of flyway java configuration service:

{% highlight java %}
@Service
public class DatabaseMigrationService {

    private DataSource dataSource;

    @Autowired
    public DatabaseMigrationService(DataSource dataSource) {
        this.dataSource = dataSource;
    }

    @PostConstruct
    public void migrateDatabase() {
        Flyway flyway = new Flyway();
        flyway.setDataSource(dataSource);
        flyway.migrate();
    }
}
{% endhighlight %}

One should also decide when to run the flyway migrations, it is generally a good idea to do so as soon as the datasource bean is constructed, I have mainly favoured the PostConstruct annotaion in my usages of flyway, to intitialize flyway in your java code one needs to create a new Flyway object and after setting the properties on the fly (if applicable) call the flyway migrate method. 

To perform tests with flyway one needs to setup a new test datasource unless if of course you wish to use the same database for testing, though I cannot think of a logical reason why one might need to do that.

Another maven dependency by the name flyway-test-extensions needs to be installed to run flyway database migrations in your integration tests, one needs to annotate each test with the annotation, FlywayTest, there are two ways in which one can do this, annotate each method in your test with the flywayTest annotation or annotate the whole class. When a whole class is annotated, flyway will only create the database once and all the methods within the class will share the same database instance (Faster!!!), where else if each method is annotated, each method will have its own database instance created by flyway and will not share with all the other tests in the class. Please note that, this is easier to do with in memory databases such as Hyper SQL and the Apache Derby databases.

{% highlight java%}
@Transactional
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(
        classes = {/* Your dependencies here */},
        loader = AnnotationConfigContextLoader.class)
@ActiveProfiles("test")
@FlywayTest
public class sampleIT {

    @Autowired
    private SessionFactory sessionFactory;
    @Autowired
    private SampleDAO sampleDAO;

    @Test
    public void testName() {

        // Your test goes here
    }
}
{% endhighlight %}


Maven dependencies for both Flyway and Flyway Test: 
{% highlight java%}
<dependency>
    <groupId>org.flywaydb</groupId>
    <artifactId>flyway-core</artifactId>
    <version>4.0</version>
 </dependency>

<dependency>
    <groupId>org.flywaydb.flyway-test-extensions</groupId>
    <artifactId>flyway-spring4-test</artifactId>
    <version>4.0</version>
    <scope>test</scope>
</dependency>
{% endhighlight %}
