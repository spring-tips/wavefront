 # Spring Tips: Wavefront


Hi, Spring fans! Welcome to another installment of Spring Tips! In this installment we'll reivsit two topics that we've adfressed in two previous videos (disteivuted tracng and metrics collection) in terms of the absolutely phenomenal [Wavefront product](https://www.wavefront.com/). 

The first video of the aforementioned two videos, dating waaay back in early 2017, looked at distributed tracing [with spring cloud sleuth and openzipkin](https://spring.io/blog/2017/02/08/spring-tips-distributed-tracing-with-zipkin). Spring cloud Sleuth is an abstraction for capturing the flow of messages from one node to another. its useful to help you see how messages move through a system. Spring cloud sleuth integrates with all the usual ingress and egrees point sin an Spring Boot application. Make an http request using either the `Restteplat` or the reactive `WebClient` or Spring Cloud Feign? it works. Receive an HTTP request to a traditional (Servelt-based) or reactive HTTP endpoint built with Spring? it wors. Send or receive a message using Spring Cloud Stream or Spring INtegration? Yep. You guessed it. It jsut worjs. You dont have to do anything, either. Just add it to the classpath and Spring CLoud Slruth does the rest. It can in turn forward the information that it captures to an out-of-band trace server like [OpenZipkin](https://zipkin.io/). Some evn provide Zipkin-compatiable proxies, like [Google Cloud StackDriver trace](https://cloud.google.com/trace/docs/zipkin). I liek that last bit as it almost fels like we've gone full circule. remember that zipkin was inpsired by among other things google 's dapper whitepaper. it's nice that we can in turn use zipkin to talk to google cloud's tracing infrastructure. Distributed tracing is ideal when  you want ot drill down into the specifics of a single request into, or out of, the system.  

The second video, harking from early 2018, looks at [collecting metrics with micrometer](https://spring.io/blog/2018/05/02/spring-tips-metrics-collection-in-spring-boot-2-with-micrometer). [Micrometer](http://micrometer.io/) is an abstraction that captures metrics - statistics - about your application that can then be made available from the spring boot actuator `/actuator/metrics` endpoint or forwarded to a timeseris database like Wavefront. Micrometer can also talk to other time series databases like AppOptics, Azure Monitor, Netflix Atlas, CloudWatch, Datadog, Dynatrace, Elastic, Ganglia, Graphite, Humio, Influx/Telegraf, JMX, KairosDB, New Relic, Prometheus, SignalFx, Google Stackdriver, StatsD. Meyrics are useful when you want to capture statistics - aggregate numbers abou the frquency a gvien http endpoint is hit, or an http endpoint returns a particualr status code or responds to a particualr http verb. 

That was a quick recap. For a more indepth look at both of these, i woudl refer you to the other Spring Tips videos. That's what they'r there for! The nice thing about Wavefront is that you wont interact with these project's APIs in this instalment because, as were about to see, the Wavefront Spring Boot interation _jsut works_. And thi sis no small surprise: the Spring Boot team worked hard on makign the Wavefront integration work seamlessly. Wavefront has other integrations for tons of ther platforms and integrations, too.

<img src = "https://raw.githubusercontent.com/spring-tips/wavefront/master/assets/integrations.png"/>



<!-- 
* goto https://www.wavefront.com/sign-up/ to sign up 
* https://longboard.wavefront.com/getstarted
* OLD https://www.wavefront.com/how-to-instrument-and-monitor-your-springboot-2-application-on-kubernetes-using-wavefront/
* https://www.wavefront.com/
* git clone https://github.com/spring-projects/spring-petclinic/
cd spring-petclinic
git checkout wavefront
mvn spring-boot:run
* this is the commit that converted a non Wavefront-aware application to a Wavefront-aware application: https://github.com/spring-projects/spring-petclinic/commit/4fdd830c0b328f885df336e1c92c2997a22ee13f
* https://wavefront.surf/tracing/service/details#_v01(g:(d:7200,ls:!t,s:1588067030,w:'2h'),p:(application:(v:spring-demos),service:(v:spring-petclinic)))

You can 2.3.SNAPSHOT and <spring-cloud.version>Hoxton.SR3</spring-cloud.version>

* `wavefront.application.name=spring-tips`
* `wavefront.application.service=reservations`

https://spring.io/blog/2018/05/02/spring-tips-metrics-collection-in-spring-boot-2-with-micrometer
https://spring.io/blog/2017/02/08/spring-tips-distributed-tracing-with-zipkin

      <dependency>
            <groupId>com.wavefront</groupId>
            <artifactId>wavefront-spring-boot-starter</artifactId>
            <version>2.0.0-SNAPSHOT</version>
        </dependency>


      <repository>
            <id>sonatype-snapshots</id>
            <name>Sonatype Snapshots</name>
            <url>https://oss.sonatype.org/content/repositories/snapshots/</url>
            <snapshots>
                <enabled>true</enabled>
            </snapshots>
        </repository> -->