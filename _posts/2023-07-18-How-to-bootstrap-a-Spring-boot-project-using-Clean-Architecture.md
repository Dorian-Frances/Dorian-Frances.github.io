---
layout: post
title: "How to bootstrap a Spring boot project using Clean Architecture"
date: 2023-07-18
tags: [bootstrap, architecture]
reading-time: 10
description: In this article, we explore the power of Hexagonal Architecture in combination with Spring Boot and Java. Discover how this architectural pattern enhances testability, separates business code from technical implementation, and promotes code modularity and scalability. While applying Hexagonal Architecture may initially seem verbose, its benefits offer a comprehensive and clean structure for your projects.
comments: true
---

# Introduction

Hello everyone ! Today we will talk about Spring and Java. More specifically, I’ll talk about how to bootstrap a spring boot project using Hexagonal
Architecture. Spring boot is a very straightforward framework, but creating a very clean bootstrap using clean architecture can be disruptive.

Hexagonal Architecture bring to the fore many concepts like :

- Testability
- Separation of business code and technical implementation
- Code modularity / scalability

Many of you could think that using hexagonal architecture to start a little project can be a little bit wordy and you are not completely wrong but in
the other hand, it also provides a comprehensive structure of your project.

> 👉 Also, I would not lie about the fact that **I do not have enough knowledge about all the different kind of architecture**. Here, I simply suggest
> a way of bootstrapping a project using hexagonal architecture. Feel free not to use it if you don’t want to. 😉

# Github repository

You can find the entire bootstrap project by clicking 📂 [this link](https://github.com/dorian-frances/df-java-template). **Feel free to use it as you
want !**

# Initialize a project

The first thing we need to do is to create our spring-boot project (I used **maven** here as my package manager).

Using IntelliJ IDEA, you might come up with something like this:

```text
.
├── pom.xml
└── src
    ├── main
    │   ├── java
    │   │   └── org
    │   │       └── example
    │   │           └── Main.java
    │   └── resources
    └── test
        └── java
```

# Target project

The application we want to create here is pretty basic, we will build a **RestAPI** with **one endpoint** that will allow us to **retrieve a user** in
a **PostgreSQL database** given its ID.

Going further, we pretty much see the elements that we will need:

- A **RestAPI** implementing the route to retrieve our *user*
- Some **business code logic**
- A local **PostgreSQL** database
- An **ORM**, simplifying database requests

# Clean Architecture

For now, we see that our code structure does not fit hexagonal architecture. To solve this, we want our code to contain those elements:

- An **application** folder: this folder will contain all the technical implementation that will consume our application (in our case, the RestAPI
  that we will build).
- An **infrastructure** folder: this folder will contain all the technical implementation that will be used by our application (in our case, the
  PostgreSQL implementation).
- A **domain** folder: this is the most important folder. This is where all your business code has to be. All the code that contains **added value**
  has to be in here.
- A **bootstrap** folder: this folder contains all the code that allow us to start our application, this is where we initialize it.

> 👉 **IMPORTANT**: *domain-driven-development* state that our domain needs to be independent from all technical implementation. The fact is, technical
> implementation **may change** over time, your **business code usually do not**. However, our domain still needs to communicate with the other parts
> of
> our application, that is why, we will create *application-domain* and *domain-infrastructure* interfaces. by doing this, we ensure better modularity
> and better testability of our application (we’ll see this later in this article).

Now, our application should look like this:

```text
.
├── application
│   ├── pom.xml
│   └── src
│       ├── main
│       │   ├── java
│       │   │   └── org.example
│       │   └── resources
│       └── test
│           └── java
├── bootstrap
│   ├── pom.xml
│   └── src
│       ├── main
│       │   ├── java
│       │   │   └── org
│       │   │       └── example
│       │   │           └── Main.java
│       │   └── resources
│       └── test
│           └── java
├── domain
│   ├── pom.xml
│   └── src
│       ├── main
│       │   ├── java
│       │   │   └── org
│       │   │       └── example
│       │   └── resources
│       └── test
│           └── java
├── infrastructure
│   ├── pom.xml
│   └── src
│       ├── main
│       │   ├── java
│       │   │   └── org
│       │   │       └── example
│       │   └── resources
│       └── test
│           └── java
└── pom.xml
```

We will now go through all the different folders to understand how we implemented each of them.

# Deep diving in our project structure

## Application

After that we implemented our rest-api-adapter, the folder has this structur:

```text
application
├── pom.xml
├── rest-api-adapter
    ├── pom.xml
    ├── src
    ├── main
    │   ├── java
    │   │   └── io.df.java.template.application.rest.api.adapter
    │   │                                   ├── controller
    │   │                                   │   └── UserController.java
    │   │                                   └── mapper
    │   │                                       ├── DfExceptionMapper.java
    │   │                                       └── UserMapper.java
    │   └── resources
    └── test
         └── java
```

The two main things that we’ll talk about here are our `UserController` and the `UserMapper` .

```java
@RestController
@Tags(@Tag(name = "User"))
@AllArgsConstructor
public class UserController implements UserApi {

    final UserFacade userFacade;

    @Override
    public ResponseEntity<GetUserDTO> getUser(UUID id) {
        try {
            final User user = userFacade.getUser(id);
            return ResponseEntity.ok(getUserFromDomainToResponse(user));
        } catch (DfException exception) {
            return mapDfExceptionToContract(() -> errorToGetUserDTO(exception), exception);
        }

    }
}
```

This is the implementation of our user endpoint. We can see that this controller **implements** the `UserApi`. This, is an interface generated by
the [openapi-generator](https://openapi-generator.tech/docs/plugins) used in our _contract-first_ methodology.

> 👉 If you look at the **README.md** file in the github repository, you will get an explanation about using _contract-first_ methodology. Basically,
> it allows us to define (in a contract) the different endpoints of our API that will be generated during **build** so that we only need to implement
> them after.

Anyway, we notice different thins in this controller, the use of a `UserFacade` that we pass in the constructor of our controller (notice
the `@AllArgsConstructor` decorator here). As I explained [here](#clean-architecture), we pass an interface to our controller that our **domain** will
then implement to increase modularity / testability of our code and independence of our *domain*.

The second thing is the use of `getUserFromDomainToResponse` method from the `UserMapper` class. We really want to make a distinction between **domain
models** and **API entity**. That is the purpose of the mapper here.

> 👉 We’ll talk about the `DfExceptionMapper` class in the [domain section](#domain).

## Domain

Our domain looks like this:

```text
domain
├── pom.xml
├── src
     ├── main
     │   ├── java
     │   │   └── io.df.java.template.domain
     │   │                       ├── exception
     │   │                       │   ├── DfException.java
     │   │                       │   └── DfExceptionCode.java
     │   │                       ├── model
     │   │                       │   └── User.java
     │   │                       ├── port
     │   │                       │   ├── in
     │   │                       │   │   └── UserFacade.java
     │   │                       │   └── out
     │   │                       │       └── UserStoragePort.java
     │   │                       └── service
     │   │                           └── UserService.java
     │   └── resources
     └── test
         └── java
           └── io.df.java.template.domain
                                 └── service
                                     └── UserServiceTest.java
```

First, our `UserService` :

```java
@AllArgsConstructor
public class UserService implements UserFacade {

    UserStoragePort userStoragePort;

    @Override
    public User getUser(UUID userId) throws DfException {
        final Optional<User> optionalUser = userStoragePort.getUser(userId);
        if (optionalUser.isEmpty()) {
            throw DfException.builder()
                    .dfExceptionCode(DfExceptionCode.USER_NOT_FOUND_EXCEPTION)
                    .errorMessage(String.format("Failed to find user with id %s", userId))
                    .build();
        }
        return optionalUser.get();
    }
}
```

This is the service that implements the interface called in the `UserController` . The same way, this service calls another interface to fetch the
user that we want in the database.

All those interfaces are called `port` and you can see that they are defined in our domain wearing the designation `port.in` and `port.out`.

In the domain, we can also find our `UserModel` . This is the object that will be manipulated in our domain. In there we can not find entities from
our API or our database. This would break hexagonal architecture principles.

Finally, we also see an `exception` folder. In this project, we defined our own exceptions allowing us to customize the exceptions that we wanna
throw. For example, in our `UserService` if we threw an `HttpNotFoundException` , this would mean that our domain is aware that we are using an HTTP
protocol as a technical implementation. Again, this breaks *hexagonal architecture* and *domain-driven-development* principles. To avoid that, we
throw customized exceptions that we’ll map INSIDE of our `rest-api-adapter` (this is the purpose of the `DfExceptionMapper` class in our `application`
folder).

This way, our domain only knows that he has to throw an exception without knowing how those are handled in the technical implementations.

## Infrastructure

```text
infrastructure
├── pom.xml
├── postgres-adapter
     ├── pom.xml
     ├── src
          ├── main
          │   ├── java
          │   │   └── io.df.java.template.infrastructure
          │   │                       └── postgres
          │   │                           ├── adapter
          │   │                           │   └── PostgresUserAdapter.java
          │   │                           ├── configuration
          │   │                           │   └── PostgresConfiguration.java
          │   │                           ├── entity
          │   │                           │   └── UserEntity.java
          │   │                           ├── mapper
          │   │                           │   └── UserEntityMapper.java
          │   │                           └── repository
          │   │                               └── UserRepository.java
          │   └── resources
          │       └── db.changelog
          │           ├── changelogs
          │           │    └── 00000000_init_user_schema.sql
          │           └── db.changelog-master.yaml
          └── test
             └── java
```

As mentioned earlier, this is where all technical implementations **used by** our application should be. Thus, in our case, we can only find
our `postgres-adapter` designed to handle relation with our database.

In this project, we used [Hibernate](https://hibernate.org/) for our ORM along with [Liquibase](https://www.liquibase.org/) to handle database
migration (I will not deep dive into the use of liquibase, but that explains the presence of the `db.changelog` directory in this adapter).

Again we can find a `mapper` directory to handle mapping between *postgres entities* and *domain models*.

Our `PostgresUserAdapter` class looks like that:

```java
@AllArgsConstructor
public class PostgresUserAdapter implements UserStoragePort {

    private final UserRepository userRepository;

    @Override
    public Optional<User> getUser(UUID userId) {
        return userRepository.findById(userId).map(UserEntityMapper::fromEntityToDomain);
    }
}
```

And the configuration of the entire module is:

```java
@Configuration
@EnableAutoConfiguration
@EnableTransactionManagement
@EnableJpaAuditing
@EntityScan(basePackages = {"io.df.java.template.infrastructure.postgres.entity"})
@EnableJpaRepositories(basePackages = {"io.df.java.template.infrastructure.postgres.repository"})
public class PostgresConfiguration {

    @Bean
    public PostgresUserAdapter postgresUserAdapter(final UserRepository userRepository) {
        return new PostgresUserAdapter(userRepository);
    }
}
```

We define here the packages where Spring will be able to find our `Entities` and `JpaRepositories` .

## Bootstrap

Finally, the bootstrap folder. As we passed interfaces as dependencies along all our application, we now have to tell our application which classed
should implement this or this interface. This is done during the runtime of our application through this `bootstrap` folder.

```text
bootstrap
├── pom.xml
├── src
    ├── main
    │   ├── java
    │   │   └── io.df.java.template.bootstrap
    │   │                       ├── DfJavaTemplateApplication.java
    │   │                       └── configuration
    │   │                           ├── DfJavaTemplateConfiguration.java
    │   │                           ├── DomainConfiguration.java
    │   │                           └── RestApiConfiguration.java
    │   └── resources
    │       └── application.yaml
    └── test
        ├── java
        |   └── io.df.java.template.bootstrap
        │                       ├── DfJavaTemplateITApplication.java
        │                       └── it
        │                           ├── AbstractDfJavaTemplateBackForFrontendApiIT.java
        │                           └── UserApiIT.java
        └── resources
            └── application-it.yaml
```

We can see in the structure of this folder different configuration files. Indeed, we will find one configuration file for our domain folder and one
for each adapters of our application (you can find the configuration folder of our `postgres-adapter` in the adapter itself).

Our `DfJavaTemplateApplication` class will then, looks like this:

```java
@SpringBootApplication
@EnableConfigurationProperties
@Import(value = {DfJavaTemplateConfiguration.class, DomainConfiguration.class, PostgresConfiguration.class, RestApiConfiguration.class})
@Slf4j
@EnableAsync
@AllArgsConstructor
public class DfJavaTemplateApplication {
    public static void main(String[] args) {
        SpringApplication.run(DfJavaTemplateApplication.class, args);
    }
}
```

> 👉 Do not forget to import all of your application configurations in the startup class so that Spring can find them and initialize each of the
> defined beans.

For example, the dependency injection for our `RestApiConfiguration` is:

```java
@Configuration
public class RestApiConfiguration {

    @Bean
    public UserController userController(final UserService userService) {
        return new UserController(userService);
    }
}
```

# Testing strategy

Obviously, we can not forget to test our application. That is why, in this project, you will find to types of tests.

## Unit Tests

An example of unit test is present in our domain folder. This type of tests are used to validate that our application behave as we want it to.

> 👉 For these tests, we do not want to reach technical implementations. Thus, we’ll have to *mock* every call to technical implementations.
> Fortunately, thanks to the use of interfaces, it is really easy to do.

For example:

```java
@Test
    void should_return_user_given_a_user_id() throws DfException {
        // Given
        final UUID userIdFake = UUID.randomUUID();
        final String usernameFake = faker.ancient().god();

        // When
        when(userStoragePortMock.getUser(userIdFake))
                .thenReturn(Optional.of(User.builder().id(userIdFake).username(usernameFake).build()));
        final User resultUser = userService.getUser(userIdFake);

        // Then
        assertThat(resultUser.getId()).isEqualTo(userIdFake);
        assertThat(resultUser.getUsername()).isEqualTo(usernameFake);
    }
```

## Integration tests

Those tests are different. **They also have more value** as they test the behaviour of our application **WITH** the use of technical implementations.
Consequently, **they are more difficult to write**. Indeed, we need to fake the startup of our application in order to enable us to fake some calls on
our API.

You’ll find an example of integration test file in the bootstrap folder.

# Continuous Integration (CI)

We also integrated a `.circleci` folder along with a `config.yml` file in this project.

```yaml
version: 2.1
machine: true

jobs:
  build-and-test:
    machine:
      image: ubuntu-2004:202107-02
      docker_layer_caching: true
    steps:
      - checkout
      - run:
          name: Install OpenJDK 17
          command: |
            sudo apt-get update && sudo apt-get install openjdk-17-jdk
            sudo update-alternatives --set java /usr/lib/jvm/java-17-openjdk-amd64/bin/java
            sudo update-alternatives --set javac /usr/lib/jvm/java-17-openjdk-amd64/bin/javac
      - run:
          name: Generate cumulative pom.xml checksum
          command: |
            find . -type f -name "pom.xml" -exec sh -c "sha256sum {} >> ~/pom-checksum.tmp" \;
            sort -o ~/pom-checksum ~/pom-checksum.tmp
      - restore_cache:
          keys:
            - df-java-template-multi-module-mvn-{{ checksum "~/pom-checksum" }}
      - run:
          name: Build
          command: ./mvnw install -T 12 -DskipTests -DskipITs
      - save_cache:
          paths:
            - ~/.m2
          key: df-java-template-multi-module-mvn-{{ checksum "~/pom-checksum" }}
      - run:
          name: Unit Tests
          command: ./mvnw test -T 12
      - run:
          name: Integration Tests
          command: ./mvnw integration-test -T 12

workflows:
  build-test-deploy:
    jobs:
      - build-and-test
```

The important things to see here are:

- The use of a real machine on CircleCI. As we used [testcontainers](https://java.testcontainers.org/) in order to make our integration tests, we have
  to use a real machine for our CI because we obviously can not launch a container image (our integration test container) inside of another image (the
  CI image).
- The workflow: basically, only one job is present in this configuration file and it does the following steps:
    - Checkout the project on Github
    - Install Java OpenJDK 17
    - Restore cache
    - Build the project
    - Save cache
    - Run the unit tests
    - Run the integration tests

# Conclusion

Though this article, I explained how I used *hexagonal architecture* and *domain-driven-development* to create my spring-boot project since I started
developing. Of course, I only have few years of experience, thus, I do not have much knowledge about other code architectures. However, I find this
architecture really educational and made me understand many key concepts of application design.

Do not hesitate to give me your feedback about this and if you want to share your knowledge about everything, you are well welcome 😉.