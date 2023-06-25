# Enum Mapping in Spring Boot | Baeldung
1\. Overview[](#overview)
-------------------------

In this tutorial, we'll explore different ways to implement case-insensitive enum mapping in Spring Boot.

First, we'll see how enums are mapped by default in Spring. Then, we'll learn how to tackle the case sensitivity challenge.

2\. Default Enum Mapping in Spring[](#formatting-instant-using-core-java)
-------------------------------------------------------------------------

Spring relies upon several built-in converters to handle string conversion when dealing with request parameters.

Typically, when we pass an enum as a request parameter**, it uses [_StringToEnumConverterFactory_](https://github.com/spring-projects/spring-framework/blob/main/spring-core/src/main/java/org/springframework/core/convert/support/StringToEnumConverterFactory.java) under the hood to convert the passed _String_ to an _enum_**_._

By design, this converter calls _Enum.valueOf(Class, String)_ **which means that the given string must match exactly one of the declared enum constants**.

[![](_assets/fslogo-green.svg)
](https://freestar.com/?utm_campaign=branding&utm_medium=banner&utm_source=baeldung.com&utm_content=baeldung_leaderboard_mid_1)

For instance, let's consider the _Level_ enum:

```null
public enum Level {
    LOW, MEDIUM, HIGH
}
```

Next, let's create a [handler method](https://www.baeldung.com/spring-handler-mappings) that accepts our enum as an argument:

```null
@RestController
@RequestMapping("enummapping")
public class EnumMappingController {

    @GetMapping("/get")
    public String getByLevel(@RequestParam(required = false) Level level){
        return level.name();
    }

}
```

Let's send a request to _http://localhost:8080/enummapping/get?level=MEDIUM_ using [CURL](https://www.baeldung.com/curl-rest):

```null
curl http://localhost:8080/enummapping/get?level=MEDIUM
```

The handler method sends back _MEDIUM_, the name of the enum constant _MEDIUM_.

Now, let's pass _medium_ instead of _MEDIUM_ and see what happens:

[![](_assets/fslogo-green.svg)
](https://freestar.com/?utm_campaign=branding&utm_medium=banner&utm_source=baeldung.com&utm_content=baeldung_leaderboard_mid_2)

AD

```null
curl http://localhost:8080/enummapping/get?level=medium
{"timestamp":"2022-11-18T18:41:11.440+00:00","status":400,"error":"Bad Request","path":"/enummapping/get"}
```

As we can see, the request is considered invalid and the application fails with an error:

```null
Failed to convert value of type 'java.lang.String' to required type 'com.baeldung.enummapping.enums.Level'; 
nested exception is org.springframework.core.convert.ConversionFailedException: Failed to convert from type [java.lang.String] to type [@org.springframework.web.bind.annotation.RequestParam com.baeldung.enummapping.enums.Level] for value 'medium'; 
...
```

Looking at the stack trace, we can see that Spring throws _ConversionFailedException._ It doesn't recognize _medium_ as an enum constant.

3\. Case-Insensitive Enum Mapping[](#formatting-instant-using-core-java-1)
--------------------------------------------------------------------------

Spring provides several convenient approaches to solve the problem of case sensitivity when mapping enums.

Let's take a close look at each approach.

### 3.1. Using _ApplicationConversionService_[](#1-using-applicationconversionservice)

The [_ApplicationConversionService_](https://docs.spring.io/spring-boot/docs/current/api/org/springframework/boot/convert/ApplicationConversionService.html) class comes with a set of configured converters and formatters.

[![](_assets/fslogo-green.svg)
](https://freestar.com/?utm_campaign=branding&utm_medium=banner&utm_source=baeldung.com&utm_content=baeldung_leaderboard_mid_3)

AD

Among these out-of-the-box converters, we find _StringToEnumIgnoringCaseConverterFactory._ **As the name implies, it converts a string into an enum in a case-insensitive manner**.

First, we need to add and configure _ApplicationConversionService_:

```null
@Configuration
public class EnumMappingConfig implements WebMvcConfigurer {
    @Override
    public void addFormatters(FormatterRegistry registry) {
        ApplicationConversionService.configure(registry);
    }
}
```

This class configures the [_FormatterRegistry_](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/format/FormatterRegistry.html) with ready-to-use converters appropriate for most Spring Boot applications.

Now, let's confirm that everything works as expected using a test case:

```null
@RunWith(SpringRunner.class)
@WebMvcTest(EnumMappingController.class)
public class EnumMappingIntegrationTest {

    @Autowired
    private MockMvc mockMvc;

    @Test
    public void whenPassingLowerCaseEnumConstant_thenConvert() throws Exception {
        mockMvc.perform(get("/enummapping/get?level=medium"))
            .andExpect(status().isOk())
            .andExpect(content().string(Level.MEDIUM.name()));
    }

}
```

As we can see, the _medium_ value passed as a parameter is successfully converted to _MEDIUM_.

### 3.2. Using Custom Converter[](#2-using-custom-converter)

Another solution would be using a custom converter. Here, we will be using the Apache Commons Lang 3 library.

First, we need to add its [dependency](https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22org.apache.commons%22%20AND%20a%3A%22commons-lang3%22):

```null
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-lang3</artifactId>
    <version>3.12.0</version>
</dependency>
```

**The basic idea here is to create a converter that converts a stringrepresentation of a _Level_ constant to a real _Level_ constant:**

[![](_assets/fslogo-green.svg)
](https://freestar.com/?utm_campaign=branding&utm_medium=banner&utm_source=baeldung.com&utm_content=baeldung_incontent_1)

AD

```null
public class StringToLevelConverter implements Converter<String, Level> {

    @Override
    public Level convert(String source) {
        if (StringUtils.isBlank(source)) {
            return null;
        }
        return EnumUtils.getEnum(Level.class, source.toUpperCase());
    }

}
```

**From a technical point of view, a custom converter is a simple class that implements the _Converter<S,T>_ interface**.

As we can see, we converted the _String_ object to uppercase. Then, we used the [_EnumUtils_](https://commons.apache.org/proper/commons-lang/apidocs/org/apache/commons/lang3/EnumUtils.html) utility class from the Apache Commons Lang 3 library to get the _Level_ constant from the upper case string.

Now, let's add the last missing piece of the puzzle. We need to tell Spring about our new custom converter. To do so, we'll use the same _FormatterRegistry_ from before. **It provides the _addConverter()_ method to register custom converters**:

```null
@Override
public void addFormatters(FormatterRegistry registry) {
    registry.addConverter(new StringToLevelConverter());
}
```

And that's it. Our _StringToLevelConverter_ is now available in the [_ConversionService_](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/core/convert/ConversionService.html).

Now, we can use it in the same way as any other converter:

```null
@RunWith(SpringRunner.class)
@SpringBootTest(classes = EnumMappingMainApplication.class)
public class StringToLevelConverterIntegrationTest {

    @Autowired
    ConversionService conversionService;

    @Test
    public void whenConvertStringToLevelEnumUsingCustomConverter_thenSuccess() {
        assertThat(conversionService.convert("low", Level.class)).isEqualTo(Level.LOW);
    }

}
```

As shown above, the test case confirms that the _“low”_ value is converted to _Level.LOW_.

### 3.3. Using Custom Property Editor[](#3-using-custom-property-editor)

Spring uses multiple built-in [property editors](https://www.baeldung.com/spring-mvc-custom-property-editor) under the hood to manage conversion between _String_ values and Java objects.

Similarly, we can create a custom property editor to map _String_ objects into _Level_ constants.

[![](_assets/fslogo-green.svg)
](https://freestar.com/?utm_campaign=branding&utm_medium=banner&utm_source=baeldung.com&utm_content=baeldung_incontent_2)

AD

For instance, let's name our custom editor _LevelEditor_:

```null
public class LevelEditor extends PropertyEditorSupport {

    @Override
    public void setAsText(String text) {
        if (StringUtils.isBlank(text)) {
            setValue(null);
        } else {
            setValue(EnumUtils.getEnum(Level.class, text.toUpperCase()));
        }
    }
}
```

As we can see, we need to extend the [_PropertyEditorSupport_](https://docs.oracle.com/en/java/javase/11/docs/api/java.desktop/java/beans/PropertyEditorSupport.html) class and override the _setAsText()_ method.

**The idea of overriding s_etAsText()_ is to convert the uppercase version of the given string into the _Level_ enum**.

It's worth noting that _PropertyEditorSupport_ provides _getAsText()_ as well. It's called when serializing a Java object to a string. So, we don't need to override it here.

We need to register our _LevelEditor_ because Spring doesn't automatically detect custom property editors. **To do that, we need to create a method annotated with _@InitBinder_ in our [Spring controller](https://www.baeldung.com/spring-controllers)**:

```null
@InitBinder
public void initBinder(WebDataBinder dataBinder) {
    dataBinder.registerCustomEditor(Level.class, new LevelEditor());
}
```

Now that we put all the pieces together, let's confirm that our custom property editor _LevelEditor_ works using a test case:

```null
public class LevelEditorIntegrationTest {

    @Test
    public void whenConvertStringToLevelEnumUsingCustomPropertyEditor_thenSuccess() {
        LevelEditor levelEditor = new LevelEditor();
        levelEditor.setAsText("lOw");

        assertThat(levelEditor.getValue()).isEqualTo(Level.LOW);
    }
}
```

Another important thing to mention here is that _EnumUtils.getEnum()_ returns the enum when found, _null_ otherwise.

So, to avoid _NullPointerException_, we need to change our handler method a little bit:

[![](_assets/fslogo-green.svg)
](https://freestar.com/?utm_campaign=branding&utm_medium=banner&utm_source=baeldung.com&utm_content=baeldung_incontent_3)

AD

```null
public String getByLevel(@RequestParam(required = false) Level level) {
    if (level != null) {
        return level.name();
    }
    return "undefined";
}
```

Now, let's add a simple test case to test this:

```null
@Test
public void whenPassingUnknownEnumConstant_thenReturnUndefined() throws Exception {
    mockMvc.perform(get("/enummapping/get?level=unknown"))
        .andExpect(status().isOk())
        .andExpect(content().string("undefined"));
}
```

4\. Conclusion[](#conclusion)
-----------------------------

In this article, we learned a variety of ways to implement case-insensitive enum mapping in Spring.

Along the way, we looked at some ways to do this using built-in and custom converters. Then, we saw how to achieve the same objective using a custom property editor.

As always, the code used in this article can be found [over on GitHub](https://github.com/eugenp/tutorials/tree/master/spring-boot-modules/spring-boot-request-params).

We’re finally running a Black Friday launch. All Courses are **30% off** until **the end of this week:**

### **[\>> GET ACCESS NOW](https://www.baeldung.com/all-courses)**