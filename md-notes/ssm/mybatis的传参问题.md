<!-- TOC -->

- [mybatis的传参问题<!-- TOC -->](#mybatis的传参问题---toc---)
    - [一.准备工作](#一准备工作)
        - [1.pom.xml文件](#1pomxml文件)
        - [2.sqlMapConfig.xml](#2sqlmapconfigxml)
        - [3.jdbc.properties](#3jdbcproperties)
        - [4.log4j.properties](#4log4jproperties)
        - [5.javabean User类](#5javabean-user类)
    - [二.@Param传参](#二param传参)
        - [1.UserDao接口](#1userdao接口)
        - [2.UserDao.xml](#2userdaoxml)
        - [3.测试类MybatisTest](#3测试类mybatistest)
    - [三.map传参](#三map传参)
        - [1.UserDao接口](#1userdao接口-1)
        - [2.UserDao.xml](#2userdaoxml-1)
        - [3.测试类MybatisTest](#3测试类mybatistest-1)
    - [四.javabean传参](#四javabean传参)
        - [1.UserDao接口](#1userdao接口-2)
        - [2.UserDao.xml](#2userdaoxml-2)
        - [3.测试类MybatisTest](#3测试类mybatistest-2)

<!-- /TOC -->

## 一.准备工作
### 1.pom.xml文件
```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.jack</groupId>
    <artifactId>mybatis_day02_paramsPass</artifactId>
    <version>1.0-SNAPSHOT</version>
    <packaging>jar</packaging>

    <dependencies>
        <!--mybatis的核心包-->
        <dependency>
            <groupId>org.mybatis</groupId>
            <artifactId>mybatis</artifactId>
            <version>3.4.5</version>
        </dependency>
        <!--数据库驱动包-->
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>5.1.18</version>
        </dependency>
        <!--日志包-->
        <dependency>
            <groupId>log4j</groupId>
            <artifactId>log4j</artifactId>
            <version>1.2.12</version>
        </dependency>
        <!--测试包-->
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.12</version>
        </dependency>
    </dependencies>

</project>
```
### 2.sqlMapConfig.xml
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<!--用来连接数据库、加载映射文件-->
<configuration>
    <!--外部配置数据库连接信息-->
    <!--方式一-->
    <properties resource="jdbc.properties"/>
    <!--方式二-->
    <!--<properties url="file:\\\E:\learn\code\ssm\mybatis_day02_daoImpl\src\main\resources\jdbc.properties"/>-->

    <!--配置别名，不区分大小写-->
    <typeAliases>
        <!--类配置别名-->
        <typeAlias type="com.jack.domain.User" alias="user"/>
        <!--包配置别名，扫描整个包-->
        <!--<package name="com.jack.domain"/>-->
    </typeAliases>
    <!--定义mysql的环境-->
    <environments default="mysql">
        <!--声明一个mysql-->
        <environment id="mysql">
            <!--事务管理器，控制事务，固定的写法JDBC-->
            <transactionManager type="JDBC"/>
            <!--定义数据源POOLED(连接池)、UNPOOLED、JNDI-->
            <dataSource type="POOLED">
                <property name="driver" value="${jdbc.driver}"/>
                <property name="url" value="${jdbc.url}"/>
                <property name="username" value="${jdbc.username}"/>
                <property name="password" value="${jdbc.password}"/>
            </dataSource>
        </environment>
    </environments>

    <!--加载配置文件-->
    <mappers>
        <!--配置文件方式-->
        <!--<mapper resource="com/jack/dao/UserDao.xml"/>-->
        <!--注解方式-->
        <!--<mapper class="com.jack.dao.UserDao"/>-->
        <!--两种方式都可以-->
        <package name="com.jack.dao"/>
    </mappers>
</configuration>
```
### 3.jdbc.properties
```properties
jdbc.driver=com.mysql.jdbc.Driver
jdbc.url=jdbc:mysql://localhost:3306/mybatis
jdbc.username=root
jdbc.password=root
```
### 4.log4j.properties
```properties
# Set root category priority to INFO and its only appender to CONSOLE.
#log4j.rootCategory=INFO, CONSOLE            debug   info   warn error fatal
log4j.rootCategory=debug, CONSOLE, LOGFILE

# Set the enterprise logger category to FATAL and its only appender to CONSOLE.
log4j.logger.org.apache.axis.enterprise=FATAL, CONSOLE

# CONSOLE is set to be a ConsoleAppender using a PatternLayout.
log4j.appender.CONSOLE=org.apache.log4j.ConsoleAppender
log4j.appender.CONSOLE.layout=org.apache.log4j.PatternLayout
log4j.appender.CONSOLE.layout.ConversionPattern=%d{ISO8601} %-6r [%15.15t] %-5p %30.30c %x - %m\n

# LOGFILE is set to be a File appender using a PatternLayout.
log4j.appender.LOGFILE=org.apache.log4j.FileAppender
log4j.appender.LOGFILE.File=e:/log/mybatis.log
log4j.appender.LOGFILE.Append=true
log4j.appender.LOGFILE.layout=org.apache.log4j.PatternLayout
log4j.appender.LOGFILE.layout.ConversionPattern=%d{ISO8601} %-6r [%15.15t] %-5p %30.30c %x - %m\n
```
### 5.javabean User类
放在main/java/com.jack.domain包下
```java
public class User {
    private Integer id;
    private String username;
    private Date birthday;
    private String sex;
    private String address;

    public Integer getId() {
        return id;
    }

    public void setId(Integer id) {
        this.id = id;
    }

    public String getUsername() {
        return username;
    }

    public void setUsername(String username) {
        this.username = username;
    }

    public Date getBirthday() {
        return birthday;
    }

    public void setBirthday(Date birthday) {
        this.birthday = birthday;
    }

    public String getSex() {
        return sex;
    }

    public void setSex(String sex) {
        this.sex = sex;
    }

    public String getAddress() {
        return address;
    }

    public void setAddress(String address) {
        this.address = address;
    }

    @Override
    public String toString() {
        return "User{" +
                "id=" + id +
                ", username='" + username + '\'' +
                ", birthday=" + birthday +
                ", sex='" + sex + '\'' +
                ", address='" + address + '\'' +
                '}';
    }
}
```
## 二.@Param传参
### 1.UserDao接口
main/java/com/jack/dao/UserDao.java
```java
public interface UserDao {

    /**
     * 根据username和id查询User
     * 第一种传参：使用@Param传递多个参数
     * 应用场景：传递参数<=5
     * @param username
     * @param id
     * @return
     */
    User selectUser(@Param("username") String username, @Param("id") Integer id);
}
```
### 2.UserDao.xml
resources/com/jack/dao/UserDao.xml
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.jack.dao.UserDao">
    <select id="selectUser" resultType="user">
        select * from user where username=#{username} and id=#{id}
    </select>
</mapper>
```
### 3.测试类MybatisTest
test/java/com/jack/test/MybatisTest.java
```java
public class MybatisTest {
    private InputStream in;
    private SqlSession sqlSession;
    private UserDao userDao;
    private User user = new User();

    @Before
    public void init() throws Exception {
        in = Resources.getResourceAsStream("sqlMapConfig.xml");
        SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(in);
        sqlSession = sqlSessionFactory.openSession();
        userDao = sqlSession.getMapper(UserDao.class);
    }

    @Test
    public void testSelectUser() {
        user = userDao.selectUser("jack", 54);
        System.out.println(user);
    }

    @After
    public void close() throws Exception {
        sqlSession.commit();
        sqlSession.close();
        in.close();
    }
}
```
## 三.map传参
### 1.UserDao接口
```java
public interface UserDao {

    /**
     * 根据username和id查询User
     * 第一种传参：使用@Param传递多个参数
     * 应用场景：传递参数<=5
     * @param username
     * @param id
     * @return
     */
    User selectUser(@Param("username") String username, @Param("id") Integer id);

    /**
     * 第二种传参：使用map传递参数
     * @param params
     * @return
     */
    User selectUser2(Map<String, Object> params);

    /**
     * 第三种传参：使用javabean
     */
    User selectUser3(User user);
}
```
### 2.UserDao.xml
```xml
<?xml version="1.0" encoding="UTF-8"?>
        <!DOCTYPE mapper
                PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
                "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.jack.dao.UserDao">
<select id="selectUser2" parameterType="map" resultType="user">
        select * from user where username=#{username} and id=#{id}
</select>
```
### 3.测试类MybatisTest
```java
public class MybatisTest {
    private InputStream in;
    private SqlSession sqlSession;
    private UserDao userDao;
    private User user = new User();

    @Before
    public void init() throws Exception {
        in = Resources.getResourceAsStream("sqlMapConfig.xml");
        SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(in);
        sqlSession = sqlSessionFactory.openSession();
        userDao = sqlSession.getMapper(UserDao.class);
    }

    @Test
    public void testSelectUser2() {
        Map<String, Object> map = new HashMap<String, Object>();
        map.put("username", "jack");
        map.put("id", 54);
        user = userDao.selectUser2(map);
        System.out.println(user);
    }

    @After
    public void close() throws Exception {
        sqlSession.commit();
        sqlSession.close();
        in.close();
    }
}
```
## 四.javabean传参
### 1.UserDao接口
```java
public interface UserDao {
    /**
     * 第三种传参：使用javabean
     */
    User selectUser3(User user);
}
```
### 2.UserDao.xml
```xml
<?xml version="1.0" encoding="UTF-8"?>
        <!DOCTYPE mapper
                PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
                "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.jack.dao.UserDao">
<select id="selectUser3" parameterType="user" resultType="user">
      select * from user where username=#{username} and id=#{id}
</select>
</mapper>
```
### 3.测试类MybatisTest
```java
public class MybatisTest {
    private InputStream in;
    private SqlSession sqlSession;
    private UserDao userDao;
    private User user = new User();

    @Before
    public void init() throws Exception {
        in = Resources.getResourceAsStream("sqlMapConfig.xml");
        SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(in);
        sqlSession = sqlSessionFactory.openSession();
        userDao = sqlSession.getMapper(UserDao.class);
    }

    @Test
    public void testSelectUser3() {
        user.setUsername("jack");
        user.setId(54);
        user = userDao.selectUser3(user);
        System.out.println(user);
    }

    @After
    public void close() throws Exception {
        sqlSession.commit();
        sqlSession.close();
        in.close();
    }
}
```
