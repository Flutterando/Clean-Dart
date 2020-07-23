# Clean Dart
Proposta de Arquitetura Limpa para o Dart/Flutter


# Início

Podemos dizer que uma arquitetura limpa pode definir o futuro do seu projeto, sabendo disso, devemos tê-la como objeto de estudo constante para que assim saibamos onde, quando e como aplica-la. 

Para essa proposta nos basearemos nas camadas da Arquitetura Limpa proposta por Robert C. Martin no livro **“Arquitetura Limpa: o Guia do Artesão Para Estrutura e Design de Software”**.


# Camadas de uma Arquitetura Limpa.

**Robert C. Martin** conclui que uma arquitetura deve conter pelo menos 4 camadas principais e independentes para ser considerada “limpa”, são elas: 
1. Regras de Negócio Corporativas
2. Regras de Negócio da Aplicação
3. Adaptadores de Interface
4. Frameworks & Drivers (Externos)


## Regras de Negócio Corporativas

São as regras de negócio cruciais para a sua aplicação, são representadas por modelos de dados denominado "Entidades", essa camada tem as regras mais sensíveis de um sistema, por isso ela está no topo das camadas. Uma "Entidade" deve ser pura, ou seja, não deve conhecer nenhuma outra camada, porém é conhecida pelas outras camadas.


## Regras de Negócio da Aplicação

São as regras que só o computador pode executar, aqui temos uma representação de comandos chamados de "Casos de Uso", e basicamente representam as ações que um usuário pode fazer na aplicação. 

Um "Caso de Uso" conhece apenas as Entidades, porém não sabe nada sobre as implementações das camadas de mais baixo nível. 

Se um "Caso de Uso" precisar acessar uma camada superior, deverá fazê-lo por meio de contratos definidos por uma interface, seguindo o “Princípio de Inversão de Dependências” do [SOLID](https://www.youtube.com/watch?v=mkx0CdWiPRA).


## Adaptadores de Interface

Essa camada é responsável por “dar suporte” para as camadas mais altas (Regras de Negócios) convertendo os dados externos em um formato que cumpra os contratos de interface definido pelas Regras de Negócios.


## Frameworks & Drivers

Todas as abstrações feitas pelas camadas mais altas foram para que você facilmente plugue artefatos externos como uma BASE DE DADOS ou uma INTERFACE DE USUÁRIO.

Essa camada normalmente sofre muitas modificações, porém com uma arquitetura limpa aplicada isso pode ser completamente indolor. 

Você poderá trocar uma API REST por outra em GraphQl sem afetar suas regras de negócio. Você poderá trocar sua interface gráfica completamente ou até mesmo trocar do Flutter para o AngularDart que as Regras de Negócio ainda assim ficarão funcionais!

Dado as descrições iremos apresentar a proposta de Arquitetura Limpa da Flutterando, a “Clean Dart”.


# Clean Dart

![Image 1](imgs/img1.png)

Usando o Flutter como exemplo teremos então quatro camadas mantendo a “Arquitetura de Plugin”, com foco principal no Domínio da Aplicação, camada esta que hospeda as 2 Regras de Negócio principais, Entidades e Casos de Uso.

![Image 1](imgs/img2.png)

A proposta de Arquitetura se propõe a desacoplar as camadas mais externas e preservar a Regra de Negócio.


## Presenter

A Camada de Presenter fica responsável por declarar as entradas, saídas e interações da aplicação. 

Usando o Flutter como exemplo, hospedaremos os Widgets, Pages e também Alguma Gerência de Estado, já no backend como exemplo, seria nesta camada onde colocaríamos os Handlers ou Commands da nossa API.


## Domain

A camada de Domain hospedará as Regras de Negócio Corporativa(Entity) e da Aplicação(Usecase).

Nossas Entidades devem ser Objetos simples podendo conter regras de validação dos seus dados por meio de funções ou ValueObjects. A Entidade não pode usar nenhum Objeto das outras camadas.

Os Casos de Uso devem executar a lógica necessária para resolver o problema. Se o Caso de Uso precisar de algum acesso externo deve ser feito por meio de contratos de interface.

Na camada Domain a única coisa que dever ser responsável apenas pela execução da lógica de negócio, não deve haver implementações de outros Objetos como Repositories ou Services dentro do Domain. 

Já em Repository como exemplo, teremos que ter apenas o contrato de interfaces e a responsabilidade de implementação desse objeto deverá ser repassado a outra camada mais baixa.


## Infrastructure(Infra)

Está camada dá suporte a camada Domain implementando suas interfaces. Para isso, essa camada se propõem a adaptar os dados externos para que possa cumprir os contratos do domínio.

Muito provavelmente nessa camada iremos implementar alguma interface de Repository ou Services que pode ou não depender de dados externos como uma API ou acesso a algum Hardware como por exemplo Bluetooth. 

Para que o Repository possa processar e adaptar os dados externos devemos criar contratos para esses serviços visando passar a responsabilidade de implementação para a camada mais baixa da nossa arquitetura.

Como sugestão, iremos criar objetos de DataSource quando quisermos acessar um dado externo, uma BaaS como Firebase ou um Cache Local usando SQLite por exemplo.
Outra sugestão seria criar objetos denominados Drivers para interfacear a comunicação com algum Hardware do dispositivo.

Os acessos externos como Datasources e Drivers devem ser implementados por outra camada, ficando apenas os Contratos de Interface nesta camada de Infra.


## External

Aqui começaremos a implementar os acessos externos e que dependem de um serviço, package ou acesso muito específico.

Basicamente a camada External deve conter tudo aquilo que terá grandes chances de ser alterado sem que você possa intervir.

No Flutter por exemplo, para cache local usamos o SharedPreferences, mas talvez em alguma estágio do projeto a implementação do SharedPreferences não seja mais suficiente para a aplicação e deve ser substituída por outro package como Hive, nesse ponto a única coisa que precisamos fazer é criar uma nova classe, implementando o Contrato esperado pela camada mais alta (que seria a Infra) e implementarmos a Lógica usando o Hive.

Um outro exemplo prático seria pensar em um Login com Firebase Auth, porém outro produto deseja utilizar um outro provider de autenticação. Bastaria apenas implementar um datasource baseado no outro provider e “Inverter a Dependência” substituindo a implementação do Firebase pela nova quando for necessário.

Os Datasources devem se preocupar apenas em “descobrir” os dados externos e enviar para a camada de Infra para serem tratados.

Da mesma forma os objetos Drivers devem apenas retornar as informações solicitadas sobre o Hardware do Device e não devem fazer tratamento fora ao que lhe foi solicitado no contrato.

# Exemplos

- [Clean Dart Login with Firebase](https://github.com/jacobaraujo7/login-firebase-clean-dart)

# Links úteis

- [Resumo do livro "Arquitetura Limpa"](https://medium.com/@deividchari/desvendando-a-arquitetura-limpa-de-uncle-bob-3e60d9aa9cce)
- [Sua Arquitetura está limpa?](https://medium.com/flutterando/sua-arquitetura-est%C3%A1-limpa-clean-architecture-no-flutter-458c68fad120)
- [Os tijolos do Clean Architecture](https://www.youtube.com/watch?v=C8mpy3pwqQc)
- [The Clean Code Blog](https://blog.cleancoder.com/uncle-bob/2012/08/13/the-clean-architecture.html)
- [Guilherme's Proposal](https://github.com/guilherme-v/flutter-clean-arch)
- [Flutter TDD Clean Architecture Course](https://www.youtube.com/watch?v=KjE2IDphA_U&list=PLB6lc7nQ1n4iYGE_khpXRdJkJEp9WOech)