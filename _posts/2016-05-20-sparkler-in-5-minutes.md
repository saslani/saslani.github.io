---
layout: post
title:  "Sparkler in 5 minutes"
date:   2016-05-20 22:53:39
categories: java
---

> Hi, I am Sarah and I am going to talk about how to get your Java app. ready to deploy in less than 10 minutes. Bare in mind I am not a fan or against any languages; rather I believe choosing a right tool for the job.

![In Search of Developers](/assets/images/sparkler_presentation/sparkler.001.jpeg#sparkler-presentation-img "In Search of Developers")

> When talking about Java you often hear Spring, Play, in old days, Struts, Hibernate, etc. Words like "simple" and "modular" haven't often been used to describe Java web application frameworks. Spring, in particular, has a user manual that's over 600 pages long!

![](/assets/images/sparkler_presentation/sparkler.002.jpeg#sparkler-presentation-img "Frameworks")

> The scary thing is that some of these frameworks are so big that people haven't usually even looked at the source code, even if you did, you might be horrified by the complication of what you find they protect themselves by treating the framework like a Big Black Box... "I don't know how it works, and I don't want to."

![](/assets/images/sparkler_presentation/sparkler.003.jpeg#sparkler-presentation-img "BIG BLACK BOX")

> Combine that with workflows that are tied to an editor, you get people writing articles that explain REST in Java in terms of how to deploy a Spring MVC WAR into JBoss using Eclipse; and people reading it like it's gospel; like it's THE WAY, not just A Way. How did they get there, and why do they stay?

![](/assets/images/sparkler_presentation/sparkler.004.jpeg#sparkler-presentation-img "IDEs")

> Suddenly you simple restful API makes you want to scream - you are becoming a *Spring* developer rather than a Java developer.

![](/assets/images/sparkler_presentation/sparkler.005.jpeg#sparkler-presentation-img "WTH")

> The way it makes me feel is as if someone jumps on my head while I am already down; wipes his snot on my face and has the guts to laugh at me!

![](/assets/images/sparkler_presentation/sparkler.006.gif#sparkler-presentation-img "snot")

> What is developer's happiness? A huge pastry shop full of freshly baked and beautifully decorated pastries?

![](/assets/images/sparkler_presentation/sparkler.007.jpeg#sparkler-presentation-img "Happiness?")

> Ask my husband, yes. What about for the rest of us?

![](/assets/images/sparkler_presentation/sparkler.008.jpeg#sparkler-presentation-img "Found it!")

> This is where Spark comes to play, no pun intended! Spark is a wrapper around Jetty that gives you a nice framework for the restful API. A few weeks ago I had to do a project in Java. When I was searching for a “lightweight” Java framework to start with, I came across Spark and within a couple of hours of working on tutorials I was ready to start my application. I used the very simple Spark framework and added make, SQL2O, flyway and RabbitMQ to make a small custom framework for my project.

![](/assets/images/sparkler_presentation/sparkler.009.jpeg#sparkler-presentation-img "Spark")

> This is how you define routes in Spark, very similar to Ruby, for example.

    public class Routes {

        public Routes(ExampleDao dao, int serverPort) {
            String version = Version.get();
            port(serverPort);
            
            get("/", (req, res) ->
                "It's time to sparkle and shine! See the README to get started. Version: " + version);

            get("/examples/:id", new GetExampleHandler(dao));
            post("/examples", new PostExampleHandler(dao));
            put("/examples/:id", new PutExampleHandler(dao));
            delete("/examples/:id", new DeleteExampleHandler(dao));
        }
    }

> For more complicated RESTs, for example, POST, you can define your Handler that extends route to do more custom work.

    public class PostExampleHandler implements Route {
    
      final static Logger logger = LoggerFactory.getLogger(PostExampleHandler.class);
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
          String message = String.format("Invalid Example: %s", e.getMessage());
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

> Functional test units are of course supported and easy to write.

    @Test
        public void postWithInvalidJsonReturnsBadRequest() throws Exception {
        String response = http.postJson(DEFAULT_HOST_URL + "/examples", "bunk body", 400);
        assertEquals("Invalid JSON: bunk body", response);
    }
    
    @Test
    public void postWithInvalidExampleReturnsBadRequest() throws Exception {
        String incompleteExample = "{\"name\" : \"foo\"}";
        String response = http.postJson(DEFAULT_HOST_URL + "/examples", incompleteExample, 400);
        assertEquals("Invalid Example: type is a required field", response);
    }
    
    @Test
    public void putWithValidJsonAndIdReturnsUpdatedExample() throws Exception {
        Example saved = dao.create(new Example("foo", "bar"));
        String response = http.putJson(DEFAULT_HOST_URL + "/examples/" + saved.getId(), "junk", 400);
        assertEquals("Example is not valid", response);
    }
    
    @Test
    public void putWithMissingIdReturnsNotFound() throws Exception {
        Example saved = dao.create(new Example("foo", "bar"));
        Example update = new Example(saved.getId(), "foo", "baz");
        String response = http.putJson(DEFAULT_HOST_URL + "/examples/" + saved.getId()+1, new Gson().toJson(update), 404);
        assertEquals("Example with id " + saved.getId()+1 +" does not exist.", response);
    }


> For those that have found functional testing to have become "Cucumbersome”, seeing an approach for end-to-end testing using nothing more than JUnit may be interesting.

![](/assets/images/sparkler_presentation/sparkler.013.jpeg#sparkler-presentation-img "Cucumbersome")

> You can also use TestSuite if you need to manage the database for tests or any other purpose.

    public class FunctionalTestSuite {
      protected static final String HOST = "localhost";
      protected static final int PORT = 4551;
      protected static final String DEFAULT_HOST_URL = String.format("http://%s:%d", HOST, PORT);
      private static CloseableHttpClient httpClient = HttpClientBuilder.create().build();
      protected static HttpTestUtils http = new HttpTestUtils(httpClient);
    
      // NB: Composing rather than inheriting here to make BeforeClass behavior
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

> Migrations are as simple as writing SQL with flyway.

    public class Migrate {
    
      public Migrate(String url) throws Exception {
        Map<String, String> params = DatabaseUrl.params(url);
        String dataSourceUrl = JdbcUrl.build(params);
        Flyway flyway = new Flyway();
        flyway.setDataSource(dataSourceUrl, params.get("user"), params.get("password"));
        flyway.migrate();
      }
    
      public static void main(String[] args) throws Exception {
        String url = System.getenv("JDBC_DATABASE_URL");
        new Migrate(url);
      }
    }

> Simple SQL!

    create table EXAMPLES (
      ID bigserial primary key,
      E_NAME varchar(255) not null,
      E_TYPE varchar(255)
    );


> Server is the entry point to the application, where I injected the details that the application is dependent on to run.

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


> Now you have a very simple, custom made framework that fits your business needs without having to learn or use all the extra baggage that comes with other frameworks "out of the box."

![](/assets/images/sparkler_presentation/sparkler.018.jpeg#sparkler-presentation-img "GOT IT!")

> Is Sparkler going to be a framework? NO! In fact, I am asking you to stay away from big frameworks; However, Sparkler can help you as a reference app, something for you to get started, and you can add or remove functionalities to make it yours.

![](/assets/images/sparkler_presentation/sparkler.019.jpeg#sparkler-presentation-img "Framework?")


> Here you go! Take it, use it, give me feedback... But most importantly, keep in mind that for the project that you are going to work on for months and maybe even years, it does worth to put the time ahead and create something that you really need. There is no such a thing as "One Framework fits All Projects!"

![](/assets/images/sparkler_presentation/sparkler.020.jpeg#sparkler-presentation-img "All done")

> Stay in touch and feel free to contact me if you have any questions!

![](/assets/images/sparkler_presentation/sparkler.021.jpeg#sparkler-presentation-img "Contact Info")

[Read more about Spark and Sparkler.](http://sarahaslanifar.com/java/2016/04/12/sparkler.html)