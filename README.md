# Rocket
Rocket is a programming language proposal designed to allow programmers to define their application business logic and state management **once** for both the front-end and the back-end while abstracting away persistence details (e.g. database schema and queries). It is a statically typed, (mostly) functional, event-driven language with syntax and semantics similar to imperative languages.

## Philosophy
- Applications written in Rocket should focus exclusively on business logic and not worry about the normal plumbing that comes with developing a modern application stack.
- Business logic and entity definitions should be written once, but accessible by the persistence layer, the application layer, and client applications.
- Developer productivity and usability are paramount, and code should be concise, readable, and idiomatic.
- Rocket should be easy to learn and get started with.
- Language syntax, design, and features should be focused on the "80%" use case.
- The language, tooling, and plugins should handle common functionality that is external to the business logic with sensible defaults, but allow developers to configure, tune and dive in if necessary (e.g. SQL queries and schema, serialization, external API calls, API communication between front-end and back-end, data caching, user authentication, transport security and encryption).
- Opinionated, concise paradigms are preferred to overly general and verbose methodologies.
- Boilerplate code should be minimized, but poorly documented and opaque "magic" code leads to misunderstanding and bugs.
- Project code structure should be flexible, but defaults and scaffolding should be highly opinionated while allowing extraction of sub-projects.
- The development environment and tooling should be easy to set up, and getting started on a new project should be very fast to allow for quick prototyping.
- Rocket applications should be decoupled from the view layer, but should be capable of simple interoperability with the UI code with minimal overhead and boilerplate.
- Performance optimization should primarily be handled by the compiler, but profile guided optimization and suggestions should be possible.
- Functionality and logic should be easily decoupled, decomposed, and tested.
- The vast majority of modern web applications are event driven and the language should reflect that.
- The language implementation should abstract away low-level concepts like parallelism, concurrency, and memory management.
- Developers should be able to write programs without worrying about where exactly the code is executed but achieve the same level of security possible in a standard stack.
- Configuration and context boundaries should be decoupled from the rest of the application.
- The compiler and tooling should prevent you from shooting yourself in the foot and provide helpful messages and suggestions.
- Validation logic and invariants should be partially checkable at compile time.
- Values should all be immutable and never null.
- Application state should be managed centrally and separately from other code; however state is not necessarily global - context specific state should be possible
- Application state changes should be managed via transactional, unidirectional data flow and consumers of the state should subscribe to the state through selectors and be notified with changes (similar to Redux).
- Application state changes should be implemented using pure functions (similar to reducers in Redux)
- IO, network, and system exceptions should be handled separately from application exceptions.
- Exceptions must be handled, but should optionally propagate to externally / to the end-user with user-friendly error messages.
- The visibility of both values and functions should be configurable both across code unit boundaries and context boundaries to prevent the leakage of sensitive information, and by default should have the most restrictive visibility.
- Interfaces should represent the external API of data structures rather than concrete classes
- Interfaces should be composable and extendable, but classes should not as it leads to brittle subclass problems
- Delegation to class instance properties is preferred to subclassing as it is explicit, less error prone, and more performant than managing a class hierarchy

## Goals
- Code and logic re-use
  - Code and logic re-use between front-end, back-end, and persistence layers
  - Compiles to Web Assembly, LLVM, Java Bytecode
  - Maintains database schema and intelligently offloads some logic to DBMS functions
- Plumbing included
  - Provide serialization, state persistence & querying, wire protocols, and transport protocols (e.g. REST API) as plugable, configurable extensions with sensible defaults that are abstracted away from the developer
- Productivity
  - Allow programmers to express their application state and logic in a readable and understandable format
  - Easy to learn with familiar syntax and semantics
  - Provide a toolchain that provides helpful errors and warnings at compile time
  - Provide a simple, familiar, testing paradigm that allows for easy creation of test cases and fixtures
- Safety
  - Static typing allows compiler to catch type errors
  - Everything is immutable, so no interior mutation bugs
  - Visibility modifiers provide encapsulation
  - Null values are not allowed
  - All errors must be handled
  - Persistence API provides optimistic concurrency control, and ACID transactions
- Security
  - All functions are executed within a *context* with associated permissions and identities
  - Objects created in a context are private to that context by default and must be explicitly shared with other contexts
  - Optionally, objects passing context boundaries can be encrypted to protect private data and comply with privacy regulations (such as [GDPR](https://en.wikipedia.org/wiki/General_Data_Protection_Regulation))
  - Optionally, objects passing context boundaries can be signed to ensure authenticity
  - Simple, secure user / principal authentication architecture with plugable implementations (OAuth, JWT, etc.)
  - Declarative, extensible, composable authorization DSL
  - Object validation checks performed at compile-time (when possible) and runtime (when crossing context boundaries)
  - Persistence layer implements a Merkle DAG -> modifications signed by the transaction context allowing for auditablity
- Performance
  - Immutability of objects allows implementations to parallelize execution safely
  - Asynchronous by default, so IO does not block execution
  - Persistence layer's Merkle DAG allows identical objects to be re-used
  - Also allows for aggressive caching of data to limit wire requests
  - Persistent Data selectors allow for batching requests
  - Lazy evaluation and memoization prevent unnecessary recomputation
  - Computed properties reduce memory footprint of persistent objects
  - Object hashing and large object fingerprinting and chunking reduce duplicate transfer of data
  - Potentially also allows for P2P or distributed data fetching
  - Allows for high quality offline user experience similar to CouchDB
  - Compiles to native code
  - Garbage collection implemented via reference counting in the compiler (similar to Rust), so very low overhead
  - Allows code to be executed where it is most efficient / performant (DB server, web server, browser)
  - Copy-on-Write and persistent data structures make immutable objects performant
  - Compiler can create temporary immutable structures to handle multiple changes and batch update
  - Static analysis can provide aggressive dead-code elimination, and module splitting for faster load times in the browser
- Interoperability
  - Provide relatively low-overhead interoperability with libraries written in other languages
  - May require some "unsafe" glue code that allows mutability within a sandboxed context
  - Bindings generated through tooling similar to Rust's [bindgen](https://github.com/rust-lang-nursery/rust-bindgen)
  - May use [GraalVM](https://www.graalvm.org/) to provide polyglot execution environment
  - Allow creation of interface bindings to interact with custom microservices or external APIs in a separate context
- Extensibility & Composability
  - Interfaces and generic type parameters allow developers to write reusable code
  - Interfaces are implicitly implemented to reduce boilerplate (similar to Go)
  - Interfaces allow for default implementations
  - Allows programmers to implement operators for arbitrary types similar to Rust (https://doc.rust-lang.org/book/first-edition/operators-and-overloading.html)
  - Rocket's interfaces are extensible and composable
  - Interfaces allow the definition of both properties and methods
  - Properties may be computed or literal properties
  - Allows for decoupled code
  - Annotations provide type metadata for the compiler - potentially allowing for versioned interfaces
- Modularity
  - Uses module system very similar to Node / ES6 modules with similar namespacing semantics
  - Access modifiers allow for encapsulation
  - Abstracted transport protocols and interfaces allow modules to be deployed as microservices without the normal overhead
- Validation
  - Validation logic can be partially checked by the compiler
  - Implemented once
- Powerful, simple, easy, familiar tooling
  - Compiler provides useful warnings, errors and messages
  - Formatter / linter to enforce consistent style
  - Executable in JIT mode to allow for REPL / exploratory coding and debugging
  - External package dependencies managed similar to NPM
    - Allows for flexible package sources (git, file, http, or rolling your own NPM-like service)
    - Requires that native dependencies be included in the package
    - Provides platform independent builds and binary packages
  - Familiar, easy to use documentation format
  - Tooling should assist with refactoring

## Non Goals
- Rocket is not intended to be a systems programming language
- While it can certainly be used as a general purpose language, it is intended for a specific use case
- Performance parity with Go / C / C++ / Java
  - While it might get there eventually, developer productivity is the primary goal
- Direct UI interaction
  - Interaction with the UI layer should be handled by a client language
  - For example, DOM interaction in the browser should be exclusively handled by Javascript

## Motivation

Writing a modern web application stack is far harder than it should be. Let's consider a fairly typical React & Redux (Javascript) + Flask & Psychopg2 (Python) + Postgres toy application that helps track product inventory and sales across different stores. First you have to decide how you want to structure your entities. Here is one way of doing it that reflects a relational database paradigm

- Product
  - id
  - name
  - category
  - price
  - [has_many items]
- Store
  - id
  - name
  - address
  - [has_many items]
  - [has_many purchases]
- Purchase
  - id
  - customer_name
  - customer_email
  - [has_many items]
  - [belongs_to store]
- Item
  - id
  - [belongs_to product]
  - [belongs_to store]
  - [belongs_to purchase?]

These entities must be defined in three different places:
- Your DB schema
- Your model layer in your Flask app
- Your redux store in your React app

What's worse is that you have to define these entities in three different programming languages each with their own semantics and provide a way to serialize and communicate these entities between each layer. So now you're designing a database schema in SQL with various foreign keys and constraints, a database communication / persistence layer in python with inline SQL queries, a REST API in Flask that serializes and deserializes entities into JSON, validates API calls, and filters parameters, creating an API client in Javascript that serializes and deserializes Javascript objects into the right shape for the API (and possibly needs to convert between snake case and camel case), implementing actions and reducers in Redux, and writing form validation logic for React.

What a headache!

You get:

- Huge amounts of code / logic replication
- Little to no type safety
- Need to test the entity code, the transport code, the validation code, the serialization code on both the front-end and the back-end
- Need to run integration tests to ensure interoperability between the various components

And we haven't even included exception handling.

What happens when:

- There's a database constraint error?
- There's an error communicating to the database?
- There's a validation error in the flask app?
- There's an authorization error in the flask app?
- There's an error communicating to the API?
- There's a form validation error in the front end?

All of these need to be handled in a sensible way, perform retries as necessary, and provide user feedback. What's more is that the developer needs to anticipate all of these errors and account for them manually and propagate them down the stack, creating yet another problem of transport, serialization and deserialization, HTTP codes, retries, etc.

And what happens if there's a bug in any layer that accidentally mutates a property of any of the entities along the way?

The true business logic of the application you are creating could be expressed in a few paragraphs of english, but you end up writing more than a thousand lines of repetitive code in 3 programming languages to achieve this. Most of the code you end up writing is glue code rather than your logic. While frameworks exist to cut down on boilerplate, every developer that writes a web application ends up writing strikingly similar code - often for each action or endpoint. This "plumbing" code ends up being repetitive, highly brittle, error prone, difficult to test, and really boring to write. While there are many excellent frameworks in various programming languages that seek to ease this pain, none (as far as I know) provide a very good solution to this problem, and each has its own drawbacks:

- Java EE / JPA / JSP
  - Highly verbose, single page application requires custom logic, so much XML
- Ruby on Rails
  - Cuts down on the SQL you have to write at the expense of entity definitions separated between ActiveRecord classes and database schema
  - Relatively concise
  - No type safety
  - You still need to implement models and serialization logic in javascript
- GraphQL / NodeJS
  - Flexible and has nice integrations with databases, allows some code re-use, but you still end up writing a lot of code, and type safety is still a challenge (even with TypeScript)
- Go
  - Good type safety, but SQL usually still written by hand
  - Similar code verbosity / replication to other back-end frameworks
  - You still need to implement models and serialization logic in javascript
- (Most) Functional Programming Languages
  - Good type safety, but SQL usually still written by hand
  - Unfamiliar syntax, and difficult to learn

## Why a new language?

Almost all of the languages used to implement application business logic in a web application were designed as general purpose programming languages with persistence, transport layers, and serialization as an add-on. Rocket is designed from the ground up for the express purpose of expressing this type of logic for a specific, but extremely common use case. Furthermore, the integration of front-end code with back-end code historically has been very difficult and fraught with issues (GWT and Kotlin are some of the better attempts at this). However, the advent of WebAssembly opens up a huge opportunity to bridge this gap. Rocket can provide semantics and functionality for this specific use case without having to worry about the excentricities of various host languages.


## State of Development

Rocket is still in concept stage, so feedback is highly appreciated. You can create a GitHub issue or email me at rocketlangfeedback@gmail.com

My current thinking for the semantics of the language is to borrow much of the safety concepts from Rust, but because everything is immutable, the complexity should be lower. From a syntax point of view, I want the language to be as familiar as possible to a newcomer, so obviously, I'm going to use a C-like syntax. However, as I've written more Typescript and Go, I've come to believe that type declarations should come after the variable name (thus allowing type inference to appear similar to explicitly typed code). I really like the concept of Rust's traits; however, I find the implementation to be very constraining (duplicate definitions, etc.). But the default implementation concept is fantastic. I think Go has jumped through a few hoops (and forced its users to as well) with respect to generics, its syntax is some of the most readable and beautiful out there, so I intend to draw a lot of inspiration from Go's syntax. While I appreciate the return to simplicity of the use of structs in Go and Rust, I think that defining methods within the type declaration is far more readable and understandable and can achieve the same semantics (with all classes being final and no interior state modification allowed).

From an implementation standpoint, I plan to use a form of [persistent data structures](https://en.wikipedia.org/wiki/Persistent_data_structure) to achieve performant immutability, and reference counting for memory management. Furthermore, persistent types should implement some kind of Merkle tree creation to achieve the type of persistence that I specified in the philosophy. Therefore, circular references are problematic. As I work through the semantics and design of the language, the first implementation is likely going to be an interpreter written in TypeScript. My rationale for this is that the prototype will still be runnable in the browser and on the server side, but I'll have a degree of type safety in terms of the design of the interpreter.

My plan is to write the language grammar / lexer in a format that will allow me to move away from Javascript, but not truly constrain me. So, my initial thinking is to write an EBNF grammar and use this node package to generate the lexer / parser (https://www.npmjs.com/package/ebnf-parser)


## Why the name Rocket?

Rocket is named after my childhood dog who passed recently:

![Alt text](rocket.jpg)
