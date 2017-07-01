# MyBatisStudy_1
[GitHub源码下载地址 : https://github.com/HChersh/MyBatisStudy_1](https://github.com/HChersh/MyBatisStudy_1)
一篇有助于快速了解MyBatis的快速入门博客，涉及单表的CRUD，快速了解MyBatis如何分别使用xml配置文件和注解两种方式实现ORM，完成POJO与关系型数据库的sql语句间的映射。
###项目Demo结构了解&&搭建准备
##### 1.目录结构 
![MyBatisStudy_1目录结构](http://upload-images.jianshu.io/upload_images/4823703-d2e7725b53800e32.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
* **pers.cherish.entity** (实体类包，与数据库表对应)
    * User.java (User实体类，属性与表字段对应)
* **pers.cherish.mapping**  (MaBatis映射相关文件包）
  * UserMapperl.java  (基于注解实现映射的接口)
  * userMapper.xml (基于xml实现映射的配置文件)
* **pers.cherish.test**  (测试包，分别测试两种实现方式)
* **pers.cherish.util**(MyBatisUtil.java根据config.xml获取SqlSessionFactory)
* **config.xml**(配置数据库连接信息，以及映射信息)
* **Referenced Libraries**下添加我们demo依赖的jar包，工程下载下来后目录bin/lib下有对应jar包，只需要config-buildpath-addJar就好

#####2.数据库#建表
目录下**MyBatisStudy_1.sql**
```
create database mybatis;
use mybatis;
CREATE TABLE users(id INT PRIMARY KEY AUTO_INCREMENT, name VARCHAR(20), age INT);
INSERT INTO users(name, age) VALUES('cherish', 21);
INSERT INTO users(name, age) VALUES('christine', 20);
``` 
###Demo搭建
#####1. 首先建立User实体类
```
package pers.cherish.entity;
/**
 * @author cherish users表所对应的实体类
 */
public class User {
	// 实体类的属性和表的字段名称一一对应
	private int id;
	private String name;
	private int age;

	public int getId() {
		return id;
	}
	public void setId(int id) {
		this.id = id;
	}
	public String getName() {
		return name;
	}
	public void setName(String name) {
		this.name = name;
	}
	public int getAge() {
		return age;
	}
	public void setAge(int age) {
		this.age = age;
	}

	@Override
	public String toString() {
		return "User [id=" + id + ", name=" + name + ", age=" + age + "]";
	}
}
```
#####2. 定义操作User实体的sql映射文件userMapper.xml
```
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<!-- 为这个mapper指定一个唯一的namespace，namespace的值习惯上设置成包名+sql映射文件名，这样就能够保证namespace的值是唯一的
例如namespace="pers.cherish.mapping.userMapper"就是pers.cherish.mapping(包名)+userMapper(userMapper.xml文件去除后缀)
 -->
<mapper namespace="pers.cherish.mapping.userMapper">
    <!-- 在select标签中编写查询的SQL语句， 设置select标签的id属性为getUser，id属性值必须是唯一的，不能够重复
    使用parameterType属性指明查询时使用的参数类型，resultType属性指明查询返回的结果集类型
    resultType="pers.cherish.entity.User"就表示将查询结果封装成一个User类的对象返回
    User类就是users表所对应的实体类
    -->
    <!-- 
        根据id查询得到一个user对象
     -->
    <select id="getUser" parameterType="int" 
        resultType="pers.cherish.entity.User">
        select * from users where id=#{id}
    </select>
    
    <!-- 创建用户(Create) -->
    <insert id="addUser" parameterType="pers.cherish.entity.User">
        insert into users(name,age) values(#{name},#{age})
    </insert>
    
    <!-- 删除用户(Remove) -->
    <delete id="deleteUser" parameterType="int">
        delete from users where id=#{id}
    </delete>
    
    <!-- 修改用户(Update) -->
    <update id="updateUser" parameterType="pers.cherish.entity.User">
        update users set name=#{name},age=#{age} where id=#{id}
    </update>
    
    <!-- 查询全部用户-->
    <select id="getAllUsers" resultType="pers.cherish.entity.User">
        select * from users
    </select>
    
</mapper>
```
#####3.完成config.xml配置文件
完成我们的数据源配置，和映射文件的配置，下面注释掉的配置，在使用注解的时候才会用到
```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE configuration PUBLIC "-//mybatis.org//DTD Config 3.0//EN" "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <environments default="development">
        <environment id="development">
            <transactionManager type="JDBC" />
            <!-- 配置数据库连接信息 -->
            <dataSource type="POOLED">
                <property name="driver" value="com.mysql.jdbc.Driver" />
                <property name="url" value="jdbc:mysql://localhost:3306/mybatis" />
                <property name="username" value="root" />
                <property name="password" value="root" />
            </dataSource>
        </environment>
    </environments>
    
    <mappers>
        <!-- 注册userMapper.xml文件， 
        userMapper.xml位于pers.cherish.mapping这个包下，所以resource写成pers.cherish.mapping/userMapper.xml-->
        <mapper resource="pers/cherish/mapping/userMapper.xml"/> 

        <!-- 注册UserMapper映射接口 之后讲注解时才会用到
        <mapper class="pers.cherish.mapping.UserMapperI"/>
        --> 
    </mappers>
       
</configuration>
```
#####4.创建我们的工具类获取SqlSessionFactory
```
package pers.cherish.util;

import java.io.InputStream;

import org.apache.ibatis.session.SqlSession;
import org.apache.ibatis.session.SqlSessionFactory;
import org.apache.ibatis.session.SqlSessionFactoryBuilder;



public class MyBatisUtil {

	/**
	 * 获取SqlSessionFactory
	 * 
	 * @return SqlSessionFactory
	 */
	public static SqlSessionFactory getSqlSessionFactory() {
		String resource = "config.xml";
		InputStream is = MyBatisUtil.class.getClassLoader().getResourceAsStream(resource);
		SqlSessionFactory factory = new SqlSessionFactoryBuilder().build(is);
		return factory;
	}

	/**
	 * 获取SqlSession
	 * 
	 * @return SqlSession
	 */
	public static SqlSession getSqlSession() {
		return getSqlSessionFactory().openSession();
	}

	/**
	 * 获取SqlSession
	 * 
	 * @param isAutoCommit
	 *            true 表示创建的SqlSession对象在执行完SQL之后会自动提交事务 false
	 *            表示创建的SqlSession对象在执行完SQL之后不会自动提交事务，这时就需要我们手动调用sqlSession.commit()提交事务
	 * @return SqlSession
	 */
	public static SqlSession getSqlSession(boolean isAutoCommit) {
		return getSqlSessionFactory().openSession(isAutoCommit);
	}
}
```
#####5.基于配置文件的测试代码
```
package pers.cherish.test;
import java.util.List;
import org.apache.ibatis.session.SqlSession;
import org.junit.Test;
import pers.cherish.entity.User;
import pers.cherish.util.MyBatisUtil;

public class TestCRUDByXmlMapper {

    @Test
    public void testAdd(){
        //SqlSession sqlSession = MyBatisUtil.getSqlSession(false);
        SqlSession sqlSession = MyBatisUtil.getSqlSession(true);
        /**
         * 映射sql的标识字符串，
         * pers.cherish.mapping.userMapper是userMapper.xml文件中mapper标签的namespace属性的值，
         * addUser是insert标签的id属性值，通过insert标签的id属性值就可以找到要执行的SQL
         */
        String statement = "pers.cherish.mapping.userMapper.addUser";//映射sql的标识字符串
        User user = new User();
        user.setName("tom");
        user.setAge(20);
        //执行插入操作
        int retResult = sqlSession.insert(statement,user);
        //手动提交事务
        //sqlSession.commit();
        //使用SqlSession执行完SQL之后需要关闭SqlSession
        sqlSession.close();
        System.out.println(retResult);
    }
    
    @Test
    public void testUpdate(){
        SqlSession sqlSession = MyBatisUtil.getSqlSession(true);
        /**
         * 映射sql的标识字符串，
         *  pers.cherish.mapping.userMapper是userMapper.xml文件中mapper标签的namespace属性的值，
         * updateUser是update标签的id属性值，通过update标签的id属性值就可以找到要执行的SQL
         */
        String statement = "pers.cherish.mapping.userMapper.updateUser";//映射sql的标识字符串
        User user = new User();
        user.setId(3);
        user.setName("tomer");
        user.setAge(20);
        //执行修改操作
        int retResult = sqlSession.update(statement,user);
        //使用SqlSession执行完SQL之后需要关闭SqlSession
        sqlSession.close();
        System.out.println(retResult);
    }
    
    @Test
    public void testDelete(){
        SqlSession sqlSession = MyBatisUtil.getSqlSession(true);
        /**
         * 映射sql的标识字符串，
         *  pers.cherish.mapping.userMapper是userMapper.xml文件中mapper标签的namespace属性的值，
         * deleteUser是delete标签的id属性值，通过delete标签的id属性值就可以找到要执行的SQL
         */
        String statement = "pers.cherish.mapping.userMapper.deleteUser";//映射sql的标识字符串
        //执行删除操作
        int retResult = sqlSession.delete(statement,5);
        //使用SqlSession执行完SQL之后需要关闭SqlSession
        sqlSession.close();
        System.out.println(retResult);
    }
    
    @Test
    public void testGetAll(){
        SqlSession sqlSession = MyBatisUtil.getSqlSession();
        /**
         * 映射sql的标识字符串，
         *  pers.cherish.mapping.userMapper是userMapper.xml文件中mapper标签的namespace属性的值，
         * getAllUsers是select标签的id属性值，通过select标签的id属性值就可以找到要执行的SQL
         */
        String statement = "pers.cherish.mapping.userMapper.getAllUsers";//映射sql的标识字符串
        //执行查询操作，将查询结果自动封装成List<User>返回
        List<User> lstUsers = sqlSession.selectList(statement);
        //使用SqlSession执行完SQL之后需要关闭SqlSession
        sqlSession.close();
        System.out.println(lstUsers);
    }
}
```
####如果我们要使用注解的方式我们需要修改下面两个地方
#####1. UserMapperI
新建我们实体数据库操作映射的接口，不需要在写实现类，mybatis会为我们创建相应的实现类
```
package pers.cherish.mapping;
import java.util.List;
import org.apache.ibatis.annotations.Delete;
import org.apache.ibatis.annotations.Insert;
import org.apache.ibatis.annotations.Select;
import org.apache.ibatis.annotations.Update;
import pers.cherish.entity.User;

/**
 * @author cherish
 * 定义sql映射的接口，使用注解指明方法要执行的SQL
 * 不需要再写imp，mybatis会更具sql语句创建对应的接口实现类
 */
public interface UserMapperI {

    //使用@Insert注解指明add方法要执行的SQL
    @Insert("insert into users(name, age) values(#{name}, #{age})")
    public int add(User user);
    
    //使用@Delete注解指明deleteById方法要执行的SQL
    @Delete("delete from users where id=#{id}")
    public int deleteById(int id);
    
    //使用@Update注解指明update方法要执行的SQL
    @Update("update users set name=#{name},age=#{age} where id=#{id}")
    public int update(User user);
    
    //使用@Select注解指明getById方法要执行的SQL
    @Select("select * from users where id=#{id}")
    public User getById(int id);
    
    //使用@Select注解指明getAll方法要执行的SQL
    @Select("select * from users")
    public List<User> getAll();
}
```
#####2.修改config.xml
申明使用注解的方式
```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE configuration PUBLIC "-//mybatis.org//DTD Config 3.0//EN" "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <environments default="development">
        <environment id="development">
            <transactionManager type="JDBC" />
            <!-- 配置数据库连接信息 -->
            <dataSource type="POOLED">
                <property name="driver" value="com.mysql.jdbc.Driver" />
                <property name="url" value="jdbc:mysql://localhost:3306/mybatis" />
                <property name="username" value="root" />
                <property name="password" value="root" />
            </dataSource>
        </environment>
    </environments>
    
    <mappers>
        <!-- 注册userMapper.xml文件， 
        userMapper.xml位于pers.cherish.mapping这个包下，所以resource写成pers.cherish.mapping/userMapper.xml
        <mapper resource="pers/cherish/mapping/userMapper.xml"/>
        -->
        <!-- 注册UserMapper映射接口 -->
        <mapper class="pers.cherish.mapping.UserMapperI"/>
    </mappers>
</configuration>
```
#####3.最后写我们基于注释的测试类就好了
```
package pers.cherish.test;
import java.util.List;
import org.apache.ibatis.session.SqlSession;
import org.junit.Test;
import pers.cherish.entity.User;
import pers.cherish.mapping.UserMapperI;
import pers.cherish.util.MyBatisUtil;

public class TestCRUDByAnnotationMapper {

	@Test
	public void testAdd() {
		SqlSession sqlSession = MyBatisUtil.getSqlSession(true);
		// 得到UserMapperI接口的实现类对象，UserMapperI接口的实现类对象由sqlSession.getMapper(UserMapperI.class)动态构建出来
		UserMapperI mapper = sqlSession.getMapper(UserMapperI.class);
		User user = new User();
		user.setName("jack");
		user.setAge(22);
		int add = mapper.add(user);
		// 使用SqlSession执行完SQL之后需要关闭SqlSession
		sqlSession.close();
		System.out.println(add);
	}

	@Test
	public void testUpdate() {
		SqlSession sqlSession = MyBatisUtil.getSqlSession(true);
		// 得到UserMapperI接口的实现类对象，UserMapperI接口的实现类对象由sqlSession.getMapper(UserMapperI.class)动态构建出来
		UserMapperI mapper = sqlSession.getMapper(UserMapperI.class);
		User user = new User();
		user.setId(3);
		user.setName("jack");
		user.setAge(26);
		// 执行修改操作
		int retResult = mapper.update(user);
		// 使用SqlSession执行完SQL之后需要关闭SqlSession
		sqlSession.close();
		System.out.println(retResult);
	}

	@Test
	public void testDelete() {
		SqlSession sqlSession = MyBatisUtil.getSqlSession(true);
		// 得到UserMapperI接口的实现类对象，UserMapperI接口的实现类对象由sqlSession.getMapper(UserMapperI.class)动态构建出来
		UserMapperI mapper = sqlSession.getMapper(UserMapperI.class);
		// 执行删除操作
		int retResult = mapper.deleteById(7);
		// 使用SqlSession执行完SQL之后需要关闭SqlSession
		sqlSession.close();
		System.out.println(retResult);
	}

	@Test
	public void testGetUser() {
		SqlSession sqlSession = MyBatisUtil.getSqlSession();
		// 得到UserMapperI接口的实现类对象，UserMapperI接口的实现类对象由sqlSession.getMapper(UserMapperI.class)动态构建出来
		UserMapperI mapper = sqlSession.getMapper(UserMapperI.class);
		// 执行查询操作，将查询结果自动封装成User返回
		User user = mapper.getById(8);
		// 使用SqlSession执行完SQL之后需要关闭SqlSession
		sqlSession.close();
		System.out.println(user);
	}

	@Test
	public void testGetAll() {
		SqlSession sqlSession = MyBatisUtil.getSqlSession();
		// 得到UserMapperI接口的实现类对象，UserMapperI接口的实现类对象由sqlSession.getMapper(UserMapperI.class)动态构建出来
		UserMapperI mapper = sqlSession.getMapper(UserMapperI.class);
		// 执行查询操作，将查询结果自动封装成List<User>返回
		List<User> lstUsers = mapper.getAll();
		// 使用SqlSession执行完SQL之后需要关闭SqlSession
		sqlSession.close();
		System.out.println(lstUsers);
	}
}

```
基本就这个样子了
