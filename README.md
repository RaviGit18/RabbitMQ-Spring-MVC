# RabbitMQ Spring MVC Application

A traditional Spring MVC web application demonstrating RabbitMQ integration with classic Spring configuration and web-based messaging interface.

## Overview

This module showcases how to integrate RabbitMQ with a traditional Spring MVC application using XML-based configuration, providing a web interface for message publishing and consumption.

## Features

### Spring MVC Integration
- **Traditional Spring MVC** architecture with XML configuration
- **Web-based Interface** for message publishing
- **RabbitMQ Integration** using Spring AMQP
- **WAR Deployment** structure
- **JSP/Servlet** support

### Key Components
- `TestController.java` - MVC controller for web interface
- `RabbitMQConsumer.java` - Message consumer implementation
- `RabbitMQConfig.java` - RabbitMQ configuration
- `Person.java` - Message model/entity
- `spring-servlet.xml` - Spring MVC configuration
- `web.xml` - Web application deployment descriptor

## Technology Stack

- **Spring Framework 5.1.5.RELEASE**
- **Spring MVC** (Web framework)
- **Spring AMQP** (RabbitMQ integration)
- **Java 8**
- **Maven**
- **JSP/Servlet API**

## Prerequisites

- Java 8 or higher
- Maven 3.6 or higher
- RabbitMQ server running on localhost:5672
- Servlet container (Tomcat 8.5+)

## Quick Start

### 1. Start RabbitMQ Server
```bash
# Using Docker
docker run -d --name rabbitmq -p 5672:5672 -p 15672:15672 rabbitmq:3-management
```

### 2. Build the Application
```bash
mvn clean package
```

### 3. Deploy to Tomcat

#### Option A: Using Maven Tomcat Plugin
```bash
mvn tomcat7:run
```

#### Option B: Manual Deployment
1. Copy `target/RabbitMQ-Spring-MVC-0.0.1-SNAPSHOT.war` to Tomcat's `webapps` directory
2. Start Tomcat server
3. Access application at `http://localhost:8080/RabbitMQ-Spring-MVC/`

### 4. Test the Application

Open your browser and navigate to:
- `http://localhost:8080/RabbitMQ-Spring-MVC/` - Home page
- `http://localhost:8080/RabbitMQ-Spring-MVC/send` - Message sending interface

## Configuration

### Web Configuration (web.xml)
```xml
<web-app>
    <servlet>
        <servlet-name>spring</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <param-value>/WEB-INF/config/spring-servlet.xml</param-value>
        </init-param>
    </servlet>
    <servlet-mapping>
        <servlet-name>spring</servlet-name>
        <url-pattern>/</url-pattern>
    </servlet-mapping>
</web-app>
```

### Spring Configuration (spring-servlet.xml)
```xml
<beans>
    <!-- Component scanning -->
    <context:component-scan base-package="com.demo.controllers" />
    
    <!-- MVC annotation support -->
    <mvc:annotation-driven />
    
    <!-- View resolver -->
    <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
        <property name="prefix" value="/WEB-INF/views/" />
        <property name="suffix" value=".jsp" />
    </bean>
</beans>
```

### RabbitMQ Configuration
```java
@Configuration
public class RabbitMQConfig {
    
    @Bean
    public ConnectionFactory connectionFactory() {
        ConnectionFactory factory = new ConnectionFactory();
        factory.setHost("localhost");
        factory.setPort(5672);
        factory.setUsername("guest");
        factory.setPassword("guest");
        return factory;
    }
    
    @Bean
    public RabbitTemplate rabbitTemplate(ConnectionFactory connectionFactory) {
        return new RabbitTemplate(connectionFactory);
    }
}
```

## Project Structure

```
src/
├── main/
│   ├── java/
│   │   └── com/
│   │       └── demo/
│   │           └── controllers/
│   │               ├── TestController.java        # MVC controller
│   │               ├── RabbitMQConsumer.java      # Message consumer
│   │               ├── RabbitMQConfig.java        # RabbitMQ config
│   │               └── Person.java                # Message model
│   └── webapp/
│       └── WEB-INF/
│           ├── config/
│           │   └── spring-servlet.xml            # Spring config
│           ├── web.xml                           # Deployment descriptor
│           ├── views/                            # JSP views (if any)
│           └── lib/                              # Dependencies
└── test/
    └── java/
        └── ...                                    # Test classes
```

## Web Interface

### Home Page (/)
Simple welcome page with application information.

### Message Sending (/send)
Web interface for sending Person messages to RabbitMQ.

#### Form Fields
- **Name**: Person's name
- **Age**: Person's age  
- **City**: Person's city

#### Example Form Submission
```html
<form action="/RabbitMQ-Spring-MVC/send" method="post">
    <input type="text" name="name" value="John Doe" />
    <input type="number" name="age" value="30" />
    <input type="text" name="city" value="New York" />
    <input type="submit" value="Send Message" />
</form>
```

## Key Components Explained

### TestController
```java
@Controller
public class TestController {
    
    @Autowired
    private RabbitTemplate rabbitTemplate;
    
    @RequestMapping("/")
    public String home() {
        return "home";
    }
    
    @RequestMapping(value = "/send", method = RequestMethod.POST)
    public String sendMessage(@RequestParam String name, 
                             @RequestParam int age, 
                             @RequestParam String city) {
        Person person = new Person(name, age, city);
        rabbitTemplate.convertAndSend("queue-1", person);
        return "success";
    }
}
```

### RabbitMQConsumer
```java
@Component
public class RabbitMQConsumer {
    
    @RabbitListener(queues = "queue-1")
    public void receiveMessage(Person person) {
        System.out.println("Received message: " + person);
        // Process the message
    }
}
```

### Person Model
```java
public class Person {
    private String name;
    private int age;
    private String city;
    
    // Constructors, getters, and setters
}
```

## Maven Configuration

### Dependencies
- **Spring Web MVC** - Web framework
- **Spring Context** - Core Spring framework
- **Spring Rabbit** - RabbitMQ integration
- **JSTL** - JSP tag library
- **Servlet API** - Web API

### Build Configuration
```xml
<build>
    <sourceDirectory>src</sourceDirectory>
    <plugins>
        <plugin>
            <artifactId>maven-war-plugin</artifactId>
            <version>2.3</version>
            <configuration>
                <warSourceDirectory>WebContent</warSourceDirectory>
                <failOnMissingWebXml>false</failOnMissingWebXml>
            </configuration>
        </plugin>
    </plugins>
</build>
```

## Deployment

### Tomcat Deployment
1. Build the WAR file: `mvn clean package`
2. Deploy to Tomcat:
   - Copy WAR to `$TOMCAT_HOME/webapps/`
   - Or use Tomcat Manager web interface
3. Access at: `http://localhost:8080/RabbitMQ-Spring-MVC/`

### Alternative Containers
- **Jetty**: Use `mvn jetty:run`
- **WildFly**: Deploy WAR file
- **WebLogic**: Deploy WAR file

## Testing

### Manual Testing
1. Deploy application to servlet container
2. Navigate to web interface
3. Fill out and submit the message form
4. Check console for consumed messages

### Integration Testing
```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(locations = {"classpath:spring-servlet.xml"})
public class RabbitMQIntegrationTest {
    
    @Autowired
    private TestController controller;
    
    @Test
    public void testMessageSending() {
        // Test message sending functionality
    }
}
```

## Monitoring and Management

### RabbitMQ Management UI
Access RabbitMQ management interface at: `http://localhost:15672`
- Monitor queues and exchanges
- View message rates
- Check connection status

### Application Logs
Check container logs for:
- Message publishing/consumption
- Connection errors
- Application startup issues

## Troubleshooting

### Common Issues
1. **Deployment Failures** - Check web.xml configuration
2. **404 Errors** - Verify URL patterns and servlet mapping
3. **RabbitMQ Connection** - Ensure server is running
4. **Message Not Received** - Check queue configuration

### Debug Tips
- Check container logs for detailed error messages
- Verify RabbitMQ management UI shows active connections
- Test with simple REST client first
- Check Spring configuration files

## Migration to Spring Boot

This application can be migrated to Spring Boot for easier deployment:

1. Add Spring Boot starter dependencies
2. Replace XML configuration with Java configuration
3. Add `@SpringBootApplication` annotation
4. Create executable JAR instead of WAR

## Best Practices

1. **Use proper error handling** in controllers
2. **Validate input data** before processing
3. **Implement logging** for debugging
4. **Use connection pooling** for production
5. **Add security** to web endpoints

## Next Steps

- Add input validation
- Implement error handling pages
- Add AJAX for dynamic content
- Create admin interface
- Add message history tracking
- Implement user authentication
