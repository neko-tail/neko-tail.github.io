---
tag: 
    - Spring Native
    - graalvm
category: Spring
---

生成提示文件，使编译完之后的程序支持反射等功能

<!--more-->

>   相关链接：
>
>   [Reachability Metadata](https://www.graalvm.org/latest/reference-manual/native-image/metadata/)
>
>   [Collect Metadata with the Tracing Agent](https://www.graalvm.org/22.3/reference-manual/native-image/metadata/AutomaticMetadataCollection/)
>
>   [aot](https://docs.spring.io/spring-framework/reference/core/aot.html)
>
>   [Using the Tracing Agent](https://docs.spring.io/spring-boot/docs/current/reference/html/native-image.html#native-image.advanced.using-the-tracing-agent)
>
>   [Custom Hints](https://docs.spring.io/spring-boot/docs/current/reference/html/native-image.html#native-image.advanced.custom-hints)

在**src/main/resources/META-INF/native-image/**目录下的提示文件会在编译时被使用，提示文件的类型和作用参考[Metadata Types](https://www.graalvm.org/latest/reference-manual/native-image/metadata/#metadata-types)

对于我们的项目，生成提示文件的方法主要以下几种：

-   自定义提示
-   Tracing Agent

## 自定义提示

>   [aot](https://docs.spring.io/spring-framework/reference/core/aot.html)
>
>   [Runtime Hints](https://docs.spring.io/spring-framework/reference/core/aot.html#aot.hints)

Spring Native在编译之前会对jar包进行aot处理，在**META-INF/native-image/<group.id>/<artifact.id>**生成提示文件

可以自定义一个实现`RuntimeHintsRegistrar`接口的类来注册需要加入提示文件的内容

RuntimeHints类的写法如下，需要在Bean上使用 `@ImportRuntimeHints` 注解来导入：

```java
import java.lang.reflect.Method;

import org.springframework.aot.hint.ExecutableMode;
import org.springframework.aot.hint.RuntimeHints;
import org.springframework.aot.hint.RuntimeHintsRegistrar;
import org.springframework.util.ReflectionUtils;

public class MyRuntimeHints implements RuntimeHintsRegistrar {

    @Override
    public void registerHints(RuntimeHints hints, ClassLoader classLoader) {
        // Register method for reflection
        Method method = ReflectionUtils.findMethod(MyClass.class, "sayHello", String.class);
        hints.reflection().registerMethod(method, ExecutableMode.INVOKE);

        // Register resources
        hints.resources().registerPattern("my-resource.txt");

        // Register serialization
        hints.serialization().registerType(MySerializableClass.class);

        // Register proxy
        hints.proxies().registerJdkProxy(MyInterface.class);
    }

}
```

一般可以将RuntimeHints类的定义和导入写在一起，如：

```java
@Configuration(proxyBeanMethods = false)
@ImportRuntimeHints(MyBatisNativeConfiguration.MyBaitsRuntimeHintsRegistrar.class)
public class MyBatisNativeConfiguration {
    
	static class MyBaitsRuntimeHintsRegistrar implements RuntimeHintsRegistrar {

        @Override
        public void registerHints(RuntimeHints hints, ClassLoader classLoader) {
            // ...
        }
    }
}
```

还可以直接使用`@RegisterReflectionForBinding`注解来绑定需要序列化和反序列化的类

次注解可以在任何bean的类或方法上使用，绑定参数中的类，使用方法如：

```java
@Component
public class OrderService {

	@RegisterReflectionForBinding(Account.class)
	public void process(Order order) {
		// ...
	}

}
```

或在一处绑定多个类：

```java
@Configuration(proxyBeanMethods = false)
@RegisterReflectionForBinding({
    UserVO.class,
	OrderVO.class,
    UserDTO.class,
    OrderDTO.class,
})
public class MyReflectionBindingConfiguration {
}
```

## Tracing Agent

>   [Tracing Agent](https://www.graalvm.org/22.3/reference-manual/native-image/metadata/AutomaticMetadataCollection/#tracing-agent)

将项目打成jar包之后，使用graalvm执行下面的命令来运行程序

```sh
java -agentlib:native-image-agent=config-output-dir=/path/output -jar xxx.jar
```

当程序停止后，会在**/path/output**目录生成提示文件，将文件放置项目的**src/main/resources/META-INF/native-image/**目录下即可

提示文件中会记录本次运行中用到的反射、序列化与反序列化、资源文件等内容，所以在程序运行时需要尽可能完整地使用功能

如果一次运行还不够的话，可以使用下面的命令来继续生成提示文件

```sh
java -agentlib:native-image-agent=config-merge-dir=/path/output -jar xxx.jar
```

当程序停止后，会在**/path/output**目录已有的提示文件基础上，继续增加新的内容
