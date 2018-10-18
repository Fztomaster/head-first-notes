# Shiro

## 一.Shiro概述

### 1.Shiro简介

Shiro官网：http://shiro.apache.org/

百度百科：https://baike.baidu.com/item/shiro/17753571?fr=aladdin

Apache Shiro是一个强大且易用的Java安全框架,执行身份验证、授权、密码和会话管理 

三个核心组件：Subject, SecurityManager 和 Realms. 

- Subject：即“当前操作用户” ，Subject代表了当前用户的安全操作 
- SecurityManager：它是Shiro框架的核心 ，管理内部组件实例，并通过它来提供安全管理的各种服务 
- Realm： Realm充当了Shiro与应用安全数据间的“桥梁”或者“连接器”。也就是说，当对用户执行认证（登录）和授权（访问控制）验证时，Shiro会从应用配置的Realm中查找用户及其权限信息 

Realm实质上是一个安全相关的DAO：它封装了数据源的连接细节，并在需要时将相关数据提供给Shiro 

### 2.Shiro入门程序HelloShiro

建立一个maven工程com.jack.shiro.shiro_demo

1.pom.xml导入shiro相关jar包

```xml
<dependencies>
<!-- 引入shiro的核心包 -->
    <dependency>
        <groupId>org.apache.shiro</groupId>
        <artifactId>shiro-core</artifactId>
        <version>1.2.4</version>
    </dependency>
    <dependency>
        <groupId>org.slf4j</groupId>
        <artifactId>slf4j-log4j12</artifactId>
        <version>1.7.25</version>
    </dependency>
</dependencies>
```

2.创建shiro.ini(配置文件放resources)

```ini
[users]
jack=123456
rose=123456
```

3.编写测试程序HelloShiro(创建com.jack.shiro包)

```java
import org.apache.shiro.SecurityUtils;
import org.apache.shiro.authc.AuthenticationException;
import org.apache.shiro.authc.UsernamePasswordToken;
import org.apache.shiro.config.IniSecurityManagerFactory;
import org.apache.shiro.mgt.SecurityManager;
import org.apache.shiro.subject.Subject;
import org.apache.shiro.util.Factory;

public class HelloShiro {
	
	public static void main(String[] args) {
		// 读取配置文件，初始化SecurityManager工厂
		Factory<SecurityManager> factory = new IniSecurityManagerFactory("classpath:shiro.ini");
		// 获取securityManager实例
		SecurityManager securityManager = factory.getInstance();
		// 把securityManager实例绑定到SecurityUtils
		SecurityUtils.setSecurityManager(securityManager);
		// 得到当前执行的用户
		Subject currentUser = SecurityUtils.getSubject();
		// 创建token令牌，用户名/密码
		UsernamePasswordToken token = new UsernamePasswordToken("jack", "123456");
		try {
			currentUser.login(token);
			System.out.println("身份认证成功!");
		} catch (AuthenticationException e) {
			e.printStackTrace();
			System.out.println("身份认证失败!");
		}
		// 退出
		currentUser.logout();
	}
}
```

## 二.身份认证

### 1.Subject认证主体

```
Subject 认证主体包含两个信息：
Principals：身份，可以是用户名，邮件，手机号码等等，用来标识一个登录主体身份；
Credentials：凭证，常见有密码，数字证书等等；
```

### 2.身份认证流程

![](./photo/shiro-1.png)

### 3.Realm和JdbcRealm

Realm：意思是域，Shiro 从 Realm 中获取验证数据；
Realm 有很多种类，例如常见的 jdbc realm，jndi realm，text realm。

### 4.demo案例

创建maven项目工程com.jack.shiro.shiro_demo2

1.pom.xml文件

```xml
 <dependencies>
 	<!-- shiro核心包 -->
 	<dependency>
		<groupId>org.apache.shiro</groupId>
		<artifactId>shiro-core</artifactId>
		<version>1.2.4</version>
    </dependency>
    <dependency>
		<groupId>org.slf4j</groupId>
		<artifactId>slf4j-log4j12</artifactId>
		<version>1.7.12</version>
	</dependency>
	<!-- c3p0连接池 -->
	<dependency>
		<groupId>c3p0</groupId>
		<artifactId>c3p0</artifactId>
		<version>0.9.1.2</version>
	</dependency>
	<dependency>
		<groupId>commons-logging</groupId>
		<artifactId>commons-logging</artifactId>
		<version>1.2</version>
	</dependency>
	<!-- mysql的驱动包 -->
	<dependency>
		<groupId>mysql</groupId>
		<artifactId>mysql-connector-java</artifactId>
		<version>5.1.37</version>
	</dependency>
 </dependencies>
```

2.jdbc_realm.ini配置文件

```ini
[main]
#采用第三方JdbcRealm连接数据库
jdbcRealm=org.apache.shiro.realm.jdbc.JdbcRealm

#实例化数据源
dataSource=com.mchange.v2.c3p0.ComboPooledDataSource

#设置参数
dataSource.driverClass=com.mysql.jdbc.Driver
dataSource.jdbcUrl=jdbc:mysql://localhost:3306/db_shiro
dataSource.user=root
dataSource.password=root

#将数据源设置到realm中
jdbcRealm.dataSource=$dataSource
securityManager.realms=$jdbcRealm
```

3.编写测试类JdbcRealmTest

```java
import org.apache.shiro.SecurityUtils;
import org.apache.shiro.authc.AuthenticationException;
import org.apache.shiro.authc.UsernamePasswordToken;
import org.apache.shiro.config.IniSecurityManagerFactory;
import org.apache.shiro.mgt.SecurityManager;
import org.apache.shiro.subject.Subject;
import org.apache.shiro.util.Factory;

public class JdbcRealmTest {
	
	public static void main(String[] args) {
		// 读取配置文件，初始化SecurityManager工厂
		Factory<SecurityManager> factory = new IniSecurityManagerFactory("classpath:jdbc_realm.ini");
		// 获取securityManager实例
		SecurityManager securityManager = factory.getInstance();
		// 把securityManager实例绑定到SecurityUtils
		SecurityUtils.setSecurityManager(securityManager);
		// 得到当前执行的用户
		Subject currentUser = SecurityUtils.getSubject();
		// 创建token令牌，用户名/密码
		UsernamePasswordToken token = new UsernamePasswordToken("jack", "123456");
		try {
			currentUser.login(token);
			System.out.println("身份认证成功!");
		} catch (AuthenticationException e) {
			e.printStackTrace();
			System.out.println("身份认证失败!");
		}
		// 退出
		currentUser.logout();
	}
}
```

