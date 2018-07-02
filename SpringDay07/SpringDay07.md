2018年6月25日 Spring Day07

##Mybatis

1.持久层的框架,支持普通的sql语句,存储过程(项目)

2.封装了底层的jdbc

3.提高开发效率

4.sql语句可以配置到xml文件中,注解

## 框架的使用
建库
	create database mydb;

1.建表

	create table t_user(
		id int auto_increment primary key,
		name  varchar(50),
		pwd  varchar(32),
		email varchar(50),
		phone varchar(32)
	)

2.新建工程

	1)新建maven工程
	2)添加web.xml
	3)添加tomcat运行环境
	4)添加依赖jar包
		mybatis.jar,mysql.jar,junit.jar

3.编写实体类User

	public class User {

		private Integer id;
		private String name;
		private String pwd;
		private String email;
		private String phone;
		....
	}

4.配置mybatis框架

	1)配置文件

	<?xml version="1.0" encoding="UTF-8"?> 
	<!DOCTYPE configuration PUBLIC "-//ibatis.apache.org//DTD Config 3.0//EN" 
		"http://ibatis.apache.org/dtd/ibatis-3-config.dtd">
	<configuration>
		<environments default="environment">
			<environment id="environment">
				<transactionManager type="JDBC" />
				<dataSource type="POOLED">
					<property name="driver" value="com.mysql.jdbc.Driver" />
					<property name="url"
						value="jdbc:mysql://localhost:3306/db" />
					<property name="username" value="root" />
					<property name="password" value="root" />
				</dataSource>
			</environment>
		</environments>
		<!-- <mappers>
			<mapper resource="StudentMapper.xml" />
			
		</mappers> -->
	</configuration>

	2)映射文件
	<!-- namespace表示命名空间,值不能重复 -->
	<mapper namespace="userDao">
		<!-- id属性, 给节点定义名字,不能重复-->
		<!-- void insertUser(User user); 
		parameterType表示方法的参数列表-->
		<insert id="addUser" 
			parameterType="cn.tedu.mybatis.bean.User">
			insert into t_user(
				name,pwd,email,phone
			)values(
				#{name},#{pwd},#{email},#{phone}
			)
			
		</insert>
		<!-- 修改t_user表信息 -->
		<!-- void updateUser(User user); -->
		<!-- update用来实现修改功能的节点 
			  id属性给update节点起名,名字唯一
			  parameterType表示参数的类型
		-->
		<update id="updateUser" 
		parameterType="cn.tedu.mybatis.bean.User">
			update  
				t_user
			set
				name = #{name},pwd = #{pwd},
				email = #{email},phone = #{phone}
		
			where
				id = #{id}
		</update>
		<!-- 根据id删除一条记录 -->
		<!-- void deleteUser(Integer id) -->
		<!-- 1.delete表示删除功能的节点 ;
			  2.如果参数列表是基本数据类型(封装类类型)
			    或者String类型,可以省略parameterType;
			  3.如果不省略,
			 可以写成parameterType="java.lang.Integer"
		-->
		<delete id="delUser">
			delete from t_user
			where
				id=#{id}
		</delete>
		<!-- 根据id查询用户信息 -->
		<!-- User selectUserById(Integer id) -->
		<!-- select用来实现查询功能
			  resultType表示记录封装的实体类类型
			   封装的规则:默认把表的字段名作为类的属性名
		 -->
		<select id="getUserById" 
			resultType="cn.tedu.mybatis.bean.User">
			select *
			from 
				t_user
			where
				id=#{id}
		</select>
		<!-- 查询表中的所有数据 -->
		<!-- List<User> selectAll() -->
		<select id="selectAll" 
			resultType="cn.tedu.mybatis.bean.User">
				select * 
				from t_user
		</select>
	</mapper>
	

5.编写持久层(使用mybatis提供的api实现对数据库表的操作)
	
	1）定义接口

	public interface UserDao {
	/**
	 * 插入t_user表的数据
	 * @param user
	 */
	void insertUser(User user);
	/**
	 * 修改t_user表的数据
	 * @param user
	 */
	void updateUser(User user);
	/**
	 * 根据id删除t_user表的数据
	 * @param id
	 */
	void deleteUser(Integer id);
	/**
	 * 根据id查询用户信息
	 * @param id
	 * @return 如果id存在,返回user,否则返回null
	 */
	User selectUserById(Integer id);
	/**
	 * 返回表中的所有数据
	 * @return
	 */
	List<User> selectAll();
	}

	2) 定义接口的实现类

	public class UserDaoImpl implements UserDao{

	public void insertUser(User user) {
		//通过mybatis提供的api方法,实现插入
		
		//3.SqlSession session=? //Connection
		SqlSession session = 
				SessionUtil.getSession();
		//第一个参数表示:namespace+id
		
		session.insert("userDao.addUser", user);
		//提交事务
		session.commit();
		//关闭session
		session.close();
	}

	public void updateUser(User user) {
		//获取session对象
		SqlSession session = 
			SessionUtil.getSession();
		//调用update方法
		session.update("userDao.updateUser",user);
		//提交事务
		session.commit();
		//关闭session
		session.close();
	}

	public void deleteUser(Integer id) {
		//获取session对象
		SqlSession session = 
				SessionUtil.getSession();
		//调用delete
		session.delete("userDao.delUser",id);
		//提交事务
		session.commit();
		//关闭session
		session.close();
		
	}

	public User selectUserById(Integer id) {
		SqlSession session =
				SessionUtil.getSession();
		User user = 
				session.selectOne("userDao.getUserById", id);
		session.close();
		return user;
	}

	public List<User> selectAll() {
		SqlSession session =
				SessionUtil.getSession();
		List<User> list =
		   session.selectList("userDao.selectAll");
		session.close();
		return list;
	}

	}


测试:

	public class TestUser {
	@Test
	public void testSelectAll(){
		UserDao dao = new UserDaoImpl();
		List<User> list = 
				dao.selectAll();
		for(User user : list){
			System.out.println(user.getEmail()+user.getName());
		}
	}
	@Test
	public void testSelectById(){
		UserDao dao = new UserDaoImpl();
		User user = 
				dao.selectUserById(2);
		System.out.println(user.getEmail()+user.getName());
	}
	@Test
	public void testDelUser(){
		UserDao dao = new UserDaoImpl();
		dao.deleteUser(1);
	}
	@Test
	public void testUpdateUser(){
		UserDao dao = new UserDaoImpl();
		User user = new User();
		user.setId(1);
		user.setName("王影");
		user.setPwd("111111");
		user.setEmail("wangying@tedu.cn");
		user.setPhone("13800138009");
		dao.updateUser(user);
	}
	@Test
	public void testInsertUser(){
		UserDao dao = new UserDaoImpl();
		User user = new User();
		user.setName("admin");
		user.setPwd("123456");
		user.setEmail("admin@tedu.cn");
		user.setPhone("13800138000");
		dao.insertUser(user);
	}

	}

##小结:

持久层:

1.定义接口

2.定义接口的实现类

3.sql语句设置到映射文件中

SqlSession提供的方法:insert update delete selectOne selectList


# 基于Mapper映射器的mybatis框架的使用(重点)

##建表

	create table t_address(
		id int auto_increment primary key,
		province varchar(50),
		city varchar(50),
		area varchar(50),
		user_address varchar(50)
	)

##创建工程
	
	1)创建maven工程
	2)添加web.xml文件
	3)添加tomcat运行环境
	4)添加依赖jar包 mysql  mybatis junit

##添加mybatis的配置文件

1.把SqlConfig.xml放到resources文件夹下;此文件配置的信息用来实现数据库连接的配置信息

2.在resources文件中,创建新的文件夹mappers,在mappers文件夹中,放置映射文件AddressMapper.xml;

##对表完成crud的操作

1.创建实体类

	public class Address {
		private Integer id;
		private String province;
		private String city;
		private String area;
		private String userAddress;

		....
	}

2.持久层

1)定义接口

	public interface AddressDao {
		void insertAddress(Address address);
		void updateAddress(Address address);
		void deleteAddress(Integer id);
		Address selectById(Integer id);
		List<Address> selectAll();

	}

2)映射文件
	
	<!-- namespace的值必须是接口名 -->
	<mapper namespace="cn.tedu.mybatis.dao.AddressDao">
		<!-- 插入数据 -->
		<!-- void insertAddress(Address address); -->
		<!-- 
			id的值必须为方法的名称
			parameterType表示方法的参数类表
		 -->
		<insert id="insertAddress" 
			parameterType="cn.tedu.mybatis.bean.Address">
				insert into t_address(
					province,city,area,user_address
				)values(
					#{province},#{city},#{area},
					#{userAddress}
				)
		</insert>
		<!-- 修改地址信息 -->
		<!-- void updateAddress(Address address); -->
	
		<update id="updateAddress"
			parameterType="cn.tedu.mybatis.bean.Address">
			update 
				t_address
			set
				province = #{province},
				city = #{city},
				area = #{area},
				user_address = #{userAddress}	
			where
				id = #{id}
		</update>
		<!-- 删除地址信息 -->
		<!-- void deleteAddress(Integer id); -->
		<delete id="deleteAddress">
			delete from t_address
			where
				id=#{id}
		</delete>
		<!-- 根据id查询地址信息 -->
		<!-- Address selectById(Integer id); -->
		<!-- 如果字段名和属性不相同;
			   那么给字段起别名,别名要和属性吗相同
		 -->
		<select id="selectById" 
		resultType="cn.tedu.mybatis.bean.Address">
			select
				id,province,city,area,
				user_address userAddress
			from
				t_address
			where
				id = #{id}
				
		</select>
	</mapper>

测试:

	public class TestAddress {
	@Test
	public void testGetById(){
		SqlSession session = 
				SessionUtil.getSession();
		AddressDao ad =
		  session.getMapper(AddressDao.class);
		System.out.println(
				ad.selectById(2));
	}
	@Test
	public void testDeleteAddress(){
		SqlSession session = 
				SessionUtil.getSession();
		AddressDao ad =
		  session.getMapper(AddressDao.class);
		ad.deleteAddress(1);
		session.commit();
		session.close();
	}
	@Test
	public void testUpdateAddress(){
		SqlSession session = 
				SessionUtil.getSession();
		AddressDao ad =
		  session.getMapper(AddressDao.class);
		Address address = new Address();
		address.setId(1);
		address.setProvince("河北省");
		address.setCity("石家庄");
		address.setArea("新华区");
		address.setUserAddress("万达广场");
		
		ad.updateAddress(address);
		session.commit();
		session.close();
	}
	@Test
	public void testInsertAddress(){
		SqlSession session = 
				SessionUtil.getSession();
		AddressDao ad =
		  session.getMapper(AddressDao.class);
		//session.insert("namespace+id",address);
		Address address = new Address();
		address.setProvince("北京市");
		address.setCity("市辖区");
		address.setArea("海淀区");
		address.setUserAddress("中鼎大厦8层");
		
		ad.insertAddress(address);
		session.commit();
		session.close();
		
	}

	}