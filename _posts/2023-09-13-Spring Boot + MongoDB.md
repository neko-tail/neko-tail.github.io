---
tag: 
    - MongoDB
category: Spring
---

在Spring Boot程序中通过spring-boot-starter-data-mongodb对MongoDB进行增删改查

<!--more-->

>   [MongoDB安装](https://www.mongodb.com/docs/manual/administration/install-community/)
>
>   [Spring Data MongoDB中文文档](https://springdoc.cn/spring-data-mongodb/)

### 安装

以Windows为例，在[下载中心](https://www.mongodb.com/try/download/community-kubernetes-operator)选择适合的版本和系统进行安装，安装过程省略，中间会提示是否要安装可视化管理工具MongoDB Compass，按需选择即可

![image-20230913083408202](https://cdn.thmaster.net/pic/202309130834299.png)

### 依赖

在pom.xml中添加以下依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-mongodb</artifactId>
</dependency>
```

在配置文件中添加以下配置

host，port和database自行替换，例：`mongodb://localhost:27017/test`

```yaml
spring:
  data:
    mongodb:
      uri: mongodb://host:port/database
```

### 使用

MongoDB的层级结构依次是 Database -> Collection -> Document

类似于MySQL中的数据库 -> 表 -> 行的结构



首先需要编写好实体类（对应于MongoDB中的Collection）

然后对于实体类，有两种进行CRUD的方式，MongoRepository和MongoTemplate

 

MongoRepository类似于MybatisPlus里的BaseMapper，可以进行简单的CRUD操作，每一个实体类配置一个对应的Repository

MongoTemplate则类似于一个MongoDB的工具类，可以进行复杂的CRUD，并且只需配置一次

#### 实体类

Document注解标记实体类，并指定collection名

例：

```java
@Document(collection = "demo")
public class Demo {
    
    private String name;
    
}
```

#### MongoRepository

MongoRepository<T, ID>接口中，T为实体类，ID为这个Document的_id字段，这个字段是MongoDB自动生成的唯一标识符

例：

```java
@Repository
public interface DemoRepository extends MongoRepository<Demo, String> {
}
```

之后就可以直接使用DemoRepository对象中的save()，findAll()等方法

#### MongoTemplate

要使用MongoTemplate只需要事先配置好一个mongoTemplate的bean即可

例：

```java
@Configuration
public class MongoConfig {
    
    @Bean
    public MongoTemplate mongoTemplate(MongoDatabaseFactory factory) {
        return new MongoTemplate(factory);
    }
    
}
```

mongoTemplate中方法的参数一般分为：

-   查询条件

    是一个Query对象，如：

    ```java
    new Query(Criteria.where("name").is("th_03")
    ```

-   更新定义

    是一个UpdateDefinition对象，如

    ```java
     Update.update("name", "th_03_updated")
    ```

-   collectionName

    使用实体类`Demo.class`或直接使用字符串`"demoi"`都可以

例：

```java
mongoTemplate.updateFirst(new Query(Criteria.where("name").is("th_03")), Update.update("name", "th_03_updated"), Demo.class);
```



