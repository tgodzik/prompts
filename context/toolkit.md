### Importing toolkit using scala-cli / scala runner directive

```scala
//> using toolkit default
```

### Filesystem and process operations using os-lib

```scala
val root: os.Path = os.temp.dir(prefix = "tk")
os.write(root / "sample.txt", "alpha\nbeta\ngamma\n")

val upper: Seq[String] =
  os.read
    .lines(root / "sample.txt")
    .map(_.toUpperCase)

os.write.over(root / "sample-upper.txt", upper.mkString("\n"))

val files: Seq[os.Path] = os.list(root).filter(os.isFile)
for file <- files do println(file)

val wc = os.proc("wc", "-l", root / "sample.txt").call()
println(wc.out.trim())

val curl = os.proc("curl", "-L", "https://git.io/fpvpS").spawn(stderr = os.Inherit)
val gzip = os.proc("gzip", "-n").spawn(stdin = curl.stdout)
val sha = os.proc("shasum", "-a", "256").spawn(stdin = gzip.stdout)
println(sha.stdout.trim())
```

### JSON handling using upickle and ujson

```scala
import upickle.default.*

case class Book(title: String, pages: Int) derives ReadWriter

val raw = """{"title":"Scala 3","pages":240}"""
val book: Book = read[Book](raw) // may throw

val fixed = book.copy(pages = book.pages + 10)

val combined: ujson.Value = ujson.Obj(
  "original" -> writeJs(book), // write to JSON AST
  "updated" -> writeJs(fixed)
)

println(ujson.write(combined, indent = 2))
```

### Http handling using sttp4 with quick mode

```scala
import sttp.client4.quick.*

val zenResponse = quickRequest
  .get(uri"https://api.github.com/zen")
  .send()

if zenResponse.isSuccess then println(zenResponse.body)
else println(s"${zenResponse.statusText} ${zenResponse.body}")

val payload = ujson.Obj("note" -> "scala-toolkit demo")
val httpBinResponse = quickRequest
  .post(uri"https://httpbin.org/post")
  .header("Content-Type", "application/json")
  .body(write(payload))
  .send()

if httpBinResponse.isSuccess then println(httpBinResponse.body)
else println(s"${httpBinResponse.statusText} ${httpBinResponse.body}")
```