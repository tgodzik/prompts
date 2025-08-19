# Writing Custom Scalafix Rules

This guide focuses on creating custom Scalafix rules for code transformations and linting.

## Rule Types

### SyntacticRule
Works on code structure without compilation information.

```scala
import scalafix.v1._
import scala.meta._

class MyRule extends SyntacticRule("MyRule") {
  override def description: String = "Replace deprecated syntax"
  override def isRewrite: Boolean = true
  
  override def fix(implicit doc: SyntacticDocument): Patch = {
    doc.tree.collect {
      case defn: Defn.Def if defn.name.value == "oldMethod" =>
        Patch.replaceTree(defn.name, Term.Name("newMethod"))
    }.asPatch
  }
}
```

### SemanticRule
Requires compilation information for type-aware transformations.

```scala
import scalafix.v1._
import scala.meta._

class TypeAwareRule extends SemanticRule("TypeAwareRule") {
  override def description: String = "Transform based on types"
  override def isRewrite: Boolean = true
  
  override def fix(implicit doc: SemanticDocument): Patch = {
    doc.tree.collect {
      case Term.Apply(Term.Select(obj, Term.Name("get")), Nil) 
        if obj.symbol.info.exists(_.signature.toString.contains("Option")) =>
        Patch.addLeft(obj.tokens.head, "/* Option.get is unsafe */ ")
    }.asPatch
  }
}
```

## Common Patterns

### Tree Matching and Transformation
```scala
// Match method definitions
case defn: Defn.Def => 
  Patch.replaceTree(defn.name, Term.Name("newName"))

// Match imports
case importStmt: Import =>
  Patch.removeTokens(importStmt.tokens)

// Match object definitions
case obj: Defn.Object if obj.name.value == "Utils" =>
  Patch.addRight(obj.tokens.last, "\n// Generated code")
```

### Working with Tokens
```scala
// Add text at specific positions
Patch.addLeft(tree.tokens.head, "prefix")
Patch.addRight(tree.tokens.last, "suffix")

// Remove tokens
Patch.removeTokens(tree.tokens)

// Replace entire tree
Patch.replaceTree(oldTree, newTree)
```

### Configuration Support
```scala
case class MyRuleConfig(
  enableFeature: Boolean = true,
  excludePatterns: List[String] = Nil
)

class ConfigurableRule(config: MyRuleConfig) extends SyntacticRule("ConfigurableRule") {
  def this() = this(MyRuleConfig())
  
  override def withConfiguration(config: Configuration): Configured[Rule] =
    config.conf.getOrElse("myRule")(MyRuleConfig.default).map(new ConfigurableRule(_))
}
```

## Linting Rules

Create diagnostic-only rules for code quality checks:

```scala
class LintingRule extends SyntacticRule("LintingRule") {
  override def description: String = "Warns about problematic patterns"
  override def isLinter: Boolean = true
  
  override def fix(implicit doc: SyntacticDocument): Patch = {
    val diagnostics = doc.tree.collect {
      case Term.Apply(Term.Name("println"), _) =>
        Diagnostic("warning", "Avoid println in production code", tree.pos)
    }
    
    Patch.lint(diagnostics)
  }
}
```
