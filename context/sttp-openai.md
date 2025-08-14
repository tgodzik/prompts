### Importing sttp-openai using scala-cli / scala runner directive

```scala
//> using dep com.softwaremill.sttp.openai::core:0.3.8
```

### Create request for inference

```scala
import sttp.openai.requests.completions.chat.ChatRequestBody.{ChatBody, ChatCompletionModel}
import sttp.openai.requests.completions.chat.message.*

// Create body of Chat Completions Request
val bodyMessages: Seq[Message] = Seq(
  Message.UserMessage(
    content = Content.TextContent("Hello!"),
  )
)

// use ChatCompletionModel.CustomChatCompletionModel("another-model") 
// for models not yet supported here
val chatRequestBody: ChatBody = ChatBody(
  model = ChatCompletionModel.GPT4oMini,
  messages = bodyMessages
)
```

All snippets below use the request defined here. Reused imports are not reimported.

### Execute request for completion using sync backend

```scala
import sttp.openai.requests.completions.chat.ChatRequestResponseData.ChatResponse
import sttp.openai.OpenAISyncClient

val apiKey = System.getenv("OPENAI_KEY")
val openAISync = OpenAISyncClient(apiKey)

// this may throw:
val chatResponse: ChatResponse = openAISync.createChatCompletion(chatRequestBody)
chatResponse.choices.headOption.map(_.message.content).foreach(println)
```

### Execute request for completion using cats backend

```scala
//> using dep org.typelevel::cats-effect:3.6.3
//> using dep com.softwaremill.sttp.client4::cats:4.0.9

import sttp.client4.httpclient.cats.HttpClientCatsBackend
import sttp.openai.OpenAIExceptions.OpenAIException
import sttp.openai.*
import cats.effect.*

val openAI = OpenAI(apiKey)

val result: IO[Option[String]] = HttpClientCatsBackend.resource[IO]().use { backend =>
  val response: IO[Either[OpenAIException, ChatResponse]] =
    openAI
      .createChatCompletion(chatRequestBody)
      .send(backend)
      .map(_.body)
  
  // rethrow goes from IO[Either[Throwable, A]] to IO[A]
  response.rethrow.map(_.choices.headOption.map(_.message.content))
}
```

### Execute streaming request with Ox (direct-style, Loom)

```scala
//> using dep com.softwaremill.sttp.openai::ox:0.3.8

import ox.*
import ox.either.orThrow
import sttp.client4.DefaultSyncBackend
import sttp.openai.streaming.ox.*

supervised:
  val backend = useCloseableInScope(DefaultSyncBackend())
  openAI
    .createStreamedChatCompletion(chatRequestBody)
    .send(backend)
    .body // this is Either[OpenAIException, Flow[ChatChunkResponse]]
    .orThrow // throw any exceptions
    .runForeach(el => println(el.orThrow))
```

### Execute streaming request with FS2

```scala
//> using dep com.softwaremill.sttp.openai::fs2:0.3.8

import sttp.openai.requests.completions.chat.ChatChunkRequestResponseData.ChatChunkResponse
import sttp.client4.httpclient.fs2.HttpClientFs2Backend
import sttp.openai.OpenAIExceptions.OpenAIException
import sttp.openai.streaming.fs2.*
import cats.effect.*
import fs2.*

val streamResult = HttpClientFs2Backend.resource[IO]().use { backend =>
  val response: IO[Either[OpenAIException, Stream[IO, ChatChunkResponse]]] =
    openAI
      .createStreamedChatCompletion[IO](chatRequestBody)
      .send(backend)
      .map(_.body)

  response.flatMap {
    case Left(exception) => IO.println(exception.getMessage)
    case Right(stream)   => stream.evalTap(IO.println).compile.drain
  }
}
```