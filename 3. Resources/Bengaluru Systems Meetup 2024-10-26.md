---
created: 2024-10-26 18:10
alias: 
---
> [!summary]-
> + 

# Bengaluru Systems Meetup 2024-10-26

## Faster and more efficient JS tooling

> ~ Srijan Paul (@injuly)

ESLint, the linter. Prettier, the autoformatter (uses CST to parse JS code and rewrites it in a prettier format). Babel, the tools that you do use most often depend on it, it's a lot of things which is a modern parser for JS which adds polyfills including some outside the JS standards (like `if` expressions).

Biggest USP for ESLint is that it supports JS, JSX, TS, TSX, Vue, Svelte etc. etc. Pretty much all of the derived languages etc. Also supports ESQuery, which is a query language to query your parse trees.

Contentions for ESLint:
+ Performance isn't great (felt most acutely in an IDE like VSCode or CI pipelines)
+ Limited to the AST. In-depth static analysis requires more.
+ [ ] SSA format
+ Very little info about control flow.
+ Falls short on security-based lints (e.g. taint analysis)

+ [ ] Rewrite of ESLint discussion: https://github.com/eslint/eslint/discussions/16557

Future of tooling: Biome (https://biomejs.dev) is a linter, formatter written for JS, written in Rust. Also Oxc https://oxc.rs (parser, linter, minifier, formatter).

What does it take to build a linter?
+ Parser (one for each dialect of JS) (see https://astexplorer.net for an interesting tool) e.g. acorn, tenko, oxc_parser
+ Scope resolver: because JS scopes are weird. (e.g. escope)
+ A uniform AST format (e.g. ESTree)
+ AST query / traversal mechanism e.g. ESTraverse, ESQuery
+ 


----

## References
+ 