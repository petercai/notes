# Enable and Disable Endpoints at Runtime With Spring Boot | Baeldung
1\. Overview[](#overview)
-------------------------

Endpoints in a Spring Boot application are the mechanisms to interact with it. At times such as during an unplanned maintenance window, we might want to temporarily limit the application's interactions with the outside.

In this tutorial, **we'll learn to enable and disable endpoints at runtime in a Spring Boot application** using a few popular libraries, such as [Spring Cloud](https://www.baeldung.com/spring-cloud-series), [Spring Actuator](https://www.baeldung.com/spring-boot-actuators), and [Apache's Commons Configuration](https://commons.apache.org/proper/commons-configuration/).

2\. Setup[](#setup)
-------------------

In this section, let's focus on setting up critical aspects for our Spring Boot project.

### 2.1. Maven Dependencies[](#maven-dependencies)

First, we'll need our Spring Boot application to **expose the _/refresh_ endpoint**,  so let's add the _[spring-boot-starter-actuator](https://search.maven.org/search?q=g:%20org.springframework.boot%20AND%20a:spring-boot-starter-actuator)_ dependency in the project's _pom.xml_ file:

```null
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
    <version>2.7.5</version>
</dependency>
```

Next, since we'll need the _@RefreshScope_ annotation later for reloading the property sources in the environment, let's add the _[spring-cloud-starter](https://search.maven.org/search?q=g:%20org.springframework.cloud%20AND%20a:spring-cloud-starter)_ dependency:

```null
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter</artifactId>
    <version>3.1.5</version>
</dependency>
```

Further, we must also **add the [BOM](https://www.baeldung.com/spring-maven-bom#2-what-is-maven-bom) for [Spring Cloud](https://search.maven.org/search?q=g:%20org.springframework.cloud%20AND%20a:spring-cloud-dependencies) in the dependency management section of the project's _pom.xml_** file so that Maven uses a compatible version of _spring-cloud-starter_:

```null
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-dependencies</artifactId>
            <version>2021.0.5</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
```

Finally, as we'll need the capability to reload a file at runtime, let's also add the _[commons-configuration](https://search.maven.org/search?q=g:%20commons-configuration%20AND%20a:commons-configuration)_ dependency:

```null
<dependency>
    <groupId>commons-configuration</groupId>
    <artifactId>commons-configuration</artifactId>
    <version>1.10</version>
</dependency>
```

### 2.2. Configuration[](#configuration)

First, let's add the configuration to the _application.properties_ file to **enable the _/refresh_ endpoint** in our application:

```null
management.server.port=8081
management.endpoints.web.exposure.include=refresh
```

Next, let's define an additional source that we can use to reload properties:

```null
dynamic.endpoint.config.location=file:extra.properties
```

Additionally, let's define the _spring.properties.refreshDelay_ property in the _application.properties_ file:

```null
spring.properties.refreshDelay=1
```

Finally, let's add two properties to the _extra.properties_ file:

```null
endpoint.foo=false
endpoint.regex=.*
```

In the later sections, we'll understand the core significance of these additional properties.

### 2.3. API Endpoints[](#api-endpoints)

First, let's define a sample **_GET_ API available at _/foo_ path**:

```null
@GetMapping("/foo")
public String fooHandler() {
    return "foo";
}
```

Next, let's define two more **_GET_ APIs available at the _/bar1_ and _/bar2_ paths**, respectively:

```null
@GetMapping("/bar1")
public String bar1Handler() {
    return "bar1";
}

@GetMapping("/bar2")
public String bar2Handler() {
    return "bar2";
}
```

In the following sections, we'll learn how to toggle an individual endpoint, such as _/foo_. Moreover, we'll also see how to toggle a group of endpoints, namely _/bar1_ and _/bar2_, identifiable through a simple regex.

### 2.4. Configuring _DynamicEndpointFilter_[](#configuring-dynamicendpointfilter)

To toggle a group of endpoints at runtime, we can use a _[Filter](https://www.baeldung.com/spring-boot-add-filter)_. By matching the request's endpoint using the _endpoint.regex_ pattern, we can allow it on success or send the _503_ HTTP response status for an unsuccessful match.

So, let's **define the _DynamicEndpointFilter_ class by extending the** _**OncePerRequestFilter**_:

```null
public class DynamicEndpointFilter extends OncePerRequestFilter {
    private Environment environment;

    
}
```

Further, we need to add the logic of pattern matching by overriding the _doFilterInternal()_ method:

```null
@Override
protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, 
  FilterChain filterChain) throws ServletException, IOException {
    String path = request.getRequestURI();
    String regex = this.environment.getProperty("endpoint.regex");
    Pattern pattern = Pattern.compile(regex);
    Matcher matcher = pattern.matcher(path);
    boolean matches = matcher.matches();

    if (!matches) {
        response.sendError(HttpStatus.SERVICE_UNAVAILABLE.value(), "Service is unavailable");
    } else {
        filterChain.doFilter(request,response);
    }
}
```

We must note the initial value of the _endpoint.regex_ property is “_.\*_” that allows all the requests through this _Filter._

3\. Toggle Using Environment Properties[](#toggle-using-env-props)
------------------------------------------------------------------

In this section, we'll learn how to do a hot reload of the environment properties from the _extra.properties_ file.

### 3.1. Reloading Configuration[](#reloading-configuration)

For this, let's start by defining a bean for _PropertiesConfiguration_ using the _FileChangedReloadingStrategy_:

```null
@Bean
@ConditionalOnProperty(name = "dynamic.endpoint.config.location", matchIfMissing = false)
public PropertiesConfiguration propertiesConfiguration(
  @Value("${dynamic.endpoint.config.location}") String path,
  @Value("${spring.properties.refreshDelay}") long refreshDelay) throws Exception {
    String filePath = path.substring("file:".length());
    PropertiesConfiguration configuration = new PropertiesConfiguration(
      new File(filePath).getCanonicalPath());
    FileChangedReloadingStrategy fileChangedReloadingStrategy = new FileChangedReloadingStrategy();
    fileChangedReloadingStrategy.setRefreshDelay(refreshDelay);
    configuration.setReloadingStrategy(fileChangedReloadingStrategy);
    return configuration;
}
```

We must note that the **source of the properties is derived using the _dynamic.endpoint.config.location_ property** in the _application.properties_ file. Additionally, the reload happens with a time delay of _1_ second, as defined by the _spring.properties.refreshDelay_ property.

Next, we need to read the endpoint-specific properties at runtime. So, let's define the _EnvironmentConfigBean_ with property-getters:

```null
@Component
public class EnvironmentConfigBean {

    private final Environment environment;

    public EnvironmentConfigBean(@Autowired Environment environment) {
        this.environment = environment;
    }

    public String getEndpointRegex() {
        return environment.getProperty("endpoint.regex");
    }

    public boolean isFooEndpointEnabled() {
        return Boolean.parseBoolean(environment.getProperty("endpoint.foo"));
    }

    public Environment getEnvironment() {
        return environment;
    }
}

```

Next, let's **create a _FilterRegistrationBean_ to register the _DynamicEndpointFilter_**:

```null
@Bean
@ConditionalOnBean(EnvironmentConfigBean.class)
public FilterRegistrationBean<DynamicEndpointFilter> dynamicEndpointFilterFilterRegistrationBean(
  EnvironmentConfigBean environmentConfigBean) {
    FilterRegistrationBean<DynamicEndpointFilter> registrationBean = new FilterRegistrationBean<>();
    registrationBean.setFilter(new DynamicEndpointFilter(environmentConfigBean.getEnvironment()));
    registrationBean.addUrlPatterns("*");
    return registrationBean;
}
```

### 3.2. Verification[](#verification)

First, let's run the application and access the _/bar1_ or _/bar2_ APIs:

```null
$ curl -iXGET http://localhost:9090/bar1
HTTP/1.1 200 
Content-Type: text/plain;charset=ISO-8859-1
Content-Length: 4
Date: Sat, 12 Nov 2022 12:46:32 GMT

bar1
```

As expected, **we get the _200 OK_ HTTP response as we have kept the initial value of the _endpoint.regex_ property to enable all the endpoints**.

Next, let's enable only the _/foo_ endpoint by changing the _endpoint.regex_ property in the _extra.properties_ file:

```null
endpoint.regex=.*/foo
```

Moving on, let's see if we're able to access the _/bar1_ API endpoint:

```null
$ curl -iXGET http://localhost:9090/bar1
HTTP/1.1 503 
Content-Type: application/json
Transfer-Encoding: chunked
Date: Sat, 12 Nov 2022 12:56:12 GMT
Connection: close

{"timestamp":1668257772354,"status":503,"error":"Service Unavailable","message":"Service is unavailable","path":"/springbootapp/bar1"}
```

As expected, the _DynamicEndpointFilter_ disabled this endpoint and sent an error response with HTTP _503_ status code.

Finally, we can also check that we're able to access the _/foo_ API endpoint:

```null
$ curl -iXGET http://localhost:9090/foo
HTTP/1.1 200 
Content-Type: text/plain;charset=ISO-8859-1
Content-Length: 3
Date: Sat, 12 Nov 2022 12:57:39 GMT

foo
```

Perfect! It looks like we've got this right.

4\. Toggle Using Spring Cloud and Actuator[](#toggle-using-cloud-actuator)
--------------------------------------------------------------------------

In this section, we'll learn an alternative approach of using the _@RefreshScope_ annotation and the actuator _/refresh_ endpoint to toggle the API endpoints at runtime.

### 4.1. Endpoint Configuration With _@RefreshScope_[](#endpoint-config-refreshscope)

First, we need to **define the configuration bean for toggling the endpoint and annotate it with _@RefreshScope_**:

```null
@Component
@RefreshScope
public class EndpointRefreshConfigBean {

    private boolean foo;
    private String regex;

    public EndpointRefreshConfigBean(@Value("${endpoint.foo}") boolean foo, 
      @Value("${endpoint.regex}") String regex) {
        this.foo = foo;
        this.regex = regex;
    }
    
}
```

Next, we need to make these properties discoverable and reloadable by creating wrapper classes such as [_ReloadableProperties_](https://www.baeldung.com/spring-reloading-properties#2-reloading-properties-instance) and _[ReloadablePropertySource](https://www.baeldung.com/spring-reloading-properties#1-reloading-environment-properties)_.

Finally, let's update our API handler to use an instance of _EndpointRefreshConfigBean_ for controlling the toggle flow:

```null
@GetMapping("/foo")
public ResponseEntity<String> fooHandler() {
    if (endpointRefreshConfigBean.isFoo()) {
        return ResponseEntity.status(200).body("foo");
    } else {
        return ResponseEntity.status(503).body("endpoint is unavailable");
    }
}
```

### 4.2. Verification[](#verification-1)

First, let's verify the _/foo_ endpoint when the value of the _endpoint.foo_ property is set to _true_:

```null
$ curl -isXGET http://localhost:9090/foo
HTTP/1.1 200
Content-Type: text/plain;charset=ISO-8859-1
Content-Length: 3
Date: Sat, 12 Nov 2022 15:28:52 GMT

foo
```

Next, let's **set the value of the _endpoint.foo_ property to _false_**_,_ and check if the endpoint is still accessible:

```null
endpoint.foo=false
```

We'll notice that the _/foo_ endpoint is still enabled. That's because we need to **reload the property sources by invoking the _/refresh_ endpoint**. So, let's do this once:

```null
$ curl -Is --request POST 'http://localhost:8081/actuator/refresh'
HTTP/1.1 200
Content-Type: application/vnd.spring-boot.actuator.v3+json
Transfer-Encoding: chunked
Date: Sat, 12 Nov 2022 15:34:24 GMT

```

Finally, let's try to access the _/foo_ endpoint:

```null
$ curl -isXGET http://localhost:9090/springbootapp/foo
HTTP/1.1 503
Content-Type: text/plain;charset=ISO-8859-1
Content-Length: 23
Date: Sat, 12 Nov 2022 15:35:26 GMT
Connection: close

endpoint is unavailable
```

We can see that the endpoint is disabled after the refresh.

### 4.3. Pros and Cons[](#pros-and-cons)

The Spring Cloud and Actuator approach has advantages and disadvantages over fetching the properties directly from the environment.

Firstly, when we rely on the _/refresh_ endpoint, we've finer control than a time-based file reloading strategy. So the application isn't making unnecessary I/O calls in the background. However, **in the case of a distributed system, we need to ensure that we're invoking the _/refresh_ endpoint for all the nodes**.

Secondly, managing a configuration bean with the _@RefreshScope_ annotation requires us to explicitly define the member variables in _EndpointRefreshConfigBean_ class to map with the property in the _extra.properties_ file. So, this approach **adds the overhead of making code changes in the configuration beans whenever we add or remove properties**.

Finally, we must also note that a script could easily resolve the first issue, and the second issue is more specific to how we're leveraging the properties. **If we're using a regex-based URL pattern with the _Filter,_ then we can control multiple endpoints with a single property** with no code changes to the configuration bean.

5\. Conclusion[](#conclusion)
-----------------------------

In this article, **we explored multiple strategies to toggle API endpoints at runtime** in a Spring Boot Application. While doing so, we leveraged some core concepts, such as [hot-reloading of properties](https://www.baeldung.com/spring-reloading-properties) and the  _@RefreshScope_ annotation.

As always, the complete source code for the tutorial is available [over on GitHub](https://github.com/eugenp/tutorials/tree/master/spring-boot-modules/spring-boot-mvc-5).

We’re finally running a Black Friday launch. All Courses are **30% off** until **next Friday:**

### **[\>> GET ACCESS NOW](https://www.baeldung.com/all-courses)**