# hibernate-spring-project
This project is an integration example of spring boot and hibernate.It has configuration for basic datasource and hikari datasource too.

Table of Contents
1. Project Structure 
2. Maven Dependencies
3. Spring Boot Configuration
4. Basic Datasource Configurations in Spring Boot
5. Hikari Datasource Configurations with Hibernate
6. Hibernate Related Configurations
7. Spring Server Implementation
8. Sample Script
9. Run Spring Boot Hibernate Application
9. Spring Boot 2.0 and Hibernate


Project Structure

Following is the project structure. We have controllers, service and dao layers. We have application.properties defined that contains configurations related to our datasource.

Maven Dependencies

spring-boot-starter-parent: It provides useful Maven defaults. It also provides a dependency-management section so that you can omit version tags for existing dependencies.

spring-boot-starter-web: It includes all the dependencies required to create a web app. This will avoid lining up different spring common project versions.

spring-boot-starter-tomcat: It enable an embedded Apache Tomcat 7 instance, by default.This can be also marked as provided if you wish to deploy the war to any other standalone tomcat.

spring-boot-starter-data-jpa: It provides key dependencies for Hibernate, Spring Data JPA and Spring ORM.

pom.xml
<parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>1.5.1.RELEASE</version>
</parent>
	
    <dependencies>
	    <dependency>
                   <groupId>org.springframework.boot</groupId>
                   <artifactId>spring-boot-starter-web</artifactId>
            </dependency>
	    <dependency>
                   <groupId>org.springframework.boot</groupId>
                   <artifactId>spring-boot-starter-tomcat</artifactId>
             </dependency>
	     <dependency>
                    <groupId>org.springframework.boot</groupId>
                    <artifactId>spring-boot-starter-security</artifactId>
	      </dependency>
	      <dependency>
                     <groupId>org.springframework.boot</groupId>
                     <artifactId>spring-boot-starter-data-jpa</artifactId>
		      <exclusions>
                             <exclusion>
                                     <groupId>org.apache.tomcat</groupId>
                                     <artifactId>tomcat-jdbc</artifactId>
                              </exclusion>
                        </exclusions>
		</dependency>
		<dependency>
                       <groupId>mysql</groupId>
                       <artifactId>mysql-connector-java</artifactId>
                </dependency>
		 <dependency>
                         <groupId>commons-dbcp</groupId>
                         <artifactId>commons-dbcp</artifactId>
		</dependency>
		
    </dependencies>
	

Spring Boot Configuration

@SpringBootApplication enables many defaults. It is a convenience annotation that adds @Configuration, @EnableAutoConfiguration, @EnableWebMvc, @ComponentScan

The main() method uses Spring Boot SpringApplication.run() method to launch an application.


Application.java
package com.harsh;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class Application {

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}


 Other Interesting Posts
Spring Data JPA Example
Spring Hibernate Integration Example
Spring Boot Actuator Complete Guide
Spring Boot Actuator Rest Endpoints Example
Spring 5 Features and Enhancements
Spring Boot Thymeleaf Example
Spring Boot Security Hibernate Example with complete JavaConfig
Securing REST API with Spring Boot Security Basic Authentication
Spring Boot Security Password Encoding using Bcrypt Encoder
Spring Security with Spring MVC Example Using Spring Boot
Websocket spring Boot Integration Without STOMP with complete JavaConfig

Basic Datasource Configurations in Spring Boot

The most convenient way to define datasource parameters in spring boot application is to make use of application.properties file. Following is our sample application.properties. Here we are using JPA based configurations and hibernate as a JPA provider.

The following configuration creates a DriverManagerDataSource which opens and closes a connection to the database when needed.It means no connection pooling is achieved.While doing so, you may have performance issues in the production. In production, it is always recommended to have datasource that supports connection pooling and to create this connection pooling datasource we require to configure custom datasource bean programatically. We will create it in next section. 
spring.datasource.url=jdbc:mysql://localhost:3306/test
spring.datasource.username=root
spring.datasource.password=root
spring.jpa.show-sql=true
spring.jpa.hibernate.ddl-auto=update
spring.jpa.hibernate.naming.physical-strategy=org.hibernate.boot.model.naming.PhysicalNamingStrategyStandardImpl
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.MySQL5InnoDBDialect

Hibernate supports 2 different naming strategies.To use Hibernate 5 default naming strategy, we have used PhysicalNamingStrategyStandardImpl. Keep a note that SpringPhysicalNamingStrategy is the default naming strategy used by spring boot.


Hikari Datasource Configurations with Hibernate

In production, it is always recommended to use datasource that supports connection pooling because database connection creation is a slow process.Here in the example we will be using HikariDatasource instead. It provides many advanced features while configuring our datasource in comparison to other datasources such as connectionTimeout, idleTimeout, maxLifetime, connectionTestQuery, maximumPoolSize and very important one is leakDetectionThreshold.It is as advanced as detecting connection leaks by itself.It is also faster and lighter than other available datasource.Following is the configuration for HikariDatasource.Make sure you comment the datasource confguration in properties file.

HikariDatasource Config
@Bean
    public DataSource dataSource() {
        HikariDataSource ds = new HikariDataSource();
        ds.setMaximumPoolSize(100);
        ds.setDataSourceClassName("com.mysql.jdbc.jdbc2.optional.MysqlDataSource");
        ds.addDataSourceProperty("url", "jdbc:mysql://localhost:3306/test");
        ds.addDataSourceProperty("user", "root");
        ds.addDataSourceProperty("password", "password");
        ds.addDataSourceProperty("cachePrepStmts", true);
        ds.addDataSourceProperty("prepStmtCacheSize", 250);
        ds.addDataSourceProperty("prepStmtCacheSqlLimit", 2048);
        ds.addDataSourceProperty("useServerPrepStmts", true);
        return ds;
    }

We can also create Hikaridatasource using DataSourceBuilder as follow.While doing so the datasource related properties can be still there in proerties file.I like this way.


@Bean
@ConfigurationProperties("spring.datasource")
public HikariDataSource dataSource() {
	return DataSourceBuilder.create().type(HikariDataSource.class).build();
}

In order to use HikariDataSource, you must include following maven dependency. Checkout the latest version here - Hikari Maven


<dependency>
            <groupId>com.zaxxer</groupId>
            <artifactId>HikariCP</artifactId>
			<version>2.7.3</version>
			</dependency>

In this case, we need to explicitly tell spring boot to use our custom datasource while creating EntityManagerfactory.Following is a sample example.


@Bean(name = "entityManagerFactory")
public EntityManagerFactory entityManagerFactory() {
	LocalContainerEntityManagerFactoryBean emf = new LocalContainerEntityManagerFactoryBean();
	emf.setDataSource(dataSource);
	emf.setJpaVendorAdapter(jpaVendorAdapter);
	emf.setPackagesToScan("com.mysource.model");
	emf.setPersistenceUnitName("default");
	emf.afterPropertiesSet();
	return emf.getObject();
}

Hibernate Related Configurations

Spring boot focusses on using JPA to persist data in relational db and it has ability to create repository implementations automatically, at runtime, from a repository interface. But here we are trying to use hibernate as a JPA provider. Hence, following configuration is required to autowire sessionFactory in our DAO class.


BeanConfig.java
package com.harsh;

import org.hibernate.SessionFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import javax.persistence.EntityManagerFactory;


@Configuration
public class BeanConfig {

	@Autowired
	private EntityManagerFactory entityManagerFactory;

	@Bean
	public SessionFactory getSessionFactory() {
	    if (entityManagerFactory.unwrap(SessionFactory.class) == null) {
	        throw new NullPointerException("factory is not a hibernate factory");
	    }
	    return entityManagerFactory.unwrap(SessionFactory.class);
	}

}


Hibernate Entity Class

Following is the entity class. The class is annotated as hibernate entity.



UserDetails.java
package com.harsh.model;

@Entity
@Table
public class UserDetails {
	
	@Id
	@Column
	@GeneratedValue(strategy = GenerationType.IDENTITY)
	private int id;
	@Column
	private String firstName;
	@Column
	private String lastName;
	@Column
	private String email;
	@Column
	private String password;
	
	//getters and setters goes here
	

Spring Server Implementation

Let us define our controller. It has one url mapping that intercepts request at /list and returns all users present in db.



UserController.java
package com.harsh.controller;

import java.util.List;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;

import com.harsh.model.UserDetails;
import com.harsh.service.UserService;

@Controller
public class UserController {
	
	@Autowired
	private UserService userService;
	
	@RequestMapping(value = "/list", method = RequestMethod.GET)
	public ResponseEntity> userDetails() {
        
		List userDetails = userService.getUserDetails();
		return new ResponseEntity>(userDetails, HttpStatus.OK);
	}

}


Defining Service Class


UserServiceImpl.java
package com.harsh.service.impl;

import java.util.List;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

import com.harsh.dao.UserDao;
import com.harsh.model.UserDetails;
import com.harsh.service.UserService;

@Service
public class UserServiceImpl implements UserService {

	@Autowired
	private UserDao userDao;

	public List getUserDetails() {
		return userDao.getUserDetails();
	}

}

Defining Dao Implementation

Let us define the dao.


UserDaoImpl.java
package com.harsh.dao.impl;

import java.util.List;

import org.hibernate.Criteria;
import org.hibernate.SessionFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;

import com.harsh.dao.UserDao;
import com.harsh.model.UserDetails;

@Component
public class UserDaoImpl implements UserDao {
	
	@Autowired
	private SessionFactory sessionFactory;

	public List getUserDetails() {
		Criteria criteria = sessionFactory.openSession().createCriteria(UserDetails.class);
		return criteria.list();
	}

}


Note:
We can also get hibernate session in following way using JPA entitymanager. But since this article is about spring boot and hibernate integration, we are injecting hibernate sessionfactory and getting session out of it. In next post we will be discussing about spring data with spring boot.


UserDaoImpl.java
@Component
public class UserDaoImpl implements UserDao {
	
    @PersistenceContext
    private EntityManager entityManager;

	public List getUserDetails() {
		Criteria criteria = entityManager.unwrap(Session.class).createCriteria(UserDetails.class);
		return criteria.list();
	}

}


Sample Script

Following are some sample DML. We will be creating some dummy user details using following insert statements.


create table User_Details (id integer not null auto_increment, email varchar(255), first_Name varchar(255), last_Name varchar(255), password varchar(255), primary key (id)) ENGINE=InnoDB;

INSERT INTO user_details(email,first_Name,last_Name,password) VALUES ('admin@admin.com','admin','admin','admin');

INSERT INTO user_details(email,first_Name,last_Name,password) VALUES ('john@gmail.com','john','doe','johndoe');

INSERT INTO user_details(email,first_Name,last_Name,password) VALUES ('sham@yahoo.com','sham','tis','shamtis');


Run Spring Boot Hibernate Application

1. Run Application.java as a java application.
2. Hit the url - http://localhost:8080/list. 
