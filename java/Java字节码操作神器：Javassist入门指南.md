# Java字节码操作神器：Javassist入门指南
### 什么是Javassist？

Javassist是一个强大的字节码操作工具，它提供了在运行时编辑Java字节码的能力。通过Javassist，开发人员可以动态地创建和修改Java类。这使得在不重新编译整个程序的情况下，能够对类进行动态修改和增强。

### 引入依赖

![](_assets/c57852bb5d834ec4b60b156f178a2948~tplv-k3u1fbpfcp-jj-mark!3024!0!0!0!q75.awebp.webp)

### 如何创建一个类？

使用Javassist创建一个类非常简单，以下是一个基本的示例：

![](_assets/ff04df71978c4878a6818106b030e6ae~tplv-k3u1fbpfcp-jj-mark!3024!0!0!0!q75.awebp.webp)

利用IDEA的反编译可以看到所创建出来的类为： ![](_assets/433996e32e14434bb69ae5d522ca25e1~tplv-k3u1fbpfcp-jj-mark!3024!0!0!0!q75.awebp.webp)
 ClassPool表示管理类的类池，可以用它来创建（makeClass）或获取（getCtClass）某个类（CtClass）。

当getCtClass()获取某个类时，比如： ![](_assets/af596a6db3554a449c05131740b717f0~tplv-k3u1fbpfcp-jj-mark!3024!0!0!0!q75.awebp.webp)

默认情况下会利用**系统类加载器**来进行加载，以下是getCtClass()中较底层的源码： ![](_assets/6f137a10e08f4455a19cdd34304f624b~tplv-k3u1fbpfcp-jj-mark!3024!0!0!0!q75.awebp.webp)

CtClass表示一个类，可以利用它来操作方法和字段。

### 如何创建方法和字段？

要在类中创建方法和字段，可以使用`CtMethod`和`CtField`类。以下是一个例子：

![](_assets/5d41597a388a49ba81b41aecea173f5a~tplv-k3u1fbpfcp-jj-mark!3024!0!0!0!q75.awebp.webp)

对应的类也变为了： ![](_assets/3a130fbb51a7437c9375a893610ee82b~tplv-k3u1fbpfcp-jj-mark!3024!0!0!0!q75.awebp.webp)

CtMethod其实有更多强大的方法，比如可以直接复制另外一个方法的方法体作为自己的方法体，比如可以设置在当前方法的前后插入额外逻辑等等。

可以直接利用CtClass得到Class对象，从而利用反射来调用方法： ![](_assets/dea84303578748af9bf1b27fbfaf8935~tplv-k3u1fbpfcp-jj-mark!3024!0!0!0!q75.awebp.webp)

### 如何修改类？

要修改类，就先获取现有的类，然后进行修改，以下是一个简单的例子： ![](_assets/878ba5fe2f0441d9b41b7e1dba567636~tplv-k3u1fbpfcp-jj-mark!3024!0!0!0!q75.awebp.webp)

采用反射调用existingMethod()方法，就能看到AOP的效果： ![](_assets/2600e42e63d84ccd90feae6e2accef72~tplv-k3u1fbpfcp-jj-mark!3024!0!0!0!q75.awebp.webp)

被修改后的类为： ![](_assets/e064b9b6bb7f439ca4af4a2632c62196~tplv-k3u1fbpfcp-jj-mark!3024!0!0!0!q75.awebp.webp)

### 如何继承另外一个类，实现一个接口？

要继承另一个类和实现一个接口，可以使用`CtClass`的`getSuperclass`和`addInterface`方法。以下是一个例子： ![](_assets/6f2136eb64bc4145a9c34737088aec7c~tplv-k3u1fbpfcp-jj-mark!3024!0!0!0!q75.awebp.webp)

新生成的类为： ![](_assets/87f706e9931141cbbdbfa7d915c650f0~tplv-k3u1fbpfcp-jj-mark!3024!0!0!0!q75.awebp.webp)

### 结语

Javassist为Java程序员提供了一个强大而灵活的工具，用于在运行时操作字节码。通过这个工具，你可以动态地创建、修改和分析类，为你的应用程序提供更高的灵活性和可扩展性。当然，在使用Javassist时，需要小心谨慎，确保对字节码的修改是正确的，以避免潜在的问题。
