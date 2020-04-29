 # Spring Tips: Wavefront

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
        </repository>