---
layout: post
title: "Microservices with Vapor"
date: 2019-02-23
categories: swift vapor
tags: vapor server-side swift microservices
image:
    path: /assets/images/microservices-with-vapor.png
---

Microservices have recently got a lot of attention and Vapor provides us with all the needed tools and APIs to create a Microservices architecture relatively easy.

Vapor has several packages providing support for the following:

* HTTP
* Redis
* SQL, MySQL, and PostgreSQL
* WebSocket
* Auth
* JWT

## What is Microservices Architecture?

A definition of Microservices from [Wikipedia](https://en.wikipedia.org/wiki/Microservices):

*Microservices are a software development technique—a variant of the service-oriented architecture (SOA) architectural style that structures an application as a collection of loosely coupled services.*

## But why?

The topic around monolith vs microservice architecture has already been debated in the software design and architecture community and is beyond the scope of this post. 

Microservices offers several benefits some of which includes:

* Clear separation of concerns
* Each service can be deployed independently
* Promotes continuous integration and delivery
* Easier to test (when implemented correctly)
* Works great with Docker and Kubernetes
* Scalable
* Enables Agile development

## API Gateway

As illustrated below, the API Gateway is the external interface and entry point to communicate with all the internal services, similar to the [Facade](https://en.wikipedia.org/wiki/Facade_pattern) pattern in object-orientated design:

![image](/assets/images/microservices-api-gateway.png)

It is the API Gateway's responsibility to check and route each request to the corresponding internal services. The API Gateway might have other responsibilities such as authentication, access control, monitoring, logging, caching, load balancing, request shaping, etc.

## Implementing a Basic API Gateway in Vapor

The Gateway Controller will check each request and route it to the corresponding service:

```swift
func boot(router: Router) throws {
    router.post("users", use: handle)
}
```

A quick and simple solution is to check the URL of the request for a prefix, in this case checking whether the request contains the `/users` prefix:

```swift
func handle(_ req: Request) throws -> Future<Response> {
    if req.http.urlString.hasPrefix("/users") {
        guard let usersHost = Environment.get("USERS_HOST") else { throw Abort(.badRequest) }
        return try handle(req, host: usersHost)
    }
    throw Abort(.badRequest)
}
```

The request is then handled by creating a new HTTP client pointing to correct `host` and endpoint:

```swift
func handle(_ req: Request, host: String) throws -> Future<Response> {
    let client = try req.make(Client.self)
    guard let url = URL(string: host + req.http.urlString) else {
        throw Abort(.internalServerError)
    }
    req.http.url = url
    req.http.headers.replaceOrAdd(name: "host", value: host)
    return client.send(req)
}
```

## Implementing a Microservice in Vapor

Each service in the Microservices architecture would expose their REST API that's consumed by other services or by the application's clients. Some services might also implement a Web UI. At runtime, each service is often containerized using Docker and have its database, configurations, etc.

The following diagram illustrates how each of the microservices communicates with each other:

![image](/assets/images/microservices-service-communication.png)

Each functional area of the application is implemented by its microservice exposing a REST API consumed by other services. For example, the user service invokes the order service to retrieve order information for the given user.

Services might also use asynchronous, message-based communication for inter-service communication using for example the [AMQP Protocol](https://www.amqp.org).

So, let's have a look at how a very basic user microservice can be implemented in Vapor.

First, we need to configure the routes for the user service:

```swift
public func routes(_ router: Router) throws {
    let userController = UserController()
    try router.register(collection: userController)
}
```

The user service has two endpoints, one to register a new user and one to retrieve an existing user:

```swift
struct UserController: RouteCollection {
    func boot(router: Router) throws {
        let usersRouter = router.grouped("users")
        
        usersRouter.post(CreateUserRequest.self, at: "", use: register)
        usersRouter.post(GetUserRequest.self, at: "login", use: getUser)
    }
}
```

Below is an example of how you can register a new user using PostgreSQL:

```swift
private extension UserController {
    func register(_ req: Request, createRequest: CreateUserRequest) throws -> Future<HTTPStatus> {
        let userRepository = try req.make(UserRepository.self)
        return try userRepository.findCount(username: createRequest.username, email: createRequest.email, on: req).flatMap { count in
            guard count == 0 else {
                throw Abort(.badRequest, reason: "A user with these credentials already exists.")
            }
            try createRequest.validate()
            let bcrypt = try req.make(BCryptDigest.self)
            let hashedPassword = try bcrypt.hash(createRequest.password)
            let user = User(username: createRequest.username, email: createRequest.email, password: hashedPassword)
            return try userRepository.save(user: user, on: req).transform(to: HTTPStatus.created)
        }
    }
}
```

As with every other architecture out there, there is no silver bullet and microservices are no exception. The Microservices architectures have many drawbacks ranging from added deployment complexity to testing which should be taken into consideration during the planning phases of your project. With that said it is still a great choice for complex, evolving applications despite drawbacks and implementation challenges.

Here are links to my GitHub repositories containing the example code:

* [vapor-microservices-api-gateway](https://github.com/rynaardb/vapor-microservices-api-gateway)
* [vapor-microservices-user-service](https://github.com/rynaardb/vapor-microservices-user-service)
