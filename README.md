 
Hi, Spring fans! Welcome to another installment of Spring Tips! In this installment, we'll revisit two topics that we've addressed in two previous videos (distributed tracing and metrics collection) in terms of the superb Tanzu [Wavefront observability](https://www.wavefront.com/) platform. 

The first video of the two videos, as mentioned above, dating way back in early 2017, looked at distributed tracing [with spring cloud sleuth and openzipkin](https://spring.io/blog/2017/02/08/spring-tips-distributed-tracing-with-zipkin). Spring Cloud Sleuth is an abstraction for capturing the flow of messages from one node to another. It's useful to help you see how messages move through a system. Spring cloud sleuth integrates with all the usual ingress and egress points in a Spring Boot application. Make an HTTP request using either the `Restteplat` or the reactive `WebClient` or Spring Cloud Feign? It works. Receive an HTTP request to a traditional (Servlet-based) or reactive HTTP endpoint built with Spring? It works. Send or receive a message using Spring Cloud Stream or Spring Integration? Yep. You guessed it. It just works. You don't have to do anything, either.

Just add the Spring Cloud Sleuth starter to the classpath, and Spring Cloud Sleuth does the rest. It can, in turn, forward the information that it captures to an out-of-band trace server like [OpenZipkin](https://zipkin.io/). Some even provide Zipkin-compatible proxies, like [Google Cloud StackDriver trace](https://cloud.google.com/trace/docs/zipkin). I like that last bit as it almost feels like we've gone full circle. Remember that Zipkin was inspired by, among other things, google 's dapper whitepaper. It's nice that we can, in turn, use Zipkin to talk to google cloud's tracing infrastructure. Distributed tracing is ideal when you want to drill down into the specifics of a single request into, or out of, the system.  

The second video from early 2018 looks at [collecting metrics with Micrometer](https://spring.io/blog/2018/05/02/spring-tips-metrics-collection-in-spring-boot-2-with-micrometer). 

[Micrometer](http://micrometer.io/) is an abstraction that captures metrics - statistics - about your application that can then be made available from the spring boot actuator `/actuator/metrics` endpoint or forwarded to a time series database like Wavefront. Micrometer can also talk to other time-series databases like AppOptics, Azure Monitor, Netflix Atlas, CloudWatch, Datadog, Dynatrace, Elastic, Ganglia, Graphite, Humio, Influx/Telegraf, JMX, KairosDB, New Relic, Prometheus, SignalFx, Google Stackdriver, StatsD. Metrics are useful when you want to capture statistics - aggregate numbers about the frequency a given HTTP endpoint is hit, or an HTTP endpoint returns a particular status code or responds to a specific HTTP verb. 

That was a quick recap. For a more in-depth look at both of these, I would refer you to the two referenced Spring Tips videos. That's why they're there! The beautiful thing about Wavefront is that you won't interact with these project's APIs in this installment because, as we're about to see, the Wavefront Spring Boot integration _jsut works_. And this is no small surprise: the Spring Boot team worked hard on making the Wavefront integration work seamlessly. Wavefront has other integrations for _tons_ of different platforms, runtimes, and projects, too. 

NOTE: I've pasted together the seemingly endless list of other integrations supported into one ginormous screenshot that you can find at the bottom of this blog. It'll be the image that's like ten times longer in length than the blog itself. 


Let's build a new project using the Spring Initializr. You must use a build of Spring Boot greater than Spring Boot 2.3.M4. Select `R2DBC`, `PostgreSQL`, `Reactive Web`, `Spring Cloud Sleuth` and `Lombok`. Click `Generate,` unzip the resulting file. When I wrote this, it worked well enough to use Spring Cloud Hoxton SR3. No need for the snapshots. Also, we need to add the Wavefront starter itself. Add the following dependency to the `pom.xml.` 

```xml
    <dependency>
        <groupId>com.wavefront</groupId>
        <artifactId>wavefront-spring-boot-starter</artifactId>
        <version>2.0.0-SNAPSHOT</version>
    </dependency>
```

The Wavefront Spring Boot starter is a `SNAPSHOT` dependency - it wouldn't be a Spring Tips video if we didn't dare to dance on the edge!  At the time of this writing, the dependency is in the Sonatype SNAPSHOTS repository. I needed to have the following Spring and Sonatype repositories in my build. Who knows, though? Perhaps everything will be GA by the time you read this. Things move quickly in these bootiful parts! 


```xml
    <repositories>
        <repository>
            <id>sonatype-snapshots</id>
            <name>Sonatype Snapshots</name>
            <url>https://oss.sonatype.org/content/repositories/snapshots/</url>
            <snapshots>
                <enabled>true</enabled>
            </snapshots>
        </repository>
        <repository>
            <id>spring-milestones</id>
            <name>Spring Milestones</name>
            <url>https://repo.spring.io/milestone</url>
        </repository>
        <repository>
            <id>spring-snapshots</id>
            <name>Spring Snapshots</name>
            <url>https://repo.spring.io/snapshot</url>
            <snapshots>
                <enabled>true</enabled>
            </snapshots>
        </repository>
    </repositories>
    <pluginRepositories>
        <pluginRepository>
            <id>spring-milestones</id>
            <name>Spring Milestones</name>
            <url>https://repo.spring.io/milestone</url>
        </pluginRepository>
        <pluginRepository>
            <id>spring-snapshots</id>
            <name>Spring Snapshots</name>
            <url>https://repo.spring.io/snapshot</url>
            <snapshots>
                <enabled>true</enabled>
            </snapshots>
        </pluginRepository>
    </pluginRepositories>
```

That all done, import the project up in your favorite IDE. 

The application is a typical reactive Spring Boot application, with which you have no doubt become familiar after watching so many other SPring Tips installments [on functional reactive HTTP endpints](https://spring.io/blog/2016/10/05/spring-tips-functional-reactive-endpoints-with-spring-framework-5-0),   [Spring Data R2DBC](https://spring.io/blog/2018/12/19/spring-tips-reactive-sql-data-access-with-spring-data-r2dbc), [testing reactive services](https://spring.io/blog/2019/02/06/spring-tips-testing-reactive-code), and more. Here's the code. 


```java
package com.example.wavefront;

import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;
import org.springframework.boot.ApplicationRunner;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.annotation.Bean;
import org.springframework.data.annotation.Id;
import org.springframework.data.repository.reactive.ReactiveCrudRepository;
import org.springframework.web.reactive.function.server.RouterFunction;
import org.springframework.web.reactive.function.server.ServerResponse;
import reactor.core.publisher.Flux;

import static org.springframework.web.reactive.function.server.RouterFunctions.route;
import static org.springframework.web.reactive.function.server.ServerResponse.ok;

@SpringBootApplication
public class WavefrontApplication {

    public static void main(String[] args) {
        SpringApplication.run(WavefrontApplication.class, args);
    }

    @Bean
    RouterFunction<ServerResponse> routes(ReservationRepository rr) {
        return route()
                .GET("/reservations", r -> ok().body(rr.findAll(), Reservation.class))
                .GET("/hello", r -> ok().bodyValue("Hi, Spring fans!"))
                .build();

    }

    @Bean
    ApplicationRunner runner(ReservationRepository reservationRepository) {
        return args -> {

            var data = Flux
                    .just("A", "B", "C", "D")
                    .map(name -> new Reservation(null, name))
                    .flatMap(reservationRepository::save);

            reservationRepository
                    .deleteAll()
                    .thenMany(data)
                    .thenMany(reservationRepository.findAll())
                    .subscribe(System.out::println);
        };
    }
}

interface ReservationRepository extends ReactiveCrudRepository<Reservation, String> {
}

@Data
@AllArgsConstructor
@NoArgsConstructor
class Reservation {

    @Id
    private String id;
    private String name;
}

```

We'll need to speicfy some configuration information for the PostgreSQL database in our `src/main/resources/application.properties`. 

```properties
spring.r2dbc.url=r2dbc:postgresql://localhost/orders
spring.r2dbc.username=orders
spring.r2dbc.password=orders
```

We'll also need to specify how our application identifies itself to Wavefront in `src/main/resources/application.properties.` Wavefront can observe multiple applications. An application, in turn, can have numerous services within it. An application name is a logical grouping of services. For our application, the name will be `spring-tips`, and the service name will be `reservations`. 

```properties
wavefront.application.name=spring-tips
wavefront.application.service=reservations
```


Aaaaannd... that's it! Start the application in your IDE or on the command line using `mvn spring-boot:run`. You'll see that the application starts up and displays some information on the console, sort of like this. 

```shell
...

To share this account, make sure the following is added to your configuration:

    management.metrics.export.wavefront.api-token=58e749b5-ee60-4c0b-988e-458c6cb77b32
    management.metrics.export.wavefront.uri=https://wavefront.surf

Connect to your Wavefront dashboard using this one-time use link:
https://wavefront.surf/us/cY69hp561D

...

```

Yep. That's it. When the Wavefront Spring Boot starter starts up, it negotiates a token for your application. So your dashboard is already ready and waiting for you by the time the application starts up. The spring boot starter is even kind enough to capture the Wavefront token for you in `~/.wavefront_token.` Subsequent runs will read that data for you. Or, you could use the Spring Boot properties printed out on the console, and it'll defer to that, too. 

> you probably don't want to have the token checked into your git repository. Instead, prefer environment variables or the Spring Cloud Config Server. Want to learn more about configuration? See [this    SPring tips video](https://spring.io/blog/2020/04/23/spring-tips-configuration) on configuration.

Note the last URL there for reference later on. But first, go to your browser and exercise some of the endpoints, [http://localhost:8080/reservations](http://localhost:8080/reservations) and [http://localhost:8080/hello](http://localhost:8080/hello). Invoke them both in the browser a half dozen times. 

Now, go to that wavefront URL, and you should see a dashboard with the data for your application. If you don't, then just wait a minute. IT takes a few minutes to see the data collected from your application. 

You'll immediately see that Wavefront has captured metrics about your application at both the application level - it knows, for example, about your HTTP requests, as well as at the JVM level, where it knows about things like your JVM's memory use. 

<img src ="https://raw.githubusercontent.com/spring-tips/wavefront/master/assets/wavefront-metrics.png" />

Click on one of those HTTP requests or go to `Application` > `Traces`, and you'll be able to drill down into the traces for your application, like this. There is a _ton_ of information there, too. 

<img src ="https://raw.githubusercontent.com/spring-tips/wavefront/master/assets/wavefront-trace.png" />

Everything I've shown you thus far is just what gets prepared for you out of the box. The real power is in the customization and the integrations. You can create custom dashboards to show all sorts of information that matters to you. Graph useful business indicators. Use plots, time series, and more. And, once you've figured out what the correct data to act on is, and figured out how to best contextualize data, then it's trivial to install integrations to alert you when its time to act. 

Speaking of integrations, I promised I'd show you some of the other integrations available. So, here it is, your moment of zen: the _insane_ list of _other_ integrations for the rare sort who is _not_ Spring Boot (huh?):

<img src = "https://raw.githubusercontent.com/spring-tips/wavefront/master/assets/integrations.png"/>

Still here? ...Why? Get! You've got places to be, production deploys to savor, growth to measure. Take what you've learned here today and integrate Wavefront into your application. If you're using [Tanzu Enterprise-ready Kubernetes Grid (Kubernetes)](https://tanzu.vmware.com/kubernetes-grid), [Tanzu Application Service (Cloud Foundry)](https://tanzu.vmware.com/application-service), or [Azure Spring Cloud](https://azure.microsoft.com/en-us/services/spring-cloud/), then this should be an especially tantalizing opportunity. Finally, an observability platform that can scale like your Spring Boot-based microservices do. So, go. Go try out Wavefront. Get to production, faster, and safer.
 