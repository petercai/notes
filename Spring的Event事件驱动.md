# Spring的Event事件驱动
### 一、前言

**目的：**  我们创建一个对象，一旦这个对象中的内容发生了变更，就会利用Event事件驱动抛出一个相应的事件，其他的观察者一旦观察到该对象的内容发生了变化，那么就可以做出响应的应对和调整！

**模式应用：**  **观察者模式**

**应用场景：**  SpringEvent在我认为是一个解决业务解耦的办法，运用了观察者模式，用于当一个业务的更改后，需要改变其他业务的状态。例如一个商品的下单，需要修改商品的库存，以及商家的消息发送等等。之前我做这种业务解耦的时候，使用的时消息队列进行解耦，但如果只是为了解耦而整合了消息队列就有点大材小用了，我认为可以使用Spring的Event事件驱动来满足需求（需要容忍宕机丢失等问题，如果无法容忍，还是需要使用消息队列）！

**说明：**  此案例为了实现，我们创建的Person对象，我们使用一个PersonChangeEvent来实现Event事件驱动，使用代码对Person进行的增删改等操作使用PeronEventPublisher来进行发布服务，使用PeronEventListener来进行监听，通过监听到对应的事件，我们来执行相应的逻辑，可以加些后续操作，例如添加缓存、更新缓存等！

### 二、代码实现

#### 2.1 **被观察的对象：**  **Person**

```kotlin
package com.ssm.user.event;

import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;

@Data
@AllArgsConstructor
@NoArgsConstructor
public class Person {

    private String name;

    private Integer age;
    
}

```

#### 2.2 **对应的Event事件驱动：**  **PersonChangeEvent**

```scala
package com.ssm.user.event;

import lombok.Data;
import lombok.Getter;
import org.springframework.context.ApplicationEvent;

@Getter 

public class PersonChangeEvent extends ApplicationEvent { 


    private Person person;

    private String op;

    public PersonChangeEvent(Person person, String op) {
        super(person);
        this.person = person;
        this.op = op;
    }

}

```

#### 2.3 **相应的Event事件发布：**  **PersonEventPublisher**

```typescript
package com.ssm.user.event;

import lombok.extern.slf4j.Slf4j;
import org.springframework.context.ApplicationEventPublisher;
import org.springframework.stereotype.Service;

@Service
@Slf4j
public class PersonEventPublisher {

    
    @Autowired
    private ApplicationEventPublisher applicationEventPublisher; 



    public void createPerson(Person person) {
        
        applicationEventPublisher.publishEvent(new PersonChangeEvent(person, "create"));
    }

    public void updatePerson(Person person) {
        applicationEventPublisher.publishEvent(new PersonChangeEvent(person, "update"));
    }
}

```

#### 2.4 **相应的Event事件监听：**  **PersonEventListener**

```typescript
package com.ssm.user.event;

import com.alibaba.fastjson.JSON;
import lombok.extern.slf4j.Slf4j;
import org.springframework.stereotype.Service;
import org.springframework.transaction.event.TransactionalEventListener;

@Service
@Slf4j
public class PersonEventListener {
    
     * @TransactionalEventListener 事务事件监听
     * 在默认情况下仅仅会被事务中发布的事件触发，如果需要在没有事务也能当作普通时间监听器触发，那么需要将fallbackExecution属性设置为true。
     */
    @TransactionalEventListener(fallbackExecution = true)
    public void listenCreateEvent(PersonChangeEvent personChangeEvent) {
        switch (personChangeEvent.getOp()) {
            case "create" :
                
                
                log.info("创建Person对象：{}", JSON.toJSONString(personChangeEvent.getPerson()));
                break;

            case "update":
                log.info("更新Person对象：{}", JSON.toJSONString(personChangeEvent.getPerson()));
                break;

            default:
                break;
        }
    }
}

```

#### 2.5 测试结果

```java
package com.ssm.user;

import com.ssm.user.event.Person;
import com.ssm.user.event.PersonEventPublisher;
import lombok.extern.slf4j.Slf4j;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.junit4.SpringRunner;

@SpringBootTest(classes = {UserApplication.class}, webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
@RunWith(SpringRunner.class)
@Slf4j
public class PersonChangeTest {

    @Autowired
    private PersonEventPublisher personEventPublisher;

    @Test
    public void test() {
        Person person = new Person("ssm", 18);
        personEventPublisher.createPerson(person);
        person.setAge(20);
        personEventPublisher.updatePerson(person);
    }
}

```

![](https://p6-xtjj-sign.byteimg.com/tos-cn-i-73owjymdk6/36e0c531804342a980f90b9dcb5f3b6b~tplv-73owjymdk6-jj-mark-v1:0:0:0:0:5o6Y6YeR5oqA5pyv56S-5Yy6IEAgU2hpU2h1b01pbmc=:q75.awebp?rk3s=f64ab15b&x-expires=1728467613&x-signature=jyg4tX%2FXZycGM6I%2Ba6AJLCBezdY%3D)

**结论：**  可以看到，我们在PeronEventPublisher和PeronEventListener中编写了创建和更新相关的操作，当我们对Person类进行操作的时候，只要调用相应的发布事件，那么在对应的监听事件就会执行相关后续操作！