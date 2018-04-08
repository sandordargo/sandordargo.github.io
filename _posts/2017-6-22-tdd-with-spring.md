---
layout: post
title: "Test driven development with Spring"
date: 2017-6-22
category: dev
tags: [java, tdd, spring]
header: "We started mob programming sessions with my new team. After a few days I already learnt a lot from a great Java dev who is in pur team."
---

Yesterday we started to sketch up a hello world REST application. Its hello service doesn't even have to return anything in its response it just has to print *hello, world!* to the logs. But how do you test that?

One of the greatest advantages of mob or pair programming is that you won't just skip some tests saying that I can't test that. You simply must.

So how do you test that you printed something to the logs?

I extended a bit what we did yesterday and upload it to [this repository](https://github.com/sandordargo/SpringTestConfigurationExample).

When you just want a unit test it is fairly easy. In Java you just inject your dependency (in this case a `PrintStream`) in the constructor. I think you'd do something similar in other languages too.

```
  @Test
  public void controllerPrintedGreetingToOutput() throws UnsupportedEncodingException {
    ByteArrayOutputStream outputStream = new ByteArrayOutputStream();
    PrintStream out = new PrintStream(outputStream);
    HelloController controller = new HelloController(out);

    controller.greet("Peter File");
    String expected = "Hello, Peter File!" + System.getProperty("line.separator");
    String actual = new String(outputStream.toByteArray(), "UTF-8");

    assertEquals(expected, actual);
  }
```

What you have to take care of is that your class under test accepts a custom `PrintStream` in its constructor, while its default output stream is `System.out`. Easy peasy, but you have multiple options.

The conventional way would be to simply use multiple constructors:

```
class HelloService {
	PrintStream outputStream;

	HelloService() {
		this(System.out);
	}

	HelloService(PrintStream outputStream) {
		this.outpuStream = outputStream;
	}

	//...
}
```

The other option is to use the features of Spring framework and declare `outputStream` with the `@Autowired` annotation.

It also means that you have to specify your configuration, otherwise you will get nice NPE's when you try to do anything with your autowired member variable.

So let's turn `BootstrapRun` from simply your main class into a configuration, by adding `@Configuration` annotation to the class and also to declare a getter for `PrintStream` with the `@Bean` annotation.

```
@SpringBootApplication
@Configuration
public class BootstrapRun {

  @Bean
  public PrintStream getPrintStream() {
    return System.out;
  }

  public static void main(String[] args) {
    SpringApplication.run(BootstrapRun.class, args);
  }
}
```

This way we don't have to pollute with multiple constructors our class in order to make it testable and to have the production behaviour at the same time. End it makes integration testing so much easier! You'll see!

Using `@Autowired` was already interesting to me, but configuring the integration test was way more enlighting.

In the integration test we wanted to instatiate our server and cell the service it exposes. In other words you don't interact directly neither with your controller nor with yout service. So you can't instatiate them with another constructor. Now it comes really handy that we used `@Autowired` for our `PrintStream` instead of having multiple constructors. Let's override the configuration!

We can do it directly in our integration test by creating a nested static class.

```
@Configuration
@Import(BootstrapRun.class)
public static class TestBootstrap {

  @Bean
  @Primary
  public PrintStream getPrintStream() {
    return new PrintStream(outputStream);
  }
}
```

There are three things to mention here.

- `@Import(BootstrapRun.class)` means that this configuration class is build based on the class passed as a parameter. It is like `extends` for a normal class.

- The second thing to mention here is the usage of `@Primary` annotation. In case you override a `@Bean` from your base config, you don't have to use `@Primary`. But as soon as in your config chain you have two functions with the same return type, Spring will complain that it doesn't know which one to use. 
In order to be explicit and clear about my intentions I prefer to use `@Primary` even if it is not mandatory.

- The last thing to mention is to say where `outputStream` comes from. As I said `TestBootstrap` is a nested class. `outputstream` is a member in the encapsulating test class so that its content can be verified easily. 


The only thing left is to use these new configurations when we start up our server. It's really easy:

```
  //...
  private static ConfigurableApplicationContext context;
  //...
  @BeforeClass
  public static void startServer() {
    context = SpringApplication.run(TestBootstrap.class);
  }

```

That's it. That's how you can validate even your logs in an integration test. You can check the whole code at [my github repository](https://github.com/sandordargo/SpringTestConfigurationExample).