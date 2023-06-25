# Java8 Collectors.toMap的两个大坑
* * *

1.  Collectors.toMap()方法的**正常使用**示例

```java
List<StudentDTO> studentDTOS = Lists.newArrayList();
studentDTOS.add(new StudentDTO(1,"xixi"));
studentDTOS.add(new StudentDTO(2,"houhou"));
studentDTOS.add(new StudentDTO(3,"maomi"));
Map<Integer, String> collect = studentDTOS.stream().collect(
	Collectors.toMap(StudentDTO::getStudentId, StudentDTO::getStudentName));
System.out.println(JSON.toJSON(collect)); 

```

* * *

一. 坑1：Duplicate Key时抛出IllegalStateException异常
---------------------------------------------

### 1\. 概述

*   按照常规Java的Map思维，往一个map里put一个已经存在的key，会把原有的key对应的value值覆盖。
*   但Java8中的Collectors.toMap()却不是这样。当key重复时，该方法默认会抛出IllegalStateException异常。

### 2\. 大坑复现

```java
public void streamToMap1() {
    List<StudentDTO> studentDTOS = Lists.newArrayList();
    studentDTOS.add(new StudentDTO(1,"xixi"));
    studentDTOS.add(new StudentDTO(1,"houhou"));
    studentDTOS.add(new StudentDTO(3,"maomi"));
    Map<Integer, String> collect = studentDTOS.stream()
    	.collect(Collectors.toMap(StudentDTO::getStudentId, StudentDTO::getStudentName));
    System.out.println(JSON.toJSON(collect)); 
}

```

*   输出结果  
    ![](_assets/20191112175259287.png)
    

### 3\. 大坑解决

1.  法1：将toMap方法修改成如下形式，这样就可以使用新的value覆盖原有value。

```java
studentDTOS.stream().collect(Collectors.toMap(StudentDTO::getStudentId, 
					StudentDTO::getStudentName,(oldValue, newValue) -> newValue));

```

*   输出结果:`{"1":"houhou","3":"maomi"}`

2.  法2：如果需要保留同一个key下所有的值，则可以对value做简单的拼接，如下：

```java
studentDTOS.stream().collect(Collectors.toMap(StudentDTO::getStudentId, 
					StudentDTO::getStudentName,(oldValue, newValue) -> oldValue + "," + newValue));

```

*   输出结果:`{"1":"xixi,houhou","3":"maomi"}`

* * *

二. 坑2：value为空时抛出NullPointerException异常
--------------------------------------

### 1\. 概述

*   当要转化的map的value值中包含空指针时， 会抛出NullPointerException异常。

### 2\. 大坑复现

```java
public void streamToMap2() {
    List<StudentDTO> studentDTOS = Lists.newArrayList();
    studentDTOS.add(new StudentDTO(1,"xixi"));
    studentDTOS.add(new StudentDTO(2,"houhou"));
    studentDTOS.add(new StudentDTO(3,null));
    Map<Integer, String> collect = studentDTOS.stream().collect(Collectors
    .toMap(StudentDTO::getStudentId, StudentDTO::getStudentName));
    System.out.println(JSON.toJSON(collect));
}

```

*   输出结果  
    ![](_assets/20191112175133516.png)
    

### 3\. 大坑解决

#### 3.1 法1：value值判空设置

1.  说明：如果是null，则设置成一个特定值。

```java
studentDTOS.stream().collect(Collectors.toMap(StudentDTO::getStudentId, studentDTO 
					-> studentDTO.getStudentName()==null?"":studentDTO.getStudentName()));

```

> 输出结果：`{"1":"xixi","2":"houhou","3":""}`

#### 3.2 法2：使用`collect(Supplier<R> supplier, BiConsumer<R, ? super T> accumulator, BiConsumer<R, R> combiner)`方法构建

1.  说明：该方法允许null值。

```java
Map<Integer, String> collect = studentDTOS.stream().collect(HashMap::new, 
		(n, v) -> n.put(v.getStudentId(), v.getStudentName()), HashMap::putAll);
for(Map.Entry<Integer, String> entry:collect.entrySet()){
    System.out.println(entry.getKey()+"="+entry.getValue());
}

```

*   输出结果

```
1=xixi
2=houhou
3=null

```

#### 3.3 使用Optional对值进行包装

```java
Map<Integer, Optional<String>> collect = studentDTOS.stream().collect(Collectors
	.toMap(StudentDTO::getStudentId,
	studentDTO -> Optional.ofNullable(studentDTO.getStudentName())));
	
for(Map.Entry<Integer, Optional<String>> entry:collect.entrySet()){
    System.out.println(entry.getKey()+"="+entry.getValue().orElse(""));
}

```

*   输出结果

```
1=xixi
2=houhou
3=

```

* * *

参考资料
----

*   [Java8 Collectors.toMap的坑](https://blog.csdn.net/u013805360/article/details/82686009)
*   [Java8 Stream 操作 Collectors.toMap(）会出现NullPointerException异常](https://blog.csdn.net/wanping321/article/details/97389347)