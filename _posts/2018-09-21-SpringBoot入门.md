---
layout:     post
title:      SpringBoot入门
subtitle:   从零开始写接口
date:       2018-09-21
author:     Edward
header-img: img/post-bg-coffee.jpeg
---
本人并不是做后台开发的，没有后端经验，在搜索了大量资料并尝试后，终于运行起来了接口服务，并用postman测试通过。
本文就是利用SpringBoot框架以及相关开发工具，以生手的方式记录实践过程。

### 一、环境搭建

#### 安装 [JDK](https://www.oracle.com/technetwork/java/javase/downloads/index.html)
JDK安装以及环境变量配置,这个没什么好说的，都会，网上也有很多
#### 安装 [IntelliJ IDEA](https://www.jetbrains.com/idea/)
开发工具，这个也没什么好说的
#### 安装 [MySQL Community Server](https://dev.mysql.com/downloads/mysql/)

1. 安装配置         
   下载完成后，解压zip包到合适的安装目录 如：D:\mysql-8.0.12,在该文件夹下创建 my.ini 配置文件，编辑 my.ini 配置以下基本信息：    
```
[mysql]
default-character-set=utf8
[mysqld]
port = 3306
basedir=C:\web\mysql-8.0.11
datadir=C:\web\sqldata
max_connections=20
character-set-server=utf8
default-storage-engine=INNODB
```
2. 初始化数据库     
   以管理员身份打开 cmd 命令行工具，切换目录：cd D:\mysql-8.0.12,也可以将该目录添加到环境变量中,执行以下命令，进行初始化：
```
mysqld --initialize --console
//执行完成后，会输出 root 用户的初始默认密码 root@localhost: APWCY5ws&hjQ
... A temporary password is generated for root@localhost: APWCY5ws&hjQ
```
3. 安装并启动
```
//安装
mysqld install
//启动
net start mysql
```
4. 登录并设置密码
```
//登录
mysql -u root -p
//修改密码
mysql> set password = '123456';
//
```
可以使用[MySQL Workbench](https://dev.mysql.com/downloads/workbench/) 或 [Navicat](https://www.navicat.com.cn/)
等数据库可视化工具进行数据库管理，具体使用方法可自行搜索。IntelliJ IDEA 自带一个简单的数据库可视化管理工具。

### 二、创建工程
1. 打开IntelliJ IDEA，创建SpringBoot工程，就是列表里的 Sping initializer，然后按照提示填写项目名，包名，路径，然后选择Web依赖，项目创建完成后，会自动生成Application类并创建程序入口
```
@SpringBootApplication
public class Application {

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```
2. 根据需求创建自己的接口类(Controller),如写一个登录接口:
```
@Controller
public class UserController {

    @RequestMapping(value = "/login", method = RequestMethod.POST)
    @ResponseBody
    public String login(String account, String password) {
        // TODO: 数据库查询以及逻辑操作
        return "登录成功";
    }
}
```

- 在类名上添加注解 `@Controller`
- GET,POST请求注解  `@RequestMapping(method = RequestMethod.POST)`
- 方法名注解 `@RequestMapping("/login")` 这个就是请求接口的方法名，如：htttp://localhost:8080/login
- 方法参数 `account, password` 就是接口参数  

这个时候启动项目，接口已经可以访问了，但这个接口只是返回了一个字符串"登录成功"，没有做任何操作，实际的接口应该是从数据库中查找数据，然后返回json  

### 三、数据库
1.首先要创建数据库，作为新手，直接用可视化工具创建就好。然后在项目resources目录下的application.properties中进行配置:
```
##端口号
server.port=8001

spring.datasource.url=jdbc:mysql://localhost:3306/demo?
spring.datasource.username=root
spring.datasource.password=123456
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
```
时区报错，可以设置时区: `spring.datasource.url=jdbc:mysql://localhost:3306/demo?serverTimezone=UTC`  
字符集问题,可以指定输出字符集`spring.datasource.url=jdbc:mysql://localhost:3306/demo?characterEncoding=UTF-8`  
也可以命令行登录进入数据库进行修改。 

2. 创建Service,使用JdbcTemplate:
```
@Service
public class UserService {

    private User rowMapper = new User();
    final JdbcTemplate template;

    public UserService(JdbcTemplate template) {
       this.template = template;
    }

    /**
     * 查询
     */
    public User findUser(String name,String psd) {
        String sql = "SELECT * FROM user WHERE name = '" + name+"' AND password = '"+psd+"'  LIMIT 1";
        return template.queryForObject(sql, rowMapper);
    }
```

其中User类要实现 RowMapper 接口,建立User类属性与表中字段的映射关系
```
public class User implements RowMapper<User> {

    private String name;
    private String password;
    private int age;

    // get set
  
    @Override
    public Coffee mapRow(ResultSet rs, int rowNum) throws SQLException {
        User map = new User();
        map.setName(rs.getInt("name"));
        map.setAge(rs.getString("age"));
         map.setPassword(rs.getString("password"));
        return map;
    }
}
```
3. 在Controller中使用Service
```
@Controller
public class UserController {

    private final UserService service;
    
    @Autowired
    public UserController(UserService service) {
        this.service = service;
    }
    
    
    @RequestMapping(value = "/login", method = RequestMethod.POST)
    @ResponseBody
    public String login(String account, String password) {
        User user = service.findUser(account,password);
        if(user==null){
        return "用户名或密码不正确";
        }
        return "登录成功";
    }
}
```
这里只是简单写一下，实际上需要进行参数判断以及根据需求进行逻辑操作；  
返回Json，结合各种Json框架，写一个返回数据模板就好了。
