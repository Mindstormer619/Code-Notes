# Kotlin Compiler
> Created: 2022-05-05 16:33

The Kotlin compiler is unique in that its frontend is built on top of it, making it easy to share the frontend with both compiler plugins and IDE plugins. For Kotlin, the goal of the frontend is to parse the written code and analyze its interpreted structure so it can generate _intermediate representation (IR)._ This IR, along with additional generated information is then fed to the backend of the compiler¹, which further analyzes, augments, and optimizes the IR before it eventually becomes machine code.

### Compiler Frontend

![[Pasted image 20220509102439.png]]

### Compiler Overview

![[Kotlin_compiler.png]]

## Parsing Phase

Lexer: Parses the source file and generates tokens.

At the parsing phase, the compiler is not concerned with _whether or not the code works_ — it is only concerned with figuring out _what was written_ in the file. The **Parsing phase** is responsible for creating syntax trees so the compiler is able to analyze and verify the code later on in the **Resolution phase.**

The basis of parsing can be seen in the following two-stage process:

1.  _Lexical analysis:_ The text file is parsed into tokens.
2.  _Syntax analysis:_ The tokens are parsed and organized into a syntax tree.

### Lexical Analysis

During lexical analysis, the Kotlin parser first creates a [KotlinLexer][klex]. This lexer scans the Kotlin file and breaks up text into a set of _tokens_ called [KtTokens][kttokens]. 

[KotlinExpressionParsing](https://github.com/gigliovale/kotlin/blob/master/compiler/frontend/src/org/jetbrains/kotlin/parsing/KotlinExpressionParsing.java) will set up a visitor to arrange these tokens into sets of expression nodes. These expression nodes are then appended to build a PSI tree via [ASTNode](https://github.com/JetBrains/intellij-community/blob/master/platform/core-api/src/com/intellij/lang/ASTNode.java).

The [_Programming Structure Interface_ (_PSI)_](https://plugins.jetbrains.com/docs/intellij/psi-elements.html#how-do-i-get-a-psi-element) is an abstraction built by JetBrains. It is sort of like a heavy-weight generic syntax tree handling text/code/language in their IDEs. These trees are in-memory representations generated during syntax analysis, and are needed for the compiler to generate additional data and recursively analyze itself for code verification in later phases. PSI is useful for compiler plugins _and_ IDEA plugins alike, since you can filter PSI to intercept specific pieces of code.

----

## References
1. https://medium.com/google-developer-experts/crash-course-on-the-kotlin-compiler-1-frontend-parsing-phase-9898490d922b


[klex]: https://github.com/JetBrains/kotlin/blob/92d200e093c693b3c06e53a39e0b0973b84c7ec5/compiler/psi/src/org/jetbrains/kotlin/lexer/KotlinLexer.java
[kttokens]: https://github.com/JetBrains/kotlin/blob/master/compiler/psi/src/org/jetbrains/kotlin/lexer/KtTokens.java