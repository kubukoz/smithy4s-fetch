# smithy4s-fetch

<!--toc:start-->
- [smithy4s-fetch](#smithy4s-fetch)
  - [Installation](#installation)
  - [Getting started](#getting-started)
  - [Contributing](#contributing)
  <!--toc:end-->

An _experimental_ [Smithy4s](https://disneystreaming.github.io/smithy4s/) client backend for [Scala.js](https://www.scala-js.org/), utilising [Fetch](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API/Using_Fetch) directly, without introducing a http4s/cats dependency.

The purpose of this library is to provide users of Smithy4s backend services a more lightweight client for the frontend – if your Scala.js frontend is not using Cats/Cats-Effect based libraries, you can communicate with your Smithy4s backend directly using Fetch, **reducing bundle size by as much as 50% in some instances**.

The library is currently only available for Scala 3, but we will welcome contributions cross-compiling it to 2.13 – it should be very easy. API surface is very minimal and designed for binary compatible evolution, so after initial round of testing and gathering community feedback, we plan to release 1.0.0 and start checking binary/Tasty compatibility for each release.

## Installation 

- **SBT**: `libraryDependencies += "tech.neander" %%% "smithy4s-fetch" % "<latest version>"`
- **Scala CLI**: `//> using dep tech.neander::smithy4s-fetch::<latest version>`
- **Mill**: `ivy"tech.neander::smithy4s-fetch::<latest version>"`

## Getting started

For example's sake, let's say we have a smithy4s service that models one of the endpoints from https://httpbin.org, defined using [smithy4s-deriving](https://github.com/neandertech/smithy4s-deriving) (note we're using [Scala CLI](https://scala-cli.virtuslab.org) for this demo):

```scala
//> using dep "tech.neander::smithy4s-deriving::0.0.2"
//> using platform scala-js
//> using scala 3.4.2
//> using option -Wunused:all

import smithy4s.*

import scala.annotation.experimental
import scala.scalajs.js.Promise

import deriving.{given, *}
import aliases.*

case class Response(headers: Map[String, String], origin: String, url: String)
    derives Schema

@experimental
trait HttpbinService derives API:
  @readonly
  @httpGet("/get")
  def get(): Promise[Response]
```

***Note** that we only need to use `@experimental` annotation because we are using smithy4s-deriving.*
*If you're creating clients for services generated by standard Smithy4s codegen, just remove all `@experimental` annotations*
*you see.*

To create a Fetch client for this service all we need to do is this:

```scala
import smithy4s_fetch.*

@main @experimental
def helloWorld = 
  val service: HttpbinService = 
    SimpleRestJsonFetchClient(
      API.service[HttpbinService],
      "https://httpbin.org"
     ).make.unliftService

  service.get().`then`(v => println(v))
```

If your locally installed Node.js version is higher than 18 (version where `fetch` was added into Node.js), then you can run this example for youreself by running `make run-example`

## Contributing

If you see something that can be improved in this library – please contribute!

This is a relatively is

Here are some useful commands:

- `make test` – run tests
- `make check-docs` – verify that snippets in `README.md` (this file) compile
- `make pre-ci` – format the code so that it passes CI check
- `make run-example` – run example from README against real https://httpbin.org
