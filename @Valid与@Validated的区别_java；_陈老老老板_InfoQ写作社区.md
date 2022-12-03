# @Valid与@Validated的区别_java；_陈老老老板_InfoQ写作社区
> 陈老老老板
> 
> 说明：工作了，学习一些新的技术栈，边学习边总结，各位一起加油。需要注意的地方都标红了，还有资源的分享. 一起加油。  

**说明：** 其实 @Valid 与 @Validated 都是做数据校验的，只不过注解位置与用法有点不同。

**不同点：** （1）@Valid 是使用 Hibernate validation 的时候使用。@Validated 是只用 Spring Validator 校验机制使用。

（2）@Valid 可以嵌套验证 @Validation 不能进行嵌套验证

（3） @Valid：可以用在方法、构造函数、方法参数和成员属性（field）上。@Validated：用在类、方法和方法参数上。但不能用于成员属性（field）。（如果 @Validated 注解在成员属性上，则会报不适用于 field 的错误。）

（4）@Valid：没有分组功能。@Validated：提供分组功能，可以在参数验证时，根据不同的分组采用不同的验证机制。

（1）@Valid 用法
------------

### a.导入依赖

SpringBoot 项目：

```
<dependency>
    <groupId>javax.validation</groupId>
    <artifactId>validation-api</artifactId>
    <version>1.1.0.Final</version>
</dependency>
 
<dependency>
    <groupId>org.hibernate</groupId>
    <artifactId>hibernate-validator</artifactId>
    <version>5.4.1.Final</version>
</dependency>
```

### b.使用前提是实体类中属性使用注解进行校验

```
package com.example.demo.pojo;

import lombok.Data;
import org.hibernate.validator.constraints.Length;
import org.hibernate.validator.constraints.NotBlank;
import org.hibernate.validator.constraints.Range;
import org.springframework.format.annotation.DateTimeFormat;

import javax.persistence.*;
import javax.validation.constraints.NotNull;
import java.io.Serializable;
import java.util.Date;


@Data
public class User implements Serializable {

    
    @NotBlank(message = "请输入名称")
    @Length(message = "名称不能超过个 {max} 字符", max = 10)
    private String username;

    
    @NotNull(message = "请输入年龄")
    @Range(message = "年龄范围为 {min} 到 {max} 之间", min = 1, max = 100)
    private String age;

}
```

### c.在 Controller 方法参数中加上 @Valid 注解

```
package com.example.demo.controller;

import com.example.demo.pojo.User;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.validation.BindingResult;
import org.springframework.validation.ObjectError;
import org.springframework.validation.annotation.Validated;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.ResponseBody;
import org.springframework.web.bind.annotation.RestController;

import javax.validation.Valid;
import java.util.List;

@RestController
public class UserController {

    public static final Logger logger = LoggerFactory.getLogger(UserController.class.getName());

    @PostMapping("/add")
    @ResponseBody
    public String add(@Validated User user, BindingResult bindingResult){
        if(bindingResult.hasErrors()){
            List<ObjectError> allErrors = bindingResult.getAllErrors();
            allErrors.forEach( v ->{
                logger.error(v.getObjectName()+"======"+v.getDefaultMessage());
            });
            return "添加失败";
        }
        return "添加成功";
    }
}
```

经过测试填写错误数据，会在控制台输出报错信息。

![](https://static001.geekbang.org/infoq/74/74d472e7097f7a4d481c2240c3b80893.png)

![](https://static001.geekbang.org/infoq/de/de5942fc149733688749977d40883824.png)

（2）@Validated 用法
----------------

### a.开启校验框架（与上面一样）

```
<dependency>
    <groupId>javax.validation</groupId>
    <artifactId>validation-api</artifactId>
</dependency>

<dependency>
    <groupId>org.hibernate.validator</groupId>
    <artifactId>hibernate-validator</artifactId>
</dependency>
```

### c.在需要开启校验功能的类上使用注解 @Validated 开启校验功能，对具体的字段设置校验规则,这里讲的是可以在类上使用 @Validated 注解，配合 xml 数据绑定。

```
@Component
@Data
@ConfigurationProperties(prefix = "servers")

@Validated
public class ServerConfig {
    
    @Max(value = 8888,message = "最大值不能超过8888")
    @Min(value = 202,message = "最小值不能低于202")
    private int port;
}
```

（3）@Validated 实现分组校验
--------------------

**注意 分组校验就是把条件加入组中，可以自由选择开启那些组的校验方式。** 

### **a.分组接口**

```
<font color="red"><b><code class="language-java">package com.example.demo.pojo;

public interface Group {

    interface Update{};

    interface FindAll{};
}
</code></b></font>
```

### **b.实体类**

```
<font color="red"><b><code class="language-java">package com.example.demo.pojo;

import lombok.Data;
import org.hibernate.validator.constraints.Length;
import org.hibernate.validator.constraints.NotBlank;
import org.hibernate.validator.constraints.Range;
import org.springframework.format.annotation.DateTimeFormat;

import javax.persistence.*;
import javax.validation.constraints.NotNull;
import java.io.Serializable;
import java.util.Date;


//lombok
@Data
public class User implements Serializable {

    //用户名
    @NotBlank(message = "请输入用户名不能为空",groups = {Group.FindAll.class})
    @Length(message = "名称不能超过个 {max} 字符", max = 10 ,groups = {Group.FindAll.class})
    private String username;

    //年龄
    @NotBlank(message = "请输入年龄不能为空",groups = {Group.Update.class})
    @Range(message = "年龄范围为 {min} 到 {max} 之间", min = 1, max = 100,groups = {Group.Update.class})
    private String age;



}
</code></b></font>
```

### **c.controller 接口：** 

****注意 @Validated 有参数 value 中写分组名称****

```
<font color="red"><b><font color="red"><b><code class="language-java">package com.example.demo.controller;

import com.example.demo.pojo.Group;
import com.example.demo.pojo.User;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.validation.BindingResult;
import org.springframework.validation.ObjectError;
import org.springframework.validation.annotation.Validated;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.ResponseBody;
import org.springframework.web.bind.annotation.RestController;

import javax.validation.Valid;
import java.util.List;

@RestController
public class UserController {

    public static final Logger logger = LoggerFactory.getLogger(UserController.class.getName());

    @PostMapping("/add")
    @ResponseBody
    //注意@Validated有参数 value中写分组名称
    public String add(@Validated(value = {Group.Update.class}) User user, BindingResult bindingResult){
        if(bindingResult.hasErrors()){
            List<ObjectError> allErrors = bindingResult.getAllErrors();
            allErrors.forEach( v ->{
                logger.error(v.getObjectName()+"======"+v.getDefaultMessage());
            });
            return "添加失败";
        }
        return "添加成功";
    }
}
</code></b></font></b></font>
```

****（4）@Valid 实现嵌套校验****
------------------------

******注:嵌套检测就是在一个 User 类中，存在另外一个 User2 类的属性。嵌套检测 User 同时也检测 User2。******

### ******a.实体类 User******

```
<font color="red"><b><font color="red"><b><b><code class="language-java">package com.example.demo.pojo;

import lombok.Data;
import org.hibernate.validator.constraints.Length;
import org.hibernate.validator.constraints.NotBlank;
import org.hibernate.validator.constraints.Range;
import org.springframework.format.annotation.DateTimeFormat;

import javax.persistence.*;
import javax.validation.Valid;
import javax.validation.constraints.NotNull;
import java.io.Serializable;
import java.util.Date;


//lombok
@Data
public class User implements Serializable {

    //用户名
    @NotBlank(message = "请输入用户名不能为空1")
    private String username;

    //年龄
    @NotBlank(message = "请输入年龄不能为空1")
    private String age;

    @Valid
    @NotNull(message = "user2不能为空1")
    private User2 user2;



}


}
</code></b></b></font></b></font>
```

### ******b.实体类 User2******

```
<font color="red"><b><font color="red"><b><b><code class="language-java">package com.example.demo.pojo;

import lombok.Data;
import org.hibernate.validator.constraints.Length;
import org.hibernate.validator.constraints.NotBlank;
import org.hibernate.validator.constraints.Range;
import org.springframework.format.annotation.DateTimeFormat;

import javax.persistence.*;
import javax.validation.Valid;
import javax.validation.constraints.NotNull;
import java.io.Serializable;
import java.util.Date;


package com.example.demo.pojo;

import lombok.Data;
import org.hibernate.validator.constraints.Length;
import org.hibernate.validator.constraints.Range;

import javax.validation.constraints.NotNull;
import java.io.Serializable;


//lombok
@Data
public class User2 implements Serializable {

    //用户名
    @Length(message = "名称不能超过个 {max} 字符2", max = 10 )
    private String username2;

    //年龄

    @Range(message = "年龄范围为 {min} 到 {max} 之间2", min = 1, max = 100)
    private String age2;



}

</code></b></b></font></b></font>
```

### ******c.Controller 类（这里使用 @Valid）******

```
<font color="red"><b><font color="red"><b><b><code class="language-java">package com.example.demo.controller;

import com.example.demo.pojo.Group;
import com.example.demo.pojo.User;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.validation.BindingResult;
import org.springframework.validation.ObjectError;
import org.springframework.validation.annotation.Validated;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.ResponseBody;
import org.springframework.web.bind.annotation.RestController;

import javax.validation.Valid;
import java.util.List;

@RestController
public class UserController {

    public static final Logger logger = LoggerFactory.getLogger(UserController.class.getName());

    @PostMapping("/add")
    @ResponseBody
    public String add(@Valid User user, BindingResult bindingResult){
        if(bindingResult.hasErrors()){
            List<ObjectError> allErrors = bindingResult.getAllErrors();
            allErrors.forEach( v ->{
                logger.error(v.getObjectName()+"======"+v.getDefaultMessage());
            });
            return "添加失败";
        }
        return "添加成功";
    }
}
</code></b></b></font></b></font>
```

> ********总结：了解这两个注解可以让你的校验数据更加方便。希望对您有帮助，感谢阅读结束语：裸体一旦成为艺术，便是最圣洁的。道德一旦沦为虚伪，便是最下流的。勇敢去做你认为正确的事，不要被世俗的流言蜚语所困扰。********