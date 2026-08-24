# Spring boot Security With JWT

![Spring Security Img](springSecurity.png)
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
### What's Simply Security is ?
The Standard Security always Combined by **Authentication & Authorization**
which always 've basic Question we need to answer **Who & What**
- Who -> Authentication context answer , We answer it with **The Identity** which simply (Username , email /Password) on other way the Login form we always face in any app
- What -> Authorization context or we can say the credentials which we allow our user to 've permissions
  , access of the user , it's the role of user in our simply we mostly simplified it in the context
  of (USER/ADMIN) each system is rounded in user/users roles and admin/admin roles each one has its
  credential which allowed to it , Admin always has a big hand in the system administration internally by the way.
---
### JWT In Spring boot Backend Development:

In spring boot there's now standard way in the framework itself support us with basic CRUD Operations
for example and standard config , as Spring boot doesn't 've even a default implementation for the
JWT Implementation which make it mainly made by us , which lead it maybe harming for us in the implementation
and also a lot of us we feel harsh in implementing it specially first time and it's normal , but
let's go and simplify it.

---
### What's Really a Token is:

![jwtConstructionImg1](jwtConstruct2.png)
![jwtConstructionImg2](jwtConstruct1.png)
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

##### Important Notes about Token:
- Token is stateless means that we don't save it in the server , so each request is treated independently not saved it's state
- We never ever put Password in the token payload , never ever as it destroy the security in case of any attack , we do every thing to make the password not accessible encrypt it even it's not saved with its value , so don't go and give it in the payload for the attackers.
- Go try check your token here if you want to see it yourself [jwt](https://www.jwt.io/).
---
### Spring Boot JWT Life Cycle:

Let's Discuss this Flow Step by step
![JwtFlow](jwtImg.png)


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




