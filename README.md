# mybatis-spring
Sample code for MyBatis and Spring integration

1. Overview

MyBatis is a popular persistence framework with support for custom SQL, Stored Procedures and advanced mappings in Java, .Net and Ruby on Rails. It eliminates much of the JDBC boiler-plate code. Like other persistence frameworks, MyBatis supports both xml based and annotation driven configurations. In this article we discuss about Abator, MyBatis v/s other ORM frameworks, when MyBatis is preferred and how to integrate MyBatis with Spring Framework.

2. Abator

Abator is a Code Generator for MyBatis. Abator introspects the database tables and generates MyBatis artifacts by eliminating the initial nuisance of setting up of objects and configuration files. Abator comes handy while implementing CRUD operations.

2.1. What does Abator generate?

Abator generates the below artifacts for MyBatis/iBatis,

Java Classes to match database table structure including primary and non primary fields, BLOB fields and classes to enable dynamic SQL
MyBatis/iBatis compatible SQL Map XML files
Supports generation of DAO classes. Abator can be configured to generate DAO classes to support Spring Framework or DAO classes those confirm to MyBatis/iBatis DAO Framework
Abator has capability to merge the existing XML configuration files with the newly generated XML files, provided both old and new files have the same name. However, it doesn't merge the Java files, based on configuration either it can overwrite or save new files with a different name.

2.2. Dependencies

Only dependency Abator has is JRE. It requires JRE version 1.4 or above. Abator requires JDBC driver implementing DatabaseMetaData interface, especially getColumns and getPrimaryKeys methods.

2.3. Setting Up and running Abator

Abator is driven by an XML configuration file and configuration file gives information to Abator on,

How to connect to database
What objects to generate and how to generate
Tables to use for Object generation
Below is the sample Abator configuration XML,

<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE abatorConfiguration
  PUBLIC "-//Apache Software Foundation//DTD Abator for iBATIS Configuration 1.0//EN"
  "http://ibatis.apache.org/dtd/abator-config_1_0.dtd">

<abatorConfiguration>
	<abatorContext id="DB2Tables" generatorSet="Java2">
		<jdbcConnection driverClass="COM.ibm.db2.jdbc.app.DB2Driver"
			connectionURL="jdbc:db2:TEST" userId="db2admin" password="db2admin">
			<classPathEntry location="/Program Files/IBM/SQLLIB/java/db2java.zip" />
		</jdbcConnection>
		<javaTypeResolver>
			<property name="forceBigDecimals" value="false" />
		</javaTypeResolver>
		<javaModelGenerator targetPackage="test.model"
			targetProject="\AbatorTestProject\src">
			<property name="enableSubPackages" value="true" />
			<property name="trimStrings" value="true" />
		</javaModelGenerator>
		<sqlMapGenerator targetPackage="test.xml"
			targetProject="\AbatorTestProject\src">
			<property name="enableSubPackages" value="true" />
		</sqlMapGenerator>
		<daoGenerator type="SPRING" targetPackage="test.dao"
			targetProject="\AbatorTestProject\src">
			<property name="enableSubPackages" value="true" />
		</daoGenerator>
		<table tableName="EMPLOYEE" domainObjectName="Employee">
			<property name="useActualColumnNames" value="true" />
			<generatedKey column="ID" sqlStatement="DB2" identity="true" />
			<columnOverride column="JOININGDATE" property="startDate" />
			<columnOverride column="NAME" jdbcType="VARCHAR" />
		</table>
	</abatorContext>
</abatorConfiguration>
Save Abator XML in a convenient location and Abator can be run by using below command (Make sure JAVA_HOME environment variable is set),

java -jar abator.jar -configfile D:\Abator\abatorConfig.xml -overwrite
Above command instructs Abator to use custom configuration file abatorConfig.xml and overwrite any previously generated Java files with the newly generated files (if names of both old and new file are same).

2.4. Tasks after running Abator

After running Abator, we need to create or modify MyBatis/iBatis artifacts.

Create or Modify SqlMapConfig.xml
Create or Modify dao.xml (If application uses MyBatis DAO framework)
2.5. Updating SqlMapConfig.xml

SqlMapConfig.xml is the master file and it specifies the database connection, transaction management scheme and MyBatis SQL map files used in the MyBatis session. Abator cannot generate these files as it doesn't know about the execution environment. Update below entries in SqlMapConfig.xml

Statement namespace must be enabled
Abator generated SQL Map Files have to be linked
For example Abator has generated an SQL Map File employee.xml, after linking SqlMapConfig.xml looks as below,

<?xml version="1.0" encoding="UTF-8"?>
  <!DOCTYPE sqlMapConfig
    PUBLIC "-//ibatis.apache.org//DTD SQL Map Config 2.0//EN"
    "http://ibatis.apache.org/dtd/sql-map-config-2.dtd">

<sqlMapConfig>
	<!-- Statement namespaces are required for Abator -->
	<settings useStatementNamespaces="true" />

	<!-- Setup the transaction manager and data source that are appropriate 
		for your environment -->
	<transactionManager type="...">
		<dataSource type="...">
		</dataSource>
	</transactionManager>

	<!-- SQL Map XML files should be listed here -->
	<sqlMap resource="Abator/xml/employee.xml" />

</sqlMapConfig>
If there are more than one SQL Map Config files those can be linked by repeating <sqlMap> tag.

2.6. Updating dao.xml

MyBatis DAO framework uses a XML configuration file commonly called as dao.xml. The framework uses this file to control database connection information and to list DAO interfaces and implementations. This file specifies the path to SqlMapConfig.xml and Abator generated DAO interfaces and classes.

Suppose Abator has generated DAO interface called IEmployeeDao and class called EmployeeDaoImpl under package com.baeldung.mybatis.dao, then dao.xml looks like below,

<?xml version="1.0" encoding="UTF-8"?>
  <!DOCTYPE daoConfig
   PUBLIC "-//ibatis.apache.org//DTD DAO Configuration 2.0//EN"
   "http://ibatis.apache.org/dtd/dao-2.dtd">

<daoConfig>
	<context>
		<transactionManager type="SQLMAP">
			<property name="SqlMapConfigResource" value="SqlMapConfig.xml" />
		</transactionManager>

		<!-- DAO interfaces and implementations should be listed here -->
		<dao interface="com.baeldung.mybatis.dao.IEmployeeDao" implementation="com.baeldung.mybatis.dao.EmployeeDaoImpl" />
	</context>
</daoConfig>
More information on Abator can be found in the link https://ibatis.apache.org/docs/tools/abator/

2.7. Alternatives for Abator

MyBatis Code Generator (MBG) is an alternative for Abator. MBG can generate code for all versions of MyBatis and iBatis version 2.2.o and above. MBG can introspect database tables and generates artifacts those can be used to access database tables. MBG will generate,

Java POJO that match table structure
MyBatis/iBatis compatible SQL Map files
Java Client objects to make use of config files and POJO classes
Similar to Abator, it has capability to merge configuration XMLs. It can either overwrite existing Java files or generate new files with unique name.

MBG is designed to run well in iterative development environments. It can be included as an Ant task or Maven Plugin in a continuous integration environment.

More information on MBG can be found in the link http://www.mybatis.org/generator/

3. MyBatis compared with other ORM Frameworks

Database access has become more easier with the invent of ORM frameworks. In Java space Hibernate, JPA and Spring Data are popular ORM frameworks. In this section let us see how MyBatis is different from other ORM frameworks,

MyBatis is not an ORM framework, it is a mapping framework. It doesn't match table to object, instead it maps result set to object. MyBatis users don'thave to worry about underlying table structure
ORMs are database independent and it is easy to switch between the databases. However, MyBatis uses SQL and is database dependent
As compared to ORM frameworks, it is easy to use Stored Procedures in MyBatis
Hibernate provide more advanced support for caching
MyBatis comes with small package and it is easy to learn
MyBatis users have to be more skilled in SQL as compared to ORM frameworks. Hiberate generates the SQL for the users based on the database it is connected with
MyBatis outperforms when we have to deal with legacy databases as compared to ORM frameworks
4. When MyBatis is preferred?

In the previous section we learnt about how MyBatis different from other ORM frameworks. Now let us see in which scenarios MyBatis is preferred,

MyBatis is well suited for applications involving legacy database and fairly complex SQL queries with high performance goals are set
Hibernate works better if application is more object-centric and if the application is more database-centric then MyBatis works better.
MyBatis performs better with complex fetch queries where just an answer is expected. Hibernate tries to load entire object graph and Lazy Initialization mechanism has to be used to optimize fetch logic, that is tricky. Hence in an analytic environment MyBatis suits better.
 5. Spring and MyBatis

mybatis-spring library seamlessly integrates MyBatis and Spring. This library allows MyBatis to participate in Spring transactions, takes care of building MyBatis mappers, SqlSessions and inject them into other beans.

In this section we are going to build a Student Enrollment Application using MySQL database with MyBatis in Spring environment.

Create a Maven project in Eclipse or STS IDE with the template of maven-archetype-webapp. Follow the below steps to create and build the application. Whole application code is present in GitHub and download link is provided later in the article. I am not providing any screenshots on setting up the project in Eclipse.

5.1. Modify pom.xml

Add below dependencies to Maven pom.xml,

<dependency>
	<groupId>org.mybatis</groupId>
	<artifactId>mybatis</artifactId>
	<version>3.1.1</version>
</dependency>
<dependency>
	<groupId>org.mybatis</groupId>
	<artifactId>mybatis-spring</artifactId>
	<version>1.1.1</version>
</dependency>
<dependency>
	<groupId>org.springframework</groupId>
	<artifactId>spring-context-support</artifactId>
	<version>3.1.1.RELEASE</version>
</dependency>
<dependency>
	<groupId>org.springframework</groupId>
	<artifactId>spring-test</artifactId>
	<version>3.1.1.RELEASE</version>
	<scope>test</scope>
</dependency>
<dependency>
	<groupId>mysql</groupId>
	<artifactId>mysql-connector-java</artifactId>
	<version>5.1.21</version>
</dependency>
<dependency>
	<groupId>javax.servlet</groupId>
	<artifactId>jstl</artifactId>
	<version>1.2</version>
</dependency>
<dependency>
	<groupId>org.springframework</groupId>
	<artifactId>spring-webmvc</artifactId>
	<version>3.2.4.RELEASE</version>
</dependency>
<dependency>
	<groupId>javax.servlet</groupId>
	<artifactId>servlet-api</artifactId>
	<version>2.5</version>
</dependency>
mybatis - for MyBatis support
mybatis-spring - MyBatis Spring integration support
jstl, spring-webmvc, servlet-api and spring-context-support - for Spring support
mysql-connector-java - for connecting to MySQL
spring-test - optional dependency and required if Spring Test is required
5.2. Modify web.xml

Modify web.xml to include below configurations,

Spring Configuration file (Create Spring config file mybatis-spring.xml under WEB-INF/config folder)
A sevlet-mapping
The final web.xml is as below,

<?xml version="1.0" encoding="UTF-8"?>
<web-app version="2.5" xmlns="http://java.sun.com/xml/ns/javaee"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_2_5.xsd">

	<servlet>
		<servlet-name>myBatisSpringServlet</servlet-name>
		<servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
		<init-param>
			<param-name>contextConfigLocation</param-name>
			<param-value>/WEB-INF/config/mybatis-spring.xml</param-value>
		</init-param>
	</servlet>

	<servlet-mapping>
		<servlet-name>myBatisSpringServlet</servlet-name>
		<url-pattern>*.html</url-pattern>
	</servlet-mapping>

	<display-name>MyBatis-Spring Integration</display-name>
</web-app>
5.3. Spring Configuration File

Create Spring Configuration file mybatis-spring.xml, add below configurations in it,

enable context, tx and mvc namespaces
Include InternalResourceViewResolver to locate the jsp files
Add data source configuration for MySQL database
Include Transaction Manager for scoping/controlling the transactions
Add SqlSessionFactory and parameters
dataSource - configured in 3rd step
typeAliasesPackage - location of model classes
mapperLocation - mapper XML files location
The final mybatis-spring.xml is as below,

<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:context="http://www.springframework.org/schema/context"
	xmlns:mvc="http://www.springframework.org/schema/mvc" xmlns:tx="http://www.springframework.org/schema/tx"
	xsi:schemaLocation="http://www.springframework.org/schema/mvc http://www.springframework.org/schema/mvc/spring-mvc-3.2.xsd
      http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
      http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx-3.1.xsd
      http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-3.1.xsd">

	<mvc:annotation-driven />
	<context:component-scan base-package="com.baeldung.spring.mybatis.controller" />
	<bean
		class="org.springframework.web.servlet.view.InternalResourceViewResolver">
		<property name="prefix" value="/WEB-INF/jsp/" />
		<property name="suffix" value=".jsp" />
	</bean>
	<bean id="dataSource"
		class="org.springframework.jdbc.datasource.DriverManagerDataSource">
		<property name="driverClassName" value="com.mysql.jdbc.Driver" />
		<property name="url" value="jdbc:mysql//localhost:3306/baeldung" />
		<property name="username" value="root" />
		<property name="password" value="admin" />
	</bean>
	<tx:annotation-driven transaction-manager="transactionManager" />
	<bean id="transactionManager"
		class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
		<property name="dataSource" ref="dataSource" />
	</bean>
	<bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
		<property name="dataSource" ref="dataSource" />
		<property name="typeAliasesPackage" value="com.baeldung.spring.mybatis.model" />
		<property name="mapperLocations" value="classpath*:com/baeldung/spring/mappers/*.xml" />
	</bean>
	<bean id="sqlSession" class="org.mybatis.spring.SqlSessionTemplate">
		<constructor-arg index="0" ref="sqlSessionFactory" />
	</bean>
	<bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
		<property name="basePackage" value="com.baeldung.spring.mybatis.mappers" />
	</bean>
</beans>
5.4. Create JSP page

Application requires couple of JSP files. JSP file code is not shown in the tutorial, these can be downloaded from the GitHub.

5.5. Create necessary Java Files

Create Model classes Student and StudentLogin,

public class Student {
	private Long id;
	private String userName;
	private String firstName;
	private String lastName;
	private String password;
	private String emailAddress;
	private Date dateOfBirth;
}
public class StudentLogin {
	private String userName;
	private String password;
}
Create Mapper class, Mapper class in MyBatis is similar to Repository class in Spring. Create an interface StudentMapper as below,

public interface StudentMapper {
	@Insert("INSERT INTO student(userName, password, firstName," + "lastName, dateOfBirth, emailAddress) VALUES"
			+ "(#{userName},#{password}, #{firstName}, #{lastName}," + "#{dateOfBirth}, #{emailAddress})")
	@Options(useGeneratedKeys = true, keyProperty = "id", flushCache = true, keyColumn = "id")
	public void insertStudent(Student student);

	@Select("SELECT USERNAME as userName, PASSWORD as password, " + "FIRSTNAME as firstName, LASTNAME as lastName, "
			+ "DATEOFBIRTH as dateOfBirth, EMAILADDRESS as emailAddress " + "FROM student WHERE userName = #{userName}")
	public Student getStudentByUserName(String userName);

}
Create Controller class. Controller class contains the routing logic for the application.

@Controller
@SessionAttributes("student")
public class StudentController {

	@Autowired
	private StudentService studentService;

	@RequestMapping(value = "/signup", method = RequestMethod.GET)
	public String signup(Model model) {
		Student student = new Student();
		model.addAttribute("student", student);
		return "signup";
	}

	@RequestMapping(value = "/signup", method = RequestMethod.POST)
	public String signup(@ModelAttribute("student") Student student, Model model) {
		if (studentService.getStudentByUserName(student.getUserName())) {
			model.addAttribute("message", "User Name exists. Try another user name");
			return "signup";
		} else {
			studentService.insertStudent(student);
			model.addAttribute("message", "Saved student details");
			return "redirect:login.html";
		}
	}

	@RequestMapping(value = "/login", method = RequestMethod.GET)
	public String login(Model model) {
		StudentLogin studentLogin = new StudentLogin();
		model.addAttribute("studentLogin", studentLogin);
		return "login";
	}

	@RequestMapping(value = "/login", method = RequestMethod.POST)
	public String login(@ModelAttribute("studentLogin") StudentLogin studentLogin) {
		boolean found = studentService.getStudentByLogin(studentLogin.getUserName(), studentLogin.getPassword());
		if (found) {
			return "success";
		} else {
			return "failure";
		}
	}
}
Create Service class. Service classes hold the business logic.

public interface StudentService {
	void insertStudent(Student student);
	boolean getStudentByLogin(String userName, String password);
	boolean getStudentByUserName(String userName);
}
@Service("studentService")
public class StudentServiceImpl implements StudentService {

	@Autowired
	private StudentMapper studentMapper;

	@Transactional
	public void insertStudent(Student student) {
		studentMapper.insertStudent(student);
	}

	public boolean getStudentByLogin(String userName, String password) {
		Student student = studentMapper.getStudentByUserName(userName);

		if (student != null && student.getPassword().equals(password)) {
			return true;
		}

		return false;
	}

	public boolean getStudentByUserName(String userName) {
		Student student = studentMapper.getStudentByUserName(userName);

		if (student != null) {
			return true;
		}

		return false;
	}
}
Setup database. Create a MySQL schema baeldung and create student table using below SQL,

CREATE TABLE `student` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT,
  `dateOfBirth` datetime NOT NULL,
  `emailAddress` varchar(255) NOT NULL,
  `firstName` varchar(255) NOT NULL,
  `lastName` varchar(255) NOT NULL,
  `password` varchar(8) NOT NULL,
  `userName` varchar(20) NOT NULL,
  PRIMARY KEY (`id`)
)
5.6. Deploy the application in Tomcat Web Server

For this demo we have chosen Tomcat 7 as the web server. Java web application can be deployed in Tomcat Server by right clicking on application and choosing "Run As -> Run on Server" in eclipse based IDE. If same has to be deployed in a remote server, then generate a war file (Right click on project and choose Export as WAR File option), ship it to the remote server and place it in appropriate Tomcat directory.

Complete code of the application can be downloaded  from GitHub.

6. Conclusion

In this tutorial we have seen what MyBatis offers and how to work with MyBatis in Spring environment. MyBatis is a lite weight, easy to learn and powerful framework to perform database operations in Java, .Net, Ruby on Rails environments. Compared to other ORM frameworks it is highly suited for database centric applications or applications involving legacy databases or applications involving complex SQL queries with high performance goals set.

7. References

https://www.tutorialspoint.com/ibatis/index.htm
http://www.mybatis.org/mybatis-3/
https://www.javacodegeeks.com
https://ibatis.apache.org/docs/tools/abator/
https://www.javacodegeeks.com/2014/02/building-java-web-application-using-mybatis-with-spring.html
