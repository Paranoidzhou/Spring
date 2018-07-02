#[回顾]

1.mybatis:持久层;封装了jdbc底层的api;映射文件(核心)编写sql语句.

2.mapper节点中,namespace="接口的名称";

3.`<insert><update><delete><select>`

4.以上4个节点中,id属性值="方法的名称"

5.都可以有parameterType属性:如果属性是基本数据类型或者String类型,可省略;如果是实体类类型必须写上

6.`<insert><update><delete>`没有resultType,只有<selete>有resultType;如果查询的方法返回类型为实体类类型或者List<实体类类型>类型,resultType的值表示记录封装的实体类类型

7.在查询时候,如果字段名和属性名不相同,有两种解决方案:

	1.给字段起别名:别名必须和属性名相同
	2.字段和属性之间做个映射

#ssm整合:部门管理

##新建工程

1.新建maven工程

2.添加web.xml

3.添加tomcat运行环境

4.添加依赖jar包

	spring-webmvc 

	mybatis
	mybatis-spring

	mysql
	commons-dbcp
	spring-jdbc
	
	junit
	jstl

5.规范包名

	cn.tedu.ssm.bean:实体类
	cn.tedu.ssm.mapper:持久层接口
	cn.tedu.ssm.service:业务层包含接口和类
	cn.tedu.ssm.controller:控制器层

6.配置文件

	1)spring-mvc.xml(spring-mvc的配置文件)

	<!-- annotation:注解, driven: 驱动的 -->
	<mvc:annotation-driven/>
	<!-- 视图解析器 -->
	<bean id="jspViewResolver" 
		class="org.springframework.web.servlet.view.InternalResourceViewResolver">
		<property name="prefix" 
			value="/web/"/><!-- 删除 jsp/ -->
		<property name="suffix" 
			value=".jsp"/>
	</bean>
	
	<context:component-scan 
		base-package="cn.tedu.ssm.controller"/>
	

	2)spring-mybatis.xml(applicationContext.xml)
	
	<!-- 配置jdbc.properties -->
		<util:properties id="jdbc" 
			location="classpath:jdbc.properties"/>
			
		<!-- 配置数据库的连接池 -->
		
		<bean id="dataSource" 
		class="org.apache.commons.dbcp.BasicDataSource">
			<property name="driverClassName" 
				       value="${jdbc.driver}"/>
			<property name="url" value="${jdbc.url}"/>
			<property name="username" value="${jdbc.username}"/>
			<property name="password" value="${jdbc.password}"></property>
			<property name="initialSize" value="${jdbc.initSize}"></property>
			<property name="maxActive" value="${jdbc.maxSize}"></property>
		</bean>
		
		<!-- spring和mybatis整合 -->
		<!-- SqlSessionFactoryBean实例化sessionFactory
			  dataSource依赖注入数据源
			  mapperLocations配置映射文件的位置
		 -->
		<bean id="factoryBean" 
			class="org.mybatis.spring.SqlSessionFactoryBean">
			<property name="dataSource" ref="dataSource"/>	
			<property name="mapperLocations" 
						 value="classpath:mappers/*.xml"/>
		</bean>
		<!-- MapperScannerConfigurer 定义持久层接口的包名 -->
		<bean id="scannerConfigurer" 
			class="org.mybatis.spring.mapper.MapperScannerConfigurer">
			<property name="basePackage" 
						 value="cn.tedu.ssm.mapper"/>
		</bean>

	3)jdbc.properties(数据库配置的属性文件)

	driver=com.mysql.jdbc.Driver
	url=jdbc:mysql://localhost:3306/mydb
	username=root
	password=root
	initSize=3
	maxSize=3

	4)在resources文件夹下新建mappers文件夹,把DeptMapper.xml拷贝到mappers文件夹中.

	DeptMapper.xml(映射文件)

	<!-- namespace的值必须是接口名 -->
	<mapper namespace="cn.tedu.ssm.mapper.DeptMapper">
		
	</mapper>

7.web.xml

	1)ContextLoaderListener---spring容器中

	2)DispatcherServlet-------spring-mvc容器中

	3)设置编码格式Filter 

	  <!-- 设置web应用的上下文 -->
	  <context-param>
	  	<param-name>contextConfigLocation</param-name>
	  	<param-value>classpath:spring-mybatis.xml</param-value>
	  </context-param>
	  <!-- spring上下文的监听器 -->
	  <listener>
	  	<listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
	  </listener>
	  <!-- 配置前端控制器 -->
	  <servlet>
	  	<servlet-name>dispatcherServlet</servlet-name>
	  	<servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
	  	<init-param>
	  		<param-name>contextConfigLocation</param-name>
	  		<param-value>classpath:spring-mvc.xml</param-value>
	  	</init-param>
	  	<load-on-startup>1</load-on-startup>
	  </servlet>
	  <servlet-mapping>
	  	<servlet-name>dispatcherServlet</servlet-name>
	  	<url-pattern>*.do</url-pattern>
	  </servlet-mapping>
	  <!-- 设置编码格式Filter -->
	  <filter>
	  	<filter-name>encodingFilter</filter-name>
	  	<filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
	  	<init-param>
	  		<param-name>encoding</param-name>
	  		<param-value>utf-8</param-value>
	  	</init-param>
	  </filter>
	  <filter-mapping>
	  	<filter-name>encodingFilter</filter-name>
	  	<url-pattern>/*</url-pattern>
	  </filter-mapping>

	
##设计表t_dept

	create table t_dept(
		id int auto_increment primary key,
		dept_name varchar(50),
		dept_loc varchar(50)
	)

## 部门管理
### 1.添加部门

###1.添加部门-持久层

新建类 Dept


1.接口中定义方法

	public interface DeptMapper{
		void insertDept(Dept dept);
	}

2.映射文件中编写sql语句
	
	在DeptMapper.xml文件中,定义`insert`节点,编写insert语句

	<insert id="insertDept" parameterType="cn.tedu.ssm.Dept">
		insert into t_dept(
			dept_name,dept_loc
		)values(
			#{deptName},#{deptLoc}
		)
	</insert>
	
测试:

###2.添加部门-业务层

1.定义接口

	public interface DeptService{
		void addDept(Dept dept);
	}

2.编写实现类:实现接口中方法

	@Service
	public class DeptServiceImpl implements DeptService{
		@Resource
		private DeptMapper deptMapper;
		public void addDept(Dept dept){
			//调用持久层的方法
			deptMapper.insertDept(dept);
		}
	}
	
测试:

### 3.添加部门-控制器层

1.显示页面

	/showAdd.do
	请求参数 : 无
	请求方式 : get/post
	响应方式 : 转发
	
	@Controller
	@ReqeustMapping("/dept")
	public class DeptController{
		@RequestMapping("/showAdd.do")
		public String showAdd(){
			return "add";
		}
	}

定义add.jsp,在web文件夹中定义add.jsp

	<form action="" method="post">
	部门名称:<input type="text" name="name" ><br>
	部门地址:<input type="text" name="loc"><br>
	<input type="submit" value="添加">
	</form>

2.定义url处理添加按钮功能

	/addDept.do
	请求参数:name,loc
	请求方式:post
	响应方式:转发       ---ok.jsp
	
	@RequestMapping("/addDept.do")
	public String addDept(String name,String loc){
		//1.调用业务层方法
		//2.return "ok"
	}

在web文件夹中定义ok.jsp


##小结

1.spring-mybatis整合

2.ssm环境的搭建

3.分析问题:页面----数据库

  解决问题:数据库----页面

4.开发步骤:1)db table 2)持久层 3)业务层   4)控制器  5)页面 
	 



















