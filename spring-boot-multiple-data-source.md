## 🧱 Integrating multiple data sources in a Spring Boot application is a common need in microservices and enterprise systems. Here’s a step-by-step explanation with code examples:

⸻

### ✅ Step 1: Define Multiple Data Sources in application.properties or application.yml

```properties
# Primary (default) DataSource
spring.datasource.url=jdbc:mysql://localhost:3306/maindb
spring.datasource.username=root
spring.datasource.password=secret
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver

# Secondary DataSource
secondary.datasource.url=jdbc:postgresql://localhost:5432/otherdb
secondary.datasource.username=postgres
secondary.datasource.password=secret
secondary.datasource.driver-class-name=org.postgresql.Driver
```

⸻

### ✅ Step 2: Define Configuration for Primary DataSource

```java
@Configuration
@EnableTransactionManagement
@EnableJpaRepositories(
    basePackages = "com.example.primary.repository",
    entityManagerFactoryRef = "primaryEntityManager",
    transactionManagerRef = "primaryTransactionManager"
)
public class PrimaryDataSourceConfig {

    @Bean
    @Primary
    @ConfigurationProperties("spring.datasource")
    public DataSource primaryDataSource() {
        return DataSourceBuilder.create().build();
    }

    @Bean
    @Primary
    public LocalContainerEntityManagerFactoryBean primaryEntityManager(
            EntityManagerFactoryBuilder builder) {
        return builder
                .dataSource(primaryDataSource())
                .packages("com.example.primary.model")
                .persistenceUnit("primary")
                .build();
    }

    @Bean
    @Primary
    public PlatformTransactionManager primaryTransactionManager(
            @Qualifier("primaryEntityManager") EntityManagerFactory emf) {
        return new JpaTransactionManager(emf);
    }
}
```

⸻

### ✅ Step 3: Define Configuration for Secondary DataSource

```java
@Configuration
@EnableJpaRepositories(
    basePackages = "com.example.secondary.repository",
    entityManagerFactoryRef = "secondaryEntityManager",
    transactionManagerRef = "secondaryTransactionManager"
)
public class SecondaryDataSourceConfig {

    @Bean
    @ConfigurationProperties("secondary.datasource")
    public DataSource secondaryDataSource() {
        return DataSourceBuilder.create().build();
    }

    @Bean
    public LocalContainerEntityManagerFactoryBean secondaryEntityManager(
            EntityManagerFactoryBuilder builder) {
        return builder
                .dataSource(secondaryDataSource())
                .packages("com.example.secondary.model")
                .persistenceUnit("secondary")
                .build();
    }

    @Bean
    public PlatformTransactionManager secondaryTransactionManager(
            @Qualifier("secondaryEntityManager") EntityManagerFactory emf) {
        return new JpaTransactionManager(emf);
    }
}
```

⸻

### ✅ Step 4: Define Entities and Repositories Separately

Keep your entities and repositories in clearly separated packages:
	•	com.example.primary.model / com.example.primary.repository
	•	com.example.secondary.model / com.example.secondary.repository

This ensures that each repository scans the correct data source.

⸻

Bonus: Using Transactional with Specific Managers

You can annotate services like this:

```java
@Transactional("secondaryTransactionManager")
public void doSomethingInSecondaryDB() {
    // logic
}
```

⸻
