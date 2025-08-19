# Writing Custom Wartremover Rules

This guide shows how to implement custom warts (linting rules) for wartremover.

## Basic Structure

All warts extend `WartTraverser` and implement the `apply` method that returns a `Traverser`:

```scala
package org.wartremover.warts

object MyCustomWart extends WartTraverser {
  def apply(u: WartUniverse): u.Traverser = {
    import u.universe._
    
    new u.Traverser {
      override def traverse(tree: Tree): Unit = {
        tree match {
          // Pattern match on AST nodes
          case pattern => 
            error(u)(tree.pos, "Error message")
            super.traverse(tree)
          case _ => 
            super.traverse(tree)
        }
      }
    }
  }
}
```

### Suppression Handling

Always check for wart annotations to allow users to suppress warnings:

```scala
case t if hasWartAnnotation(u)(t) =>
  // Skip this tree - it's suppressed
```

### Synthetic Code Detection

Use `isSynthetic(u)(tree)` to ignore compiler-generated code:

```scala
val synthetic = isSynthetic(u)(tree)
tree match {
  case ValDef(mods, _, _, _) if mods.hasFlag(Flag.MUTABLE) && synthetic =>
    // Ignore synthetic mutable vals (pattern matching artifacts)
```

### Error Reporting

Use the helper methods for consistent error reporting:

```scala
error(u)(tree.pos, "Description of the problem")
warning(u)(tree.pos, "Warning message")
```

## Example Implementations

### Simple Literal Detection

Forbid magic numbers:

```scala
object NoMagicNumbers extends WartTraverser {
  def apply(u: WartUniverse): u.Traverser = {
    import u.universe._
    
    new u.Traverser {
      override def traverse(tree: Tree): Unit = {
        tree match {
          case t if hasWartAnnotation(u)(t) =>
            
          case Literal(Constant(n: Int)) if n != 0 && n != 1 =>
            error(u)(tree.pos, s"Magic number $n is not allowed")
            super.traverse(tree)
            
          case _ => super.traverse(tree)
        }
      }
    }
  }
}
```


### Type-Based Detection

Forbid specific types or type patterns:

```scala
object NoMutableCollections extends WartTraverser {
  def apply(u: WartUniverse): u.Traverser = {
    import u.universe._
    
    val mutableCollectionTypes = Set(
      "scala.collection.mutable.ListBuffer",
      "scala.collection.mutable.ArrayBuffer",
      "scala.collection.mutable.Set",
      "scala.collection.mutable.Map"
    )
    
    new u.Traverser {
      override def traverse(tree: Tree): Unit = {
        tree match {
          case t if hasWartAnnotation(u)(t) =>
            
          case ValDef(_, _, tpt, _) if mutableCollectionTypes.contains(tpt.tpe.typeSymbol.fullName) =>
            error(u)(tree.pos, s"Mutable collection ${tpt.tpe.typeSymbol.fullName} is forbidden")
            super.traverse(tree)
            
          case _ => super.traverse(tree)
        }
      }
    }
  }
}
```
