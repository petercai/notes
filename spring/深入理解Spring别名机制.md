# 深入理解Spring别名机制 
概述
--

在 spring 容器中，允许通过名称或别名来获取 bean ，这个能力来自于顶层接口 `AliasRegistry`，分析类下属的关系图，可以看到，几乎所有主要容器都直接或间接的实现了 `AliasRegistry` 接口。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a39f93b3c0ef4be69b98dbe8625b04d5~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp)

`AliasRegistry` 的结构非常简单，主要的类就是 `AliasRegistry` 接口与他的实现类 `SimpleAliasRegistry`，后续的实现类基本都直接或间接的继承了 `SimpleAliasRegistry`。

本文将基于 spring 源码 `5.2.x` 分支，围绕 `SimpleAliasRegistry` 解析 spring 的别名机制是如何实现的。

1.AliasRegistry
---------------

在 spring 的容器中，一个 bean 必须至少有一个名称，而一个名称可以有多个别名，别名亦可以有别名，若我们把这个最原始的名称称为 `id`，则结构可以有：

```text
id -> id's alias1 -> alias of id's alias1 ... ...
   -> id's alias2 -> alias of id's alias2 ... ...

```

通过 bean 的 `id`，或与该 `id` 直接、间接相关的别名，都可以从容器中获取到对应的 bean。

`AliasRegistry` 的作用就是定义这一套规则：

```java
public interface AliasRegistry {
    
	void registerAlias(String name, String alias);
    
	void removeAlias(String alias);
    
	boolean isAlias(String name);
    
	String[] getAliases(String name);
}

```

2.SimpleAliasRegistry
---------------------

`SimpleAliasRegistry` 是直接基于 `AliasRegistry` 接口提供的简单实现类，他在内部维护了一个 `Map` 集合，以别名作为 key，别名对应的原名称作为 value：

```java
public class SimpleAliasRegistry implements AliasRegistry {

    
    private final Map<String, String> aliasMap = new ConcurrentHashMap<>(16);

}

```

在 `SimpleAliasRegistry` 中，我们在上文所提到的 `id` ，被称为标准名称 `CanonicalName`。

### 2.1.注册别名

```java
@Override
public void registerAlias(String name, String alias) {
    Assert.hasText(name, "'name' must not be empty");
    Assert.hasText(alias, "'alias' must not be empty");
    
    synchronized (this.aliasMap) {
        
        if (alias.equals(name)) {
            this.aliasMap.remove(alias);
            if (logger.isDebugEnabled()) {
                logger.debug("Alias definition '" + alias + "' ignored since it points to same name");
            }
        }
        else {
            String registeredName = this.aliasMap.get(alias);
            
            if (registeredName != null) {
                
                if (registeredName.equals(name)) {
                    
                    return;
                }

                
                if (!allowAliasOverriding()) {
                    throw new IllegalStateException("Cannot define alias '" + alias + "' for name '" +
                                                    name + "': It is already registered for name '" + registeredName + "'.");
                }
                if (logger.isDebugEnabled()) {
                    logger.debug("Overriding alias '" + alias + "' definition for registered name '" +
                                 registeredName + "' with new target name '" + name + "'");
                }
            }
            
            checkForAliasCircle(name, alias);
            
            
            this.aliasMap.put(alias, name);
            if (logger.isTraceEnabled()) {
                logger.trace("Alias definition '" + alias + "' registered for name '" + name + "'");
            }
        }
    }
}

```

**是否允许覆盖别名对应的原名称**

在 `registerAlias` 方法中，调用了 `allowAliasOverriding` 方法，该方法在 `SimpleAliasRegistry` 中固定返回 `true` ：

```java
protected boolean allowAliasOverriding() {
    return true;
}

```

该方法决定一个已经有对应原名称的别名，是否允许被重新被对应到另一个原名称。跟 `HashMap` 中的 `afterNodeXXX` 方法一样，这里很明显是留给子类重写的钩子方法。

**检查是否存在循环引用**

在 `registerAlias` 方法中，调用了 `checkForAliasCircle` 以检查别名是否存在 `a -> b -> a` 这样的循环引用：

```java
protected void checkForAliasCircle(String name, String alias) {
    if (hasAlias(alias, name)) {
        throw new IllegalStateException("Cannot register alias '" + alias +
                                        "' for name '" + name + "': Circular reference - '" +
                                        name + "' is a direct or indirect alias for '" + alias + "' already");
    }
}

public boolean hasAlias(String name, String alias) {
    String registeredName = this.aliasMap.get(alias);
    
    
    return ObjectUtils.nullSafeEquals(registeredName, name) 
        
        || (registeredName != null && hasAlias(name, registeredName));
}

```

因为在 `SimpleAliasRegistry` 中，一个名称作为另一个名称的别名后，该名称仍然允许有别的名称作为它的别名。

举个例子，实际场景中，可能会出现 `a -> b -> c` 这种情况，对应在 `aliasMap` 中则存在 `b -> a` 与 `c -> b` 的引用关系。

此时，若想要确定 `c` 是否是 `a` 的别名，就需要在通过 `c` 取出 `b` 后，再向上递归根据 `b` 找到 `a`，把每一级的关系却判断一遍。

**为什么要加锁**

理解了循环引用的检查，也就不难理解加锁的必要性了。实际使用中，`SimpleAliasRegistry` 基本是作为单例使用，此时不可避免的可能会存在多线程调用 `registerAlias` 的可能性，即便 `aliasMap` 使用的 `ConcurrentHashMap` 本身是线程安全的，但是这不能阻止出现 `a -> b` 和 `b -> a` 两个操作的同时发生，此时 `a` 与 `b` 构成了循环引用，而每个线程中的 `checkForAliasCircle` 却未必能够正确的检测出问题。

### 2.2.获取名称

**获取全部别名**

```java
@Override
public String[] getAliases(String name) {
    List<String> result = new ArrayList<>();
    synchronized (this.aliasMap) {
        retrieveAliases(name, result);
    }
    return StringUtils.toStringArray(result);
}

private void retrieveAliases(String name, List<String> result) {
    this.aliasMap.forEach((alias, registeredName) -> {
        if (registeredName.equals(name)) {
            result.add(alias);
            retrieveAliases(alias, result);
        }
    });
}

```

获取别名也很好理解，首先加锁同样的为了避免删除的同时注册别名导致的一些问题，然后递归调用的 `retrieveAliases` 也是为了防止出现“别名还有别名”的情况，`getAliases` 最终将返回该名称与该名称构成直接或间接引用关系的别名。

**获取标准名称**

```java
public String canonicalName(String name) {
    String canonicalName = name;
    
    String resolvedName;
    do {
        resolvedName = this.aliasMap.get(canonicalName);
        if (resolvedName != null) {
            canonicalName = resolvedName;
        }
    }
    while (resolvedName != null);
    return canonicalName;
}

```

该方法的作用是通过一个别名获得对应的标准名称。举个例子，假如现在存在如下别名引用关系：`a -> b -> c -> d`，则从 `a`、`b`、`c`、`d` 任意一个名称开始调用该方法，则最终都会返回 `a`。

### 2.3.移除别名

```java
@Override
public void removeAlias(String alias) {
    synchronized (this.aliasMap) {
        String name = this.aliasMap.remove(alias);
        if (name == null) {
            throw new IllegalStateException("No alias '" + alias + "' registered");
        }
    }
}

```

### 2.4.名称转换

有时候，直接注册到 `SimpleAliasRegistry` 的名称和对应别名未必就是最想要，还需要替换占位符或者进行一些大小写转换的操作，这就需要通过 `resolveAliases` 方法来完：

```java
public void resolveAliases(StringValueResolver valueResolver) {
    Assert.notNull(valueResolver, "StringValueResolver must not be null");
    synchronized (this.aliasMap) {
        Map<String, String> aliasCopy = new HashMap<>(this.aliasMap);
        aliasCopy.forEach((alias, registeredName) -> {
            
            String resolvedAlias = valueResolver.resolveStringValue(alias);
            String resolvedName = valueResolver.resolveStringValue(registeredName);
            
            if (resolvedAlias == null || resolvedName == null || resolvedAlias.equals(resolvedName)) {
                this.aliasMap.remove(alias);
            }
            
            else if (!resolvedAlias.equals(alias)) {
                String existingName = this.aliasMap.get(resolvedAlias);
                
                if (existingName != null) {
                    if (existingName.equals(resolvedName)) {
                        
                        this.aliasMap.remove(alias);
                        return;
                    }
                    throw new IllegalStateException(
                        "Cannot register resolved alias '" + resolvedAlias + "' (original: '" + alias +
                        "') for name '" + resolvedName + "': It is already registered for name '" +
                        registeredName + "'.");
                }
                
                checkForAliasCircle(resolvedName, resolvedAlias);
                this.aliasMap.remove(alias);
                this.aliasMap.put(resolvedAlias, resolvedName);
            }
            else if (!registeredName.equals(resolvedName)) {
                this.aliasMap.put(alias, resolvedName);
            }
        });
    }
}

```

其中，`StringValueResolver` 是一个函数式接口，它用于将一个字符串转为另一个字符串，也就是 `resolve` 的过程：

```java
@FunctionalInterface
public interface StringValueResolver {
	@Nullable
	String resolveStringValue(String strVal);
}

```

`resolveAliases` 的过程基本与注册别名的方法 `registerAlias` 一致，本质上就是将原有的别名的与名称转换后再次建立映射关系，然后移除旧的映射关系。