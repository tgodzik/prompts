### Importing sttp-openai using scala-cli / scala runner directive

```scala
//> using toolkit default
````

### Basic testing with munit

```scala
class MyTests extends munit.FunSuite:
  test("sum of two integers") {
    val obtained = 2 + 2
    val expected = 4
    assertEquals(obtained, expected)
  }

  // catching exceptions
  import java.nio.file.NoSuchFileException

  test("read missing file") {
    val missingFile = os.pwd / "missing.txt"
    intercept[NoSuchFileException] {
      os.read(missingFile)
    }
  }
  
  // resource management
  val usingTempFile: FunFixture[os.Path] = FunFixture(
    setup = _ => os.temp(prefix = "file-tests"),
    teardown = tempFile => os.remove(tempFile)
  )
  usingTempFile.test("overwrite on file") { tempFile =>
    os.write.over(tempFile, "Hello, World!")
    val obtained = os.read(tempFile)
    assertEquals(obtained, "Hello, World!")
  }

  // compile errors
  test("should not compile") {
    compileErrors("val x = 1 = 1")
  }
```