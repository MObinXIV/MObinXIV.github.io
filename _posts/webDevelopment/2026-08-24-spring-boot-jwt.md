# Spring boot Security With JWT

![Spring Security Img](/assets/springSecurity.png)
## Introduction
Security is not a feature , it's a responsibility.
In every app we developers implement the most important thing we must care about is the security of
our app, as it's so important to keep the user always secured from attacks , keeping his important data
protected always is a developer responsibility , actually developer must restrict the normal user from
actions which will lead his data to be stolen which is a real problem.
here we 're going to discuss spring boot security , what really a basic security is, also
we 'll discuss the standard JWT authentication with spring boot. without wasting anytime , let's
go and see.
---
## What's Simply Security is ?
The Standard Security always Combined by **Authentication & Authorization**
which always 've basic Question we need to answer **Who & What**
- Who -> Authentication context answer , We answer it with **The Identity** which simply (Username , email /Password) on other way the Login form we always face in any app
- What -> Authorization context or we can say the credentials which we allow our user to 've permissions
  , access of the user , it's the role of user in our simply we mostly simplified it in the context
  of (USER/ADMIN) each system is rounded in user/users roles and admin/admin roles each one has its
  credential which allowed to it , Admin always has a big hand in the system administration internally by the way.
---
## JWT In Spring boot Backend Development:

In spring boot there's now standard way in the framework itself support us with basic CRUD Operations
for example and standard config , as Spring boot doesn't 've even a default implementation for the
JWT Implementation which make it mainly made by us , which lead it maybe harming for us in the implementation
and also a lot of us we feel harsh in implementing it specially first time and it's normal , but
let's go and simplify it.

---


## What's Really a Token is:

![jwtConstructionImg1](/assets/jwtConstruct2.png)
![jwtConstructionImg2](/assets/jwtConstruct1.png)
- As we see this an example for the token the client receives `eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiYWRtaW4iOnRydWUsImlhdCI6MTUxNjIzOTAyMn0.KMUFsIDTnFmyG3nMiGM6H9FNFUROf3wh7SmqJp-QV30`
- Any token has absolute 3 parts -> **Header**,**Payload** & **Signature**
1. **Header** -> `{
   "alg": "HS256",
   "typ": "JWT"
   }` as we see , simply it contains `alg`(encryption algorithm) HS256(HMAC using SHA-256 — a symmetric algorithm)
   & the type as a metadata verify the type of token is JWT token.
2. **Payload** -> `{
   "sub": "aliii@gmail.com",
   "iat": 1787523892,
   "exp": 1787532532,
   "authorities": [
   "USER"
   ]
   }` , Payload is simply our info  , `sub` verifies the token which 've been generated belong to whom
   in our case our subject we use is the user email , `iat` simply tells when the token issued at ,
   `exp` is the expiration date of the token it tells when the token 'll be expired for the purpose of security polices to always force the user to  get a fresh token
   , `authorities` is a claim verifies the role of our client to permit credentials for the authenticated user
3. **Signature** -> `404E635266556A586E3272357538782F413F4428472B4B6250645367566B59` , the signature is simply a key which you can generated using any key generation site , it's main role is to check the integrity of the token if there's any single modification of it or not if there any difference the server rejects it immediately.

#### Important Notes about Token:
- Token is stateless means that we don't save it in the server , so each request is treated independently not saved it's state
- We never ever put Password in the token payload , never ever as it destroy the security in case of any attack , we do every thing to make the password not accessible encrypt it even it's not saved with its value , so don't go and give it in the payload for the attackers.

---


## Spring Boot JWT Life Cycle:

Let's Discuss this Flow Step by step
![JwtFlow](/assets/jwtImg.png)


- It starts when user sends HTTP request to our backend application which is running in spring boot.
- Our work is going with filters(Filters are simply interceptors that sit in front of your controllers and process every incoming HTTP request before it reaches your actual endpoint logic) actually.
- So when you ask to access our controller , **JwtAuthFilter** checks if you 've a Jwt token or not , if the user has no token, we send to him **403**(Forbidden) response with exception (Token doesn't exist)
- In Case we 've the token, we go through some validation steps.
- **JwtAuthFilter** start the process of extraction the user details from the token subject , it first extracts **userEmail or username** the subject which we provide in the payload
  , then the **UserDetailsService** go to check if the subject exists in the database or not , case it's not exists **UserDetailsService** sends to **JwtAuthFilter** the user is not found , then **JwtAuthService**
  sends a **401 Unauthorized Exception** (**User Doesn't exist**).
- In case **UserDetailsService** provides the  **JwtAuthService** with acceptance.
- **JwtAuthService** go validate the **expiration date** in the payload , case  the token has been
  expired , then **JwtAuthService** sends **401 exception** (Invalid Jwt token).
- In Case everything goes right , **JwtAuthService** updates **SecurityContextHolder**(it's simply the place which 'll set  the user authenticated and give him the credentials to its role)
- once the **SecurityContextHolder** update the status of user to be authenticated it 'll go to the **DispatcherServlet**(it's simply the part which is corresponding to dispatch the request to the corresponding controller)
  picks the **response** send it back to you with http status (200 accepted).
  So this was our Big Picture.

## Implementation

Now Let's go  in The Practical Part see how can we Do these steps in spring boot.



### Configuration Requirements:
On your machine you need to setup and configure this:
- Java SE 21 IDE 
- (IntelliJ IDEA)
- Postman (or Curl)
- PostgreSQL v17
- Docker

### Code Implementation:

### Project Structure

```plaintext
springBootSecurityJwt
├── src
│   ├── main
│   │   ├── java
│   │   │   └── com.mobin.springbootsecurityjwt
│   │   │       ├── authentication
│   │   │       │   ├── AuthenticationController.java
│   │   │       │   ├── AuthenticationRequest.java
│   │   │       │   ├── AuthenticationResponse.java
│   │   │       │   ├── AuthenticationService.java
│   │   │       │   └── RegistrationRequest.java
│   │   │       │
│   │   │       ├── common
│   │   │       │   └── BaseAuditingEntity.java
│   │   │       │
│   │   │       ├── config
│   │   │       │   ├── ApplicationAuditAware.java
│   │   │       │   └── BeansConfig.java
│   │   │       │
│   │   │       ├── email
│   │   │       │   ├── EmailService.java
│   │   │       │   └── EmailTemplateName.java
│   │   │       │
│   │   │       ├── handler
│   │   │       │   ├── BusinessErrorCodes.java
│   │   │       │   ├── ExceptionResponse.java
│   │   │       │   └── GlobalExceptionHandler.java
│   │   │       │
│   │   │       ├── role
│   │   │       │   ├── Role.java
│   │   │       │   └── RoleRepository.java
│   │   │       │
│   │   │       ├── security
│   │   │       │   ├── JwtFilter.java
│   │   │       │   ├── JwtService.java
│   │   │       │   ├── SecurityConfig.java
│   │   │       │   └── UserDetailsServiceImpl.java
│   │   │       │
│   │   │       ├── user
│   │   │       │   ├── Token.java
│   │   │       │   ├── TokenRepository.java
│   │   │       │   ├── User.java
│   │   │       │   └── UserRepository.java
│   │   │       │
│   │   │       └── SpringBootSecurityJwtApplication.java
│   │   │
│   │   └── resources
│   │       ├── templates
│   │       │   └── activate_account.html
│   │       ├── application.yml
│   │       └── application-dev.yml
│   │
│   └── test
│       └── java
│           └── com.mobin.springbootsecurityjwt
│               └── SpringBootSecurityJwtApplicationTests.java
│
└── pom.xml
.....
```

### DataBase Schema:

![JwtFlow](/assets/springSecurityDb.png)

#### Flow summary: 

- User / Role -> M-N relationship with User_Role Bridge Table between them achieving m-n relationship.
- User/Token -> 1->n relationship as user can many token.

### Code Implementation: 

let's now go to the code implementation step by step. 

#### 1. Create our project

Go to [Spring Initializer](https://start.spring.io/)

![springIoPic](/assets/springIo.png)
 
and here our pom file , as there's dependencies like the **JWT dependencies** which as we say not built-in spring boot even 

here's `pom.xml` config

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>4.1.0</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
    <groupId>com.mobin</groupId>
    <artifactId>springSecurity</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>springSecurity</name>
    <description>springSecurity</description>
    <url/>
    <licenses>
        <license/>
    </licenses>
    <developers>
        <developer/>
    </developers>
    <scm>
        <connection/>
        <developerConnection/>
        <tag/>
        <url/>
    </scm>
    <properties>
        <java.version>21</java.version>
    </properties>
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-jpa</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-security</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-thymeleaf</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-validation</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-webmvc</artifactId>
        </dependency>
        <dependency>
            <groupId>org.postgresql</groupId>
            <artifactId>postgresql</artifactId>
            <scope>runtime</scope>
        </dependency>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>
        <dependency>
            <groupId>io.jsonwebtoken</groupId>
            <artifactId>jjwt-api</artifactId>
            <version>0.12.5</version>
        </dependency>
        <dependency>
            <groupId>org.springdoc</groupId>
            <artifactId>springdoc-openapi-starter-webmvc-ui</artifactId>
            <version>2.8.6</version>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-mail</artifactId>
        </dependency>
        <!-- JWT Implementation -->
        <dependency>
            <groupId>io.jsonwebtoken</groupId>
            <artifactId>jjwt-impl</artifactId>
            <version>0.12.5</version>
            <scope>runtime</scope>
        </dependency>

        <!-- JWT Jackson Serializer/Deserializer -->
        <dependency>
            <groupId>io.jsonwebtoken</groupId>
            <artifactId>jjwt-jackson</artifactId>
            <version>0.12.5</version>
            <scope>runtime</scope>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-jpa-test</artifactId>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-security-test</artifactId>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-thymeleaf-test</artifactId>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-validation-test</artifactId>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-webmvc-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <executions>
                    <execution>
                        <id>default-compile</id>
                        <phase>compile</phase>
                        <goals>
                            <goal>compile</goal>
                        </goals>
                        <configuration>
                            <annotationProcessorPaths>
                                <path>
                                    <groupId>org.projectlombok</groupId>
                                    <artifactId>lombok</artifactId>
                                </path>
                            </annotationProcessorPaths>
                        </configuration>
                    </execution>
                    <execution>
                        <id>default-testCompile</id>
                        <phase>test-compile</phase>
                        <goals>
                            <goal>testCompile</goal>
                        </goals>
                        <configuration>
                            <annotationProcessorPaths>
                                <path>
                                    <groupId>org.projectlombok</groupId>
                                    <artifactId>lombok</artifactId>
                                </path>
                            </annotationProcessorPaths>
                        </configuration>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>

</project>
```

#### 2. Configure Docker container and application.yml:

Here we are work with docker compose file to configure our full app containers  and also we containerize The DB
, and the application.yml file is for the app settings.

 - Here's the docker-compose file configurations which , we run it using the command `docker-compose -d`

`docker-compose.yml`

```dockerfile
services:
  postgres:
    container_name: postgres-sql-bsn
    image: postgres
    environment:
      POSTGRES_USER: username
      POSTGRES_PASSWORD: password
      PGDATA: /var/lib/postgresql/data
      POSTGRES_DB: book_social_network

    volumes:
      - postgres:/data/postgres
    ports:
      - 5433:5432
    networks:
      - spring-demo
    restart: unless-stopped

  mail-dev:
    container_name: mail-dev-bsn
    image: maildev/maildev
    ports:
      - 1080:1080
      - 1025:1025

networks:
  spring-demo:
    driver: bridge
volumes:
  postgres:
    driver: local
```
- Note -> this file with its whole commands, I 'll explain separately in an article ,but simply this a configuration for the database and mailing containers in our app
- let's now see application settings which lands in `application.yml`
 we 'll separate our profiles as root one , and dev one , if we in the future want also to include production one
`application.yml` 

```yaml
spring:
  profiles:
    active: dev

  servlet:
    multipart:
      max-file-size: 50MB
springdoc:
  default-produces-media-type: application/json
server:
  servlet:
    context-path: /api/v1/  # prefix all the end points with this end
```

`application-dev.yml`

```yaml
spring:
  datasource:
    url: jdbc:postgresql://localhost:5433/book_social_network
    username: username
    password: password
    driver-class-name: org.postgresql.Driver
  jpa:
    hibernate:
      ddl-auto: update
    show-sql: false
    properties:
      hibernate:
        format_sql: true
    database: postgresql
    database-platform: org.hibernate.dialect.PostgreSQLDialect
  mail:
    host: localhost
    port: 1025
    username: mohamed
    password: mohamed
    properties:
      mail:
        smtp:
          trust: "*"
          auth: true
          starttls:
            enable: true
            connectiontimeout: 5000
            timeout: 3000
            writetimeout: 5000
application:
  security:
    jwt:
      secret-key: 404E635266556A586E3272357538782F413F4428472B4B6250645367566B59
      expiration: 8640000
  mailing:
    frontend:
      activation-url: http//localhost:4200/activate-account
```
- Briefly -> we setup our database settings , mail settings , and put some private variables ones which securely must be hide

#### 3. Implementing the code business logic:

- Security of the app lay in **2** basic standards which the whole system 'll round about them:
1. **Registration**
2. **Login**
let's go see what happens
#### 3.1 Create our entities:
- Simply entities is the representation of our database tables in the code it annotated with `@Entity` annotation.
In the security context the first entity we always implement first is the `User.Java` entity with our basic user data we all know as email , password and the name we can work with any data else according to the use case also
, keep in mind *always* when we go to secure our app in spring boot our `Uaser.java` must implements `UserDetails` interface and als `Pricipal` interface

let's simply clarify their roles 

- `UserDetails` -> main Job is to make **Spring Security** user info which achieved through providing the functionalities of `username` , `password` , `user roles` & the account credential availabilities which achieved through `accountLocked` and `accountEnabled`
- `Principal` -> it's main Job is to tell `who is logged in right now` to provide him his own access to the controllers in our system
here this code achieve this simply take a look at it , in each `@Override` ones comes from `UserDetails` & `Principal`

`User.Java`
```java
package com.mobin.booknetworkapi.user;

import com.mobin.booknetworkapi.common.BaseAuditingEntity;
import com.mobin.booknetworkapi.role.Role;
import jakarta.persistence.*;
import lombok.*;
import org.jspecify.annotations.Nullable;
import org.springframework.security.core.GrantedAuthority;
import org.springframework.security.core.authority.SimpleGrantedAuthority;
import org.springframework.security.core.userdetails.UserDetails;

import javax.security.auth.Subject;
import java.security.Principal;
import java.util.Collection;
import java.util.List;
import java.util.stream.Collectors;

@Getter
@Setter
@Builder
@AllArgsConstructor
@NoArgsConstructor
@Entity
@Table(name="_user")
public class User extends BaseAuditingEntity implements UserDetails, Principal{
    @Id
    @GeneratedValue
    private Integer id;
    private String firstName;
    private String lastName;
    private String dateOfBirth;
    @Column(unique = true)
    private String email;
    private String password;
    private boolean accountLocked;
    private boolean enabled;
    @ManyToMany(fetch =FetchType.EAGER)
    private List<Role> roles;
    @Override
    public String getName() {
        return email;
    }

    @Override
    public boolean implies(Subject subject) {
        return Principal.super.implies(subject);
    }

    @Override
    public Collection<? extends GrantedAuthority> getAuthorities() {
        return this.roles
                .stream()
                .map(r-> new SimpleGrantedAuthority(r.getName()))
                .collect(Collectors.toList());
    }

    @Override
    public @Nullable String getPassword() {
        return password;
    }

    @Override
    public String getUsername() {
        return email;
    }

    @Override
    public boolean isAccountNonExpired() {
        return true;
    }

    @Override
    public boolean isAccountNonLocked() {
        return !accountLocked;
    }

    @Override
    public boolean isCredentialsNonExpired() {
        return true;
    }

    @Override
    public boolean isEnabled() {
        return enabled;
    }
    public String getFullName(){
        return firstName +" "+lastName;
    }
}
```
after this we apply the  roles in our system and this achieved  all things goes according to our 
let's go and see our role entity

`Role.java`

```java
package com.mobin.booknetworkapi.role;

import com.fasterxml.jackson.annotation.JsonIgnore;
import com.mobin.booknetworkapi.common.BaseAuditingEntity;
import com.mobin.booknetworkapi.user.User;
import jakarta.persistence.*;
import lombok.*;

import java.util.List;

@Getter
@Setter
@Builder
@AllArgsConstructor
@NoArgsConstructor
@Entity
public class Role extends BaseAuditingEntity {
    @Id
    @GeneratedValue
    private Integer id;
    @Column(unique = true)
    private String name;
    // establish the relation between user & roles
    @ManyToMany(mappedBy = "roles")
    @JsonIgnore
    List<User> users;

}
```

now as we see we configured our `M-N` relationship between User and Role through these snippets 


```java
@ManyToMany(fetch =FetchType.EAGER)
    private List<Role> roles;
```
```java
@ManyToMany(mappedBy = "roles")
    @JsonIgnore
    List<User> users;
```

here we achieve the relation between `user` and `role`
as we always do some important things when we fetch using `FetchType.EAGER` it helps us to fetch the roles eagerly which is suitable to our case 
also there's other important annotation `@JsonIgnore` it main work to ignore the users when we fetch from the role side which serves the logical thinking sa it's not needed and 'll  consume time.
- Also, Important consideration we see `@ManyToMany` annotation makes Hibernate, persist for us the bridge table instead of making it ourselves.

let's now see our `Token.Java` entity 

`Token.java`
```java
package com.mobin.booknetworkapi.user;

import com.mobin.booknetworkapi.common.BaseAuditingEntity;
import jakarta.persistence.*;
import lombok.*;

import java.time.LocalDateTime;

@Getter
@Setter
@Builder
@AllArgsConstructor
@NoArgsConstructor
@Entity
public class Token{
    @Id
    @GeneratedValue
    private Integer id;
    private String token;
    private LocalDateTime createdAt;
    private LocalDateTime expiresAt;
    private LocalDateTime validatedAt;
    @ManyToOne
    @JoinColumn(name = "userId", nullable = false)
    private User user;
}
```
as we see its simply contain the token data, and also we also persist our `1-N` user-token relationship

Let's come to see Really what is `BaseAuditingEntity.java` it's an **Auditing Entity Listener** its main role is helping us
to track `created/modified` cases without doing this `manually` ,Here I package it As `Refactoring` to the code instead of typing them in each entity , We `Inherit` it in our entities

let's see the code and discuss the important notes about it

`BaseAuditingEntity`

```java
package com.mobin.booknetworkapi.common;

import jakarta.persistence.Column;
import jakarta.persistence.EntityListeners;
import jakarta.persistence.MappedSuperclass;
import lombok.AllArgsConstructor;
import lombok.Getter;
import lombok.NoArgsConstructor;
import lombok.Setter;
import org.springframework.data.annotation.CreatedBy;
import org.springframework.data.annotation.CreatedDate;
import org.springframework.data.annotation.LastModifiedBy;
import org.springframework.data.annotation.LastModifiedDate;
import org.springframework.data.jpa.domain.support.AuditingEntityListener;

import java.time.LocalDateTime;

@MappedSuperclass
@NoArgsConstructor
@AllArgsConstructor
@Getter
@Setter
@EntityListeners(AuditingEntityListener.class)
public class BaseAuditingEntity {
    @CreatedDate
    @Column(name = "created_date",nullable = false,updatable = false)
    private LocalDateTime createdDate;
    @LastModifiedDate
    @Column(name = "last_modified_date",insertable = false)
    private LocalDateTime updatedDate;
    @CreatedBy
    @Column(name = "created_by",nullable = false,updatable = false)
    private String createdBy;
    @LastModifiedBy
    @Column(name = "last_modified_by", insertable = false)
    private String lastModifiedBy;
}
```
let's now discuss the important annotation which seems important:

```java
@MappedSuperclass
```
- this one holds the 4 common auditing fields we need (which also in our class as annotations):
1. createdDate -> when the record was created 
2. updatedDate -> when it was last modified 
3. createdBy -> who created it 
4. lastModifiedBy -> who last modified it

We are not finished yet, we need now to inform spring about our auditing & this done through theses steps 
in `SpringBootApplication.java` the entry point in our app 

```java
package com.mobin.booknetworkapi;

import com.mobin.booknetworkapi.role.Role;
import com.mobin.booknetworkapi.role.RoleRepository;
import jakarta.persistence.EntityListeners;
import org.springframework.boot.CommandLineRunner;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.annotation.Bean;
import org.springframework.data.jpa.domain.support.AuditingEntityListener;
import org.springframework.data.jpa.repository.config.EnableJpaAuditing;
import org.springframework.scheduling.annotation.EnableAsync;

@EnableJpaAuditing(auditorAwareRef = "applicationAuditAware")
@SpringBootApplication
@EntityListeners(AuditingEntityListener.class)
@EnableAsync
public class SpringSecurityApplication {

    public static void main(String[] args) {
        SpringApplication.run(SpringSecurityApplication.class, args);
    }
    @Bean
    public CommandLineRunner runner(RoleRepository roleRepository) {
        return args -> {
            if(roleRepository.findByName("USER").isEmpty())
                roleRepository.save(Role.builder().name("USER").build());
        };
    }
}
```
we've some work to discuss 

```java
@EnableJpaAuditing(auditorAwareRef = "applicationAuditAware")
@EntityListeners(AuditingEntityListener.class)
```
- `@EntityListeners(AuditingEntityListener.class)` ->This tells **JPA** Watch this entity's lifecycle events (before insert/before update) and auto-fill the audit fields. To achieve the auditing automatically , it's the one that triggered `createdDate` & `updatedDate`.
- `@EntityListeners(AuditingEntityListener.class)` -> its' simply answers the question **Who is the current user** and we achieve this through the configuration of `ApplicationAuditAware.java` which we 'll discuss in seconds

before diving to the `ApplicationAuditAware` let's discuss what does this code do

```java
@Bean
    public CommandLineRunner runner(RoleRepository roleRepository) {
        return args -> {
            if(roleRepository.findByName("USER").isEmpty())
                roleRepository.save(Role.builder().name("USER").build());
        };
    }
```

this one inserts by default the role `USER` to the client who uses or system , we persist it be default for the clients of our system.

Now let's see how we configure `ApplicationAuditAware.java` 

`ApplicationAuditAware.java`

```java
package com.mobin.booknetworkapi.config;

import org.springframework.data.domain.AuditorAware;
import org.springframework.security.authentication.AnonymousAuthenticationToken;
import org.springframework.security.core.Authentication;
import org.springframework.security.core.context.SecurityContextHolder;
import org.springframework.stereotype.Component;

import java.util.Optional;

@Component
public class ApplicationAuditAware implements AuditorAware<String> {

    @Override
    public Optional<String> getCurrentAuditor() {
        Authentication authentication = SecurityContextHolder.getContext().getAuthentication();

        if (authentication == null
                || !authentication.isAuthenticated()
                || authentication instanceof AnonymousAuthenticationToken) {
            return Optional.of("SYSTEM");
        }

        return Optional.of(authentication.getName());
    }
}
```

here we annotated the class as `@Component` to make spring see its work to tell who is the current logged-in user , we get the current loggedIn one
if there's no current loggedIn one we return `SYSTEM` if there's a one we send the `username` , and it always implements AuditorAware interface.

#### 3.2 Create our Repositories :

- Repository -> is simply interface always extends `JPAReopsitory` , to help us persist the data from The DB and do our `CRUD` operations & it's annotated with `@Repository`

let's discuss each repository simply 

`UserRepository.java`
```java
package com.mobin.booknetworkapi.user;

import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.stereotype.Repository;

import java.util.Optional;

@Repository
public interface UserRepository extends JpaRepository<User,Integer> {
    Optional<User> findByEmail(String email);
}
```
we've only one function helps use to get the user email from the Database.

`RoleRepository.java`

```java
package com.mobin.booknetworkapi.role;

import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.stereotype.Repository;

import java.util.Optional;

@Repository
public interface RoleRepository extends JpaRepository<Role,Integer> {
    Optional<Role>findByName(String role);
}
```
as an email it contains function to retrieve the user role from the db.

`TokenRepository.java`

```java
package com.mobin.booknetworkapi.user;

import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.stereotype.Repository;

import java.util.Optional;

@Repository
public interface TokenRepository extends JpaRepository<Token, Integer> {
    Optional <Token> findByToken(String token);
}
```
simply the repo got one function to help us get the token.

- something to say the repository has a lot of build-in functionalities help us to deal with the Db, we put this and `JPA` understand our purpose and add them to built-in functions.

#### 3.3 Configure our Security Logic:

Now, The Journey begins to be something executable, without wasting any time let's Configure our security.

`Visual Summary`

```plaintext
Incoming Request
      ↓
Is URL in WHITE_LIST? ── Yes → permitAll() → Controller
      ↓ No
JwtFilter runs (addFilterBefore)
      ↓
Extract & validate JWT → Set Authentication in SecurityContext
      ↓
authorizeHttpRequests checks: authenticated? ── No → 401/403
      ↓ Yes
Controller handles request
```


`ScurityConfig.java`
```java
package com.mobin.booknetworkapi.security;

import lombok.RequiredArgsConstructor;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.authentication.AuthenticationProvider;
import org.springframework.security.config.annotation.method.configuration.EnableMethodSecurity;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.config.annotation.web.configurers.AbstractHttpConfigurer;
import org.springframework.security.web.SecurityFilterChain;
import org.springframework.security.web.authentication.UsernamePasswordAuthenticationFilter;
import static org.springframework.security.config.Customizer.withDefaults;
import static org.springframework.security.config.http.SessionCreationPolicy.*;

@Configuration
@EnableWebSecurity
@RequiredArgsConstructor
@EnableMethodSecurity(securedEnabled = true)
public class SecurityConfig {
    private final JwtFilter jwtAuthFilter;
    private final AuthenticationProvider authenticationProvider;
    private static final String[] WHITE_LIST_URL = {
            "/auth/**",
            "/v3/api-docs",
            "/v3/api-docs/**",
            "/swagger-resources",
            "/swagger-resources/**",
            "/configuration/ui",
            "/configuration/security",
            "/swagger-ui/**",
            "/webjars/**",
            "/ws/**",
            "/swagger-ui.html"
    };
    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        http
                .cors(withDefaults())
                 .csrf(AbstractHttpConfigurer::disable)
                .authorizeHttpRequests(req->
                        req.requestMatchers(WHITE_LIST_URL).permitAll()
                                .anyRequest().authenticated()
                )
                .sessionManagement(session->session.sessionCreationPolicy(STATELESS))
                .authenticationProvider(authenticationProvider)
                .addFilterBefore(jwtAuthFilter, UsernamePasswordAuthenticationFilter.class)
                ;
        return http.build();
    }
}
```
This class which is annotated with `@Configuration`(an annotation tells spring to load this bean in the startup)
is the central configuration of  entire security system 
— it tells Spring Security exactly how to handle every incoming request: 
what's public, what needs a token, how sessions work, and which filter checks the JWT.

let's first discuss about some of our basic annotations here:

 - `@EnableWebSecurity` -> Activates Spring Security for the whole app, without it we can't prevent taking actions in our system without being secured.
 - `@EnableMethodSecurity(securedEnabled = true)` -> Allows us to secure individual methods, as for our authorities we 'll need to give specific controllers admin only access for example(`@PreAuthorize("hasRole('ADMIN')")`).

- let's clarify these fields in the class

```java
    private final JwtFilter jwtAuthFilter;
    private final AuthenticationProvider authenticationProvider;
```
We Inject these 2 (using `@RequiredArgsConstructor` form lombok) fields in our class for this purpose (we 'll discuss their implementation):
- `jwtAuthFilter` -> It's a filter of type(`JwtFilter`) that intercepts every request, extracts the JWT from the header, validates it, and manually authenticates the user before Spring's default login filter runs.
- `authenticationProvider` -> It's a component responsible for  verifying **credentials** (checks username/password against the database via UserDetailsService + PasswordEncoder), during mainly login (/auth/login).

- The story of the remaining snippets
```java
 private static final String[] WHITE_LIST_URL = {
            "/auth/**",
            "/v3/api-docs",
            "/v3/api-docs/**",
            "/swagger-resources",
            "/swagger-resources/**",
            "/configuration/ui",
            "/configuration/security",
            "/swagger-ui/**",
            "/webjars/**",
            "/ws/**",
            "/swagger-ui.html"
    };
    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        http
                .cors(withDefaults())
                 .csrf(AbstractHttpConfigurer::disable)
                .authorizeHttpRequests(req->
                        req.requestMatchers(WHITE_LIST_URL).permitAll()
                                .anyRequest().authenticated()
                )
                .sessionManagement(session->session.sessionCreationPolicy(STATELESS))
                .authenticationProvider(authenticationProvider)
                .addFilterBefore(jwtAuthFilter, UsernamePasswordAuthenticationFilter.class)
                ;
        return http.build();
```
- `White List` -> simply contains the end points which we allow to access without authentication, as for example the user needn't to be authenticated during `singnup/login`.
- `securityFitlerChain(HttpSecurity http)` -> A bean defines the actual security rules/pipeline applied to every HTTP request.

Let's discuss the remaining part of code 

```java
http
                .cors(withDefaults())
                 .csrf(AbstractHttpConfigurer::disable)
                .authorizeHttpRequests(req->
                        req.requestMatchers(WHITE_LIST_URL).permitAll()
                                .anyRequest().authenticated()
                )
                .sessionManagement(session->session.sessionCreationPolicy(STATELESS))
                .authenticationProvider(authenticationProvider)
                .addFilterBefore(jwtAuthFilter, UsernamePasswordAuthenticationFilter.class)
                ;
        return http.build();
```
- `.cors(withDefaults())` -> Here, we enable Enables CORS (Cross-Origin Resource Sharing) using default settings, it's simply for the purpose of front-end accessing our api.
- `.csrf(AbstractHttpConfigurer::disable)` -> AS `csrf attacks` done in session/cookie , we disable it as we work with JWT.
  - `.authorizeHttpRequests(req->
    req.requestMatchers(WHITE_LIST_URL).permitAll()
    .anyRequest().authenticated()
    )` -> `Core Access Role`,Simply this apply our `White-List` credentials open the access to any one matches it otherwise refuse it till it provides `JWT` token.
- `.sessionManagement(session -> session.sessionCreationPolicy(STATELESS))` -> Tells Spring Security not to create or use HTTP sessions , to remain our token `stateless`.
- `.addFilterBefore(jwtAuthFilter, UsernamePasswordAuthenticationFilter.class);` -> **The most important line** for JWT Inserts your jwtAuthFilter **before Spring's default username/password filter in the filter chain**.
- `return http.build();` -> Builds and returns the finalized security configuration as a bean.

Let's simplify our `JwtFilter.java`

`JwtFilter.java`

```java
package com.mobin.booknetworkapi.security;
import jakarta.servlet.FilterChain;
import jakarta.servlet.ServletException;
import jakarta.servlet.http.HttpServletRequest;
import jakarta.servlet.http.HttpServletResponse;
import lombok.NonNull;
import lombok.RequiredArgsConstructor;
import org.springframework.security.authentication.UsernamePasswordAuthenticationToken;
import org.springframework.security.core.context.SecurityContextHolder;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.security.core.userdetails.UserDetailsService;
import org.springframework.security.web.authentication.WebAuthenticationDetailsSource;
import org.springframework.stereotype.Service;
import org.springframework.web.filter.OncePerRequestFilter;
import java.io.IOException;
import static org.springframework.http.HttpHeaders.*;

@Service
@RequiredArgsConstructor
public class JwtFilter extends OncePerRequestFilter {
    private final JwtService jwtService;
    private final UserDetailsService userDetailsService;
    @Override
    protected void doFilterInternal(@NonNull HttpServletRequest request,
                                    @NonNull HttpServletResponse response,
                                    @NonNull FilterChain filterChain) throws ServletException, IOException {
        // in case we 're going good in the auth
        if(request.getServletPath().contains("/api/v1/auth")) {
            filterChain.doFilter(request, response);
            return;
        }
        final String authHeader = request.getHeader(AUTHORIZATION);
        final  String jwt;
        final String userEmail;
        if(authHeader==null || !authHeader.startsWith("Bearer ")) {
            filterChain.doFilter(request, response);
            return;
        }
        jwt = authHeader.substring(7);
        userEmail = jwtService.extractUsername(jwt);
        if(userEmail!=null && SecurityContextHolder.getContext().getAuthentication()==null){
            UserDetails userDetails = userDetailsService.loadUserByUsername(userEmail);
            if(jwtService.isTokenValid(jwt, userDetails)) {
                UsernamePasswordAuthenticationToken authToken = new UsernamePasswordAuthenticationToken(
                        userDetails, null, userDetails.getAuthorities());
                authToken.setDetails(new WebAuthenticationDetailsSource().buildDetails(request));
                SecurityContextHolder.getContext().setAuthentication(authToken);
            }
        }
        filterChain.doFilter(request, response);
    }
}
```
This is the `heart of JWT` authentication.It runs on every single request 
(except whitelisted ones) and is responsible for -> reading 
the token, validating it, and telling Spring Security 
"this user is authenticated" — manually, without a session. Let's see 
the flow simply, and discuss only the critical points.
`JwtFilter.java`
```plaintext
Request comes in
      ↓
Is it /api/v1/auth/**? ── Yes → skip filter → go straight through
      ↓ No
Has "Authorization: Bearer ..." header? ── No → pass through (will be rejected later if protected)
      ↓ Yes
Extract token → extract username
      ↓
Is user already authenticated in this request? ── Yes → skip
      ↓ No
Load UserDetails from DB (via UserDetailsService)
      ↓
Is token valid for this user? (signature + expiration) ── No → skip
      ↓ Yes
Build Authentication object → set it in SecurityContextHolder
      ↓
Continue filter chain → Controller (now "logged in" for this request)
```

Last part in our configurations is `BeanConfig.java`, it just wires up the core building blocks that the 
rest of your security system (SecurityConfig, JwtFilter, login logic) depends on. Think of it as 
the "factory" that produces the tools everyone else uses.

`BeanConfig.java`
```java
package com.mobin.booknetworkapi.config;

import lombok.RequiredArgsConstructor;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.authentication.AuthenticationManager;
import org.springframework.security.authentication.AuthenticationProvider;
import org.springframework.security.authentication.dao.DaoAuthenticationProvider;
import org.springframework.security.config.annotation.authentication.configuration.AuthenticationConfiguration;
import org.springframework.security.core.userdetails.UserDetailsService;
import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;
import org.springframework.security.crypto.password.PasswordEncoder;

@Configuration
@RequiredArgsConstructor
public class BeansConfig {
    private final UserDetailsService userDetailsService;
    @Bean
    public AuthenticationProvider authenticationProvider() {
        DaoAuthenticationProvider autProvider = new DaoAuthenticationProvider(userDetailsService);
        autProvider.setPasswordEncoder(passwordEncoder());
        return autProvider;
    }

    @Bean
    public AuthenticationManager authenticationManager(AuthenticationConfiguration config) {
        return config.getAuthenticationManager();
    }

    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }
}
```

`BeanConfig.java`
```plaintext
                    ┌─────────────────────┐
                    │  UserDetailsService  │  (loads user from DB)
                    └──────────┬───────────┘
                               │ used by
                               ▼
                    ┌─────────────────────┐
    PasswordEncoder │ AuthenticationProvider│ ← also uses PasswordEncoder
    (BCrypt)  ───────►  (DaoAuthenticationProvider)
                    └──────────┬───────────┘
                               │ used by
                               ▼
                    ┌─────────────────────┐
                    │ AuthenticationManager│  ← called manually at login
                    └──────────┬───────────┘
                               │
                               ▼
                    Login Service (/auth/login)
                    authenticationManager.authenticate(...)
```
this the flow simply , it says that
BeansConfig centralizes the core security beans used throughout 
the app: **PasswordEncoder** (BCrypt, for hashing/verifying passwords), 
**AuthenticationProvider** (connects UserDetailsService + PasswordEncoder to 
actually validate login credentials), and **AuthenticationManager** 
(the entry point your login service calls to trigger that validation). 
Keeping these as beans allow them to be injected wherever needed  
`SecurityConfig`, the login/auth service, and the registration service 
without duplicating logic.

#### 3.4 Create Services & DTOs :

##### 3.4.1 Services:

###### JwtService: 
This class is the JWT toolbox it's responsible for creating 
tokens (at login) and reading/validating them (in JwtFilter, on every request). 
It's the only class that actually touches the JWT library directly.

`JwtService.java`
```java
package com.mobin.booknetworkapi.security;

import io.jsonwebtoken.Claims;
import io.jsonwebtoken.Jwts;
import io.jsonwebtoken.SignatureAlgorithm;
import io.jsonwebtoken.io.Decoders;
import io.jsonwebtoken.security.Keys;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.security.core.GrantedAuthority;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.stereotype.Service;

import java.security.Key;
import java.util.ArrayList;
import java.util.Date;
import java.util.HashMap;
import java.util.Map;
import java.util.function.Function;

import static io.jsonwebtoken.SignatureAlgorithm.HS256;

@Service
public class JwtService {
    @Value("${application.security.jwt.expiration}")
    private long jwtExpiration;
    @Value("${application.security.jwt.secret-key}")
    private String secretKey;

    public String generateToken(UserDetails userDetails) {
        return generateToken(new HashMap<>(), userDetails);
    }
    public String extractUsername(String token) {
        return extractClaim(token, Claims::getSubject);
    }

    private <T> T extractClaim(String token, Function<Claims,T> claimsResolver) {
        final  Claims claims = extractAllClaims(token);
        return claimsResolver.apply(claims);
    }

    private Claims extractAllClaims(String token) {
        return Jwts
                .parser()
                .setSigningKey(getSignKey())
                .build()
                .parseClaimsJws(token)
                .getBody();
    }

    public   String generateToken(Map<String,Object> claims, UserDetails userDetails) {
        return buildToken(claims,userDetails,jwtExpiration);
    }

    private String buildToken(Map<String, Object> extraClaims,
                              UserDetails userDetails,
                              long jwtExpiration) {
        var authorities = userDetails.getAuthorities()
                .stream()
                .map(GrantedAuthority::getAuthority)
                .toList();
        return Jwts
                .builder()
                .setClaims(extraClaims)
                .setSubject(userDetails.getUsername())
                .setIssuedAt(new Date(System.currentTimeMillis()))
                .setExpiration(new Date(System.currentTimeMillis()+jwtExpiration))
                .claim("authorities",authorities)
                .signWith(getSignKey(), HS256)
                .compact();
    }

    public boolean isTokenValid(String token , UserDetails userDetails) {
        final String username = extractUsername(token);
        return (username.equals(userDetails.getUsername())) && !isTokenExpired(token);
    }

    private boolean isTokenExpired(String token) {
        return extractExpiration(token).before(new Date());
    }

    private Date extractExpiration(String token) {
        return extractClaim(token, Claims::getExpiration);
    }


    private Key getSignKey() {
        byte[] keyBytes = Decoders.BASE64.decode(secretKey);
        return Keys.hmacShaKeyFor(keyBytes);
    }
}
```

`JwtService.java`
```plaintext
═══════════════════════════════════════════════════════════════
                 PART 1: GENERATE TOKEN (at Login)
═══════════════════════════════════════════════════════════════

AuthService              JwtService                    Jwts Builder
    │                        │                               │
    │ generateToken(userDetails)                              │
    ├───────────────────────►│                               │
    │                        │                               │
    │                        │ userDetails.getAuthorities()  │
    │                        │ → convert to List<String>     │
    │                        │   ["ROLE_USER"]                │
    │                        │                               │
    │                        │ buildToken(claims, userDetails, expiration)
    │                        ├──────────────────────────────►│
    │                        │                               │
    │                        │        setSubject(username)   │
    │                        │        setIssuedAt(now)        │
    │                        │        setExpiration(now+X)    │
    │                        │        claim("authorities", roles)
    │                        │                               │
    │                        │        getSignKey()            │
    │                        │        (decode Base64 secret)  │
    │                        │        signWith(key, HS256)    │
    │                        │                               │
    │                        │◄──────────────────────────────┤
    │  ◄── JWT String ───────┤        .compact()              │
    │                        │                               │


═══════════════════════════════════════════════════════════════
               PART 2: VALIDATE TOKEN (every Request - inside JwtFilter)
═══════════════════════════════════════════════════════════════

JwtFilter                JwtService                  Jwts Parser
    │                        │                               │
    │ extractUsername(token) │                               │
    ├───────────────────────►│                               │
    │                        │ extractAllClaims(token)        │
    │                        ├──────────────────────────────►│
    │                        │                               │
    │                        │        setSigningKey(getSignKey())
    │                        │        parseClaimsJws(token)   │
    │                        │        ⚠️ if signature is wrong → Exception
    │                        │                               │
    │                        │◄──────────────────────────────┤
    │                        │ claims.getSubject()             │
    │◄───────────────────────┤ → "user@email.com"              │
    │                        │                               │
    │                        │                               │
    │ isTokenValid(token, userDetails)                        │
    ├───────────────────────►│                               │
    │                        │                               │
    │                        │ extractUsername(token)         │
    │                        │  == userDetails.getUsername()?  │
    │                        │           AND                  │
    │                        │ isTokenExpired(token)?          │
    │                        │  → extractExpiration(token)     │
    │                        │  → .before(new Date())          │
    │                        │                               │
    │◄───────────────────────┤                               │
    │  true / false          │                               │
```

Explanation of our Flow:

`JwtService` is the engine responsible for everything related to 
the token itself — creation, decoding, and validation. 
It works in two clear stages:
1. Generating the Token (at Login):
- Takes the `UserDetails` of the user who just authenticated successfully
- Builds a token containing: the username (as subject), issued date,
 expiration date, and the user's roles/authorities.
- Signs the token with a secret key using the `HS256` algorithm 
 so if anyone changes even a single character, 
the signature becomes invalid and the token gets rejected
2. Validating the Token (on every incoming Request, inside JwtFilter):
- `Decodes` the token and verifies the signature is intact 
(proving it was genuinely issued by the server).
- Extracts the username from it , using `extractUsername` function.
- Confirms that username matches the expected user and that 
 the token hasn't expired yet **isTokenValid**.
this was the story of our primary service in the logic we can say.


##### AuthenticationService: 

AuthenticationService handles the **three core steps** of 
the auth lifecycle: registration, login, and account activation.

let's now see the code and make a diagram, then explain it.

`AuthenticationService.java`
```java
package com.mobin.booknetworkapi.authentication;
import com.mobin.booknetworkapi.email.EmailService;
import com.mobin.booknetworkapi.role.RoleRepository;
import com.mobin.booknetworkapi.security.JwtService;
import com.mobin.booknetworkapi.user.Token;
import com.mobin.booknetworkapi.user.TokenRepository;
import com.mobin.booknetworkapi.user.User;
import com.mobin.booknetworkapi.user.UserRepository;
import jakarta.mail.MessagingException;
import jakarta.transaction.Transactional;
import lombok.RequiredArgsConstructor;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.security.authentication.AuthenticationManager;
import org.springframework.security.authentication.UsernamePasswordAuthenticationToken;
import org.springframework.security.core.userdetails.UsernameNotFoundException;
import org.springframework.security.crypto.password.PasswordEncoder;
import org.springframework.stereotype.Service;

import java.security.SecureRandom;
import java.time.LocalDateTime;
import java.util.HashMap;
import java.util.List;

import static com.mobin.booknetworkapi.email.EmailTemplateName.*;

@Service
@RequiredArgsConstructor
public class AuthenticationService {
    private final RoleRepository roleRepository;
    private final PasswordEncoder passwordEncoder;
    private final UserRepository userRepository;
    private final TokenRepository tokenRepository;
    private final EmailService emailService;
    private final AuthenticationManager authenticationManager;
    private final JwtService jwtService;
    @Value("${application.mailing.frontend.activation-url}")
    private String activationUrl;
    public void register(RegistrationRequest request) throws MessagingException {
        // get the user role
        var userRole = roleRepository.findByName("USER")
                .orElseThrow(() -> new IllegalStateException("User role not found"));
        // assign the user
        var user = User.builder()
                .firstName(request.getFirstName())
                .lastName(request.getLastName())
                .email(request.getEmail())
                .password(passwordEncoder.encode(request.getPassword()))
                .accountLocked(false)
                .enabled(false)
                .roles(List.of(userRole))
                .build();
        userRepository.save(user);
        sendValidationEmail(user);
    }

    private void sendValidationEmail(User user) throws MessagingException {
        var newToken = generateAndSaveActivationToken(user);
        emailService.sendEmail(
                user.getEmail(),
                user.getFullName(),
                ACTIVATE_ACCOUNT,
                activationUrl,
                newToken,
                "Account Activation"
        );
    }

    private String generateAndSaveActivationToken(User user) {
        String generatedToken = generateActivationToken(6);
        var token = Token.builder()
                .token(generatedToken)
                .createdAt(LocalDateTime.now())
                .expiresAt(LocalDateTime.now().plusMinutes(15))
                .user(user)
                .build();
        tokenRepository.save(token);
        return generatedToken;
    }

    private String generateActivationToken(int length) {
        String characters = "0123456789";
        StringBuilder codeBuilder = new StringBuilder();
        SecureRandom random = new SecureRandom();
        for(int i = 0; i < length; i++) {
            int randomIndex = random.nextInt(characters.length());
            codeBuilder.append(characters.charAt(randomIndex));
        }
        return codeBuilder.toString();
    }

    public AuthenticationResponse login(AuthenticationRequest request) {
        var auth = authenticationManager.authenticate(new UsernamePasswordAuthenticationToken(request.getEmail(), request.getPassword()));
        var claims = new HashMap<String,Object>();
        var user = ((User) auth.getPrincipal());
        var jwtToken = jwtService.generateToken(claims,user);
        return AuthenticationResponse.builder().token(jwtToken).build();
    }

    public void activateAccount(String token) throws MessagingException {
        Token savedToken = tokenRepository.findByToken(token).orElseThrow(() -> new RuntimeException("Invalid token"));
        // case the token had been expired
        if(LocalDateTime.now().isAfter(savedToken.getExpiresAt())) {
            sendValidationEmail(savedToken.getUser());
            throw new RuntimeException("Activation token has expired , a new token has been sent to the same email address");
        }
        var user = userRepository.findById(savedToken.getUser().getId()).orElseThrow(()->new UsernameNotFoundException("User not found"));
        user.setEnabled(true);
        userRepository.save(user);
        savedToken.setValidatedAt(LocalDateTime.now());
        tokenRepository.save(savedToken);
    }
}
```

`AuthenitcationService.java`
```plaintext
═══════════════════════════════════════════════════════════════
                 PART 1: REGISTER (Sign Up)
═══════════════════════════════════════════════════════════════

Client         AuthenticationService      RoleRepository/UserRepo    EmailService
  │                     │                          │                     │
  │ POST /auth/register │                          │                     │
  ├────────────────────►│                          │                     │
  │                     │ findByName("USER")       │                     │
  │                     ├─────────────────────────►│                     │
  │                     │◄─────────────────────────┤                     │
  │                     │                          │                     │
  │                     │ passwordEncoder.encode(password)                │
  │                     │ build User (enabled=false, accountLocked=false)│
  │                     │                          │                     │
  │                     │ userRepository.save(user)│                     │
  │                     ├─────────────────────────►│                     │
  │                     │                          │                     │
  │                     │ generateActivationToken(6)  → e.g. "482913"    │
  │                     │ save Token (expires in 15 min)                  │
  │                     ├─────────────────────────►│                     │
  │                     │                          │                     │
  │                     │ emailService.sendEmail(activationUrl + token)  │
  │                     ├──────────────────────────┼────────────────────►│
  │◄────────────────────┤                          │                     │
  │  ✅ registered, check email                     │                     │


═══════════════════════════════════════════════════════════════
                 PART 2: LOGIN
═══════════════════════════════════════════════════════════════

Client         AuthenticationService    AuthenticationManager      JwtService
  │                     │                        │                     │
  │ POST /auth/login    │                        │                     │
  │ {email, password}   │                        │                     │
  ├────────────────────►│                        │                     │
  │                     │ authenticationManager.authenticate(           │
  │                     │   UsernamePasswordAuthenticationToken)        │
  │                     ├───────────────────────►│                     │
  │                     │                        │ (delegates to        │
  │                     │                        │  AuthenticationProvider,
  │                     │                        │  checks DB + password)
  │                     │◄───────────────────────┤                     │
  │                     │ auth.getPrincipal()     │                     │
  │                     │ → cast to User          │                     │
  │                     │                        │                     │
  │                     │ jwtService.generateToken(claims, user)        │
  │                     ├─────────────────────────┼────────────────────►│
  │                     │◄─────────────────────────┼────────────────────┤
  │◄────────────────────┤  JWT token              │                     │
  │  { token: "..." }   │                        │                     │


═══════════════════════════════════════════════════════════════
                 PART 3: ACTIVATE ACCOUNT
═══════════════════════════════════════════════════════════════

Client         AuthenticationService      TokenRepository       UserRepository
  │                     │                        │                     │
  │ GET /auth/activate?token=482913               │                     │
  ├────────────────────►│                        │                     │
  │                     │ findByToken(token)      │                     │
  │                     ├───────────────────────►│                     │
  │                     │◄───────────────────────┤                     │
  │                     │                        │                     │
  │                     │ is expired? (now > expiresAt)                │
  │                     ├── YES ──► resend validation email             │
  │                     │           throw "token expired"               │
  │                     │                        │                     │
  │                     ├── NO ───► continue                            │
  │                     │                        │                     │
  │                     │ findById(user.id)       │                     │
  │                     ├─────────────────────────┼───────────────────►│
  │                     │◄─────────────────────────┼───────────────────┤
  │                     │ user.setEnabled(true)   │                     │
  │                     │ userRepository.save(user)                     │
  │                     │                        │                     │
  │                     │ savedToken.setValidatedAt(now)                │
  │                     │ tokenRepository.save(savedToken)              │
  │                     ├───────────────────────►│                     │
  │◄────────────────────┤                        │                     │
  │  ✅ account activated │                        │                     │

```
- Let's now discuss our 3 steps:

1. Register:
- Fetches the default "USER" role from the Db.
- Builds a new User with the password hashed via 
 `PasswordEncoder` (never stored in plain text as we discussed)
,and creates the account as `enabled = false` by default(not usable yet).
- Saves the user, then generates a random 6-digit activation code
with  a 15-minute expiration, and `emails` it to the user.

2. Login:
- Delegates the actual credential check to `AuthenticationManager`.
 authenticate(...), which internally uses the 
AuthenticationProvider (from BeansConfig) to verify the email/password against the Db.
- If successful, the Authentication object 
holds the authenticated User as its `principal`.
- That user is passed to JwtService.generateToken(...) 
to produce a signed `JWT`, which is returned to the 
 client — this is the token the 
client will attach to **every future request**.

3. Activate Account:
- Looks up the activation token sent to the `user's email`.
- If the token is expired, a new activation email is sent and the request is rejected.
- If valid, the user's account is flipped to 
`enabled = true` (they can now log in), 
and the token is marked as validated.

##### EmailService: 
`EmailService.java`  main job is to send an activation token to user's email in `SignUp` process.

let's see the code and discuss it's main functionalities.

`EmailService.java`

```java
package com.mobin.booknetworkapi.email;
import jakarta.mail.MessagingException;
import jakarta.mail.internet.MimeMessage;
import lombok.RequiredArgsConstructor;
import org.springframework.mail.javamail.JavaMailSender;
import org.springframework.mail.javamail.MimeMessageHelper;
import org.springframework.scheduling.annotation.Async;
import org.springframework.stereotype.Service;
import org.thymeleaf.context.Context;
import org.thymeleaf.spring6.SpringTemplateEngine;
import java.util.HashMap;
import java.util.Map;
import static java.nio.charset.StandardCharsets.*;
import static org.springframework.mail.javamail.MimeMessageHelper.MULTIPART_MODE_MIXED;

@Service
@RequiredArgsConstructor
public class EmailService {
    private final JavaMailSender mailSender; // spring interface for sending mails via SMTP
    private final SpringTemplateEngine templateEngine;// thymeleaf engine takes html tmp+ data to produce html page

    @Async
    public void sendEmail(String to , String username,EmailTemplateName emailTemplate,String confirmationUrl,String activationCode,String subject) throws MessagingException {
        String templateName;
        if(emailTemplate==null){
            templateName = "emailTemplate";
        }else{
            templateName = emailTemplate.getName();
        }
        MimeMessage mimeMessage = mailSender.createMimeMessage();
        MimeMessageHelper helper = new MimeMessageHelper(mimeMessage,
                MULTIPART_MODE_MIXED,
                UTF_8.name()
        );
        Map<String, Object> properties = new HashMap<>();
        properties.put("username", username);
        properties.put("confirmationUrl", confirmationUrl);
        properties.put("activationCode", activationCode);

        Context context = new Context();
        context.setVariables(properties);
        helper.setFrom("contact@mobin.com");
        helper.setTo(to);
        helper.setSubject(subject);
        String template = templateEngine.process(templateName, context);
        helper.setText(template, true);
        mailSender.send(mimeMessage);
    }
}
```
- `@Async` -> it's an annotation runs on a separate thread, so AuthenticationService doesn't wait for the email to send before responding to the client.


```java
   templateName = emailTemplate.getName(); // -> "activate_account"
```
it picks the template name , which in our case name is `activate_account.html`


```java
   MimeMessage mimeMessage = mailSender.createMimeMessage();
   MimeMessageHelper helper = new MimeMessageHelper(mimeMessage, MULTIPART_MODE_MIXED, UTF_8.name());
```
MimeMessageHelper lets you build a proper **HTML email** (not just plain text).
---
```java
   properties.put("username", username);
   properties.put("confirmationUrl", confirmationUrl);
   properties.put("activationCode", activationCode);
   context.setVariables(properties);
```
here we simply **inject** this data inside the our `activate_account.html` template.
---
```java
   String template = templateEngine.process(templateName, context);
```
Here we render final `activate_account.html` page.

---
```java
   helper.setFrom("anyEmail"); helper.setTo(to); helper.setSubject(subject);
   helper.setText(template, true); 
   mailSender.send(mimeMessage);
```
Set email metadata & send , via `SMTP`.

--- 
And this our ,our `activate_account.html` which only renders our data.


---
##### 3.4.2 DTOs:
`DTO` is simply a way to sent secure response and get secure responses , to not pass only the data we need to send 
and get only the data required to get in the response as there's a sensitive data we tend to not send in the response for example `passowrd`, so in the service you see `AutenticationRequest` for example it's the request with only the fields we wanna the client send.

- let's look how these `DTOs` looks:
`AuthenticationRequest.java`

```java
package com.mobin.booknetworkapi.authentication;

import jakarta.validation.constraints.Email;
import jakarta.validation.constraints.NotBlank;
import jakarta.validation.constraints.Size;
import lombok.*;

@Getter
@Setter
@Builder
@AllArgsConstructor
@NoArgsConstructor
public class AuthenticationRequest {
    @Email(message = "Email is not formatted")
    @NotBlank(message = "email is mandatory")
    private String email;
    @NotBlank(message = "Password is mandatory")
    @Size(min = 8, message = "Password should be at least 8 character long")
    private String password;
}
```

`RegisterationRequest.java`
```java
package com.mobin.booknetworkapi.authentication;

import jakarta.validation.constraints.Email;
import jakarta.validation.constraints.NotBlank;
import jakarta.validation.constraints.Size;
import lombok.*;

@Getter
@Setter
@Builder
@AllArgsConstructor
@NoArgsConstructor
public class RegistrationRequest {
  @NotBlank(message = "Firstname is mandatory")
  private String firstName;
  @NotBlank(message = "Lastname is mandatory")
  private String lastName;
  @Email(message = "Email is not formatted")
  @NotBlank(message = "email is mandatory")
  private String email;
  @NotBlank(message = "Password is mandatory")
  @Size(min = 8, message = "Password should be at least 8 character long")
  private String password;
}
```
`AuthenticationResponse.java`
```java
package com.mobin.booknetworkapi.authentication;

import lombok.Builder;
import lombok.Getter;
import lombok.Setter;

@Getter
@Setter
@Builder
public class AuthenticationResponse {
    private String token;
}
```
#### 3.4 Finally Establish our controllers :

Controllers are simply the **end-points** which client sends request through it ,then it send it to the server and give him the response
Let's now see our main controller which contains `Signup/Login`.

```java
package com.mobin.booknetworkapi.authentication;

import io.swagger.v3.oas.annotations.tags.Tag;
import jakarta.mail.MessagingException;
import jakarta.validation.Valid;
import lombok.RequiredArgsConstructor;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;

@RestController
@RequestMapping("auth")
@RequiredArgsConstructor
@Tag(name="Authentication")
public class AuthenticationController {
    private final AuthenticationService authenticationService;
    @PostMapping("/register")
    @ResponseStatus(HttpStatus.ACCEPTED)
    public ResponseEntity<?> register(@RequestBody @Valid RegistrationRequest request) throws MessagingException {
        authenticationService.register(request);
        return  ResponseEntity.accepted().build();
    }

    @PostMapping("/login")
    public ResponseEntity<AuthenticationResponse>login(@RequestBody @Valid AuthenticationRequest request){
            return ResponseEntity.ok(authenticationService.login(request));
    }

    @GetMapping("activate-account")
    public void confirm(@RequestParam String token) throws MessagingException {
        authenticationService.activateAccount(token);
    }
}
```

