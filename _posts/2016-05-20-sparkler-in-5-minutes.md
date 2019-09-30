---
layout: post
title:  "Sparkler in 5 minutes"
date:   2016-05-20 22:53:39
categories: Java
---

![In Search of Developer Happiness](/img/sparkler_presentation/sparkler.001.jpeg#sparkler-presentation-img "In Search of Developers")


In my last post, I [introduced Sparkler](http://sarahaslanifar.com/java/2016/04/12/sparkler.html), a reference app for how I'm using the Spark web application framework to deploy Java-based API's and microservices. I've recently given a few five-minute lightning talks about Sparker based on the concepts in this article.


First of all, why Java and not something "modern" like Clojure, Ruby, or Python? To start, I'm not a *fan* of any programming language or framework: I believe choosing the right tool for the job. Part of what it means to be *the right tool* is matching the skills of the developers involved and the tolerance of the organization for uncertainty.

In many enterprises, Java-based applications are familiar to both development and ops teams. As such, I wanted a way to quickly build Java-based API's that preserved the productivity and flow I'd experienced in some of the dynamic language ecosystems.

![](/img/sparkler_presentation/sparkler.002.jpeg#sparkler-presentation-img "Frameworks")

When talking about building Java web applications, you often hear about solutions like Spring, Play, Struts, Hibernate, etc. "Simple" and "modular" aren't usually the adjectives people choose to describe these creations. Spring, in particular, has a user manual that's over 600 pages long!

The scary thing is that some of these frameworks are so big that people haven't usually even looked at the source code. Even if you did, you might be horrified by the complication of what you find. People mostly resort to treating the framework like a Big Black Box as a coping mechanism..."I don't know how it works, and I don't want to know."

![](/img/sparkler_presentation/sparkler.003.jpeg#sparkler-presentation-img "BIG BLACK BOX")

Combine this large frameworks with workflows that are tied to an editor, you get people writing articles that explain REST in Java in terms of how to deploy a Spring MVC WAR into JBoss using Eclipse without talking about any of the high-level design principles. People quote these articles like it's THE WAY, not just "A Way". How did they get there, and why do they stay?

![](/img/sparkler_presentation/sparkler.004.jpeg#sparkler-presentation-img "IDEs")

Suddenly, you find yourself becoming a *Spring* developer rather than a Java developer. Your simple restful API makes you want to scream.

![](/img/sparkler_presentation/sparkler.005.jpeg#sparkler-presentation-img "WTH")

It makes me feel like someone is jumping on my head while I am already down, wiping snot on my face with a laugh!

![](/img/sparkler_presentation/sparkler.006.gif#sparkler-presentation-img "snot")

What is developer's happiness? A huge pastry shop full of freshly baked and beautifully decorated pastries?

![](/img/sparkler_presentation/sparkler.007.jpeg#sparkler-presentation-img "Happiness?")

Ask my husband, and you'll get a "Yes". What about for the rest of us?

![](/img/sparkler_presentation/sparkler.008.jpeg#sparkler-presentation-img "Found it!")

Enter Spark, a wrapper around Jetty that gives you a nice framework for a restful API. A few weeks ago I had to do a project in Java. When I was searching for a small, simple Java framework to start with, I came across Spark. Within a couple of hours of working on tutorials I was ready to start my application. I added make, Sql2o, Flyway, and RabbitMQ to compose a modular, loosely coupled custom framework for my project.

![](/img/sparkler_presentation/sparkler.009.jpeg#sparkler-presentation-img "Spark")

HTTP routes are defined in a manner very similar to Sinatra or Rails:

{% highlight java %}
public class Routes {

    public Routes(ExampleDao dao, int serverPort) {
        String version = Version.get();
        port(serverPort);

        get("/", (req, res) ->
            "It's time to sparkle and shine! See the README to
                get started. Version: " + version);

        get("/examples/:id", new GetExampleHandler(dao));
        post("/examples", new PostExampleHandler(dao));
        put("/examples/:id", new PutExampleHandler(dao));
        delete("/examples/:id", new DeleteExampleHandler(dao));
    }
}
{% endhighlight %}

For more complicated verbs like POST, you can define a Handler class:

{% highlight java %}
public class PostExampleHandler implements Route {

  final static Logger logger = LoggerFactory
    .getLogger(PostExampleHandler.class);

  private final ExampleDao dao;

  public PostExampleHandler(ExampleDao dao) {
    this.dao = dao;
  }

  @Override
  public Object handle(Request req, Response res) throws Exception {
    String body = req.body();
    logger.info("creating an example: " + body);
    Example example;
    try {
      example = new Gson().fromJson(body, Example.class);
      example.validate();
    } catch (ExampleException e) {
      res.status(400);
      String message = String.format("Invalid Example: %s",
        e.getMessage());
      logger.warn(message);
      return message;
    } catch (JsonSyntaxException e) {
      res.status(400);
      String message = String.format("Invalid JSON: %s", body);
      logger.warn(message);
      return message;
    }

    Example saved = dao.create(example);
    logger.info("example created with id: " + saved.getId());
    res.status(201);
    return String.valueOf(saved.getId());
  }
}
{% endhighlight %}

Functional test units are easy to write.

{% highlight java %}
    @Test
        public void postWithInvalidJsonReturnsBadRequest()
            throws Exception {
        String response = http
            .postJson(DEFAULT_HOST_URL +
                "/examples",
                "bunk body",
                400);
        assertEquals("Invalid JSON: bunk body", response);
    }

@Test
public void postWithInvalidExampleReturnsBadRequest()
    throws Exception {
    String incompleteExample = "{\"name\" : \"foo\"}";
    String response = http
        .postJson(DEFAULT_HOST_URL +
            "/examples",
            incompleteExample,
            400);
    assertEquals("Invalid Example: type is a required field",
        response);
}

@Test
public void putWithValidJsonAndIdReturnsUpdatedExample()
    throws Exception {
    Example saved = dao.create(new Example("foo", "bar"));
    String response = http
        .putJson(DEFAULT_HOST_URL +
            "/examples/" +
            saved.getId(),
            "junk",
            400);
    assertEquals("Example is not valid", response);
}

@Test
public void putWithMissingIdReturnsNotFound()
    throws Exception {
    Example saved = dao.create(new Example("foo", "bar"));
    Example update = new Example(saved.getId(), "foo", "baz");
    String response = http
        .putJson(DEFAULT_HOST_URL +
            "/examples/" +
            saved.getId()+1,
            new Gson().toJson(update),
            404);
    assertEquals(
        "Example with id " + saved.getId()+1 +" does not exist.",
        response);
}
{% endhighlight %}

For those that have found functional testing to have become "Cucumbersome”, seeing an approach for end-to-end testing using nothing more than JUnit may be interesting.

![](/img/sparkler_presentation/sparkler.013.jpeg#sparkler-presentation-img "Cucumbersome")

You can also use a superclass if you need to manage the database for tests or any other purpose:

{% highlight java %}
public class FunctionalTestSuite {
  protected static final String HOST = "localhost";
  protected static final int PORT = 4551;
  protected static final String DEFAULT_HOST_URL =
    String.format("http://%s:%d", HOST, PORT);
  private static CloseableHttpClient httpClient =
    HttpClientBuilder.create().build();
  protected static HttpTestUtils http = new HttpTestUtils(httpClient);

  // NB: Composing rather than inheriting here to make
  // BeforeClass behavior
  // in this class behave as expected. JUnit would normally run the
  private static DatabaseTestRunner dbTest = new DatabaseTestRunner();

  protected static ExampleDao dao = dbTest.getExampleDao();

  @BeforeClass
  public static void setup() throws Exception {
    DatabaseTestRunner.migrateDatabase();
    new Routes(dao, PORT);
    Spark.awaitInitialization();
  }

  @AfterClass
  public static void stopServer() throws Exception {
    Spark.stop();
    httpClient.close();
  }

  @Before
  public void before() throws Exception {
    dbTest.clearDatabase();
  }
}
{% endhighlight %}

Migrations with Flyway are as simple as writing SQL:

{% highlight java %}
public class Migrate {

  public Migrate(String url) throws Exception {
    Map<String, String> params = DatabaseUrl.params(url);
    String dataSourceUrl = JdbcUrl.build(params);
    Flyway flyway = new Flyway();
    flyway.setDataSource(dataSourceUrl,
        params.get("user"),
        params.get("password"));
    flyway.migrate();
  }

  public static void main(String[] args) throws Exception {
    String url = System.getenv("JDBC_DATABASE_URL");
    new Migrate(url);
  }
}
{% endhighlight %}

SQL is already a DSL for schema definition:

{% highlight java %}
create table EXAMPLES (
  ID bigserial primary key,
  E_NAME varchar(255) not null,
  E_TYPE varchar(255)
);
{% endhighlight %}

The Server class is the entry point to the application, where I inject application dependencies:

{% highlight java %}
public class Server {    
  public static void main(String[] args) throws Exception {
    int serverPort = Integer.parseInt(System.getenv("PORT"));
    String url = System.getenv("JDBC_DATABASE_URL");

    Map<String, String> params = DatabaseUrl.params(url);
    Sql2o db = Sql2oFactory.create(params);
    ExampleDao dao = new ExampleDao(db);
    new Routes(dao, serverPort);
  }
}
{% endhighlight %}

Now you have a very simple, custom made framework that fits your business needs without having to learn or use all the extra baggage that comes with other frameworks "out of the box."

![](/img/sparkler_presentation/sparkler.018.jpeg#sparkler-presentation-img "GOT IT!")

Is Sparkler going to be a framework? NO! In fact, I am asking you to stay away from big frameworks; However, Sparkler can help you as a reference app, something for you to get started, and you can add or remove features to make it yours.

![](/img/sparkler_presentation/sparkler.019.jpeg#sparkler-presentation-img "Framework?")


So there you have it! Take it, use it, give me feedback... But most importantly, keep in mind that for the project that you are going to work on for months and maybe even years, it's worth the time to create something that you and your team understand and really need. There is no such a thing as "One Framework fits All Projects!"

![](/img/sparkler_presentation/sparkler.020.jpeg#sparkler-presentation-img "All done")

Stay in touch and feel free to contact me if you have any questions!

![](/img/sparkler_presentation/sparkler.021.jpeg#sparkler-presentation-img "Contact Info")

[Read the previous post about Spark and Sparkler](http://sarahaslanifar.com/java/2016/04/12/sparkler.html)

[Check out the Sparkler source](https://github.com/saslani/sparkler)

[Find me on LinkedIn](https://www.linkedin.com/in/felfeli)

[Email me at sarah@testedminds.com](mailto://sarah@testedminds.com)
