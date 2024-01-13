# Go Code Standards and Application Structure

**Last Updated:** 01-31-2023


**Author:** [Lin Trieu](https://github.com/LinTrieu)

## Abstract


I prepared this document as an internal RFC document for my previous engineering department. The RFC has been anonymized such that no company-specific information remains. The format of this document adheres to standard RFC style conventions with `MUST` and `SHOULD` imperatives. For further information on this, there is an RFC on what is an RFC, found (unironically) [here](https://www.rfc-editor.org/rfc/rfc7322).  

The intention of this document is to serve as a set of guiding principles for writing Go code and building domain-driven scalable applications. 


## Motivation


The main objective of this document is to help developers follow the same set of common rules when writing Go code. It will specifically aim at standardizing Go services by introducing guidelines on application architecture and best practices to improve code quality and consistency.


This document will also help onboarding developers to understand how to contribute to Go repositories.


### Objectives:


1. Consistency and standardization in internal Go services
2. Easy to understand, navigate, and reason for any new developer coming to the application
3. Easy to change, loosely-coupled
4. Easy to test
5. Structure should reflect how the software works




## Design


## Go Code Standards
Go language constructs and widely adopted conventions for best practice


### Code Style


* We `MUST` follow the "Go coding style guide" and write clear, idiomatic Go code according to the [standard documentation](https://golang.org/doc/).
* We `MUST` automate code style checks and linting using [golangci-lint](https://github.com/golangci/golangci-lint)
* We `MUST` upper-case acronyms, such as `ServeHTTP`.


### Modules


* We `MUST` organize our Go code in packages.
* We `MUST` name packages in lowercase convention e.g. ‘storedpaymentmethod’.
* We `MUST` name packages in singular and avoid plural. e.g. ‘paymentmethod’, not ‘paymentmethods’.
* We `MUST` name packages using short and representative names and  `MUST NOT` use meaningless, generic names such as`util`, `misc` or `common`. Other engineers should be able to identify the purpose of the package from its name.
* We `SHOULD` name packages based on what it provides, not what it contains.
* We `SHOULD` avoid package name collisions by using unique naming to differ from other internal packages or standard libraries.


### Files


* We `SHOULD` name Go files in the convention `a_file_for_go.go`.


### Interfaces


* We `MUST` use interfaces to specify and define behavior.
* Interfaces `SHOULD` be kept as small as possible so only methods required are defined. The bigger the interface, the weaker the abstraction.
* We `MUST` name interface arguments within an interface definition
* We `SHOULD` name one-method interfaces with its method name ending in an -er suffix, e.g. sourceGetter
* We `SHOULD` define interfaces next to the code that needs them (Go interfaces do not need to be explicitly implemented).


### Functions / Methods


* Functions `MUST` be named using `MixedCaps`/`mixedCaps`, and `MUST NOT` be named using `names_with_underscores`.
* We `SHOULD` use the same method receiver name for every method on that type.
* We `SHOULD NOT` use return named parameters, unless there is a specific requirement to do so.


### Constants


* We `SHOULD` name constants using camelCase for private constants, and PascalCase for public constants.


### Variables


* We `SHOULD` use short, descriptive variable names, where it does not compromise on code readability.
   * `var i int` instead of `var index int`
* We `SHOULD NOT` use redundant long names within their context. A name's length should not exceed its information content.
* We `MUST` use a consistent variable declaration style throughout the application.
* When declaring and initializing a variable to its zero value, we `SHOULD` use the `var` keyword.
   * e.g. `var foo string`
* When declaring and initializing a variable to a non-zero value, we `SHOULD` use the short variable declaration operator `:=`. 
   * e.g `foo := "bar"`


### Unit testing


* We `SHOULD` have a unit test file for each Go file and unit tests `SHOULD` cover all exported functional methods.
* Test files `SHOULD` be in the `{packagename}_test` package in the directory of the files that they test.
* We `SHOULD` aim to only test functionality exported from a package.
* Each test function `MUST` be named as `TestXxx`, and 'Xxx' `MUST NOT` start with a lowercase letter e.g. `TestFoo(t *testing.T)`
* Every project `SHOULD` have defined test coverage which is reviewed once a PR is created.


### Structs
* We `SHOULD` use constructors over plain struct initialization, particularly on 'struct layers' (NewHandler, NewRepository, NewService...).
   * `h := NewHandler(foo)` instead of `h := Handler{foo}`.
* We `MUST` name constructor functions using the pattern `New{StructName}`. For example, a Service constructor should be named NewService.




## Application Architecture


### Guiding Principles 


* The application architecture `MUST` be loosely coupled with the separation of concerns.
* The application architecture `MUST` abstract away implementation details and limit how code structures refer to each other, keeping related things within a boundary.
* We `MUST` package the code in these groups and place them into distinct “layers”.
* We `SHOULD` adhere to the dependency inversion principle in which outer layers (implementation details) can refer to inner layers (abstractions), but inner layers only depend on interfaces.
* Dependencies `MUST` point only inwards, and inner layers `MUST NOT` reference outer layers.


### Directories


* `/cmd` - main application for the project. This directory contains the application entry point (s) that will be compiled into a binary executable(s).
* `/pkg` - library code that is open to use for external applications to import.
   * This includes generated Go code for gRPC as it contains both server and client code which is used by both the project it is in and external projects.
* `/internal` - private application code, arranged in layers (detailed below).
   * Additional structure can be added to `internal` by separating shared and non-shared internal code. Actual application code can go in an `internal/app` directory whilst the code shared by those apps in an `internal/pkg` directory. Note that this is optional and should be evaluated based on whether an extra layer of nesting would be beneficial given the size of the application.
   * For more details on this refer to: https://github.com/golang-standards/project-layout


### Internal Package


* We `MUST` structure the internal directory into clear business domain objects by package.
* We `SHOULD` structure our business domain objects into the following layers, as outlined below.


### Handler


* This layer `MUST` only be responsible for serving requests in or out of a package.
* This layer `SHOULD` be aware of the presentation model and the functions should fulfill the Service interface.
* The handler `SHOULD` validate request-related data before being passed to a service.
* Dependencies: service, adaptor


### Adapter


* This layer `MUST` provide functions that adapt and translate internal models to external models and vice versa. The adapter enables your application to communicate externally with HTTP or gRPC clients, files readers/writers and publishers/subscribers, etc.
* We `SHOULD` use adapter functions to decouple internal models from external models.
* Dependencies: models


### Service


* This layer `MUST` contain the internal business domain logic by pulling in data from repositories.
* Dependencies: repository, other service layers


### Repository


* This layer `MUST` be a data access layer which is an interface for executing CRUD operations to a datastore and it `MUST NOT` contain business logic.
* This layer `SHOULD` contain data access logic, and `SHOULD` interact only with one database table/model.
* Where we implement multiple datastores for a single model, we `SHOULD` create two separate repository files in the same package, both with the suffix `*_repository.go`. For example, for the same repository model using a MySQL Database for permanent storage and memcached for cache.
* Dependencies: database pool


### Models


* This layer `MUST` represent a collection of data structure definitions and `MUST NOT` contain business-domain logic.
* We `MUST` define and store models within its relevant package. We `SHOULD NOT` store models within a generic `models` package, in order to avoid coupling and circular referencing.
* We `SHOULD` store generated presentation models (usually from gRPC) under `pkg/api/`.


### Other


* Middleware.go
   * This layer `MUST` provide abstracted code that is executed either on the Server before the request is passed onto the user’s application code, or on the client around the user call.
   * We `SHOULD` develop and use internal libraries and middleware that implement common re-usable patterns of generic functionality such as logging, retries, monitoring, and tracing.
  
* Validator
   * This layer `MUST` provide validation within the handler, and validate the input/request before passing data to the service layer.


### Example Structure


```
internal
|        
└───<business-domain-object-one>
│   │   repository.go
│   │   service.go
│   │   handler.go
│   │   adapter.go
│   │   model.go
│   │
│   └───<business-domain-object-one-child>
│       │   repository.go
│       │   service.go
│       │   handler.go
│       │   adapter.go
│  
└───<business-domain-object-two>
│   │   repository.go
│   │   service.go
│   │   handler.go
│   │   adapter.go
│   │
└───<business-domain-object-three>
   │   repository.go
   │   service.go
   │   handler.go
   │   adapter.go
   ...  


```


## Go Libraries


* We `SHOULD` use standardized, established libraries instead of re-inventing the wheel where re-usable functionality has already been created.
   * These cover areas such as, but not limited to: logging, SQL tooling, token authentication, AWS system management, and HTTP/gRPC Server routing. There are further internal libraries that should be referred to.
* We `MUST` document and refer to these Go Services and Library dependencies, so that the internal or third-party functionality is used consistently across our application ecosystem.




## Drawbacks
* This is just one approach to structuring a Go project. In the case of smaller application ecosystems, the cost-benefit overhead should be considered. In some cases, another approach may also be more relevant depending on the specific project’s needs. However, in this trade-off, we have opted to favor consistency and standardization in our Go services, for motivations detailed under Section 2.
* Where we diverge from the specificities of the RFC, we `MUST` still adhere to the RFC's Guiding Principles.


## Alternatives
* Another option is to follow the application structure and principles of one of the Go frameworks such as [Revel | full-stack web framework for Go](https://github.com/revel/revel) and [Gin-Gonic | HTTP web framework written in Go](https://github.com/gin-gonic/gin).

