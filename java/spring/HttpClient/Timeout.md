
# 1. response timeout

To set the **socket timeout** (also known as **read timeout**) for `http-outgoing-0` connections in a Spring Boot application using **Apache HttpClient5**, you need to configure the **response timeout** on the HTTP client.

### 🔧 What is `http-outgoing-0`?

`http-outgoing-0` is the default thread name used by Apache HttpClient for outgoing HTTP connections. If you're seeing this in logs or thread dumps, it refers to the thread handling an HTTP request.

---

### ✅ How to Set Socket Timeout (Read Timeout)

In **Apache HttpClient5**, the socket timeout is configured using:

```java
RequestConfig requestConfig = RequestConfig.custom()
    .setResponseTimeout(Timeout.ofSeconds(20)) // socket timeout
    .build();
```

In a **Spring Boot 3.4.6** app, you typically set this when building the `HttpClient` or the `RestTemplate`:

```java
@Bean
public RestTemplate restTemplate() {
    RequestConfig config = RequestConfig.custom()
        .setConnectTimeout(Timeout.ofSeconds(20))
        .setConnectionRequestTimeout(Timeout.ofSeconds(5))
        .setResponseTimeout(Timeout.ofSeconds(20)) // socket timeout
        .build();

    CloseableHttpClient client = HttpClients.custom()
        .setDefaultRequestConfig(config)
        .build();

    HttpComponentsClientHttpRequestFactory factory = new HttpComponentsClientHttpRequestFactory(client);
    return new RestTemplate(factory);
}
```

---

### 🧩 Summary of Timeout Types

|Timeout Type|HttpClient5 Method|Description|
|---|---|---|
|Connection Timeout|`setConnectTimeout(...)`|Time to establish TCP connection|
|Connection Request Timeout|`setConnectionRequestTimeout(...)`|Time to get a connection from pool|
|**Socket Timeout** (Read Timeout)|`setResponseTimeout(...)`|Time to wait for data after connection|

---

Would you like a working example or a test endpoint to verify this behavior?
