# 适配器模式_后端_InfoQ写作社区
> 一般做业务开发，不太容易有大量使用设计模式的场景。这里总结一下在业务开发中使用较为频繁的设计模式。当然语言为 Java，基于 Spring 框架。

### 1 适配器模式(Adapter Pattern)

已存在的接口、服务，跟我们所需、目的接口不兼容时，我们需要通过一定的方法将二者进行兼容适配。一个常见的例子，家用电源(国标)220V，而手机标准输入一般为 5V，此时我们便需要一个适配器来将 220V 转换为 5V 使用。

适配器模式一般有 3 个角色：

*   Target: 目标接口
    
*   Adaptee: 需要进行适配的类(受改造者)
    
*   Adapter: 适配器(将 Adaptee 转为 Target)
    

这个出现的场景其实挺多，但实际完全按照适配器模式编写代码的场景可能并不多。简单业务场景，直接就将适配、兼容代码混杂在业务代码中了。并没有将其摘出来处理。大部分业务代码，可能后续并不会再做扩展之类，过度设计反而会降低可读性并增加代码的复杂性。

适配器模式一般分为`类适配器`和`对象适配器`。类适配器的话，使用继承方式实现：`class Adapter extends Adaptee implements Target`；而对象适配器的话，则使用组合方式实现。这种情况更灵活一些。毕竟大家都推荐多用组合少用继承。

在学生发生约课完课等事件时，我们需要将部分数据同步到外部 CRM 系统中。课程的话，按班级类型分为：1v1，小班课、大班课。不同的班级类型课程数据有所不同。事件上报时，并不是全量数据，有些数据需要消费者按需查询。如课程课程编号、名称、预约上课时间等。

不同班级类型的课程由三个不同的 FeignClient(Adaptee)提供服务，而我们想要的就是查询课程相关信息(Target)。

为了模拟服务提供者，我们 Mock 如下服务。

```
@Data
@Builder
public class OneClass {
    
    private String lessonNo;
    
    private String lessonName;

    
    private String one;
}

@Data
@Builder
public class SmallClass {
    
    private String lessonNo;
    
    private String lessonName;

    
    private String small;
}

@Data
@Builder
public class BigClass {
    
    private String lessonNo;
    
    private String lessonName;

    
    private String big;
}

public interface RemoteClassClient {

    default OneClass getOne() {
        return OneClass.builder().lessonNo("one").lessonName("1V1").build();
    }

    default SmallClass getSmall() {
        return SmallClass.builder().lessonNo("small").lessonName("小班课").build();
    }

    default BigClass getBig() {
        return BigClass.builder().lessonNo("big").lessonName("大班课").build();
    }
}

public class RemoteClassClientImpl implements RemoteClassClient {
}
```

该服务统一由`RemoteClassClient`对外提供各个班级类型的查询服务。

> ClassService (Target 目标接口、及目标对象)

```
* 课程信息
 */
@Data
@Builder
public class ClassInfoBO {
    
    private String type;

    
    private String classId;
    
    private String lessonNo;
    
    private String lessonName;
}


 * 目标接口
 */
public interface ClassService {

    boolean match(String classType);

    ClassInfoBO getClassInfo(String classId);
}
```

下面我们就需要几个适配器来完成适配。

#### 1.1 对象适配器

> OneClassAdapter

```
* 1v1适配器
 */
@Component
@RequiredArgsConstructor
public class OneClassAdapter implements ClassService {
    private static final String TYPE = "1";

    private final RemoteClassClient classClient;

    @Override
    public boolean match(String classType) {
        return TYPE.equals(classType);
    }

    @Override
    public ClassInfoBO getClassInfo(String classId) {
        final OneClass one = classClient.getOne();
        return ClassInfoBO.builder()
                .type("1")
                .classId(classId)
                .lessonNo(one.getLessonNo()).lessonName(one.getLessonName())
                .build();
    }
}
```

> SmallClassAdapter

```
* 小班课适配器
 */
@Component
@RequiredArgsConstructor
public class SmallClassAdapter implements ClassService {
    private static final String TYPE = "2";

    private final RemoteClassClient classClient;

    @Override
    public boolean match(String classType) {
        return TYPE.equals(classType);
    }

    @Override
    public ClassInfoBO getClassInfo(String classId) {
        final SmallClass small = classClient.getSmall();
        return ClassInfoBO.builder()
                .type("2")
                .classId(classId)
                .lessonNo(small.getLessonNo()).lessonName(small.getLessonName())
                .build();
    }
}
```

> BigClassAdapter

```
* 大班课适配器
 */
@Component
@RequiredArgsConstructor
public class BigClassAdapter implements ClassService {
    private static final String TYPE = "3";

    private final RemoteClassClient classClient;

    @Override
    public boolean match(String classType) {
        return TYPE.equals(classType);
    }

    @Override
    public ClassInfoBO getClassInfo(String classId) {
        final BigClass big = classClient.getBig();
        return ClassInfoBO.builder()
                .type("3")
                .classId(classId)
                .lessonNo(big.getLessonNo()).lessonName(big.getLessonName())
                .build();
    }
}
```

至此，适配器完成。可以根据具体场景选择不同的适配器，去适配当前的场景。再来个适配器工厂类。

> ClassAdapterFactory

```
* 课程信息适配器工厂
 */
@Service
@RequiredArgsConstructor
public class ClassAdapterFactory {
    private final List<ClassService> classServiceList;

    ClassService getAdapter(String classType) {
        return classServiceList.stream()
                .filter(cs -> cs.match(classType)).findFirst()
                .orElse(null);
    }
}
```

#### 1.2 类适配器

仅以其一举例。这种适配器不如对象适配器灵活。

```
* 小班课适配器(类适配器)
 */
@Component
public class SmallClassAdapter2 extends RemoteClassClientImpl implements ClassService {
    private static final String TYPE = "2";

    @Override
    public boolean match(String classType) {
        return TYPE.equals(classType);
    }

    @Override
    public ClassInfoBO getClassInfo(String classId) {
        final SmallClass small = super.getSmall();
        return ClassInfoBO.builder()
                .type("2")
                .classId(classId)
                .lessonNo(small.getLessonNo()).lessonName(small.getLessonName())
                .build();
    }
}
```

可以看到，我们通过`继承`的方式，使`SmallClassAdapter2`具备了`RemoteClassClientImpl`查询课程信息的能力。跟`SmallClassAdapter`也没有啥太大区别。

#### 1.3 单测

```
@SpringBootTest
class ClassServiceTest {

    @Autowired
    private ClassAdapterFactory adapterFactory;

    @Test
    void testOne() {
        String classType = "1";
        String classId = "11111111";
        Optional.ofNullable(adapterFactory.getAdapter(classType)).ifPresent(ad -> {
            final ClassInfoBO classInfo = ad.getClassInfo(classId);
            assertEquals("one", classInfo.getLessonNo());
        });
    }

    @Test
    void testSmall() {
        String classType = "2";
        String classId = "22222222";
        Optional.ofNullable(adapterFactory.getAdapter(classType)).ifPresent(ad -> {
            final ClassInfoBO classInfo = ad.getClassInfo(classId);
            assertEquals("small", classInfo.getLessonNo());
        });
    }

    @Test
    void testBig() {
        String classType = "3";
        String classId = "33333333";
        Optional.ofNullable(adapterFactory.getAdapter(classType)).ifPresent(ad -> {
            final ClassInfoBO classInfo = ad.getClassInfo(classId);
            assertEquals("big", classInfo.getLessonNo());
        });
    }
}
```

### 2 思考

适配器模式能带来什么？

1.  感觉最主要的还是带了一种解决方案(😅当然设计模式本来就是如此)。其次是对某些场景提供了一种规范，加入没有这个设计模式，我们也能有解决方案，但具体实现可能千奇百怪(😅当然设计模式本来就是从千奇百怪中提炼出来的)。
    
2.  设计模式不是银弹。避免滥用。
    

跟策略模式有啥区别？

1.  单从实现方式(套路)来说。不能说毫无区别，简直就是一模一样。
    
2.  细想一下，`Adaptee`在策略模式中，并不是必然存在的。它的目的是不同策略的具体实现和调用分开。而适配器模式的重点在于`Adaptee -> Target`的适配。
    

* * *

封面图来源: [https://refactoring.guru/design-patterns/adapter](https://xie.infoq.cn/link?target=https%3A%2F%2Frefactoring.guru%2Fdesign-patterns%2Fadapter)  

* * *

```
echo '5Y6f5Yib5paH56ugOiDmjpjph5Eo5L2g5oCO5LmI5Zad5aW26Iy25ZWKWzkyMzI0NTQ5NzU1NTA4MF0pL+aAneWQpihscGUyMzQp' | base64 -d
s
```