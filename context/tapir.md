### Importing tapir with direct style server using scala-cli / scala runner directive

```scala
//> using dep com.softwaremill.sttp.tapir::tapir-core:1.11.42
//> using dep com.softwaremill.sttp.tapir::tapir-netty-server-sync:1.11.42
//> using dep ch.qos.logback:logback-classic:1.5.18
```

### Serving requests

```scala
import ox.*
import sttp.tapir.*
import sttp.tapir.server.netty.sync.NettySyncServer

@main def main(): Unit =
  val helloWorld = endpoint.get
    .in("hello")
    .in(query[String]("name"))
    .out(stringBody)
    .handleSuccess(name => s"Hello, $name!")

  supervised:
    val serverBinding = useInScope(NettySyncServer().addEndpoint(helloWorld).start())(_.stop())
    println(s"You can now make requests to http://${serverBinding.hostName}:${serverBinding.port}/hello?name=...!")
    never
```

### Serving static content

```scala
//> using dep com.softwaremill.sttp.tapir::tapir-files:1.11.42

import sttp.shared.Identity
import sttp.tapir.emptyInput
import sttp.tapir.files.*
import sttp.tapir.server.netty.sync.NettySyncServer

@main def staticContentFromFilesNettyServer(): Unit =
  NettySyncServer()
    .port(8080)
    .addEndpoints(staticFilesServerEndpoints[Identity](emptyInput)("/var/www"))
    .startAndWait()
```