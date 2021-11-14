# Spring Boot Admin

### Run
mvn spring-boot:run -Dspring-boot.run.profiles=local

### client configuration

#### with spring boot 1.x
1. Enable actuator endpoints for example : <br/>
enable all :
<pre>
management.endpoints.enabled-by-default=true
</pre>
enable specific : 
<pre>
management.endpoints.enabled-by-default=false
management.endpoint.info.enabled=true
</pre>
2. Add SBA server address for example spring.boot.admin.url= http://localhost:8086 <br/>

3. Reconfigure the api path for Spring Boot Admin Client 1.5.x:
application.yml
<pre>
spring.boot.admin.api-path: instances
</pre>
#### with spring boot 2.x
1. Expose endpoints <br/>
Spring Boot 2 most of the endpoints aren’t exposed via http by default, we expose all of them. For production you should carefully choose which endpoints to expose :<br/> 
By default : health <br/>
Expose all : 
<pre>
management.endpoints.web.exposure.include=* 
</pre>
Expose specific : 
<pre>
management.endpoints.jmx.exposure.include=health,info
</pre>

NB : application developed by spring boot 2 no need to add client. because we use DiscoveryClient

### http trace 
You have to this configuration in order to view request traces :
<pre>
import org.springframework.boot.actuate.trace.http.HttpTraceRepository;
import org.springframework.boot.actuate.trace.http.InMemoryHttpTraceRepository;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class HttpTraceActuatorConfiguration {

	@Bean
	public HttpTraceRepository httpTraceRepository() {
		return new InMemoryHttpTraceRepository();
	}

}
</pre>

#### Customizing the HttpTraceRepository
You can add your custom http trace repository for example trace only get requests :
<pre>
import org.springframework.boot.actuate.trace.http.HttpTrace;
import org.springframework.boot.actuate.trace.http.HttpTraceRepository;
import org.springframework.stereotype.Repository;

import java.util.Collections;
import java.util.List;
import java.util.concurrent.atomic.AtomicReference;

@Repository
public class CustomTraceRepository implements HttpTraceRepository {

	AtomicReference<HttpTrace> lastTrace = new AtomicReference<>();

	@Override
	public List<HttpTrace> findAll() {
		return Collections.singletonList(lastTrace.get());
	}

	@Override
	public void add(HttpTrace trace) {
		if ("GET".equals(trace.getRequest().getMethod())) {
			lastTrace.set(trace);
		}
	}
}
</pre>
