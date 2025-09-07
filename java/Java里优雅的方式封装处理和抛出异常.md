# Java里优雅的方式封装处理和抛出异常
今天我们来分享点关于异常处理和封装的不同方式。下文所有的代码都进行了省略操作，如果有问题欢迎来评论区交流

一、定义自定义枚举类
----------

```java
@EqualsAndHashCode(callSuper = true)
@Data
@Accessors(chain = true)
public class ServiceException extends RuntimeException {
    
    private int code;
    
    
    private String message;

    
    public ServiceException(int code, String message) {
        setCode(code).setMessage(message);
    }
}

```

此时，我们可以使用 `throw` 关键词来进行传统异常的抛出：

```java
throw new ServiceException(401, "你的身份凭证已过期");

```

太难受了，如果很多地方抛出这个异常的话，状态码成了魔法值，还要写一堆文案。

**又不太想定义八百个异常类，那更麻烦了**

继续改造：

二、异常枚举封装
--------

我们把上面的异常代码和消息封装到枚举里面，再通过 `ServiceException` 的新构造来抛出：

```java
@Getter
@AllArgsConstructor
public enum ServiceError {
    PARAM_MISSING(4001, "缺少必要的请求参数"),
    UNAUTHORIZED(401, "获取你的身份信息失败，请重新登录后再试"),
    FORBIDDEN(403, "无权操作"),
    FORBIDDEN_EXIST(4031, "添加失败，数据已存在"),
    DATA_NOT_FOUND(404, "没有查到相关的数据"),
    

    private final int code;
    private final String message;
}

```

然后为自定义业务异常类 `ServiceException` 添加两个可以传入 `ServiceError` 枚举项的构造方法：

```java
public class ServiceException extends RuntimeException {
    
    public ServiceException(@NotNull ServiceError serviceError) {
        setCode(serviceError.getCode()).setMessage(serviceError.getMessage());
    }

    public ServiceException(@NotNull ServiceError serviceError, String message) {
        setCode(serviceError.getCode()).setMessage(message);
    }
    
}

```

那么接下来，我们抛出异常就不会出现魔法值了，而且同一个错误代码，也可以在不同场景下写不同的错误信息文案了：

```java
throw new ServiceException(ServiceError.UNAUTHORIZED);
throw new ServiceException(ServiceError.UNAUTHORIZED, "小子，你掉线了");

```

三、控制器接管异常抛出
-----------

接下来，我们就可以使用 `@ControllerAdvice` 和 `@ExceptionHandler` 注解来处理异常了：

```java
@ControllerAdvice
@ResponseStatus(HttpStatus.OK)
@ResponseBody
public class ExceptionInterceptor {
    
     * <h2>处理自定义异常</h2>
     */
    @ExceptionHandler(ServiceException.class)
    public Json systemExceptionHandle(@NotNull ServiceException exception) {
        return Json.error(exception).setData(exception.getData());
    }

    
}

```

这样，我们的异常抛出给前端，就会是我们定义的标准 `JSON` 格式了。

不过，抛出异常的地方还是 **不够优雅**。

比如：

```java
if(Objects.isNull(user)){
    throw new ServiceException(ServiceError.USER_NOT_FOUND);
}

```

我们能不能再精简一下上面的代码呢？

能！

四、封装断言类型异常抛出
------------

首先我们分析下，`ServiceError枚举类` 和 `ServiceException类` 都存在 `code` `message` 属性，我们先抽离一个 `IException` 接口，把 `code` `message` 属性抽象出来：

```java
public interface IException {
    
    int getCode();

    
    String getMessage();
}

```

然后，我们将 `ServiceError` 枚举类和 `ServiceException` 类都实现这个接口：

```java
public enum ServiceError implements IException {
    
}

public class ServiceException extends RuntimeException implements IException {
    
}

```

这样，我们就保持了异常和枚举都可以正常使用 `code` `message` 属性，并且 `ServiceError` 枚举类和 `ServiceException` 类也保持了统一。

接着，我们直接在这个异常类里做默认实现，来创建并抛出异常：

```java
public interface IException {
    
    int getCode();

    
    String getMessage();

    
    private @NotNull ServiceException create() {
        return new ServiceException(getCode(), getMessage());
    }

    
    default void stop() {
        stop(getMessage());
    }

    
    default void stop(String message) {
        throw create().setMessage(message);
    }
    
    
    default void stopWhen(boolean condition) {
        stopWhen(condition, getMessage());
    }
    
    
    default void stopWhen(boolean condition, String message) {
        if (condition) {
            stop(message);
        }
    }
}

```

这样就美滋滋了，我们再来看看 **三、控制器接管异常抛出** 中的判断空抛出代码，我们就可以这么改造了：

*   改造前，三行代码

```java
if(Objects.isNull(user)){
    throw new ServiceException(ServiceError.USER_NOT_FOUND);
}

```

*   改造后，一行代码

```java
ServiceError.USER_NOT_FOUND.stopWhen(Objects.isNull(user));

```

五、是不是美滋滋？
---------

我们还能再过分一点，添加很多的默认实现，比如：

```java
public interface IException {
    
    default void stopWhenNull(Object object) {
        stopWhenNull(object, getMessage());
    }

    default void stopWhenNull(Object object, String message) {
        if(Objects.isNull(object)){
            stop(message);
        }
    }

    
    default void stopWhenEquals(Object object1, Object object2) {
        if(Objects.equals(object1, object2)){
            stop();
        }
    }

    
}

```

每个调用的人就会很舒坦：

```java
ServiceError.USER_NOT_FOUND.stopWhenNull(user, "用户信息不能为空");
ServiceError.USER_TARGET_ERROR.stopWhenEquals(fromUserId, toUserId, "发送人和收件人不能是同一个人");

```

如果是判了 `null` 异常后，后续不再需要 `IDEA` 飘黄提示可能为空，可以参考我们之前的一篇文章 [你可能都不知道,JetBrains提供了这些好玩又实用的注解](https://juejin.cn/post/7441796089529860122 "https://juejin.cn/post/7441796089529860122") ，添加一些 `JetBrains` 的注解来实现。

六、总结
----

本文的部分代码可能因为篇幅原因被我删减掉了，如果想查看完整的开源代码，可以关注这个 Github 仓库：

[github.com/HammCn/AirP…](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2FHammCn%2FAirPower4J "https://github.com/HammCn/AirPower4J")

好了，今天的分享就到这里，如果对你有帮助，欢迎 关注 点赞 收藏 三连～

Bye.