# Retry with Delay in RxJava | Baeldung
  

We’re finally running a Black Friday launch. All Courses are **30% off** until **next Friday:**

### **[\>> GET ACCESS NOW](https://www.baeldung.com/all-courses)**

1\. Overview[](#overview)
-------------------------

In this tutorial, **we'll see how to use retry with delay in RxJava**. When working with _Observable_, it is quite likely to get an _error_ instead of a _success_. In such cases, we can use _retry_ with delay to reattempt after some time.

2\. Project Setup[](#project-setup)
-----------------------------------

To begin this, let's create a Maven or a Gradle project. Here, **we're using a Maven-based project**. Let's add the _rxjava_ dependency to our _pom.xml_ file:

```null
<dependency>
    <groupId>io.reactivex.rxjava2</groupId>
    <artifactId>rxjava</artifactId>
    <version>2.2.21</version>
</dependency>
```

The dependency can be found on the [Maven Central Page](https://search.maven.org/artifact/io.reactivex.rxjava2/rxjava/2.2.21/jar).

3\. Retry in the Observable Lifecycle[](#retry-in-the-observable-lifecycle)
---------------------------------------------------------------------------

_Retry_ is used when an _Observable_ source emits an error. This is a resubscription to the source in the hopes of getting completed without errors in the next attempt. Therefore, _retry()_ and its other variants come into the picture only when the _Observable_ emits an error.

### 3.1. Overloaded Variants of Retry[](#1-overloaded-variants-of-retry)

Apart from the _retry()_ method discussed in the above section, RxJava provides two overloaded variants, _retry(long)_ and _retry(Func2)_.

The _retry(long)_ returns an _Observable_ that mirrors the source _Observable_. We resubscribe to this mirror only as long as it calls _onError_ the specified number of times.

The _retry(Func2)_ method takes a function that accepts a _Throwable_ as input and produces a _boolean _as output. Here, we resubscribe to the mirror only if it returns _true _for the specific _Exception _and the retry count.

### 3.2. _RetryWhen_ vs _Retry_[](#2-retrywhen-vs-retry)

The _retryWhen(Func1)_ provides us with an option to include custom logic to determine whether or not we should resubscribe to the mirror of the original _Observable_. This method accepts a function that takes the _exception _thrown by the _onError _handler.

If the function emits an item, we resubscribe to the mirror, while we don't if it produces an o_nError_ notification.

4\. Code Examples[](#code-examples)
-----------------------------------

Let's now watch these concepts in action with the help of a few code snippets.

### 4.1. Successful _Observable_[](#1-successful-observable)

If an _Observable_ call succeeds, then the Observer's _onNext_ method is invoked. Let's see an example:

```null
@Test
public void givenObservable_whenSuccess_thenOnNext(){
    Observable.just(remoteCallSuccess())
        .subscribe(success -> {
            System.out.println("Success");
            System.out.println(success);
        }, err -> {
            System.out.println("Error");
            System.out.println(err);
        });
}
```

The method _remoteCallSuccess() _completes successfully and returns a _String_. In this case, the _onNext()_ method of the subscriber is invoked, printing success messages.

### 4.2. Erroneous Observable[](#2-erroneous-observable)

Let's introduce a new utility function called _remoteCallError()_ that mocks calling a remote server. This method is vulnerable to errors. Therefore, our _retryWhen_ logic would kick in:

```null
@Test
public void givenObservable_whenError_thenOnError(){
    Observable.just(remoteCallError())
        .subscribe(success -> {
            System.out.println("Success");
            System.out.println(success);
        }, err -> {
            System.out.println("Error");
            System.out.println(err);
        });
}
```

In this case, the _remoteCallError() _method causes an error. Therefore, the _onError() _method of the subscriber gets called. Consequently, error messages get printed.

### 4.3. Retrying with Delay[](#3-retrying-with-delay)

We can add a delay during the resubscription process:

```null
@Test
public void givenError_whenRetry_thenCanDelay(){
    Observable.just(remoteCallError())
        .retryWhen(attempts -> {
            return attempts.flatMap(err -> {
                if (customChecker(err)) {
                    return Observable.timer(5000, TimeUnit.MILLISECONDS);
                } else {
                    return Observable.error(err);
                }
            });
        });
}
```

Similar to the previous case, the _remoteCallError() _method causes an error. Here, however, instead of notifying the observer's _onError() _method, we give this _Observable_ a chance to resubcribe. For this, we pass the exception to a _customChecker() _method, which returns a _boolean_. If the response is _true_, we return a new _Observable _with a delay of 5000 milliseconds (5 seconds). Otherwise, we return an error and notify the observer.

5\. Conclusion[](#conclusion)
-----------------------------

In this tutorial, we saw how to use _retryWhen_ and some of its related functions. We understood the cases under which _retry_ and _retryWhen_ help us solve our issues. The source code is available [over on GitHub](https://github.com/eugenp/tutorials/tree/master/rxjava-modules/rxjava-core).

We’re finally running a Black Friday launch. All Courses are **30% off** until **next Friday:**

### **[\>> GET ACCESS NOW](https://www.baeldung.com/all-courses)**