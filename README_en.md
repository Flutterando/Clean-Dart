# Clean Dart
Clean architecture proposal for Dart/Flutter.

# Introduction

We can say that a clean architecture might define the future of your project. Knowing that, it's our role to study constantly in order to know where, when and how to apply it.

This proposal is based on Robert C. Martin's **“Clean Architecture: A Craftsman's Guide to Software Structure and Design”** principles and it's layers structure approach.

# Clean Architecture layers

**Robert C. Martin** states that, to be considered "clean", an architecture must have at least 4 main and independent layers. They are:

1. Enterprise Business Rules
2. Application Business Rules
3. Interface Adapters
4. Frameworks & Drivers (External)

![Image 3](imgs/img3.png)


## Enterprise Business Rules

They are the most sensitive rules of a system and, thus, the highest-level layer. They are represented by data models called **entities**. 

An **entity** must be pure. This means that it cannot have any knowledge about the layers below it. On the other hand, all other layers know about the **entities**.

## Application Business Rules

They are the rules that are exclusive and specific to your application and can only be executed by the target device. They are expressed in commands called **use cases**, which, roughly speaking, represents any action the user can perform within your application.

An **use case** only knows the **entities** layer, and know nothing about the lower layers.

If an **use case** needs to access a higher layer, it should be done with the [**Dependency Inversion Principle**](https://en.wikipedia.org/wiki/Dependency_inversion_principle) in mind.

## Interface Adapters

This layer is responsible for acting as a bridge between the higher layers and the external data. It helps the external data to communicate with the higher layers in a way that respects the interface contracts defined in the business rules.


## Frameworks & Drivers

All the higher-level layers abstractions were designed specially top improve the decoupling between them and the external artifacts. This makes easier to switch them whenever you want, in a plug & play fashion.

The frameworks & drivers layer is highly volatile, and is constantly changing. Within a clean architecture, however, these changes may be completely painless and safe, leaving your business rules untouched.

We can, then, switch from our Rest API to a GraphQL one, from our UI to another one, or even from Flutter itself to AngularDart. The business rules will keep working as before, as you won't need to change them not even a little.

That said, let's present our proposal for a Flutterando Clean Architecture, namely, **Clean Dart**.


# Clean Dart

![Image 1](imgs/img1.png)

By using Flutter as an example, we have four layers, keeping the "plugin architecture", with the main focus on the Application Domain. In this layer inhabits the two main business rules, the **entities** and the **usecases**.

![Image 1](imgs/img2.png)

This architecture proposes to dissociate the external layers and preserve the business rules.

## Presenter

The **Presenter** layer is responsible to declare the I/O and the interactions of the application.

If we take Flutter as an example, this layer would contain the Widgets, Pages and the State Management. On the other hand, if we were dealing with the backend, this layer would be where we would have the Handlers or Commands of our API.

## Domain

The **Domain** layer will contain our **core business rules** (entity) and **application-specific business rules** (usecases).

Our **entities** must be simple objects, that may or not have validation rules for its data through functions or ValueObjects. **The entity must not depend on any object of the other layers.**

The **usecases** must run the necessary logic to solve a specific problem. If the **usecase** needs the any external access, this access may be done through interface contacts that will be implemented by the lower-level layers.

The **Domain** must be responsible only for the execution of the business rules. It must not have any other object implementations, like repositories or services.

Taking a repository as example, we will have only the interface contract to this repository. The implementation of this contract must be done by a lower-level layer.

## Infrastructure (Infra)

This layer supports the **Domain** layer by implementing its interfaces. To do this, it have to adapt the external data so that it fullfill the domain contracts.

This layer will, probably, have the implementation for some repository or service interface that can't depend on external data, like an API, or the access to some hardware, like a Bluetooth device.

For the repository to be able to process and adapt the external data we must create contracts for these services, aiming to defer the implementation responsibility to a lower-level layer in our architecture.

Our suggestion is to create **DataSource** object when we want to access external data, that is, for example, a BaaS like Firebase or a SQLite-based local cache. Another suggestion is to create **Driver** objects to interface the communication between your application and some device hardware.

The external accesses like data sources and drivers must be implemented by another layer, leaving only the interface contracts in this layer.

## External

Here we implement the external accesses that depends on a hardware, package or highly-specific access.

Basically, the **External** layer must contain everything that is expected to be highly volatile and constantly changed.

In Flutter, for instance, we use `shared_preferences` for local cache. However, it may be that, in a later stage of the project, `shared_preferences` won't be able to meet the requirements of our application and we will want to replace it with another package, like `hive`. When this happens, all we need to do is to implement, using the logic inherent to `hive`, a new instance of the contract that the infrastructure layer expects.

Another pragmatic example would be to think in a login system based on Firebase Auth. Another product, however, want to use other authentication provider. To make this substitution it would be as simple as implementing a data source based on this new provider and "invert the dependency", using this implementation instead of the Firebase one's when need.

The **data sources** must only worry about discovering the external data and sending it to the infra layer, where they will be dealt.

Likewise, the **drivers** objects must only provide the device hardware info that is required by the contract, and not deal with anything else.

# Tips

## Think on layers

When you are about to start developing, start thinking about layers. We shouldn't worry with what the **Presenter** or **External** layers have, for example. If we start thinking by the external layers we may be eventually misguided by them. Thus, we should get used to develop each layer, from the most internal to the most external.

It may be that, in the beginning of your "clean" journey, some of these layers may seem useless. This happens when our thinking is not yet based **on layers** (or, maybe, because your business rule is too much simple for this).

## Unit Testing is your new UI

It's very common to developers to create your apps views before anything, so they help them to test their business rules. However, we already have a more proper tool for this, and a place specially designed for this kind of test.

Developing in a "clean" way is completely related with **TDD** (Test-Driven Development), as the **Presenter** layer is going to be one of the latest that we are going to think and develop.

## Spent your time dealing with possible errors

**It's better to let a Exception be thrown than to handle it in a generic way...**
A good tip is to use some handling-enforcing approach, like the `Either` class from `dartz` library.

The `Either` class may receive two distinct data, a `Left` one, representing an error, and a `Right` one, representing the actual expected result. This reduces a lot the need to manually handle the exceptions with **try-catch**, which is error-prone, in higher layers.

## Don't fall in the temptation of bypassing a layer

Sometimes you may have a very simple **usecase**, that will simply pass the data to the **repository**, like, for example, a CRUD where all you need to do is to validate if the data is being correctly received and yield it to the **repository** to do its work.

It may seem weird to have a class that have a single method which only function is to validate the data and send it to another class, but you are going to see that this will become quite useful when you are maintaining your project. It's not uncommon that your **usecase** borns small like this, but in the near future it grows bigger and more complex.

An example of this case is when you are using Firebase. The Firebase package only returns a Stream, and you could, as well, simply put it directly in your **view**. However, if someday you need to remove Firebase from your project and replace it with an alternative, you will have to remake your entire view, or even your entire project.

That said, don't fall in the temptation of calling the **repository** directly from your **controller**, or to use Firebase directly in your **view**. You will be breaking your architecture laws and will eventually regret of your decision.

# Sign up!

We would like to have your feedback!

If you like your "Clean Dart" architecture proposal, leave a **star** in this repository. This is the same as signing a "clean manifest" agreeing with our proposal!

We are open to suggestions and improvements on this documentation!
If you want to contribute, open an [issue](https://github.com/Flutterando/Clean-Dart/issues). Our team will be very happy with your interest in improving this tool for the community. Also, feel free to open a **pull request** with fixes to this proposal documentation.

# Examples

- [Clean Dart Login with Firebase, MobX and Modular](https://github.com/jacobaraujo7/login-firebase-clean-dart)
- [Clean Dart Github Search with BLoC and Modular](https://github.com/Flutterando/clean-dart-search-bloc)
- [Clean Dart Github Search with MobX and Modular](https://github.com/jacobaraujo7/clean-dart-search-mobx)
- [Clean Dart Github Search with Flutter Command](https://github.com/aquilarafa/clean_dart_flutter_command)
- [Chat WebSocket with Get_It and Cubit](https://github.com/rodrigorahman/flutter_curso_chat_websocket) 

# Useful links

- [Resumo do livro "Arquitetura Limpa"](https://medium.com/@deividchari/desvendando-a-arquitetura-limpa-de-uncle-bob-3e60d9aa9cce)
- [Sua Arquitetura está limpa?](https://medium.com/flutterando/sua-arquitetura-est%C3%A1-limpa-clean-architecture-no-flutter-458c68fad120)
- [Os tijolos do Clean Architecture](https://www.youtube.com/watch?v=C8mpy3pwqQc)
- [The Clean Code Blog](https://blog.cleancoder.com/uncle-bob/2012/08/13/the-clean-architecture.html)
- [Guilherme's Proposal](https://github.com/guilherme-v/flutter-clean-arch)
