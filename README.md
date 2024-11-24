
在Java后端开发中，根据前端返回的字段名动态查询数据库是一种常见的需求。这种需求通常通过使用反射和动态SQL来实现。下面是一个完整的代码示例，它展示了如何根据前端返回的字段名动态查询数据库中的数据。


## 一、根据前端返回的字段名动态查询数据库中的数据示例


### 1\.准备工作


#### （1）数据库设置：


* 假设我们有一个名为users的表，结构如下：



```
CREATE TABLE users (
    id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(255),
    email VARCHAR(255),
    age INT
);

```
* 插入一些测试数据：



```
INSERT INTO users (name, email, age) VALUES ('Alice', 'alice@example.com', 30);
INSERT INTO users (name, email, age) VALUES ('Bob', 'bob@example.com', 25);

```


#### （2）依赖库：


* 使用`MySQL`作为数据库。
* 使用`JDBC`进行数据库连接。
* 使用`Spring Boot`简化配置和依赖管理（但示例中不涉及复杂的Spring框架功能，只关注核心逻辑）。


### 2\.核心代码


#### (1\)数据库连接配置


首先，在`src/main/resources`目录下创建一个`application.properties`文件，配置数据库连接信息：



```
spring.datasource.url=jdbc:mysql://localhost:3306/your_database_name
spring.datasource.username=your_database_username
spring.datasource.password=your_database_password
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver

```

#### (2\) 动态查询工具类


创建一个工具类`DynamicQueryUtil`，用于根据字段名生成动态SQL并执行查询：



```
import java.sql.*;
import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;
import java.util.Map;
 
import org.springframework.beans.factory.annotation.Value;
import org.springframework.stereotype.Component;
 
@Component
public class DynamicQueryUtil {
 
    @Value("${spring.datasource.url}")
    private String url;
 
    @Value("${spring.datasource.username}")
    private String username;
 
    @Value("${spring.datasource.password}")
    private String password;
 
    public List> queryByFields(List fields, String tableName) {
        List> results = new ArrayList<>();
        String sql = "SELECT " + String.join(", ", fields) + " FROM " + tableName;
 
        try (Connection conn = DriverManager.getConnection(url, username, password);
             Statement stmt = conn.createStatement();
             ResultSet rs = stmt.executeQuery(sql)) {
 
            while (rs.next()) {
                Map row = new HashMap<>();
                for (String field : fields) {
                    row.put(field, rs.getObject(field));
                }
                results.add(row);
            }
 
        } catch (SQLException e) {
            e.printStackTrace();
        }
 
        return results;
    }
}

```

#### (3\)控制器类


创建一个控制器类`UserController`，用于接收前端请求并调用动态查询工具类：



```
import java.util.List;
import java.util.Map;
 
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.*;
 
@RestController
@RequestMapping("/api/users")
public class UserController {
 
    @Autowired
    private DynamicQueryUtil dynamicQueryUtil;
 
    @GetMapping("/query")
    public List> queryUsers(@RequestParam List fields) {
        return dynamicQueryUtil.queryByFields(fields, "users");
    }
}

```

#### (4\)启动类


创建一个Spring Boot启动类：



```
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
 
@SpringBootApplication
public class DynamicQueryApplication {
 
    public static void main(String[] args) {
        SpringApplication.run(DynamicQueryApplication.class, args);
    }
}

```

### 3\.运行示例


(1\)启动Spring Boot应用。


(2\)使用浏览器或Postman等工具发送GET请求到`http://localhost:8080/api/users/query?fields=name,email`。


(3\)响应结果应类似于：



```
[
    {
        "name": "Alice",
        "email": "alice@example.com"
    },
    {
        "name": "Bob",
        "email": "bob@example.com"
    }
]

```

### 4\.注意事项


(1\)**安全性**：在实际应用中，需要对前端传入的字段名和表名进行校验，防止SQL注入攻击。


(2\)**性能**：频繁拼接SQL字符串并创建连接可能对性能有影响，应考虑使用连接池和缓存机制。


(3\)**扩展性**：可以使用更高级的ORM框架（如MyBatis或Hibernate）来简化数据库操作，并增强安全性和性能。


## 二、更详细的代码示例


下面是一个更详细的代码示例，它包含了完整的Spring Boot项目结构，并详细解释了每一步。这个示例将展示如何创建一个简单的Spring Boot应用，该应用能够根据前端请求的字段名动态地查询数据库中的数据。


### 1\.项目结构



```
dynamic-query-example
├── src
│   ├── main
│   │   ├── java
│   │   │   └── com
│   │   │       └── example
│   │   │           └── dynamicquery
│   │   │               ├── DynamicQueryApplication.java
│   │   │               ├── controller
│   │   │               │   └── UserController.java
│   │   │               ├── service
│   │   │               │   └── UserService.java
│   │   │               ├── service.impl
│   │   │               │   └── UserServiceImpl.java
│   │   │               ├── util
│   │   │               │   └── DynamicQueryUtil.java
│   │   │               └── model
│   │   │                   └── User.java
│   │   ├── resources
│   │   │   ├── application.properties
│   │   │   └── data.sql  (可选，用于初始化数据库)
│   └── test
│       └── java
│           └── com
│               └── example
│                   └── dynamicquery
│                       └── DynamicQueryApplicationTests.java
└── pom.xml

```

### 2\.`pom.xml`


首先，确保我们的`pom.xml`包含了必要的依赖项，如Spring Boot Starter Web和MySQL Connector。



```
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0modelVersion>
    <groupId>com.examplegroupId>
    <artifactId>dynamic-query-exampleartifactId>
    <version>0.0.1-SNAPSHOTversion>
    <packaging>jarpackaging>
    <name>dynamic-query-examplename>
    <description>Demo project for Spring Bootdescription>
    <parent>
        <groupId>org.springframework.bootgroupId>
        <artifactId>spring-boot-starter-parentartifactId>
        <version>2.7.0version>
        <relativePath/> 
    parent>
    <dependencies>
        <dependency>
            <groupId>org.springframework.bootgroupId>
            <artifactId>spring-boot-starter-webartifactId>
        dependency>
        <dependency>
            <groupId>mysqlgroupId>
            <artifactId>mysql-connector-javaartifactId>
            <scope>runtimescope>
        dependency>
        <dependency>
            <groupId>org.springframework.bootgroupId>
            <artifactId>spring-boot-starter-data-jpaartifactId>
        dependency>
        <dependency>
            <groupId>org.springframework.bootgroupId>
            <artifactId>spring-boot-starter-testartifactId>
            <scope>testscope>
        dependency>
    dependencies>
    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.bootgroupId>
                <artifactId>spring-boot-maven-pluginartifactId>
            plugin>
        plugins>
    build>
project>

```

### 3\.`application.properties`


配置数据库连接信息。



```
spring.datasource.url=jdbc:mysql://localhost:3306/your_database_name?useSSL=false&serverTimezone=UTC
spring.datasource.username=your_database_username
spring.datasource.password=your_database_password
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
spring.jpa.hibernate.ddl-auto=update
spring.jpa.show-sql=true
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.MySQL5Dialect

```

### 4\.数据模型 `User.java`


定义一个简单的`User`类，它对应数据库中的`users`表。



```
package com.example.dynamicquery.model;
 
import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.GenerationType;
import javax.persistence.Id;
 
@Entity
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String name;
    private String email;
    private Integer age;
 
    // Getters and Setters
}

```

### 5\.动态查询工具类 `DynamicQueryUtil.java`


这个类将负责根据字段名生成SQL并执行查询。注意，这个示例中我们不会直接使用它，而是用JPA来演示更现代的方法。但为了完整性，我还是提供了这个类的代码。



```
java复制代码

// ... (省略了DynamicQueryUtil类的代码，因为它在这个示例中不会被直接使用)

```

**注意**：在实际应用中，我们应该使用JPA的`EntityManager`或Spring Data JPA的`JpaRepository`来执行动态查询，而不是直接使用JDBC。下面的`UserServiceImpl`类将展示如何使用JPA来实现动态查询。


### 6\. 服务接口 `UserService.java`


定义一个服务接口。



```
package com.example.dynamicquery.service;
 
import java.util.List;
import java.util.Map;
 
public interface UserService {
    List> findUsersByFields(List fields);
}

```

### 7\.服务实现类 `UserServiceImpl.java`


首先，我们假设 `User` 类已经存在，并且包含了 `id`, `name`, `email`, 和 `age` 属性，以及相应的getter和setter方法。这里我们不会完整地展示 `User` 类，因为它通常很简单，只是包含一些基本的字段和JPA注解。


接下来，我们完善 `User_` 类，使其能够作为JPA元模型的模拟。在实际项目中，我们会使用JPA提供的元模型生成器来自动生成这些类。


以下是完整的 `UserServiceImpl.java` 代码，包括一个简化的 `User` 类定义和完善的 `User_` 类：



```
package com.example.dynamicquery.model;
 
import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.GenerationType;
import javax.persistence.Id;
 
@Entity
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String name;
    private String email;
    private Integer age;
 
    // Getters and Setters
    public Long getId() {
        return id;
    }
 
    public void setId(Long id) {
        this.id = id;
    }
 
    public String getName() {
        return name;
    }
 
    public void setName(String name) {
        this.name = name;
    }
 
    public String getEmail() {
        return email;
    }
 
    public void setEmail(String email) {
        this.email = email;
    }
 
    public Integer getAge() {
        return age;
    }
 
    public void setAge(Integer age) {
        this.age = age;
    }
}
 
---
 
package com.example.dynamicquery.service.impl;
 
import com.example.dynamicquery.model.User;
import com.example.dynamicquery.service.UserService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
 
import javax.persistence.EntityManager;
import javax.persistence.TypedQuery;
import javax.persistence.criteria.*;
import java.util.*;
 
@Service
public class UserServiceImpl implements UserService {
 
    @Autowired
    private EntityManager entityManager;
 
    @Override
    public List> findUsersByFields(List fields) {
        if (fields == null || fields.isEmpty()) {
            throw new IllegalArgumentException("Fields list cannot be null or empty");
        }
 
        CriteriaBuilder cb = entityManager.getCriteriaBuilder();
        CriteriaQuery cq = cb.createQuery(Object[].class);
        Root user = cq.from(User.class);
 
        List> selections = new ArrayList<>();
        for (String field : fields) {
            switch (field) {
                case "id":
                    selections.add(user.get("id"));
                    break;
                case "name":
                    selections.add(user.get("name"));
                    break;
                case "email":
                    selections.add(user.get("email"));
                    break;
                case "age":
                    selections.add(user.get("age"));
                    break;
                default:
                    throw new IllegalArgumentException("Unknown field: " + field);
            }
        }
 
        cq.select(cb.array(selections.toArray(new Selection[0])));
        TypedQuery query = entityManager.createQuery(cq);
 
        List results = query.getResultList();
        List> userList = new ArrayList<>();
        for (Object[] result : results) {
            Map userMap = new HashMap<>();
            for (int i = 0; i < fields.size(); i++) {
                userMap.put(fields.get(i), result[i]);
            }
            userList.add(userMap);
        }
 
        return userList;
    }
 
    // 静态内部类用于模拟JPA的元模型（Metamodel），实际项目中应使用自动生成的元模型
    // 注意：在实际项目中，我们不需要手动编写这个类，JPA提供者会自动为我们生成。
    // 这里只是为了演示目的而包含它。
    private static class User_ {
        // 这些字段是模拟的，实际中应由JPA工具自动生成
        public static final SingularAttribute id = mockAttribute("id");
        public static final SingularAttribute name = mockAttribute("name");
        public static final SingularAttribute email = mockAttribute("email");
        public static final SingularAttribute age = mockAttribute("age");
 
        // 模拟方法，实际中不存在
        private static  SingularAttribute mockAttribute(String name) {
            return null; // 实际返回的是由JPA提供者生成的SingularAttribute实例
        }
    }
 
    // 注意：上面的mockAttribute方法是为了编译通过而添加的，实际代码中应该移除。
    // 在实际项目中，我们应该直接使用JPA提供的元模型类，而不是这个模拟的User_类。
    // 由于这个示例是为了演示动态查询，我们保留了User_类，但在实际应用中应忽略它。
}

```

**重要说明**：


（1）在实际项目中，我们应该使用JPA提供者（如Hibernate）自动生成的元模型类，而不是上面的 `User_` 类。这些类通常位于与实体类相同的包中，并且以 `_` 后缀命名（例如，`User_`）。


（2）上面的 `mockAttribute` 方法是为了编译通过而添加的，实际代码中应该移除。这个方法在实际项目中不存在，因为它只是模拟了JPA元模型的行为。


（3）在调用 `user.get(...)` 时，我们直接使用了字符串属性名（例如 `"id"`, `"name"` 等）。在实际项目中，我们应该使用JPA元模型类中的静态字段来引用这些属性，以提高类型安全性和重构能力。例如，我们应该使用 `User_.id` 而不是 `"id"`。但是，由于我们在这个示例中模拟了元模型，所以我们直接使用了字符串。


（4）在实际项目中，我们可能还需要处理一些额外的边界情况，比如字段名的大小写敏感性、空值处理等。


（5）考虑到性能和安全性，动态查询应该谨慎使用，并确保传入的字段名是经过验证和授权的。


## 三、内置的`http.server`模块来创建一个基本的HTTP服务器


这里将以Python语言编写一个简单的Web服务器为例，使用内置的`http.server`模块来创建一个基本的HTTP服务器。这个示例将展示如何启动一个服务器，并在特定端口上监听请求，然后返回一个简单的HTML响应。


### 1\.代码示例



```
# 导入必要的模块
from http.server import SimpleHTTPRequestHandler, HTTPServer
 
# 定义一个自定义的请求处理器类，继承自SimpleHTTPRequestHandler
class MyRequestHandler(SimpleHTTPRequestHandler):
    def do_GET(self):
        # 设置响应状态码
        self.send_response(200)
        
        # 设置响应头
        self.send_header('Content-type', 'text/html')
        self.end_headers()
        
        # 准备响应体
        response_body = """
        
        
        
            Simple Web Server
        
        
            # Hello, World!


            This is a simple web server running on Python.


        
        
        """
        
        # 发送响应体
        self.wfile.write(response_body.encode('utf-8'))
 
# 定义服务器的地址和端口
server_address = ('', 8080)
 
# 创建HTTP服务器对象，传入服务器地址和请求处理器类
httpd = HTTPServer(server_address, MyRequestHandler)
 
# 打印服务器启动信息
print("Starting httpd server on port 8080...")
 
# 启动服务器，开始监听请求
httpd.serve_forever()

```

### 2\.代码说明：


（1）**导入模块**：首先，我们导入了`SimpleHTTPRequestHandler`和`HTTPServer`模块，这两个模块是Python标准库中用于创建HTTP服务器的。


（2）**定义请求处理器**：我们创建了一个名为`MyRequestHandler`的类，继承自`SimpleHTTPRequestHandler`。在这个类中，我们重写了`do_GET`方法，用于处理GET请求。


（3）**设置响应**：在`do_GET`方法中，我们首先设置了响应的状态码（200表示成功），然后设置了响应头（指定内容类型为HTML），最后准备了响应体（一个简单的HTML页面）。


（4）**启动服务器**：我们定义了服务器的地址和端口（这里监听所有接口的8080端口），然后创建了`HTTPServer`对象，并传入服务器地址和我们自定义的请求处理器类。最后，调用`serve_forever`方法启动服务器，使其开始监听请求。


### 3\.运行代码：


将上述代码保存为一个Python文件（例如`simple_server.py`），然后在命令行中运行该文件：



```
bash复制代码

python simple_server.py

```

服务器启动后，我们可以在浏览器中访问`http://localhost:8080`，我们将看到一个简单的HTML页面，显示“Hello, World!”和一条消息说明这是一个简单的Web服务器。


这个示例展示了如何使用Python标准库中的模块创建一个基本的Web服务器，并处理HTTP GET请求。根据需要，我们可以进一步扩展这个示例，添加更多的请求处理方法，处理POST请求，或者从请求中获取参数等。


 本博客参考[西部世界官网](https://tianchuang88.com)。转载请注明出处！
