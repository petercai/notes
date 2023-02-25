# Thread.sleep() vs Awaitility.await()
  

If you have a few years of experience in the Java ecosystem and you'd like to share that with the community, have a look at our [**Contribution Guidelines**](https://www.baeldung.com/expand-java-contribution-guidelines).

**1\. Overview**[](#overview)
-----------------------------

In this tutorial, we’ll compare two ways to handle asynchronous operations in Java. **First, we’ll see how the _sleep()_ method works on [_Thread_](https://www.baeldung.com/java-start-thread)s. Then, we’ll try to achieve the same functionality with the features provided by the [Awaitility](https://www.baeldung.com/awaitility-testing) library.** Along the way, we can see how these solutions compare and which is more suitable for our use case.

**2\. Use Cases**[](#use-cases)
-------------------------------

**The _sleep()_ and _await()_ methods can be particularly useful when we’d like to wait for the completion of an asynchronous operation.** For example, our application might send messages to a message broker or queue. In this case, we don’t know precisely when the message is received on the other end. Another use case can be calling an API endpoint and waiting for a specific result. For example, we send a request to a service, it starts a long-running task, and we wait for it to finish.

**In our example application, we’ll create a simple service that tracks the status of our requests. We’ll check if a request is in the required state after a given amount of time.**

**3\. Application Setup**[](#application-setup)
-----------------------------------------------

Let’s create an asynchronous service that handles the requests. We also need a way to fetch the status of these requests to be able to verify it afterward:

```null
public class RequestProcessor {

    private Map<String, String> requestStatuses = new HashMap<>();

    public String processRequest() {
        String requestId = UUID.randomUUID().toString();
        requestStatuses.put(requestId, "PROCESSING");

        ScheduledExecutorService executorService = Executors.newSingleThreadScheduledExecutor();
        executorService.schedule((() -> {
            requestStatuses.put(requestId, "DONE");
        }), getRandomNumberBetween(500, 2000), TimeUnit.MILLISECONDS);

        return requestId;
    }

    public String getStatus(String requestId) {
        return requestStatuses.get(requestId);
    }

    private int getRandomNumberBetween(int min, int max) {
        Random random = new Random();
        return random.nextInt(max - min) + min;
    }
}
```

This service uses [Java’s _ScheduledExecutorService_](https://www.baeldung.com/java-executor-service-tutorial#ScheduledExecutorService) to delay a command that changes the status of the request to _“DONE”._ It waits a random amount of time between one half-second and two seconds.

**We’ll create unit tests to showcase the two approaches for checking the async operation's result.**

**4\. Plain Java**[](#plain-java)
---------------------------------

First, let’s use the plain Java approach and pause the execution on the thread.

**In this case, we can set the amount of time we’d like to wait in milliseconds.** Let’s create our test class and the first unit test:

```null
@DisplayName("Request processor")
public class RequestProcessorUnitTest {

    RequestProcessor requestProcessor = new RequestProcessor();

    @Test
    @DisplayName("Wait for completion using Thread.sleep")
    void whenWaitingWithThreadSleep_thenStatusIsDone() throws InterruptedException {
        String requestId = requestProcessor.processRequest();

        Thread.sleep(2000);

        assertEquals("DONE", requestProcessor.getStatus(requestId));
    }
}
```

In this test case, we call the _processRequest()_ method of our _RequestProcessor_ to start a request. Then, we have to wait before getting the status via the request ID. We’re waiting for a status change because we expect it to be done.

**When we use _Thread.sleep()_, we have to make sure that we wait enough time before checking the result.** In our case, we know that our request is processed in a maximum of two seconds. **In real life, though, it’s more difficult to figure out the correct amount of time to wait.**


**5\. Awaitility**[](#awaitility)
---------------------------------

**We can also use [Awaitility](https://www.baeldung.com/awaitility-testing), a library that provides an easy-to-read API for testing this kind of code.**

First of all, we add the [_awaitility_ dependency](https://search.maven.org/search?q=g:org.awaitility%20AND%20a:awaitility) to our _pom.xml_:

```null
<dependency>
    <groupId>org.awaitility</groupId>
    <artifactId>awaitility</artifactId>
    <version>4.2.0</version>
    <scope>test</scope>
</dependency>
```

Now, we can create our test case that utilizes the new functionalities:

```null
@Test
@DisplayName("Wait for completion using Awaitility")
void whenWaitingWithAwaitility_thenStatusIsDone() {
    String requestId = requestProcessor.processRequest();

    Awaitility.await()
      .until(() -> requestProcessor.getStatus(requestId), not(equalTo("PROCESSING")));

    assertEquals("DONE", requestProcessor.getStatus(requestId));
}
```

This test case starts the same way as the previous one. **But, instead of sleeping for a fixed two seconds, we have a conditional statement. In this case, we sleep until the request isn’t in _“PROCESSING”_ status anymore.** After this, we use the same assertion to make sure that the status has the expected value.

We can provide additional options, too. For example, we can configure that we'd like to wait at most two seconds:


```null
Awaitility.await()
  .atMost(2, TimeUnit.SECONDS)
  .until(() -> requestProcessor.getStatus(requestId), not(equalTo("PROCESSING")));
```

**In the background, Awaitility uses [polling](https://github.com/awaitility/awaitility/wiki/Usage#polling) to check whether the given statement is true or false. We can increase or decrease the poll interval, but the [default value](https://github.com/awaitility/awaitility/wiki/Usage#defaults) is 100 milliseconds.** In other words, Awaitility checks the condition every 100 milliseconds. Let’s add an initial poll delay as well because we know that the status can’t change earlier than 500 milliseconds:

```null
Awaitility.await()
  .atMost(2, TimeUnit.SECONDS)
  .pollDelay(500, TimeUnit.MILLISECONDS)
  .until(() -> requestProcessor.getStatus(requestId), not(equalTo("PROCESSING")));
```

**6\. Comparison**[](#comparison)
---------------------------------

As we saw, both approaches can work just fine for our use case. However, there are some advantages and disadvantages that we should be aware of.

**It’s fairly simple to use _sleep()_ to pause a thread, but we don’t have much control after we send it to sleep.** The operation that we’re waiting for might finish immediately afterward and we’d still have to wait for the whole pre-defined duration.

**On the other hand, Awaitility lets us have a more fine-grained configuration.** As soon as the condition's fulfilled, the thread resumes execution, and this can improve performance.

**The _sleep()_ method is available in Java by default, while Awaitility is a library that needs to be added to our project.** We have to consider this when choosing the solution. **It’s more obvious to use the built-in method, but we can have much more readable code with the domain-specific language.**

**7\. Conclusion**[](#conclusion)
---------------------------------

In this article, we discussed two different approaches for handling asynchronous operations in Java. We focused on testing, but these examples can be used in other parts of the code as well.

**First, we used Java’s built-in solution to pause execution on a thread with the _sleep()_ method.** It’s easy to use, but we have to provide the sleep duration in advance.

**Then, we compared it with the Awaitility library, which provides a domain-specific language for handling this kind of situation.** It results in more readable code, but we have to learn how to use it.


As always, the source code for these examples is available [over on GitHub](https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-concurrency-basic-3).