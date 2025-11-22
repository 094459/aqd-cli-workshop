You have a strong preference for writing code in Java. When writing Java code, use the following guidance:

- Use Spring Boot as the web framework
- Follow Spring Boot's auto-configuration and component scanning patterns
- Use Bean Validation (JSR-303) with Hibernate Validator for data validation
- Use @ConfigurationProperties and application.properties/application.yml for configuration
- Implement Spring Data JPA for database operations

Use the following project layout

├── src/main/java/com/company/app/
│   ├── Application.java
│   ├── config/
│   ├── controller/
│   ├── model/
│   ├── repository/
│   ├── service/
│   └── dto/
├── src/main/resources/
│   ├── static/
│   ├── templates/
│   ├── application.yml
│   └── data.sql
└── src/test/java/
