# ECMAScript proposal: Types as Comments

## _NOTE: This is a fork of https://github.com/giltayar/proposal-types-as-comments but I haven't updated all files. I'm just exploring alternate syntax for types._

This proposal aims to enable developers to add annotations to their JavaScript code, allowing those annotations to read by a type checker, transpiler or other tool that is _external to JavaScript_.
At runtime, a JavaScript engine ignores them, treating the annotations as comments.

The aim of this proposal is enable static tools like [TypeScript](https://www.typescriptlang.org/), [Flow](https://flow.org/), and others to integrate into JavaScript without any need for transpilation.

## Proposal

> The following is a _strawperson_ proposal. Please treat it as such.

### Annotations
Annotations are prefixed with `::`. If the next character is a valid identifier start then the annotation goes until an identifier would normally terminate. If `::` is followed by a brace `{[(` then the annotation continues until the ballanced brace closes. Limitation: this assumes that all annotation forms use ballanced braces.


```js

let x::string = 'hello';

x = 100;
```
In the example above, `x` is annotated with the value `string`.
Tools such as TypeScript can utilize that annotation, and might choose to error on the statement `x = 100`;
however, a JavaScript engine that follows this proposal would execute every line here without error.
This is because annotations do not change the semantics of a program, and are equivalent to comments.

### Pragma

To signal to annotation consumers that they should parse the annotations a pragma statement can be set:

```js
::@ts_check
```

where `::@` designates the start of the pragma and what follows is an identifier for the annotation consumer which can be any valid js identifier. This again is treated as a comment by the js runtime but is offered as a starting convention to prevent collision of tools. This pragma applies to the scope in which the pragma is inserted be that the global scope or a particular function.

## Examples

### TypeScript


```ts
class Point {
    public readonly x: number
}
```
Using Annotations:
```js
class Point {
    x::(public readonly number)
}
```

If these are permitted as part of this proposal, the semantics would be to ignore them all, treating the above class the same as the following.

```js
class Point {
    x;
}
```

TypeScript could choose to prefer another encoding of the same type information

```ts
class Point {
  ::public ::readonly x::number
}
```
These forms are all valid JS but may or not be valid for the annotation consumer.

Multiline example
```ts
::(interface Customer {
    id: number;
    name: string;
})
```

## FAQ

### Does JavaScript need static type-checking?

Given how much effort organizations and teams have put into building type-checkers and adopting them, the answer is **yes**.
Perhaps not every developer will reach for static type-checking, **and that's okay** - that's why this proposal makes type annotations entirely optional;
however, the ecosystem demand for using types is undeniable.

TypeScript has done a great job of demonstrating this - it's gained very widespread use with broad signals that people want to keep using it.
It's opt-in, but it has a major presence in the ecosystem, and these days TypeScript support is seen as a huge advantage for libraries.

The question is not whether JS should have types, but rather "how should JS work with types?"
One valid answer is that the current ecosystem provides sufficient support where types are stripped out separately ahead-of-time, but this proposal may provide advantages over that approach.

### Why not unofficially build TS checking and compilation into various systems?

A number of systems, such as [ts-node](https://github.com/TypeStrong/ts-node) and [deno](https://deno.land/manual/getting_started/typescript), have tried this.
Apart from startup performance issues, a common problem is compatibility across versions and modes, in type checking semantics, grammar, and transpilation output. 
This proposal would not subsume the needs for all of those features, but it would provide one compatible syntax and semantics to unify around for many needs.

### Why not stick to existing JS comment syntax?

Although it is possible to define types in existing JavaScript comments, as Closure and TypeScript's JSDoc mode do, this syntax is much more verbose and unergonomic.

Consider the fact that JSDoc type annotations were present in Closure Compiler prior to TypeScript, and that TypeScript's JSDoc support has been present for years now.
While types in JSDoc has grown over the years, most type-checked JavaScript is still written in TypeScript files with TypeScript syntax.
This is similarly the case with Flow, where Flow's type-checker can analyze Flow-like syntax in existing JavaScript comments, but most Flow users continue to just use direct annotation/declaration syntax.

### Don't all JS developers transpile anyway? Will it really help to remove the type-desugaring step?

The JavaScript ecosystem has been slowly moving back to a transpilation-less future. The sunsetting of IE11 and the rise of evergreen browsers that implement the latest JavaScript standard means that developers can once again run standard JavaScript code without transpilation.
The advent of native ES modules in the browser and in Node.js also means that, at least in development, the ecosystem is working its way to a future where even bundling is not necessary.
Node.js developers in particular, have historically avoided transpilation, and are today torn between the ease of development that is brought by no transpilation, and the ease of development that languages like TypeScript bring.

Implementing this proposal means that we can add type systems to this list of "things that don't need transpilation anymore" and bring us closer to a world where transpilation is optional and not a necessity.

### Does this proposal make all TypeScript programs valid JavaScript?

No. In fact typescript programs would have to be modified to be valid JS programs. This could conceivably be done via a code transformation however.

### Does this proposal make all Flow programs valid JavaScript?

Same as for TypeScript

### What about `.d.ts` files and "libdef" files?

*Declaration files* and *library definition* files are used by TypeScript and Flow respectively as a kind of "header" file that describes values, their types, and where they exist.
This proposal can safely ignore them as it does not need to interpret the semantics of the type information inside them.
TypeScript and Flow will continue reading and analyzing these files as they do today.

### Does this proposal mean that TypeScript developers would have to modify their codebases?

No. TypeScript can continue to be TypeScript, with no compatibility impact or changes to codebases.
This proposal would give developers the _option_ to restrict themselves to a particular subset of TypeScript which would run as JavaScript without transpilation.

Developers may still want to use TypeScript syntax for other reasons:

- Use of certain syntax features which are not supported in JavaScript (e.g., `enum`s and parameter properties)
- Compatibility with existing codebases which may run into certain syntax edge cases that are handled differently
- Non-standard extensions/reinterpretations of JavaScript (e.g. experimental legacy decorators, decorator metadata, or \[\[Set]] semantics for fields)

If developers decide to migrate an existing TypeScript codebase to use the JavaScript syntax specified in this proposal, the goal of this proposal is that the modifications would be minimal.
We believe that many developers would be motivated by removing a build step, but others could decide to stick with TypeScript compilation and enjoy the full power of the language.

### How should tools work with JavaScript type syntax?

Given the fact that some TypeScript features are [out of scope](#intentional-omissions), and that standard JavaScript will not evolve as fast as TypeScript or support its variety of configurations, there will continue to be an advantage for many tools to support TypeScript in its fuller form, beyond what is potentially standardized as JavaScript.

Some tools currently need a "plugin" or option to get TypeScript support working.
This proposal would mean that these tools could support type syntax by default, forming a standard, versionless, always-on common base for type syntax.
Full and prescriptive support for TypeScript, Flow, etc. could be remain an opt-in mode on top of that.

### What about compatibility with ReasonML, PureScript, and other statically typed languages that compile to JavaScript?

While these languages _compile_ to JavaScript, and have static typing, they are not supersets of JavaScript, and thus are not relevant to this proposal.

### Will the ability to deploy annotated source code directly result in bloated applications?

Returning to a world where code does not strictly need to be compiled prior to being used in production means that developers may end up deploying more code than is necessary.
Hence larger payloads over-the-wire for remotely served apps, and more text to parse at load time.

However this situation already exists.
Today, many users omit a build step and ship large amounts of comments and other extraneous information, e.g. unminified code.
It remains a best practice to perform an ahead-of-time optimization step on code destined for production if the use-case is performance-sensitive.

### How could runtime-checked types be added in future?

The annotations could conceivably be used as a macro language to do code-generation as part of some transpilation step but this would not be considered part of this specification as that would be the purview of the annotation consumers.

## Prior Art

### Other languages that have optional erasable type syntax

When Python decided to add a gradual type system to the language, it did it in two steps.
First, a proposal for type annotations was created in [PEP-3107 - Function Annotations](https://www.python.org/dev/peps/pep-3107/) that specified parameter types and function return types in Python
Several years later, the places in which types could occur was expanded in [Python PEP-484 - Type Hints](https://www.python.org/dev/peps/pep-0484/).


The proposal here differs significantly from Python's types, as the types in this proposal are entirely ignored, not evaluated as expressions or accessible at runtime as metadata. This difference is largely motivated by the existing community precedent, where JS type systems do not tend to use JS expression grammar for their types, so it is not possible to evaluate them as such.

[Ruby 3 has now also implemented RBS](https://www.ruby-lang.org/en/news/2020/12/25/ruby-3-0-0-released/): type definitions that sit _beside_ the code
and are not part of it.

#### Languages that add type systems onto JavaScript

[TypeScript](https://www.typescriptlang.org/docs/handbook/intro.html), [Flow](https://flow.org/en/docs/), and [Hegel](https://hegel.js.org/docs) are languages that implement type systems above standard JavaScript.

#### Ability to add type systems to JavaScript via comments

Both TypeScript and Flow enable developers to write JavaScript code and incorporate
types as comments that the JavaScript runtime ignores.

For Flow, these are [Flow comment types](https://flow.org/en/docs/types/comments/), and
for TypeScript these are [JSDoc comments](https://www.typescriptlang.org/docs/handbook/type-checking-javascript-files.html).

See the author's blog post on their positive experience with TypeScript's JSDoc comments [here](https://gils-blog.tayar.org/posts/jsdoc-typings-all-the-benefits-none-of-the-drawbacks/).

Closure Compiler's type checking works entirely via JSDoc comments ([docs](https://developers.google.com/closure/compiler/docs/js-for-compiler)). The Closure Compiler team has received many requests for an in-line type syntax, but was hesitant to do this without a standard.

#### Relevant proposals and discussions in TC39

TC39 has previously discussed [guards](https://web.archive.org/web/20141214075910/http://wiki.ecmascript.org/doku.php?id=strawman:guards), which form a new, stronger type system.

Previously, Sam Goto led discussions around an [optional types proposal](https://github.com/samuelgoto/proposal-optional-types) which aimed to unify syntax and semantics across the type-checkers.
Trying to find agreement across type-checkers, along with defining a sufficient subset in both syntax and semantics meant that there were difficulties with this approach.
An evolution of this plan was [pluggable types](https://github.com/samuelgoto/proposal-pluggable-types) which was [inspired by Gilad Bracha's ideas on pluggable type systems](http://bracha.org/pluggableTypesPosition.pdf).
This proposal is extremely similar to the pluggable types proposal, but leans a bit more heavily on the idea of viewing types as comments, and comes at a time with broader adoption of type-checking and a more mature type-checking ecosystem.

The [optional types proposal repository contains links to other prior discussions around types in JavaScript](https://github.com/samuelgoto/proposal-optional-types/blob/master/FAQ.md#tc39-discussions).
