# Custom Auto-Configuration with Spring Boot

### 1. Overview

Simply put, the Spring Boot auto-configuration helps us automatically configure a Spring application based on the dependencies that are present on the classpath.

This can make development faster and easier by eliminating the need to define certain beans included in the auto-configuration classes.
In the following section, we’ll look at creating our custom Spring Boot auto-configuration.

### 2. Maven Dependencies

Let’s start with the dependencies:

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <version>8.0.19</version>
</dependency>
```

The latest versions of spring-boot-starter-data-jpa and mysql-connector-java can be downloaded from Maven Central.


### 3. Creating a Custom Auto-Configuration

In order to create a custom auto-configuration, we need to create a class annotated as @Configuration and register it.
Let’s create a custom configuration for a MySQL data source:

```java
@Configuration
public class MySQLAutoconfiguration {
    //...
}
```

Next, we need to register the class as an auto-configuration candidate.
We do this by adding the name of the class under the key org.springframework.boot.autoconfigure.EnableAutoConfiguration in the standard file resources/META-INF/spring.factories:

```java
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
com.baeldung.autoconfiguration.MySQLAutoconfiguration
```

If we want our auto-configuration class to have priority over other candidates, we can add the @AutoConfigureOrder(Ordered.HIGHEST_PRECEDENCE) annotation.


We design the auto-configuration using classes and beans marked with @Conditional annotations so that we can replace the auto-configuration or specific parts of it.
Note that the auto-configuration is only in effect if we don’t define the auto-configured beans in the application. If we define our bean, it will override the default one.

### 3.1. Class Conditions

Class conditions allow us to specify that we want to include a configuration bean if a specified class is present using the @ConditionalOnClass annotation, or if a class is absent using the @ConditionalOnMissingClass annotation.
Let’s specify that our MySQLConfiguration will load only if the class DataSource is present, in which case we can assume the application will use a database:

```java
@Configuration
@ConditionalOnClass(DataSource.class)
public class MySQLAutoconfiguration {
    //...
}
```

### 3.2. Bean Conditions

If we want to include a bean only if a specified bean is present or not, we can use the @ConditionalOnBean and @ConditionalOnMissingBean annotations.

To look at this, let’s add an entityManagerFactory bean to our configuration class.
First, we’ll specify that we only want to create this bean if a bean called dataSource is present and if a bean called entityManagerFactory is not already defined:

```java
@Bean
@ConditionalOnBean(name = "dataSource")
@ConditionalOnMissingBean
public LocalContainerEntityManagerFactoryBean entityManagerFactory() {
    LocalContainerEntityManagerFactoryBean em
      = new LocalContainerEntityManagerFactoryBean();
    em.setDataSource(dataSource());
    em.setPackagesToScan("com.baeldung.autoconfiguration.example");
    em.setJpaVendorAdapter(new HibernateJpaVendorAdapter());
    if (additionalProperties() != null) {
        em.setJpaProperties(additionalProperties());
    }
    return em;
}
```

Let’s also configure a transactionManager bean that will load only if we haven’t already defined a bean of type JpaTransactionManager:

```java
@Bean
@ConditionalOnMissingBean(type = "JpaTransactionManager")
JpaTransactionManager transactionManager(EntityManagerFactory entityManagerFactory) {
    JpaTransactionManager transactionManager = new JpaTransactionManager();
    transactionManager.setEntityManagerFactory(entityManagerFactory);
    return transactionManager;
}
```

### 3.3. Property Conditions

We use the @ConditionalOnProperty annotation to specify if a configuration loads based on the presence and value of a Spring Environment property.
First, let’s add a property source file for our configuration that will determine where the properties will be read from:

```java
@PropertySource("classpath:mysql.properties")
public class MySQLAutoconfiguration {
    //...
}
```

We can configure the main DataSource bean that we’ll use to create connections to the database so that it will load only if a property called usemysql is present.
We can use the attribute havingValue to specify certain values of the usemysql property that have to be matched.
Now let’s define the dataSource bean with default values that connect to a local database called myDb if we set the usemysql property to local:

```java
@Bean
@ConditionalOnProperty(
  name = "usemysql", 
  havingValue = "local")
@ConditionalOnMissingBean
public DataSource dataSource() {
    DriverManagerDataSource dataSource = new DriverManagerDataSource();
 
    dataSource.setDriverClassName("com.mysql.cj.jdbc.Driver");
    dataSource.setUrl("jdbc:mysql://localhost:3306/myDb?createDatabaseIfNotExist=true");
    dataSource.setUsername("mysqluser");
    dataSource.setPassword("mysqlpass");

    return dataSource;
}
```

If we set the usemysql property to custom, we’ll configure the dataSource bean using custom properties values for the database URL, user and password:

```java
@Bean(name = "dataSource")
@ConditionalOnProperty(
  name = "usemysql", 
  havingValue = "custom")
@ConditionalOnMissingBean
public DataSource dataSource2() {
    DriverManagerDataSource dataSource = new DriverManagerDataSource();
        
    dataSource.setDriverClassName("com.mysql.cj.jdbc.Driver");
    dataSource.setUrl(env.getProperty("mysql.url"));
    dataSource.setUsername(env.getProperty("mysql.user") != null 
      ? env.getProperty("mysql.user") : "");
    dataSource.setPassword(env.getProperty("mysql.pass") != null 
      ? env.getProperty("mysql.pass") : "");
        
    return dataSource;
}
```

The mysql.properties file will contain the usemysql property:

usemysql=local

An application that uses the MySQLAutoconfiguration may need to override the default properties. In this case, it just needs to add different values for the mysql.url, mysql.user and mysql.pass properties and the usemysql=custom line in the mysql.properties file.

### 3.4. Resource Conditions

Adding the @ConditionalOnResource annotation means that the configuration loads only when a specified resource is present.
Let’s define a method called additionalProperties() that will return a Properties object containing Hibernate-specific properties to be used by the entityManagerFactory bean, only if the resource file mysql.properties is present:

```java
@ConditionalOnResource(
  resources = "classpath:mysql.properties")
@Conditional(HibernateCondition.class)
Properties additionalProperties() {
    Properties hibernateProperties = new Properties();

    hibernateProperties.setProperty("hibernate.hbm2ddl.auto", 
      env.getProperty("mysql-hibernate.hbm2ddl.auto"));
    hibernateProperties.setProperty("hibernate.dialect", 
      env.getProperty("mysql-hibernate.dialect"));
    hibernateProperties.setProperty("hibernate.show_sql", 
      env.getProperty("mysql-hibernate.show_sql") != null 
      ? env.getProperty("mysql-hibernate.show_sql") : "false");
    return hibernateProperties;
}
```

We can add the Hibernate-specific properties to the mysql.properties file:

```txt
mysql-hibernate.dialect=org.hibernate.dialect.MySQLDialect
mysql-hibernate.show_sql=true
mysql-hibernate.hbm2ddl.auto=create-drop
```

### 3.5. Custom Conditions

Let’s say we don’t want to use any of the conditions available in Spring Boot.
We can also define custom conditions by extending the SpringBootCondition class and overriding the getMatchOutcome() method.

Let’s create a condition called HibernateCondition for our additionalProperties() method that will verify whether a HibernateEntityManager class is present on the classpath:

```java
static class HibernateCondition extends SpringBootCondition {

    private static String[] CLASS_NAMES
      = { "org.hibernate.ejb.HibernateEntityManager", 
          "org.hibernate.jpa.HibernateEntityManager" };

    @Override
    public ConditionOutcome getMatchOutcome(ConditionContext context, 
      AnnotatedTypeMetadata metadata) {
 
        ConditionMessage.Builder message
          = ConditionMessage.forCondition("Hibernate");
        return Arrays.stream(CLASS_NAMES)
          .filter(className -> ClassUtils.isPresent(className, context.getClassLoader()))
          .map(className -> ConditionOutcome
            .match(message.found("class")
            .items(Style.NORMAL, className)))
          .findAny()
          .orElseGet(() -> ConditionOutcome
            .noMatch(message.didNotFind("class", "classes")
            .items(Style.NORMAL, Arrays.asList(CLASS_NAMES))));
    }
}
```

Then we can add the condition to the additionalProperties() method:

```java
@Conditional(HibernateCondition.class)
Properties additionalProperties() {
  //...
}
```

### 3.6. Application Conditions

We can also specify that the configuration can load only inside/outside a web context. In order to do this, we can add the @ConditionalOnWebApplication or @ConditionalOnNotWebApplication annotation.

### 4. Testing the Auto-Configuration

Let’s create a very simple example to test our auto-configuration.
We will create an entity class called MyUser and a MyUserRepository interface using Spring Data:

```java
@Entity
public class MyUser {
    @Id
    private String email;

    // standard constructor, getters, setters
}

public interface MyUserRepository 
  extends JpaRepository<MyUser, String> { }
```

In order to enable auto-configuration, we can use one of the @SpringBootApplication or @EnableAutoConfiguration annotations:

```java
@SpringBootApplication
public class AutoconfigurationApplication {
    public static void main(String[] args) {
        SpringApplication.run(AutoconfigurationApplication.class, args);
    }
}
```

Next, let’s write a JUnit test that saves a MyUser entity:

```java
@RunWith(SpringJUnit4ClassRunner.class)
@SpringBootTest(
  classes = AutoconfigurationApplication.class)
@EnableJpaRepositories(
  basePackages = { "com.baeldung.autoconfiguration.example" })
public class AutoconfigurationLiveTest {

    @Autowired
    private MyUserRepository userRepository;

    @Test
    public void whenSaveUser_thenOk() {
        MyUser user = new MyUser("user@email.com");
        userRepository.save(user);
    }
}
```

Since we didn’t define our DataSource configuration, the application will use the auto-configuration we created to connect to a MySQL database called myDb.

The connection string contains the createDatabaseIfNotExist=true property, so the database does not need to exist. However, the user mysqluser, or the one specified through the mysql.user property if it is present, needs to be created.
We can check the application log to see that we’re using the MySQL data source:

web - 2017-04-12 00:01:33,956 [main] INFO  o.s.j.d.DriverManagerDataSource - Loaded JDBC driver: com.mysql.cj.jdbc.Driver

### 5. Disabling Auto-Configuration Classes

Let’s say we want to exclude the auto-configuration from loading.
We could add the @EnableAutoConfiguration annotation with exclude or excludeName attribute to a configuration class:

```java
@Configuration
@EnableAutoConfiguration(
  exclude={MySQLAutoconfiguration.class})
public class AutoconfigurationApplication {
    //...
}
```

We can also set the spring.autoconfigure.exclude property:

spring.autoconfigure.exclude=com.baeldung.autoconfiguration.MySQLAutoconfiguration

### 6. Conclusion

In this article, we’ve shown how to create a custom Spring Boot auto-configuration.
The full source code of the example can be found over on GitHub.
The JUnit test can be run using the autoconfiguration profile mvn clean install -Pautoconfiguration.
