# Guide to Resilience4j With Spring Boot | Baeldung
1\. Overview[](#overview)
-------------------------

[Resilience4j](https://www.baeldung.com/resilience4j) is a lightweight fault tolerance library that provides a variety of fault tolerance and stability patterns to a web application. In this tutorial, we'll **learn how to use this library with a simple Spring Boot application**.

2\. Setup[](#setup)
-------------------

In this section, let's focus on **setting up critical aspects for our Spring Boot project**.

### 2.1. Maven Dependencies[](#maven-dependencies)

Firstly, we'll need to add the _[spring-boot-starter-web](https://search.maven.org/search?q=g:org.springframework.boot%20AND%20a:spring-boot-starter-web)_ dependency to bootstrap a simple web application:

```null
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
<dependency>
```

Next, **we'd need the [_resilience4j-spring-boot2_](https://search.maven.org/search?q=resilience4j-spring-boot2) and _[spring-boot-starter-aop](https://search.maven.org/search?q=spring-boot-starter-aop)_ dependencies to use the features from the Resilience-4j library using annotations** in our Spring Boot application:

```null
<dependency>
    <groupId>io.github.resilience4j</groupId>
    <artifactId>resilience4j-spring-boot2</artifactId>
</dependency>

<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-aop</artifactId>
</dependency>
```

Additionally, we'd also need to add the _[spring-boot-starter-actuator](https://search.maven.org/artifact/org.springframework.boot/spring-boot-starter-actuator)_ dependency to monitor the application's current state through a set of exposed endpoints:

```null
<dependency>
    <groupId>org.springframework.boot</groupId> 
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

Lastly, let's add the _[wiremock-jre8](https://search.maven.org/search?q=g:com.github.tomakehurst%20AND%20a:wiremock-jre8)_ dependency, as it'll help us in testing our REST APIs using a [mock HTTP server](https://www.baeldung.com/introduction-to-wiremock):

```null
<dependency>
    <groupId>com.github.tomakehurst</groupId>
    <artifactId>wiremock-jre8</artifactId>
    <scope>test</scope>
</dependency>
```

### 2.2. _RestController_ and External API Caller[](#)

While using different features of the Resilience4j library, our web application needs to interact with an external API. So, let's go ahead and add a bean for the _[RestTemplate](https://www.baeldung.com/rest-template) _that will help us make API calls.

```null
@Bean
public RestTemplate restTemplate() {
    return new RestTemplateBuilder().rootUri("http://localhost:9090")
      .build();
}
```

Next, let's define the _ExternalAPICaller_ class as a _Component_ and use the _restTemplate_ bean as a member:

```null
@Component
public class ExternalAPICaller {
    private final RestTemplate restTemplate;

    @Autowired
    public ExternalAPICaller(RestTemplate restTemplate) {
        this.restTemplate = restTemplate;
    }
}
```

After this, we can **define the _ResilientAppController_ class that exposes REST API endpoints and internally uses the _ExternalAPICaller_ bean to call external API**:

```null
@RestController
@RequestMapping("/api/")
public class ResilientAppController {
    private final ExternalAPICaller externalAPICaller;
}
```

### 2.3. Actuator Endpoints[](#3-actuator-endpoints)

We can **expose health endpoints via the [Spring Boot actuator](https://www.baeldung.com/spring-boot-actuators) to know the exact state of the application** at any given time.

So, let's add the configuration to the _application.properties_ file and enable the endpoints:

```null
management.endpoints.web.exposure.include=*
management.endpoint.health.show-details=always

management.health.circuitbreakers.enabled=true
management.health.ratelimiters.enabled=true
```

Additionally, as and when we need, we'll add feature-specific configuration in the same _application.properties_ file.

### 2.4. Unit Test[](#unit-test)

Our web application will call an external service in a real-world scenario. However, we can **simulate the existence of such a running service by starting an external service using the _WireMockExtension_ class**.

So, let's define _EXTERNAL\_SERVICE_ as a static member in the _ResilientAppControllerUnitTest_ class:

```null
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
class ResilientAppControllerUnitTest {

    @RegisterExtension
    static WireMockExtension EXTERNAL_SERVICE = WireMockExtension.newInstance()
      .options(WireMockConfiguration.wireMockConfig()
      .port(9090))
      .build();
```

Additionally, let's add an instance of _TestRestTemplate_ to call the APIs:

```null
@Autowired
private TestRestTemplate restTemplate;
```

### 2.5. Exception Handler[](#exception-handler)

The Resilience4j library would protect the service resources by throwing an exception depending on the fault tolerance pattern in context. However, these exceptions should translate to an HTTP response with a meaningful status code for the client.

So, let's **define the _ApiExceptionHandler_ class to hold handlers for different exceptions**:

```null
@ControllerAdvice
public class ApiExceptionHandler {
}
```

We'll add handlers in this class as we explore different fault tolerance patterns.

3\. Circuit Breaker[](#circuit-breaker)
---------------------------------------

The **[circuit breaker pattern](https://www.baeldung.com/resilience4j#circuit-breaker) protects a downstream service** by restricting the upstream service from calling the downstream service during a partial or complete downtime.

Let's start by exposing the _/api/circuit-breaker_ endpoint and adding the _@CircuitBreaker_ annotation:

```null
@GetMapping("/circuit-breaker")
@CircuitBreaker(name = "CircuitBreakerService")
public String circuitBreakerApi() {
    return externalAPICaller.callApi();
}
```

As required, we also need to define the _callApi()_ method in the _ExternalAPICaller_ class for calling an external endpoint _/api/external_:

```null
public String callApi() {
    return restTemplate.getForObject("/api/external", String.class);
}
```

Next, let's add the configuration for the circuit breaker in the _application.properties_ file:

```null
resilience4j.circuitbreaker.instances.CircuitBreakerService.failure-rate-threshold=50
resilience4j.circuitbreaker.instances.CircuitBreakerService.minimum-number-of-calls=5
resilience4j.circuitbreaker.instances.CircuitBreakerService.automatic-transition-from-open-to-half-open-enabled=true
resilience4j.circuitbreaker.instances.CircuitBreakerService.wait-duration-in-open-state=5s
resilience4j.circuitbreaker.instances.CircuitBreakerService.permitted-number-of-calls-in-half-open-state=3
resilience4j.circuitbreaker.instances.CircuitBreakerService.sliding-window-size=10
resilience4j.circuitbreaker.instances.CircuitBreakerService.sliding-window-type=count_based
```

Essentially, the configuration would allow 50% of failed calls to the service in the [closed state](https://www.baeldung.com/resilience4j#1-circuit-breakers-states-and-settings), after which it'll open the circuit and start rejecting requests with the _CallNotPermittedException_. So, it'd be a good idea to add a handler for this exception in the _ApiExceptionHandler_ class:

```null
@ExceptionHandler({CallNotPermittedException.class})
@ResponseStatus(HttpStatus.SERVICE_UNAVAILABLE)
public void handleCallNotPermittedException() {
}
```

Finally, let's test the _/api/circuit-breaker_ API endpoint by simulating a scenario of downstream service downtime using _EXTERNAL\_SERVICE:_

```null
@Test
public void testCircuitBreaker() {
    EXTERNAL_SERVICE.stubFor(WireMock.get("/api/external")
      .willReturn(serverError()));

    IntStream.rangeClosed(1, 5)
      .forEach(i -> {
          ResponseEntity response = restTemplate.getForEntity("/api/circuit-breaker", String.class);
          assertThat(response.getStatusCode()).isEqualTo(HttpStatus.INTERNAL_SERVER_ERROR);
      });

    IntStream.rangeClosed(1, 5)
      .forEach(i -> {
          ResponseEntity response = restTemplate.getForEntity("/api/circuit-breaker", String.class);
          assertThat(response.getStatusCode()).isEqualTo(HttpStatus.SERVICE_UNAVAILABLE);
      });
    
    EXTERNAL_SERVICE.verify(5, getRequestedFor(urlEqualTo("/api/external")));
}
```

We can notice that the first five calls failed as the downstream service was down. After that, the circuit switches to an open state. Hence, the subsequent five attempts are rejected with the _503_ HTTP status code without actually calling the underlying API.

4\. Retry[](#rate-limiter)
--------------------------

**The [retry pattern](https://www.baeldung.com/resilience4j#retry) provides resiliency to a system by recovering from transient issues**. Let's start by adding the _/api/retry_ API endpoint with _@Retry_ annotation:

```null
@GetMapping("/retry")
@Retry(name = "retryApi", fallbackMethod = "fallbackAfterRetry")
public String retryApi() {
    return externalAPICaller.callApi();
}
```

Further, we can **optionally supply a fallback mechanism when all the retry attempts fail**. In this case, we've provided the _fallbackAfterRetry _as a fallback method:

```null
public String fallbackAfterRetry(Exception ex) {
    return "all retries have exhausted";
}
```

Next, let's update the _application.properties_ file to add the configuration that'll govern the behavior of retries:

```null
resilience4j.retry.instances.retryApi.max-attempts=3
resilience4j.retry.instances.retryApi.wait-duration=1s
resilience4j.retry.metrics.legacy.enabled=true
resilience4j.retry.metrics.enabled=true
```

As such, we're planning to retry for a maximum of 3 attempts, each with a delay of _1s_.

Finally, let's test the retry behavior of the _/api/retry_ API endpoint:

```null
@Test
public void testRetry() {
    EXTERNAL_SERVICE.stubFor(WireMock.get("/api/external")
      .willReturn(ok()));
    ResponseEntity<String> response1 = restTemplate.getForEntity("/api/retry", String.class);
    EXTERNAL_SERVICE.verify(1, getRequestedFor(urlEqualTo("/api/external")));

    EXTERNAL_SERVICE.resetRequests();

    EXTERNAL_SERVICE.stubFor(WireMock.get("/api/external")
      .willReturn(serverError()));
    ResponseEntity<String> response2 = restTemplate.getForEntity("/api/retry", String.class);
    Assert.assertEquals(response2.getBody(), "all retries have exhausted");
    EXTERNAL_SERVICE.verify(3, getRequestedFor(urlEqualTo("/api/external")));
}
```

We can notice that in the first scenario, there were no issues, so a single attempt was sufficient. On the other hand, when there was an issue, there were three attempts, after which the API responded via the fallback mechanism.

5\. Time Limiter[](#bulkhead)
-----------------------------

We can **use the [time limiter pattern](https://www.baeldung.com/resilience4j#time-limiter) to set a threshold timeout value for async calls made to external systems**.

Let's add the _/api/time-limiter_ API endpoint that internally calls a slow API:

```null
@GetMapping("/time-limiter")
@TimeLimiter(name = "timeLimiterApi")
public CompletableFuture<String> timeLimiterApi() {
    return CompletableFuture.supplyAsync(externalAPICaller::callApiWithDelay);
}
```

Further, let's simulate the delay in the external API call by adding a sleep time in the _callApiWithDelay()_ method:

```null
public String callApiWithDelay() {
    String result = restTemplate.getForObject("/api/external", String.class);
    try {
        Thread.sleep(5000);
    } catch (InterruptedException ignore) {
    }
    return result;
}
```

Next, we need to provide the configuration for the _timeLimiterApi_ in the _application.properties_ file:

```null
resilience4j.timelimiter.metrics.enabled=true
resilience4j.timelimiter.instances.timeLimiterApi.timeout-duration=2s
resilience4j.timelimiter.instances.timeLimiterApi.cancel-running-future=true
```

We can note that the threshold value is set to 2s. After which, the Resilience4j library internally cancels the async operation with a _TimeoutException_. So, let's **add a handler for this exception in the _ApiExceptionHandler_ class to return an API response with the _408_ HTTP status code**:

```null
@ExceptionHandler({TimeoutException.class})
@ResponseStatus(HttpStatus.REQUEST_TIMEOUT)
public void handleTimeoutException() {
}
```

Finally, let's verify the configured time limiter pattern for the _/api/time-limiter_ API endpoint:

```null
@Test
public void testTimeLimiter() {
    EXTERNAL_SERVICE.stubFor(WireMock.get("/api/external").willReturn(ok()));
    ResponseEntity<String> response = restTemplate.getForEntity("/api/time-limiter", String.class);

    assertThat(response.getStatusCode()).isEqualTo(HttpStatus.REQUEST_TIMEOUT);
    EXTERNAL_SERVICE.verify(1, getRequestedFor(urlEqualTo("/api/external")));
}
```

As expected, since the downstream API call was set to take more than 5 seconds to complete, we witnessed a timeout for the API call.

6\. Bulkhead[](#timelimiter)
----------------------------

**The [bulkhead pattern](https://www.baeldung.com/resilience4j#bulkhead) limits the maximum number of concurrent calls to an external service.**

Let's start by adding the _/api/bulkhead_ API endpoint with the _@Bulkhead_ annotation:

```null
@GetMapping("/bulkhead")
@Bulkhead(name="bulkheadApi")
public String bulkheadApi() {
    return externalAPICaller.callApi();
}
```

Next, let's define the configuration in the _application.properties_ file to control the bulkhead functionality:

```null
resilience4j.bulkhead.metrics.enabled=true
resilience4j.bulkhead.instances.bulkheadApi.max-concurrent-calls=3
resilience4j.bulkhead.instances.bulkheadApi.max-wait-duration=1
```

By this, we want to limit the maximum number of concurrent calls to _3_, so each thread can wait only for _1ms_ if the bulkhead is full. After which, the requests be rejected with the _BulkheadFullException_ exception. Further, we'd want to return a meaningful HTTP status code to the client, so let's add an exception handler:

```null
@ExceptionHandler({ BulkheadFullException.class })
@ResponseStatus(HttpStatus.BANDWIDTH_LIMIT_EXCEEDED)
public void handleBulkheadFullException() {
}
```

Finally, let's test the bulkhead behavior by calling five requests in parallel:

```null
@Test
public void testBulkhead() throws InterruptedException {
    EXTERNAL_SERVICE.stubFor(WireMock.get("/api/external")
      .willReturn(ok()));
    Map<Integer, Integer> responseStatusCount = new ConcurrentHashMap<>();

    IntStream.rangeClosed(1, 5)
      .parallel()
      .forEach(i -> {
          ResponseEntity<String> response = restTemplate.getForEntity("/api/bulkhead", String.class);
          int statusCode = response.getStatusCodeValue();
          responseStatusCount.put(statusCode, responseStatusCount.getOrDefault(statusCode, 0) + 1);
      });

    assertEquals(2, responseStatusCount.keySet().size());
    assertTrue(responseStatusCount.containsKey(BANDWIDTH_LIMIT_EXCEEDED.value()));
    assertTrue(responseStatusCount.containsKey(OK.value()));
    EXTERNAL_SERVICE.verify(3, getRequestedFor(urlEqualTo("/api/external")));
}
```

We notice that **only three requests were successful, whereas the other requests were rejected with the _BANDWIDTH\_LIMIT\_EXCEEDED _HTTP status code**.

7\. Rate Limiter[](#retry)
--------------------------

**The [rate limiter pattern](https://www.baeldung.com/resilience4j#rate-limiter) limits the rate of requests to a resource.**

Let's start by adding the _/api/rate-limiter_ API endpoint with the _@RateLimiter_ annotation:

```null
@GetMapping("/rate-limiter")
@RateLimiter(name = "rateLimiterApi")
public String rateLimitApi() {
    return externalAPICaller.callApi();
}
```

Next, let's define the configuration for the rate limiter in the _application.properties_ file:

```null
resilience4j.ratelimiter.metrics.enabled=true
resilience4j.ratelimiter.instances.rateLimiterApi.register-health-indicator=true
resilience4j.ratelimiter.instances.rateLimiterApi.limit-for-period=5
resilience4j.ratelimiter.instances.rateLimiterApi.limit-refresh-period=60s
resilience4j.ratelimiter.instances.rateLimiterApi.timeout-duration=0s
resilience4j.ratelimiter.instances.rateLimiterApi.allow-health-indicator-to-fail=true
resilience4j.ratelimiter.instances.rateLimiterApi.subscribe-for-events=true
resilience4j.ratelimiter.instances.rateLimiterApi.event-consumer-buffer-size=50
```

With this configuration, we want to limit the API calling rate to _5_ _req/min_ without waiting. **After reaching the threshold for the allowed rate, requests will be rejected with the _RequestNotPermitted_ exception**. So, let's define a handler in the _ApiExceptionHandler_ class for translating it to a meaningful HTTP status response code:

```null
@ExceptionHandler({ RequestNotPermitted.class })
@ResponseStatus(HttpStatus.TOO_MANY_REQUESTS)
public void handleRequestNotPermitted() {
}
```

Lastly, let's test our rate-limited API endpoint with _50_ requests:

```null
@Test
public void testRatelimiter() {
    EXTERNAL_SERVICE.stubFor(WireMock.get("/api/external")
      .willReturn(ok()));
    Map<Integer, Integer> responseStatusCount = new ConcurrentHashMap<>();

    IntStream.rangeClosed(1, 50)
      .parallel()
      .forEach(i -> {
          ResponseEntity<String> response = restTemplate.getForEntity("/api/rate-limiter", String.class);
          int statusCode = response.getStatusCodeValue();
          responseStatusCount.put(statusCode, responseStatusCount.getOrDefault(statusCode, 0) + 1);
      });

    assertEquals(2, responseStatusCount.keySet().size());
    assertTrue(responseStatusCount.containsKey(TOO_MANY_REQUESTS.value()));
    assertTrue(responseStatusCount.containsKey(OK.value()));
    EXTERNAL_SERVICE.verify(5, getRequestedFor(urlEqualTo("/api/external")));
}
```

As expected, **only five requests were successful, whereas all the other requests failed with the _TOO\_MANY\_REQUESTS_ HTTP status code**.

8\. Actuator Endpoints[](#actuator-endpoints)
---------------------------------------------

We've configured our application to support actuator endpoints for monitoring purposes. Using these endpoints, we can determine how the application behaves over time using one or more of the configured fault tolerance patterns.

Firstly, we can generally **find all the exposed endpoints using a GET request to the _/actuator_ endpoint**:

```null
http:
{
    "_links" : {
        "self" : {...},
        "bulkheads" : {...},
        "circuitbreakers" : {...},
        "ratelimiters" : {...},
        ...
    }
}
```

We can see a JSON response with fields like _bulkheads_, _circuit breakers_, _ratelimiters_, and so on. Each field provides us specific information depending on its association with a fault tolerance pattern.

Next, let's take a look at the fields associated with the retry pattern:

```null
"retries": {
  "href": "http://localhost:8080/actuator/retries",
  "templated": false
},
"retryevents": {
  "href": "http://localhost:8080/actuator/retryevents",
  "templated": false
},
"retryevents-name": {
  "href": "http://localhost:8080/actuator/retryevents/{name}",
  "templated": true
},
"retryevents-name-eventType": {
  "href": "http://localhost:8080/actuator/retryevents/{name}/{eventType}",
  "templated": true
}
```

Moving on, let's inspect the application to see the list of retry instances:

```null
http:
{
    "retries" : [ "retryApi" ]
}
```

As expected, we can see the _retryApi_ instance in the list of configured retry instances.

Finally, let's make a GET request to the _/api/retry_ API endpoint through a browser and **observe the retry events using the _/actuator/retryevents_ endpoint**:

```null
{
    "retryEvents": [
    {
        "retryName": "retryApi",
        "type": "RETRY",
        "creationTime": "2022-10-16T10:46:31.950822+05:30[Asia/Kolkata]",
        "errorMessage": "...",
        "numberOfAttempts": 1
    },
    {
        "retryName": "retryApi",
        "type": "RETRY",
        "creationTime": "2022-10-16T10:46:32.965661+05:30[Asia/Kolkata]",
        "errorMessage": "...",
        "numberOfAttempts": 2
    },
    {
        "retryName": "retryApi",
        "type": "ERROR",
        "creationTime": "2022-10-16T10:46:33.978801+05:30[Asia/Kolkata]",
        "errorMessage": "...",
        "numberOfAttempts": 3
    }
  ]
}
```

Since the downstream service is down, we can see three retry attempts with a wait time of _1s_ between any two attempts. It's just like we configured it.

9\. Conclusion[](#conclusion)
-----------------------------

In this article, we learned about how to use the Resilience4j library in a Sprint Boot application. Additionally, **we dived deep into several fault tolerance patterns such as circuit breaker, rate limiter, time limiter, bulkhead, and retry.**

As always, the complete source code for the tutorial is available [over on GitHub](https://github.com/eugenp/tutorials/tree/master/spring-boot-modules/spring-boot-libraries-2).