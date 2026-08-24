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
#### 3.1 Create our entities & Repositories :
In the security context the first entity we always implement first is the `User.Java` entity with our basic user data we all know as email , password and the name we can work with any data else according to the use case also
, keep in mind *always* when we go to secure our app in spring boot our `Uaser.java` must implements `UserDetails` interface and als `Pricipal` interface

let's simply clarify there roles 

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
if there's no current loggedIn one we return `SYSTEM` if there a one we send the `username` , and it always implements AuditorAware interface.


