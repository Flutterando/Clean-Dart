# Clean Dart
Dart/Flutter向けのクリーンアーキテクチャ提案

# Introduction

クリーンアーキテクチャは、プロジェクトの未来を定義すると言えます。そのため、私たちの役割は、それをどこに、いつ、どのように適用するかを知るために、常に勉強することです。

この提案は、Robert C. Martinの **“Clean Architecture: A Craftsman's Guide to Software Structure and Design”** の原則と、そのレイヤー構造アプローチに基づいています。

# Clean Architecture layers

**Robert C. Martin**は、アーキテクチャが「クリーン」であるためには、少なくとも4つの主要かつ独立したレイヤーを持っている必要があると述べています。それらは次のとおりです。

1. Enterprise Business Rules : 企業の就業規則
2. Application Business Rules : コンピュータ上のプログラムの就業規則
3. Interface Adapters : 異なるレイヤーをつなげる適合器
4. Frameworks & Drivers (External) : プログラムの枠組みとOSとDevice間のソフトウェア（外部）

![Image 3](imgs/img3.png)


## Enterprise Business Rules : 企業の就業規則

これは、システムの最も感度が高く、最も高いレベルのレイヤーです。これらは、**エンティティ**と呼ばれるデータモデルによって表されます。

エンティティは、純粋でなければなりません。これは、それが下のレイヤーについての知識を持っていないことを意味します。一方、他のすべてのレイヤーは、**エンティティ**について知っています。

## Application Business Rules : コンピュータ上のプログラムの就業規則

これらは、排他的で特定のアプリケーションに固有のルールであり、ターゲットデバイスでのみ実行できます。これらは、**ユースケース** と呼ばれるコマンドで表され、大まかに言えば、ユーザーがアプリケーション内で実行できるアクションを表します。

**ユースケース** は、**エンティティ** レイヤーのみを知っており、下位レイヤーについては何も知りません。

もし **ユースケース** が上位レイヤーにアクセスする必要がある場合、それは [**依存性逆転の原則**](https://ja.wikipedia.org/wiki/%E4%BE%9D%E5%AD%98%E6%80%A7%E9%80%86%E8%BB%A2%E3%81%AE%E5%8E%9F%E5%89%87) を考慮して行われるべきです。

## Interface Adapters : 異なるレイヤーをつなげる適合器

このレイヤーは、上位レイヤーと外部データの間の橋渡しとしての役割を果たします。これは、ビジネスルールで定義されたインターフェース契約を尊重する方法で、外部データが上位レイヤーと通信するのを助けます。

## Frameworks & Drivers (External) : プログラムの枠組みとOSとDevice間のソフトウェア（外部）

全ての上位レイヤーの抽象化は、それらと外部アーティファクトとの間の結合を改善するために特に設計されました。これにより、プラグ＆プレイのスタイルでいつでもそれらを切り替えることが容易になります。

このレイヤーは非常に揮発性が高く、常に変化しています。しかし、クリーンアーキテクチャの中では、これらの変更は完全に痛みなく安全に行うことができ、ビジネスルールは変更されません。

Rest APIからGraphQL APIに、UIから別のUIに、FlutterからAngularDartに切り替えることができます。ビジネスルールは以前と同じように機能し続け、それらを変更する必要はありません。

そうは言っても、具体例がないと理解が難しいかもしれません。そこで、Flutterando Clean Architecture、つまり **Clean Dart** の提案をご紹介します。

# Clean Dart

![Image 1](imgs/img1.png)

Flutterを例にとって、4つのレイヤーがあり、メインのフォーカスはアプリケーションドメインにあります。このレイヤーには、2つの主要なビジネスルール、**エンティティ** と **ユースケース** が存在します。

![Image 1](imgs/img2.png)

このアーキテクチャは、外部レイヤーを分離し、ビジネスルールを保持することを提案しています。

## Presenter

**プレゼンター** レイヤーは、アプリケーションのI/Oと相互作用を宣言する責任があります。

もし、Flutterを例にとると、このレイヤーには、ウィジェット、ページ、およびステート管理が含まれます。一方、バックエンドを扱う場合、このレイヤーはAPIのハンドラーやコマンドが存在する場所になります。

## Domain

**ドメイン** レイヤーは、**エンティティ** と **ユースケース** を含む **ビジネスルール** を持っています。

**エンティティ** は、データの検証ルールや関数、または `ValueObjects` を通じてデータの検証ルールを持つかもしれませんが、シンプルなオブジェクトである必要があります。**エンティティは、他のレイヤーのオブジェクトに依存してはいけません。**

**ユースケース** は、特定の問題を解決するために必要なロジックを実行しなければなりません。もし **ユースケース** が外部アクセスを必要とする場合、このアクセスは、下位レイヤーによって実装されるインターフェースコンタクトを介して行われるかもしれません。

**ドメイン** は、ビジネスルールの実行のみに責任を持たなければなりません。リポジトリやサービスのような他のオブジェクトの実装を持ってはいけません。

責任を持つ例として、リポジトリを取ると、このリポジトリにはインターフェース契約のみがあります。この契約の実装は、下位レイヤーによって行われなければなりません。

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
- [Clean Dart Pokedex with BLoC and Modular](https://github.com/umpedetiago/pokedex_with_cleandart.git) 

# Useful links

- [Resumo do livro "Arquitetura Limpa"](https://medium.com/@deividchari/desvendando-a-arquitetura-limpa-de-uncle-bob-3e60d9aa9cce)
- [Sua Arquitetura está limpa?](https://medium.com/flutterando/sua-arquitetura-est%C3%A1-limpa-clean-architecture-no-flutter-458c68fad120)
- [Os tijolos do Clean Architecture](https://www.youtube.com/watch?v=C8mpy3pwqQc)
- [The Clean Code Blog](https://blog.cleancoder.com/uncle-bob/2012/08/13/the-clean-architecture.html)
- [Guilherme's Proposal](https://github.com/guilherme-v/flutter-clean-arch)
