## JdbcTemplate
### 构造方法
```java
	JdbcTemplate(DataSource dataSource); // c3p0、druid
```
### 常用方法
```java
	void execute(String sql); // 执行DCL和DDL
	int update(String sql, Object... params); // 执行DML：insert、update、delete
	queryXxx(); // 执行DQL
```
### JdbcTemplate使用
* 导入jar包
	* JdbcTemplate相关jar包：5个
		* spring-beans-5.0.0.RELEASE.jar
		* spring-core-5.0.0.RELEASE.jar
		* spring-jdbc-5.0.0.RELEASE.jar
		* spring-tx-5.0.0.RELEASE.jar
		* commons-logging-1.2.jar
	* 连接池的包
		* mysql-connector-java-5.1.37-bin.jar
	* 数据库驱动包
		* c3p0-0.9.2-pre5.jar
		* mchange-commons-java-0.2.3.jar
		* druid-1.1.0.jar
* 执行sql操作
	* int execute(String sql, Object... params);
	* queryXxx();
### DQL相关的API(重点)
#### 查询所有数据, 封装成List<Bean>，常用
```java
	 List<JavaBean> beans = template.query(String sql, new BeanPropertyRowMapper<>(JavaBean.class), Object... params);
```
#### 查询一条数据, 封装成JavaBean，常用
```java
	JavaBean bean = template.queryForObject(String sql, new BeanPropertyRowMapper<>(JavaBean.class), Object... params);
```
#### 查询单值，常用
```java
	long num = template.queryForObject(String sql, Long.class, Object... params);
```
#### 查询一条数据封装成Map，了解
```java
	Map<字段名, 字段值> map = template.queryForMap(String sql, Object... params); // 一行数据就是一个Map
```
#### 查询多条数据封装成List，了解
```java
	List<Map<字段名, 字段值>> list = template.queryForList(String sql, Object... params); // List里的每一个元素就是一行数据的Map
```
### JdbcTemplate Demo
#### c3p0-config.xml配置文件
```xml
	<c3p0-config>
    <!-- 使用默认的配置获取连接池对象 -->
    <default-config>
        <!--连接参数-->
        <property name="driverClass">com.mysql.jdbc.Driver</property>
        <property name="jdbcUrl">jdbc:mysql://localhost:3306/test</property>
        <property name="user">root</property>
        <property name="password">root</property>

        <!--连接池参数-->
        <property name="initialPoolSize">5</property>
        <property name="maxPoolSize">10</property>
        <property name="checkoutTimeout">2000</property>
        <property name="maxIdleTime">1000</property>
    </default-config>
</c3p0-config>
```
#### druid.properties配置文件
```properties
	driverClassName=com.mysql.jdbc.Driver
	url=jdbc:mysql://127.0.0.1:3306/user
	username=root
	password=root
	initialSize=5
	maxActive=10
	maxWait=3000
	minIdle=3
```
#### JdbcUtils工具类-c3p0
```java
	public class JdbcUtils {
		// 创建连接池对象，自动从src/下加载c3p0-config.xml
        private static ComboPooledDataSource dataSource = new ComboPooledDataSource();
        // 获取连接
        public static Connection getConnection() throws SQLException {
            return dataSource.getConnection();
        }
        // 获取连接池对象
        public static DataSource getDataSource() {
            return dataSource;
        }
        // 释放资源
        public static void close(Result rs, Statement stmt, Connection conn) {
            if (rs != null) {
                try {
                    rs.close();
                } catch(SQLException e) {
                    e.printStackTrace();
                }
            }
            if (stmt != null) {
                try {
                    stmt.close();
                } catch(SQLException e) {
                    e.printStackTrace();
                }
            }
            if (conn != null) {
                try {
                    conn.close();
                } catch(SQLException e) {
                    e.printStackTrace();
                }
            }
        }
	}
```
#### JdbcUtils工具类-druid
```java
public class JdbcUtils {
    private static Properties prop = null;
    private static DataSource dataSource = null;
    // 获取连接
    public static Connection getConnection() throws Exception {
    	return dataSource.getDataSource();
    }
    // 获取数据库连接池
    public static DataSource getDataSource() {
        InputStream input = JdbcUtils.class.getClassLoader().getResourceAsStream("druid.properties");
        try {
        prop = new Properties();
        prop.load(input);
        dataSource = DruidDataSourceFactory.createDataSource(prop);
        } catch(Exception e) {
        e.printStackTrace();
        }
        return dataSource();
    }
    // 释放资源
    public static void close(ResultSet resultSet, Statement statement, Connection connection){
        if (resultSet != null) {
            try {
                resultSet.close();
            } catch (SQLException e) {
                    e.printStackTrace();
            }
     	}

        if (statement != null) {
            try {
                statement.close();
            } catch (SQLException e) {
                e.printStackTrace();
            }
		}

        if (connection != null) {
            try {
                connection.close();
            } catch (SQLException e) {
                e.printStackTrace();
            }
        }
    }
}
```
#### JdbcTemplateDml.java
```java
public class JdbcTemplateDml {
    private JdbcTemplate template = new JdbcTemplate(jdbcUtils.getDataSource);
    @Test
    public void testInsert() {
        int i = template.update("insert into user(uname, password) values(?, ?)", "jack", "123456");
        System.out.println(i == 1);
    }
    @Test
    public void testUpdate() {
        int i = template.update("update user set uname=? where uid=1", "jack");
        System.out.println(i == 1);
    }
    @Test
    public void testDelete() {
        int i = template.update("delete from student where id=?", 1);
        System.out.println(i == 1);
    }
}
```
#### JdbcTemplateDql.java
```java
public class JdbcTemplateDql {
    private JdbcTemplate template = new JdbcTemplate(JdbcUtils.getDataSource());
    private String sql;
    // 查询单值
    @Test
    public void test() {
        sql = "select max(age) from student";
        Integer i = template.queryForObject(sql, Integer.class);
        System.out.println(i);
    }
    // 查询一条数据, 不推荐
    @Test
    public void test2() {
        sql = "select * from student where id=2";
        Map<String, Object> map = template.queryForMap(sql);
        Set<Map.Entry<String, Object>> entries = map.entrySet();
        for (Map.Entry<String, Object> entry : entries) {
            String key = entry.getKey();
            Object value = entry.getValue();
            System.out.println(key + ":" + value);
        }
    }
    // 查询多条数据，不推荐
    @Test
    public void test3() {
        sql = "select * from student where score=?";
        List<Map<String, Object>> maps = template.queryForList(sql, 90);
        for (Map<String, Object> map : maps) {
            for (Map.Entry<String, Object> entry : map.entrySet()) {
                String key = entry.getKey();
                Object value = entry.getValue();
                System.out.println(key + ":" + value);
            }
        }
    }
    // 查询一条数据，封装成一个Student对象，推荐使用
    @Test
    public void test4() {
    	sql = "select * from student where id=?";
        Student student = template.queryForObject(sql, new BeanPropertyRowMapper<>(Student.class), 2);
    }
    // 查询多条数据，封装成List<Student>，推荐使用
    @Test
    public void test5() {
    	sql = "select * from student";
        List<Student> list = template.query(sql, new BeanPropertyRowMapper<>(Student.class));
        for (Student s : list) {
            System.out.println(s);
        }
    }
    // 查询多条数据，封装成List<Student>，不推荐
    @Test
    public void test6() {
        sql = "select * from student";
        List<Student> list = template.query(sql, new RowMapper<Student>(){
            @Override
            public Student mapRow(Result rs, int i) throws SQLException {
                Student s = new Student();
                s.setId(rs.getInt("id"));
                s.setName(rs.getString("name"));
                s.setAge(rs.getInt("age"));
                s.setScore(rs.getInt("score"));
                return s;
            }
        });
        for (Student s : list) {
            System.out.println(s);
        }
    }
}
```