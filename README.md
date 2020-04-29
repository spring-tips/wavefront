 # Spring Tips: Wavefront


Hi, Spring fans! Welcome to another installment of Spring Tips! In this installment we'll reivsit two topics that we've adfressed in two previous videos (disteivuted tracng and metrics collection) in terms of the absolutely phenomenal [Wavefront product](https://www.wavefront.com/). 

The first video of the aforementioned two videos, dating waaay back in early 2017, looked at distributed tracing [with spring cloud sleuth and openzipkin](https://spring.io/blog/2017/02/08/spring-tips-distributed-tracing-with-zipkin). Spring cloud Sleuth is an abstraction for capturing the flow of messages from one node to another. its useful to help you see how messages move through a system. Spring cloud sleuth integrates with all the usual ingress and egrees point sin an Spring Boot application. Make an http request using either the `Restteplat` or the reactive `WebClient` or Spring Cloud Feign? it works. Receive an HTTP request to a traditional (Servelt-based) or reactive HTTP endpoint built with Spring? it wors. Send or receive a message using Spring Cloud Stream or Spring INtegration? Yep. You guessed it. It jsut worjs. You dont have to do anything, either. Just add it to the classpath and Spring CLoud Slruth does the rest. It can in turn forward the information that it captures to an out-of-band trace server like [OpenZipkin](https://zipkin.io/). Some evn provide Zipkin-compatiable proxies, like [Google Cloud StackDriver trace](https://cloud.google.com/trace/docs/zipkin). I liek that last bit as it almost fels like we've gone full circule. remember that zipkin was inpsired by among other things google 's dapper whitepaper. it's nice that we can in turn use zipkin to talk to google cloud's tracing infrastructure. Distributed tracing is ideal when  you want ot drill down into the specifics of a single request into, or out of, the system.  

The second video, harking from early 2018, looks at [collecting metrics with micrometer](https://spring.io/blog/2018/05/02/spring-tips-metrics-collection-in-spring-boot-2-with-micrometer). [Micrometer](http://micrometer.io/) is an abstraction that captures metrics - statistics - about your application that can then be made available from the spring boot actuator `/actuator/metrics` endpoint or forwarded to a timeseris database like Wavefront. Micrometer can also talk to other time series databases like AppOptics, Azure Monitor, Netflix Atlas, CloudWatch, Datadog, Dynatrace, Elastic, Ganglia, Graphite, Humio, Influx/Telegraf, JMX, KairosDB, New Relic, Prometheus, SignalFx, Google Stackdriver, StatsD. Meyrics are useful when you want to capture statistics - aggregate numbers abou the frquency a gvien http endpoint is hit, or an http endpoint returns a particualr status code or responds to a particualr http verb. 

That was a quick recap. For a more indepth look at both of these, i woudl refer you to the two referenced Spring Tips videos. That's what they'r there for! The nice thing about Wavefront is that you wont interact with these project's APIs in this instalment because, as were about to see, the Wavefront Spring Boot interation _jsut works_. And thi sis no small surprise: the Spring Boot team worked hard on makign the Wavefront integration work seamlessly. Wavefront has other integrations for _tons_ of ther platforms and integrations, too. 

NOTE: I've pasted together the seemingly endless list of other integrations supported into one ginormous screnshot that you can find at the bottom of this blog. It'll be the image that's like ten times longer in length than the blog itself. 


Let's build a new project usin th SPring iNitialzr. It's important that you use a build of Spring Boot greater than Spring Boot 2.3.M4. Select `R2DBC`, `PostgreSQL`, `Reactive Web`, `Spring Cloud Sleuth` and `Lombok`. Click `Generate`, unzip the resulting file. When I wrote this, it worked well enough to use Spring Cloud Hoxton SR3. No need fo rhte snapshots. Also, we ned to add the Wavefront starter itself. Add the follwoin depennecy to the `pom.xml`. 

```xml
    <dependency>
        <groupId>com.wavefront</groupId>
        <artifactId>wavefront-spring-boot-starter</artifactId>
        <version>2.0.0-SNAPSHOT</version>
    </dependency>
```

This is a `SNAPSHOT` depenency - it wouldn't be a Spring Tips video if we didn't dare to dance on the edge!  At the time of this writing, the depenency is in the Sonatype SNAPSHOTS repository. I needed to have the following Spring and Sonatype repositories in my build. Who knows though? Perhaps everything will be GA by the time you read this. Things move quickly in these bootiful parts! 


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

The application is a typical reactive Spring Boot application, with which you have no doubt become familiar after watching so many other SPring Tips installments [on fucntional reactive HTTP endpints](https://spring.io/blog/2016/10/05/spring-tips-functional-reactive-endpoints-with-spring-framework-5-0),   [Spring Data R2DBC](https://spring.io/blog/2018/12/19/spring-tips-reactive-sql-data-access-with-spring-data-r2dbc), [testing reactive services](https://spring.io/blog/2019/02/06/spring-tips-testing-reactive-code), and more. Here's the code. 


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

We'll also need to specify how our appkicatino identifies itself to Wavefront, also in `src/main/resources/application.properties`. Wavefront can observe multiple applications. An appllication in turn can have multiple services within it. An application name is a logical grouping of services. For our application, the  application name will be `spring-tips`, and the service name will be `reservations`. 

```properties
wavefront.application.name=spring-tips
wavefront.application.service=reservations
```


Aaaaannd... that's it! Start the application in your IDE or on the comnand ine using `mvn spring-boot:run`. You'll see that the application starts up and displays some information on the console, sort of liek this. 

```shell
...
Your existing Wavefront account information has been restored from disk.

To share this account, make sure the following is added to your configuration:

	management.metrics.export.wavefront.api-token=58e749b5-ee60-4c0b-988e-458c6cb77b32
	management.metrics.export.wavefront.uri=https://wavefront.surf

Connect to your Wavefront dashboard using this one-time use link:
https://wavefront.surf/us/cY69hp561D

...

```

Yep. That's it. IT's done. When the Spring Boot starter starts up, it negotiates a token for your application. So your dashboard is already ready and waiting for you by the time the applicatin starts up. The spring boot starter is even nice enough to catpture the Wavefront token for you in `~/.wavefront_token`. Subsequent runs will read that data for you. Or, you could use the SPring Boot properties printed out on the console and itll defer to that, too. 

> you probably don't want to have the token checked into your git repositor. Instead, prefer environment variables or the Spring Cloud Cnofig Srrver. Want to learn more about configuration? See [this    SPring tips video](https://spring.io/blog/2020/04/23/spring-tips-configuration) on configuration.

Note that last uRL there for reference later on. But first, go to your browser and exercise some of the endpoints, [http://localhost:8080/reservations](http://localhost:8080/reservations) and [http://localhost:8080/hello](http://localhost:8080/hello). Invoke them both in the browser a half dozen times. 

Now, go to that wavefront URL and you should see a dashboard with the data for your application. If you don't, then just wait a minute. IT takes a few minutes to see the data collected from yur application. 


You'll immediately see that Wavefront has captured metrics about your application at bth the application level - it knows, for example, about your HTTP requets, as well as at the JVM level, where it knows about things like your jVM's memory use. 

<img src ="https://raw.githubusercontent.com/spring-tips/wavefront/master/assets/wavefront-metrics.png" />

Click on one of those HTTP requests or go to Application > Traces and you'll be able to drill down into the traces for your application, like this. There is a _ton_ of information there, too. 

<img src ="https://raw.githubusercontent.com/spring-tips/wavefront/master/assets/wavefront-trace.png" />

Everything i've shown you thus far is jsut what gets prepared for you out of the box. the real power is in the customization and the integratins. You can created custom dashboards to show all sorts of informatino that matters to you. Graph useful business indicators. Use plots, time series, and more. And, once you've figured out what the right data to act on is, and figured out how to best contextualize data, then it's trivial to install inegrations to alert you when its time to act. 

Speaking of integrations, I promised i'd show you some of the other integrations avaialble. So, here it is, your moment of zen: the _insane_ list of _other_ integrations for the rare sort not Spring Boot:

<img src = "https://raw.githubusercontent.com/spring-tips/wavefront/master/assets/integrations.png"/>


 