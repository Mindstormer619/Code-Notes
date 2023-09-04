# VertX
> Created: 2023-02-16 13:03

Eclipse Vert.x is a tool-​kit for build­ing **re­ac­tive** ap­pli­ca­tions on the JVM. Re­ac­tive ap­pli­ca­tions are both **scal­able** as work­loads grow, and **re­silient** when fail­ures arise. A re­ac­tive ap­pli­ca­tion is **re­spon­sive** as it keeps la­tency under con­trol by mak­ing ef­fi­cient usage of sys­tem re­sources, and by pro­tect­ing it­self from er­rors.

Pro­cess­ing more con­cur­rent con­nec­tions with less threads is pos­si­ble when you use **asyn­chro­nous I/O**. In­stead of block­ing a thread when an I/O op­er­a­tion oc­curs, we move on to an­other task which is ready to progress, and re­sume the ini­tial task later when it is ready.

Vert.x mul­ti­plexes con­cur­rent work­loads using **event loops**.

![[event-loop.f2afd59d.svg]]

Code that runs on event loops should not per­form block­ing I/O or lengthy pro­cess­ing. But don’t worry if you have such code: Vert.x has worker threads and APIs to process events back on an event loop.

We know that asyn­chro­nous pro­gram­ming re­quires more ef­forts. At the core of Vert.x, we sup­port **call­backs** and **promises/fu­tures**, the lat­ter being a sim­ple and el­e­gant model for chain­ing asyn­chro­nous op­er­a­tions.

Ad­vanced re­ac­tive pro­gram­ming is pos­si­ble with [Rx­Java](https://github.com/ReactiveX/RxJava), and if you pre­fer some­thing closer to tra­di­tional im­per­a­tive pro­gram­ming, then we are happy to pro­vide you with first-​class sup­port of [Kotlin corou­tines](https://kotlinlang.org/docs/reference/coroutines-overview.html).

Vert.x of­fers tools to keep la­tency under con­trol, in­clud­ing a sim­ple and ef­fi­cient **cir­cuit breaker**.

```kotlin
class MainVerticle : AbstractVerticle() {
  override fun start() {
    // Create a Router
    val router = Router.router(vertx)

    // Mount the handler for all incoming requests at every path and HTTP method
    router.route().handler { context ->
      // Get the address of the request
      val address = context.request().connection().remoteAddress().toString()
      // Get the query parameter "name"
      val queryParams = context.queryParams()
      val name = queryParams.get("name") ?: "unknown"
      // Write a json response
      context.json(
          json {
            obj(
              "name" to name,
              "address" to address,
              "message" to "Hello $name connected from $address"
            )
          }
      )
    }

    // Create the HTTP server
    vertx.createHttpServer()
        // Handle every request using the router
        .requestHandler(router)
        // Start listening
        .listen(8888)
        // Print the port
        .onSuccess { server ->
          println("HTTP server started on port " + server.actualPort())
        }
  }
}
```

Book: https://www.manning.com/books/vertx-in-action

----

## References
1. https://vertx.io/introduction-to-vertx-and-reactive/