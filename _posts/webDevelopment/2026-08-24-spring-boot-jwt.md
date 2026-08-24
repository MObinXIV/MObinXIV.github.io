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






