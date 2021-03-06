---
layout:     post
title:      SpringBoot整合mybatis注解版和druid连接池的使用
subtitle:   
date:       2019-10-23
author:     canloveme
header-img: img/home-bg-o.jpg
catalog: true
tags:
    - Blog
---


## SpringBoot整合Mybatis以及使用druid连接池的使用

### 1、引入依赖文件

**注意springboot没有自带的mybatis的starter**

```xml
<dependency>
     <groupId>org.mybatis.spring.boot</groupId>
     <artifactId>mybatis-spring-boot-starter</artifactId>
     <version>2.1.1</version>
 </dependency>
```

这是mybatis官方写的springboot启动器

引入druid连接池依赖和mysql驱动

```xml
 <!-- druid -->
 <dependency>
     <groupId>com.alibaba</groupId>
     <artifactId>druid-spring-boot-starter</artifactId>
     <version>1.1.10</version>
  </dependency>
<dependency>
   <groupId>mysql</groupId>
   <artifactId>mysql-connector-java</artifactId>
   <scope>runtime</scope>
</dependency>
```



### 2、配置数据源

```yaml
spring:
  datasource:
    type: com.alibaba.druid.pool.DruidDataSource
    url: jdbc:mysql://localhost:3306/springboot?      	serverTimezone=UTC
    username: root
    password: 123456
    driver-class-name: com.mysql.cj.jdbc.Driver
```

使用type:可指定连接池，因使用的mysql版本为8.0所以url中连接数据库时需加上`serverTimezone=UTC`来解决时区的问题，此时的driver也要使用新Driver加上cj。

### 3、对应的entity和mapper

我们在这里使用一个user表为例。插入两条测试数据。

```sql
-- ----------------------------
-- Table structure for user
-- ----------------------------
DROP TABLE IF EXISTS `user`;
CREATE TABLE `user` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT,
  `username` varchar(255) DEFAULT NULL,
  `password` varchar(255) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=3 DEFAULT CHARSET=utf8;

-- ----------------------------
-- Records of user
-- ----------------------------
INSERT INTO `user` VALUES ('1', 'admin', 'admin');
INSERT INTO `user` VALUES ('2', 'csq', '123456');

```

#### 对应的entity和mapper

```java
@Data
public class User {
    private Integer id;
    private String username;
    private String password;
}

```

这里使用到了import lombok.Data;可在创建工程时加入依赖。

![1571758327227](C:\Users\20161123\AppData\Roaming\Typora\typora-user-images\1571758327227.png)

关于lombok的具体使用可参考官网。

```java
@Mapper
public interface UserMapper {

    /**
     * 根据id查找user
     * @param id id
     * @return 返回user
     */
    @Select("select * from user where id = #{id}")
    public User getUserById(Integer id);

    /**
     * 插入user
     * @param user 插入的实体
     * @return 返回插入的数据
     */
    @Options(useGeneratedKeys = true,keyProperty ="id")
    @Insert("insert into user(username,password) values(#{username},#{password})")
    public int insert(User user);
    
}
```

其中@select、@insert等注解为mybatis的注解，其中带上相应的sql语句。

@Options表示主键。相应注解可查看mybatis在github上的文档。

 https://mybatis.org/mybatis-3/zh/getting-started.html 

使用mapper注解标注此类为一个mybatis的mapper类。也可以在启动类上使用@MapperScan标注mapper所在的包一次性注入。

```java
@MapperScan("cn.csq.springbootdruid.mapper")
@SpringBootApplication
public class SpringBootDruidApplication {

    public static void main(String[] args) {
        SpringApplication.run(SpringBootDruidApplication.class, args);
    }
}

```

### 4、测试和contorller

这时可在springboot自带的test类中进行userMapper的测试。

```java
@SpringBootTest
public class UserMapperTest {
    @Autowired
    UserMapper userMapper;

    @Test
    public  void getUserById(){
        User user = userMapper.getUserById(1);
        System.out.println(user);
    }
}
```

其中特别需要注意的是测试方法不要有传入参数。

![G}[16LUO7{G7MP]0X5OMLSY](E:\QQ文件\MobileFile\Image\G}[16LUO7{G7MP]0X5OMLSY.png)

此时可知getUserById方法没有错误，userMapper的注解可以使用。

可以使用Controller来进行使用。

```java
@RestController
public class userController {
    @Autowired
    UserMapper userMapper;

    @GetMapping("/user/{id}")
    public User getUserById(@PathVariable("id") Integer id) {
        return userMapper.getUserById(id);
    }

    @GetMapping("/user")
    public User insert(User user) {
        userMapper.insert(user);
        return user;
    }
}
```

![K43_@LN70T4T$2M1469W$23](E:\QQ文件\MobileFile\Image\K43_@LN70T4T$2M1469W$23.png)

测试成功。crud省略。

### 5、druid的使用

配置好druid后，可使用druid管理很多东西。

![5FGS$S@[]CSC@1TK@WS%1MX](E:\QQ文件\MobileFile\Image\5FGS$S@[]CSC@1TK@WS%1MX.png)

6、整合xml版的mybatis

将这部分和MyBatis Generator的使用放一起吧。

==不管使用注解版还是xml版，将mapper注入是最重要的。==使用mapper注解或mapperscan
