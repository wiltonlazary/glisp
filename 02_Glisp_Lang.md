# GLISP – Language Description

> **Authority:** This document is the authoritative technical description. No abstractions or proposals beyond what is written here.

---

# Part I: Foundation

## §1 Philosophy

GLISP emerged from recognizing that different programming paradigms (command-line, functional, object-oriented, reactive, effect-driven, Lisp) aren't fundamentally different - they're variations on structured data transformation wearing different syntactic clothes. The parser doesn't need different modes because there aren't different modes, just different patterns emerging from the same underlying mechanics.

The language follows thermodynamic principles - simple local rules compose to create complex behaviors. Just as thermodynamics provides fundamental laws governing energy transformations without dictating specific configurations, GLISP provides core parsing rules that enable all surface syntaxes and computational patterns without prescribing one style.

GLISP recognizes the profound truth that **everything is struct transformation** - functions receive input structs and produce output structs. Effects transform execution context. Reactive streams transform time-ordered data. Types are struct templates with defaults and constraints. Abstract structs are incomplete transformations waiting to be filled. All computation reduces to structured data undergoing mechanical transformation.

## §2 Overview

GLISP (Grafted Lisp) is a **homoiconic, multi-paradigm** surface language with a **Lisp-like core (Lispian)**.

GLISP's syntax design isn't accidental - it's carefully engineered to enable single-pass, no-backtracking parsing while supporting the full spectrum of computational patterns. This is why:
* Grafts require delimiter touching (no ambiguity)
* Glue requires no spaces (immediately recognizable)
* Spaced transformers are distinct from glued ones
* Section triggers are unambiguous
* Everything can be determined from local context

The syntax and parser constraints work together by design to support all programming paradigms uniformly.

Key properties:
* **Style-flexible parser** — all surface styles transform to the same core
* **Gradual typing** — dynamic ⟷ static spectrum with structural contracts
* **Configurable syntax** — directives customize parsing behavior for all patterns
* **Traceable transformations** — source offsets preserved through all transforms
* **Deterministic round-trip** — `parse ↦ print ↦ parse` yields identical AST
* **Streaming capable** — can parse infinite sources incrementally
* **Memory-aware** — ownership and resource management integrated
* **Reactive-native** — streams and channels as first-class constructs
* **Async-transparent** — asynchronous operations use identical syntax

Surface styles expressible through transformations include:
Command-line, Data-oriented (DOP), Functional-first (FFP), Object-oriented (OOP), Reactive, ML-like, Nim-like, Python-like, C-like, Lisp-like, and combinations thereof within the same program.

## §3 Core Concepts

### 3.1 Unified Data Model

**Datum** ∈ {`AtomDatum`, `ListDatum`}

* **AtomDatum** — contiguous non-whitespace characters (identifiers, numbers, transformers, abstract markers)
* **ListDatum** — delimited sequence with preserved delimiter pair and semantic context

All structures are:
1. **Positions** (0, 1, 2, ...) - the universal substrate
2. **Names** (optional aliases to positions)
3. **Values** (content at positions, may be abstract with `?`)
4. **Context** (execution, data, memory, reactive)

### 3.2 Transform Priority

Transform priority emerges from parsing phases:
1. **System patterns** (`./`, `../`, `://`, `~/`) - highest precedence
2. **Glue transformers** - during atom parsing (`x:y`, `&data`, `data~>stream`)
3. **Grafts** - zero whitespace between structures (`f[x]`, `handler{body}`)
4. **Spaced transformers** - during buffer processing (whitespace on both sides)
5. **Section/structure materialization** - at boundaries

### 3.3 Transformer Spacing Rules

**Spaced:** Requires whitespace on BOTH sides
- Section trigger: `x : y` (spaced colon)
- Infix: `a + b` (spaced plus)
- Associative: `x = y` (spaced equals)
- Effect: `x ! y` (spaced exclamation)

**Glued:** NO whitespace on specified side(s)
- Prefix glue: `'x`, `--x`, `&data` (glued left)
- Suffix glue: `x:y`, `x.field`, `x~>y` (glued right)
- LHS-glued: `x: y`, `x:\n  y` (no left space, any right)

### 3.4 Computational Contexts

GLISP recognizes four fundamental computational contexts, all emerging from the same parsing mechanics:

* **Execution Context** — Sequential computation (sections with `{}`)
* **Data Context** — Structural data (structures with `[]`)
* **Memory Context** — Resource management (memory marks)
* **Reactive Context** — Time-ordered transformation (streams)

---

# Part II: Stream Parser

## §4 Parser Model

The GLISP parser transforms character streams into abstract syntax trees through mechanical pattern recognition. The parser applies local transformations based on configuration, never needing to understand semantics of any computational pattern - whether functional, object-oriented, reactive, or memory-managed.

**Key principles:**
* **Streaming** — Processes input character by character
* **Mechanical** — Applies deterministic transformations based on patterns
* **Configurable** — Behavior controlled by directives for all paradigms
* **Local** — Makes decisions based on local context (near-by short-range forces)
* **Universal** — Same rules handle all computational patterns

**Mental model:** Characters ↦ Patterns ↦ Transformations ↦ AST (supporting all paradigms)

## §4.1 System Patterns and Command-Line Fluidity

GLISP recognizes that file paths and URLs are **system-level primitives** that transcend language configuration. These patterns are fundamental to all computational contexts and must be preserved as single atoms.

**Design Principle:**

Command-line fluidity requires natural expression of system resources across all paradigms:
```glisp
# Natural system interaction in any context
cat ./tmp.txt                    # Command-line
data~>save("./output.txt")       # Reactive
async val load: [file] => <- read(file)  # Async
arena temp(megabyte) : read("~/config")  # Memory context
```

System patterns (`./`, `../`, `:/`, `~/`) are recognized before any configured transformers, ensuring filesystem and network paths remain intact in all computational contexts.

## §5 Configuration

GLISP behavior is controlled via directives that support all computational paradigms:

```glisp
#directive="#"                                             # Directive prefix
#comment="#"                                               # Comment prefix (requires space after)
#glue={
  "prefix": {
    "quote": "'"        # Quote - literal AST
    "quasiquote": "''"  # Quasiquote - AST template  
    "unquote": "~"      # Unquote in quasiquote
    "splice": "~*"      # Splice in quasiquote
    "meta": "@"         # Meta section prefix
    "spread": "..."     # Spread prefix (exemplar macro)
    "param": "--"       # Parameter prefix
    "lens": "lens"      # Lens creation prefix
    # Memory marks
    "own": "&"          # Ownership transfer (default)
    "share": "share&"   # Reference-counted sharing
    "borrow": "borrow&" # Temporary access
    "copy": "copy&"     # Explicit copy
    "acquire": "acquire&" # Manual allocation
    "release": "release&" # Manual deallocation
    "pin": "pin&"       # Memory pinning
    "unpin": "unpin&"   # Memory unpinning
  }
  "suffix": {
    "nav": "."         # Navigation/field access
    "cascade": ".."    # Cascade (returns original object)
    "pair": ":"        # Key-value pair
    "tag": "::"        # Type tagging
    "stream": "~>"     # Stream transformation
    "channel": "<~>"   # Bidirectional channel
    "bind": "<-"       # Monadic bind
    "send": "->"       # Message send
    "lens_get": "@"    # Lens operation
  }
}
#string={"escaped":"\"\"" "raw":"``"}                     # String delimiters
#separator=[","]                                          # Separators (like whitespace)
#sequence=[";"]                                           # Sequence continuation
#declaration=["val" "var" "rec" "export" "async" "arena" "type"]  # Declaration keywords
#bitstring={"open":"<[" "close":"]>"}                     # Bit string delimiters
#section={"trigger":":" "materialize":"{}"}               # Section configuration
#structure={"trigger":"·" "materialize":"[]"}             # Structure configuration
#effect={"trigger":"!" "materialize":"{}"}                # Effect configuration
#bracket=["()" "[]" "{}" "⟅⟆" "<[]>"]                     # Recognized bracket pairs
#apply=["()"]                                             # Function application delimiters
#array=["[||]"]                                           # Array delimiters
#block=["{}"]                                             # Code block delimiters
#struct=["[]"]                                            # Struct delimiters
#binary=["<[]>"]                                          # Binary pattern delimiters
#infix=[
  "+"    # addition
  "-"    # subtraction
  "*"    # multiplication
  "/"    # division
  "=="   # equality
  "!="   # inequality
  "<="   # less than or equal
  ">="   # greater than or equal
  "<"    # less than
  ">"    # greater than
  "&&"   # logical and
  "||"   # logical or
  "in"   # membership test
  "is"   # type check
  "as"   # type cast
  "or"   # sum type / logical or
  "and"  # product type / logical and
  "has"  # constraint check
  "where" # guard condition
]
#associative={
  "left": [
    "="    # mutation
    "<-"   # monadic extraction
    "=>"   # lambda - params => body
    "|>"   # pipe first argument leftward
    "|"    # pipe last argument leftward
    ">>"   # pipe with placeholder _l
    "!>"   # async CPS transformation
    "&>"   # lazy evaluation
    "!!>"  # parallel execution
    "~>"   # stream transformation
    "<~>"  # bidirectional channel pipe
    "with" # context operations
    "handle" # handler operations
  ]
  "right": [
    "<|"   # pipe first argument rightward
    "|<"   # pipe last argument rightward
    "<<"   # pipe with placeholder _r
    "<~"   # channel receive
    "throws" # error propagation
    "raises" # exception flow
  ]
}
#sibling=[
  ["try" "catch" "finally"]
  ["if" "elseif" "else"]
  ["with_handler" "with_effect" "with_context"]
  ["match" "case" "default"]
  ["for" "while" "until" "async_for"]
]                                                          # Sibling keyword groups
```

**Evaluator conventions (NOT parser configuration):**
* Atoms starting with `*` may be varargs/handlers/properties - evaluator interprets
* The `*` prefix in `*rest`, `*get`, `*age`, `*handle`, `*perform` is just part of the atom name
* Parser sees these as regular atoms, evaluator assigns meaning
* Abstract marker `?` is just an atom to parser, evaluator interprets as incomplete field

**Note on synthesized lists:** When the parser creates lists during transformations, it uses appropriate delimiters based on context: transformations that create function applications use delimiters from `#apply`, while materialized sections use delimiters from `#section.materialize`, materialized structures use delimiters from `#structure.materialize`, and materialized effects use delimiters from `#effect.materialize`.

## §6 Character Processing

### 6.1 Shebang

If first two characters are `#!`, entire first line is treated as comment. Not configurable, applies regardless of directive/comment settings.

### 6.2 Comments

**Start:** `<C> ` where `<C>` is configured prefix followed by single space
**Range:** from start to newline (exclusive)
**Effect:** discarded before parsing

Comments do not break glue operations:
```glisp
x:: # comment
  Type
↦ (:: x Type)

&data # memory mark
  |> process
↦ (|> (& data) process)

data~> # stream
  transform
↦ (~> data transform)
```

### 6.3 Whitespace and Separators

* **Whitespace** — spaces, tabs, newlines separate datums
* **Tabs** — forbidden in leading indentation
* **Separators** — characters in `#separator` (default: `,`) act like whitespace, terminating values and constructs
* **Sequence markers** — characters in `#sequence` (default: `;`) act as statement boundaries, completing the current item and starting the next item at the same level in the parent section

### 6.4 Strings

Multiple string types with configurable delimiters:

**Escaped strings** — Process escape sequences (`\n`, `\t`, `\\`). Default: `"..."`

**Raw strings** — Literal content, escape delimiter by doubling. Default: `` `...` ``

**Binary strings** — Bit-level patterns. Default: `<[...]>`

**Prefixed strings** — Atom glued to string opening becomes processor:
```glisp
f"Hello ${name}"    ↦ (f "Hello ${name}")
sql`SELECT * FROM`  ↦ (sql "SELECT * FROM")
s"User: ${id}"      ↦ (s "User: ${id}")  # String interpolation (exemplar template)
regex"[a-z]+"       ↦ (regex "[a-z]+")
```

### 6.5 Directives

Form: `<D>datum` where `<D>` is directive prefix (default `#`)
Prefix glued to following datum. May alter parsing behavior by scope.

### 6.6 TOP

The source body (TOP) is a section. Follows standard section mechanics.

---

# Part III: Core Syntax

> **Note:** This part describes the syntactic patterns that emerge from the parser mechanics. The parser doesn't "understand" these constructs - they arise naturally from mechanical transformations that support all computational paradigms.

## §7 Sections

### 7.1 Principles

Sections are undelimited lists that the parser is always within:
* Parser starts inside the TOP section
* Sections collect items until boundaries (newlines, sequence markers)
* Undelimited items (sequences of atoms) are wrapped with `#apply` delimiters
* Already-delimited items (lists) remain as-is
* Can be inline (single line) or block (indented)
* Support all computational contexts (execution, memory, reactive)

### 7.2 Triggers

Section behavior after:
* Section trigger from `#section.trigger` (spaced) — child section (doesn't appear in AST)
* `;` — statement boundary at same level (doesn't appear in AST)
* `@` — meta section prefix
* Glue suffix transformers creates sections (`:`, etc.)
* Associative transformers (`=`, `=>`, `|>`, `<|`, `~>`, etc.) — when followed by newline or multiple elements

**Syntactic triggers:** The section trigger (configured as `:` by default) and `;` are purely syntactic markers. They control section grouping but do not appear in the resulting AST:

```glisp
# Section trigger creates child section
if condition : expr      ↦ (if condition {expr})

# ; continues current section
print x ; print y        ↦ {(print x) (print y)}

# Neither appear in AST, unlike transformers:
x = value                 ↦ (= x value)        # = appears
x => expr                 ↦ (=> x expr)        # => appears
data |> func             ↦ (|> data func)     # |> appears
data ~> transform        ↦ (~> data transform) # ~> appears
```

These syntactic triggers enable multi-statement and nested structures for all paradigms without adding wrapper functions to the AST.

### 7.3 Singleton Rule

Section with exactly one element:
* If element is ListDatum ↦ section collapses to that list
* If element is non-list ↦ wrapped in materialize delimiter

```glisp
if p : (print "ok")    ↦ (if p (print "ok"))      # Collapses
if p : 42              ↦ (if p {42})               # Wrapped
async val f: (fetch()) ↦ (: (async (val f)) (fetch))  # Collapses
```

### 7.4 Multi-Element Rule

Section with multiple elements materializes using `#section.materialize` delimiter:

```glisp
if p :
  print 1
  print 2
↦ (if p {(print 1) (print 2)})

async val operation: [data] =>
  val processed: process(data)
  save(processed)
↦ (: (async (val operation)) (=> [data] {(: (val processed) (process data)) (save processed)}))
```

### 7.5 Section Materialization

Sections materialize (get finalized) when the parser has nothing more to add to them:
* At EOF for TOP section
* At closing delimiter for explicit lists
* At next section trigger (`:`, `;`, associative transformer, etc.)
* At dedent for indented sections
* At memory context changes
* At reactive context shifts

Materialization applies singleton/multi-element rules and wraps in appropriate delimiters.

### 7.6 Section Type Tagging

When materializing a section, the parser mechanically applies these positional rules for `::Type` where `::` has no left operand (standalone tag transformer):

1. **Last item in section** ↦ tags the entire section
2. **Single item in section** ↦ section collapses to that item (per singleton rule)
3. **Any other position** ↦ remains as separate item

```glisp
# Last item - tags the section
[x] =>
  val doubled: x * 2
  doubled
  ::Int
↦ (=> [x] (:: {(: (val doubled) (* x 2)) doubled} Int))

# Works with all computational contexts
async val fetch_user: [id] =>
  val user: <- api.get(id)
  user
  ::User
↦ (: (async (val fetch_user)) (=> [id] (:: {(: (val user) (<- (. api get id))) user} User)))

# Memory context
arena temp(megabyte) :
  process()
  ::Result
↦ (arena temp megabyte (:: {process} Result))
```

### 7.7 Sibling Grouping

When the parser encounters the first keyword of a sibling group (from `#sibling`) followed by a section trigger, it collects subsequent siblings at the same indentation into a single form. This advanced mechanical rule **uses near-by short-range forces** to detect siblings at the same indentation level.

#### 7.7.1 Grouping Rules

* First keyword triggers collection
* Siblings must be at same indentation level
* Siblings must follow in sequence (non-siblings break the group)
* Each sibling can appear multiple times (e.g., multiple `elseif`, `catch`)
* Order preserved in AST
* **The parser uses local context** to detect sibling keywords at the same indentation level

#### 7.7.2 Examples

**If-elseif-else:**
```glisp
if condition :
  then-body
elseif other :
  elif-body
else :
  else-body

↦ (if {condition} {then-body}
    (elseif {other} {elif-body})
    (else {else-body}))
```

**Try-catch-finally:**
```glisp
try :
  body
catch e :
  handler
finally :
  cleanup

↦ (try {body}
    (catch e {handler})
    (finally {cleanup}))
```

**Handler context:**
```glisp
with_handler [*fetch: handle_fetch] :
  operation()
with_effect [*cache: use_cache] :
  fallback()

↦ (with_handler [(: *fetch handle_fetch)] {operation}
    (with_effect [(: *cache use_cache)] {fallback}))
```

### 7.8 Binding Context in Blocks

In block context, `:` creates bindings rather than pairs:

```glisp
{
  x: 10                          # Binding (immutable)
  var y: 20                      # Mutable binding
  result: x + y                  # Binding
  async val data: <- fetch()     # Async binding
  stream: clicks~>throttle(100)  # Stream binding
  &buffer: acquire&memory(1024)  # Memory binding
}
# Parser produces: {(: x 10) (: (var y) 20) (: result (+ x y)) 
#                   (: (async (val data)) (<- (fetch))) (: stream (~> clicks (throttle 100)))
#                   (: (& buffer) (acquire& (memory 1024)))}
# Evaluator interprets as bindings in all contexts
```

## §8 Structures

### 8.1 Principles

Structures are undelimited value collections parallel to sections:
* Parser enters structure mode via structure trigger `·` (spaced)
* Structures collect data items until boundaries
* Apply same singleton/materialization rules as sections
* Can be inline (single line) or block (indented)
* Support abstract fields with `?` markers

### 8.2 Triggers

Structure behavior after:
* Structure trigger from `#structure.trigger` (spaced) — child structure
* Any boundary that would close a section also closes a structure

### 8.3 Singleton Rule

Structure with exactly one element:
* If element is ListDatum ↦ structure collapses to that list
* If element is non-list ↦ wrapped with materialize delimiter

```glisp
point · [10 20]    ↦ (point [10 20])      # Collapses
value · 42         ↦ (value [42])          # Wrapped
```

### 8.4 Multi-Element Rule

Structure with multiple elements materializes using `#structure.materialize`:

```glisp
point · 10 20 30
↦ (point [10 20 30])

# With memory and async
async_config ·
  timeout: 5000
  retries: 3
  buffer: share&cache
↦ (async_config [(: timeout 5000) (: retries 3) (: buffer (share& cache))])
```

### 8.5 Nested Structures

```glisp
matrix ·
  row · 1 2 3
  row · 4 5 6
↦ (matrix [(row [1 2 3]) (row [4 5 6])])

# With computational contexts
reactive_system ·
  stream · clicks~>debounce(100)
  memory · &buffer
  state · count:0
↦ (reactive_system [(stream [(~> clicks (debounce 100))])
                     (memory [(& buffer)])
                     (state [(: count 0)])])
```

## §9 Abstract Structs

### 9.1 Core Concept

Abstract structs are structs with holes (undefined fields marked with `?`) that must be filled before instantiation. This unifies all forms of structural contracts through a single mechanism.

### 9.2 Abstract Fields

An abstract struct contains one or more fields marked with `?`, indicating required implementations:

```glisp
# Abstract struct with holes
Drawable: [
  draw: ?                   # Abstract - must be provided
  position: [x:0 y:0]       # Concrete - has default
  visible: true             # Concrete - has default
]

# Cannot instantiate abstract structs
d: Drawable[]               # ERROR: Cannot instantiate abstract struct
                            # Missing implementation for: draw
```

### 9.3 Spread Inheritance

The spread transformer `...` mechanically carries forward both concrete and abstract fields:

```glisp
# Spreading inherits all fields including holes
Sprite: [
  ...Drawable               # Inherits draw:?, position, visible
  image: nil
  # Still abstract - 'draw' not provided
]

# Completing the abstract struct
ConcreteSprite: [
  ...Sprite
  draw: [self ctx] =>       # Fills the hole
    if self.visible :
      ctx.render(self.image, self.position)
]

# Now can instantiate
s: ConcreteSprite[]         # OK - all holes filled
```

### 9.4 Progressive Concretization

Abstract structs support gradual refinement from abstract to concrete:

```glisp
# Highly abstract with handlers
Service: [
  start: ?
  stop: ?
  restart: ?
  *handle_request: ?        # Handler hole
  async_fetch: ?            # Async operation hole
]

# Partially concrete with some defaults
WebService: [
  ...Service
  port: 8080
  host: "localhost"
  restart: [self] => {      # Fills one hole
    self.stop()
    self.start()
  }
  # start, stop, *handle_request, async_fetch still abstract
]

# Fully concrete
HttpServer: [
  ...WebService
  start: [self] => bind_port(self.port)
  stop: [self] => close_port(self.port)
  *handle_request: [self req] => route_request(req)
  async_fetch: [self url] => async val result: <- http_get(url)
]
```

### 9.5 Structural Contracts as Abstract Structs

All contract types unify under abstract structs:

```glisp
# Comparable contract
Comparable: [
  (<): ?
  (==): ?
]

# Container contract  
Container: [
  len: ?
  get: ?
  set: ?
  iter: ?
]

# Storage contract
Storage: [
  get: ?
  set: ?
  delete: ?
  has: ?
]

# Enumerable with default implementations
Enumerable: [
  # Abstract - must be provided
  each: ?
  
  # Concrete - default implementations using abstract fields
  map: [self f] =>
    val result: []
    self.each([x] => result.append(f(x)))
    result
    
  count: [self] =>
    var n: 0
    self.each([_] => n = n + 1)
    n
]
```

## §10 Glue Transformers

Glue transformers attach to adjacent datums with no intervening space. GLISP supports both traditional and memory management glue patterns:

* **Prefix glue** — transformer before operand (configured in `#glue.prefix`)
* **Suffix glue** — transformer after operand (configured in `#glue.suffix`)

Error if EOF follows any glued transformer.

### 10.1 Prefix Glue Transformers

#### 10.1.1 Quote `'` (prefix)
* Forms: `'expr`
* Transform: `'expr` ↦ `(' expr)`
* Literal AST after transformations

#### 10.1.2 Quasiquote `''` (prefix)
* Forms: `''expr`
* Transform: `''expr` ↦ `('' expr)`
* AST template for macros

#### 10.1.3 Unquote `~` (prefix)
* Forms: `~expr`
* Transform: `~expr` ↦ `(~ expr)`
* Use within quasiquote

#### 10.1.4 Splice `~*` (prefix)
* Forms: `~*expr`
* Transform: `~*expr` ↦ `(~* expr)`
* Splice within quasiquote

#### 10.1.5 Meta `@` (prefix)
* Forms: `@expr`
* Opens inline meta section

#### 10.1.6 Spread `...` (prefix)
* Forms: `...expr`
* Transform: `...expr` ↦ `(... expr)`
* Spreads collections or imports (exemplar macro)

#### 10.1.7 Param `--` (prefix)
* Forms: `--expr`
* Transform: `--expr` ↦ `(-- expr)`
* Creates parameters

#### 10.1.8 Lens `lens` (prefix)
* Forms: `lens expr`
* Transform: `lens expr` ↦ `(lens expr)`
* Creates lens operations

#### 10.1.9 Memory Marks (prefix)
* `&expr` ↦ `(& expr)` — Ownership transfer
* `share&expr` ↦ `(share& expr)` — Reference counting
* `borrow&expr` ↦ `(borrow& expr)` — Temporary access
* `copy&expr` ↦ `(copy& expr)` — Explicit copy
* `acquire&expr` ↦ `(acquire& expr)` — Manual allocation
* `release&expr` ↦ `(release& expr)` — Manual deallocation
* `pin&expr` ↦ `(pin& expr)` — Memory pinning
* `unpin&expr` ↦ `(unpin& expr)` — Memory unpinning

### 10.2 Suffix Glue Transformers

#### 10.2.1 Tag `::` (LHS-glued)

* Forms: `expr::T`, `expr:: T`, `expr::\n  T`
* Right-associative: `a::T1::T2` ↦ `(:: a (:: T1 T2))`

#### 10.2.2 Pair `:` (LHS-glued)

Note: This is the glued pair transformer, different from the spaced section trigger configured in `#section.trigger`.

* Forms: `x:1`, `x: y`, `x:\n  y`
* Creates pair: `(: x y)`
* In block context, evaluator interprets as binding

#### 10.2.3 Navigation `.` (LHS-glued)

* Left-associative chains
* Returns result of access/call
```glisp
expr.field.method()  ↦ ((. (. expr field) method))
```

#### 10.2.4 Cascade `..` (LHS-glued)

* Left-associative chains
* Always returns original LHS object 

```glisp
widget..padding: 8..color: blue
↦ (.. (.. widget (: padding 8)) (: color blue))
# Returns widget after setting padding and color
```

#### 10.2.5 Stream `~>` (LHS-glued)

* Forms: `expr~>transform`
* Transform: `expr~>transform` ↦ `(~> expr transform)`
* Creates stream transformations

```glisp
data~>filter(pred)~>map(f)  ↦ (~> (~> data (filter pred)) (map f))
```

#### 10.2.6 Channel `<~>` (LHS-glued)

* Forms: `expr<~>other`
* Transform: `expr<~>other` ↦ `(<~> expr other)`
* Creates bidirectional channels

#### 10.2.7 Bind `<-` (LHS-glued)

* Forms: `expr<-monad`
* Transform: `expr<-monad` ↦ `(<- expr monad)`
* Monadic extraction/binding

#### 10.2.8 Send `->` (LHS-glued)

* Forms: `expr->target`
* Transform: `expr->target` ↦ `(-> expr target)`
* Message passing

#### 10.2.9 Lens Get `@` (LHS-glued)

* Forms: `lens@obj`
* Transform: `lens@obj` ↦ `(@ lens obj)`
* Lens access operation

### 10.3 Lispian Neutrality

If glue transformer appears as list head, it's a normal atom (no glue semantics):
```glisp
(:: x T)   ↦ (:: x T)    # No transformation
x::T       ↦ (:: x T)     # Glue transformation

(~> data f) ↦ (~> data f) # No transformation  
data~>f     ↦ (~> data f) # Glue transformation

(& data)    ↦ (& data)    # No transformation
&data       ↦ (& data)    # Glue transformation
```

## §11 Grafts

### 11.1 Head-Graft (HG) - Universal for All Brackets

Pattern: `atom[...]`, `atom{...}`, `atom(...)`, `atom⟅...⟆`, `atom<[...]>` 
Transform: Atom becomes head of following list, **preserving delimiter type**

```glisp
f[x]         ↦ (f [x])         # Preserves []
f{x}         ↦ (f {x})         # Preserves {}  
f(x)         ↦ (f (x))         # Preserves ()
f⟅x⟆         ↦ (f ⟅x⟆)         # Preserves ⟅⟆
f<[x:8]>     ↦ (f <[x:8]>)     # Preserves <[]>
```

All bracket pairs from `#bracket` configuration participate uniformly in head-graft.

**Applications across all paradigms:**
```glisp
# Function calls
function(args)    ↦ (function args)

# Constructors
Point[x:10 y:20]  ↦ (Point [x:10 y:20])

# Handlers
handler[handlers]  ↦ (handler [handlers])

# Async operations
async{body}       ↦ (async {body})

# Stream operations
source<[pattern]> ↦ (source <[pattern]>)

# Lens operations
lens[path]        ↦ (lens [path])
```

### 11.2 Tail-Graft (TG)

List opening glued to previous list appends to it:
```glisp
f[x]{y}     ↦ (f [x] {y})      # { is glued
handler[handlers]{body} ↦ (handler [handlers] {body})
async[config]{operation} ↦ (async [config] {operation})
```

## §12 Structs

### 12.1 Struct Recognition

Lists with delimiter `#struct` (default `[]`) are structs containing any mix of:
- Positional items
- Pairs `(: name value)`
- Parameters `(-- name)`
- Varargs (atoms starting with `*`)
- Abstract markers (`?`)
- Handlers (atoms starting with `*` in handler context)

```glisp
[name:"Alice" age:30]    # Struct with pairs
↦ [(: name "Alice") (: age 30)]

[--host --port]               # Struct with parameters
↦ [(-- host) (-- port)]

[x y *rest]                   # Struct with varargs
↦ [x y *rest]                 # *rest is just an atom

[draw:? bounds:[0 0]]         # Abstract struct
↦ [(: draw ?) (: bounds [0 0])]

[*handle:? *perform:?]        # Handler struct
↦ [(: *handle ?) (: *perform ?)]

[10 20 30]                    # Positional struct
↦ [10 20 30]
```

### 12.2 Parser Behavior

The parser mechanically identifies struct context by delimiter. All `[]` lists are structs for the evaluator to interpret with appropriate semantics for each computational context.

## §13 Infix Transformers

### 13.1 Configuration

Transformers listed in `#infix` are left-associative. Spacing required (whitespace on both sides).

### 13.2 Left-Fold Rule

All infix transformers fold left without precedence. This mechanical rule preserves evaluation order in the AST:

```glisp
a + b * c    ↦ (* (+ a b) c)
# Clear order: add first, then multiply

a has b where c  ↦ (where (has a b) c)
# Clear order: check has, then apply where condition
```

### 13.3 Lispian Neutrality

If infix transformer appears as list head, no transformation:
```glisp
(+ 1 2)      ↦ (+ 1 2)
(has obj field) ↦ (has obj field)
```

## §14 Associative Transformers

### 14.1 Configuration

Transformers listed in `#associative` fold based on their associativity and create sections. Spacing required (whitespace on both sides).

### 14.2 Section Behavior

Associative transformers create sections when followed by:
- Newline (indented content)
- Multiple elements before next transformer

Single inline elements do not trigger section creation:

**Inline (no sections):**
```glisp
data |> clean |> validate |> save
↦ (|> (|> (|> data clean) validate) save)

data ~> throttle ~> process ~> output
↦ (~> (~> (~> data throttle) process) output)
```

**With newlines (sections created):**
```glisp
data |>
  log("starting")
  validate
|> process
↦ (|> (|> data {(log "starting") validate}) process)

stream ~>
  debounce(100)
  filter(active)
~> update_ui
↦ (~> (~> stream {(debounce 100) (filter active)}) update_ui)
```

### 14.3 Lispian Neutrality

If transformer appears as list head, no transformation:
```glisp
(|> data f)  ↦ (|> data f)
(~> stream transform) ↦ (~> stream transform)
```

## §15 Mutation `=`

The `=` associative transformer is exclusively for mutation of existing mutable bindings:

```glisp
{
  var x: 10      # : introduces mutable binding
  x = 20         # = mutates existing binding
  
  y: 30          # : introduces immutable binding
  y = 40         # ERROR: cannot mutate immutable
  
  var stream: clicks~>throttle(100)
  stream = clicks~>debounce(200)  # Reassign stream
  
  var buffer: &initial_data
  buffer = &new_data  # Reassign ownership
}
```

## §16 Lambda Syntax

The `=>` associative transformer creates lambda expressions. It takes left operand as struct pattern and right operand as body:

```glisp
[x y] => x + y                      ↦ (=> [x y] (+ x y))
[x *rest] => process(x, rest)       ↦ (=> [x *rest] (process x rest))
```

**Note:** Function parameters MUST use struct syntax. Parentheses are not valid:
```glisp
[x y] => x + y    # ✓ Correct
(x y) => x + y    # ✗ ERROR: Invalid syntax
```

Like other associative transformers, `=>` creates sections when followed by newlines or multiple elements:

```glisp
# Lambda with async operations
[url] =>
  val response: <- async val fetch: [u] => <- http_get(u)
  val data: <- response.json()
  data
↦ (=> [url] {(: (val response) (<- (: (async (val fetch)) (=> [u] (<- (http_get u))))))
             (: (val data) (<- (. response json)))
             data})

# Lambda with memory management
[&data] =>
  val processed: process(&data)
  share&processed
↦ (=> [(& data)] {(: (val processed) (process (& data)))
                   (share& processed)})
```

## §17 Quoting

Quoting uses prefix glue from `#glue.prefix`:

* **Quote** (`'`) — literal AST after transformations
* **Quasiquote** (`''`) — AST template for macros
* **Unquote** (`~`) — unquote within quasiquote
* **Splice** (`~*`) — splice within quasiquote

Quasiquote demonstrates the structured macro mechanism, where AST nodes are manipulated directly while preserving all computational contexts.

## §18 Spread

The spread prefix glue `...` from `#glue.prefix` is the **exemplar macro** demonstrating synthetic AST generation with full tracing:

* **Form:** `...expr`
* **Transform:** `...expr` ↦ `(... expr)`
* **Usage:** Spreads collections or import bindings into scope

```glisp
...import("./utils")  ↦ (... (import "./utils"))
...[1 2 3]           ↦ (... [1 2 3])

# Spread generates synthetic AST bindings with full tracing
val config: [host:"localhost" port:8080 async_timeout:5000]
...config
# Synthetic bindings maintain complete trace:
# host ↦ spread-site ↦ config.host ↦ original-definition
# port ↦ spread-site ↦ config.port ↦ original-definition
# async_timeout ↦ spread-site ↦ config.async_timeout ↦ original-definition
```

Spread demonstrates how all macros must maintain complete source tracing through expansion. Each synthetic binding preserves the full chain from invocation site through all transformations to original source.

## §18.1 String Interpolation

String interpolation `s""` is the **exemplar template** demonstrating text-based code generation with full tracing:

```glisp
s"Hello ${name}, you are ${age} years old"
↦ (s "Hello ${name}, you are ${age} years old")

# Template processor for async operations
async_s"Fetch from ${url} with timeout ${timeout}"
↦ (async_s "Fetch from ${url} with timeout ${timeout}")
```

String interpolation demonstrates how templates (text-based) differ from macros (AST-based) while both maintain full provenance tracing across all computational contexts.

## §19 Varargs and Handlers

Atoms starting with `*` are recognized by the evaluator in different contexts:

### 19.1 Varargs (Function Parameters)
```glisp
# Function parameters
print: [msg *args] => ...     # *args is just an atom to parser

# Pattern matching  
match list :
  [] : "empty"
  [x *xs] : "has items"        # *xs is just an atom to parser
```

### 19.2 Handlers (Abstract Structs)
```glisp
# Handler definition using abstract struct
Handler: [
  *perform: ?                  # Abstract handler
  *handle: ?                   # Abstract handler
  state: []                    # Concrete field
]

# Handler implementation
LogHandler: [
  ...Handler
  *perform: [operation *args] => dispatch_operation(operation, args)
  *handle: [request] => log_and_continue(request)
]
```

### 19.3 Properties (Object Context)
```glisp
# Property definition
Person: [
  _age: 0
  *age: {                      # Property with getter/setter
    get: [self] => self._age
    set: [self value] => 
      if value >= 0 : self._age = value
      else : error("Age must be non-negative")
  }
]
```

The parser treats all `*` prefixed atoms as regular atoms. The evaluator interprets them based on context.

---

# Part IV: Declarations and Types

## §20 Declarations

### 20.1 Configuration

```glisp
#declaration=["val" "var" "rec" "type" "export" "async" "arena"]
```

* `val` — Immutable declaration keyword
* `var` — Mutable declaration keyword
* `rec` — Recursive declaration keyword
* `type` — Type declaration keyword
* `export` — Export declaration keyword
* `async` — Async declaration keyword
* `arena` — Arena memory context keyword

### 20.2 Syntax

Declarations use colon for binding introduction:

```glisp
val x: 5                           # Immutable value
var count: 0                       # Mutable value
val add: [x y] => x + y            # Immutable function
async val fetch_user: [id] =>      # Async function
  <- api.get(s"/users/${id}")
  
type Point: [x::Int y::Int]        # Type declaration

export val public: 10              # Exported immutable value

arena temp(megabyte) :             # Arena memory context
  process_with_temp_allocations()
```

### 20.3 Parsing

```glisp
val x: 10           ↦ (: (val x) 10)
var y: 20           ↦ (: (var y) 20)
val f: [a b] => a + b   ↦ (: (val f) (=> [a b] (+ a b)))

async val fetch: [url] => <- http_get(url)
↦ (: (async (val fetch)) (=> [url] (<- (http_get url))))

# Async sugar: async alone implies val
async fetch: [url] => <- http_get(url)
↦ (: (async (val fetch)) (=> [url] (<- (http_get url))))

export val z: 30    ↦ (: (export (val z)) 30)
export [x y z]      ↦ (export [x y z])

arena frame(size) : body
↦ (arena frame size {body})
```

### 20.4 Declaration Ordering

Declaration keywords have a specific ordering when combined:

1. `export` (if present)
2. `async` (if present)
3. Core declaration (`val`, `var`, `rec`, `type`)

```glisp
export async val fetch: [url] => ...
↦ (: (export (async (val fetch))) ...)

# Invalid orderings:
async export val fetch: ...  # ERROR: export must come first
val async fetch: ...         # ERROR: async must come before val
```

### 20.5 No Special Context

Declaration keywords don't disable head-graft or create special parsing contexts:

```glisp
val f: [x y] => x + y  # Parses as (: (val f) (=> [x y] (+ x y)))

# IMPORTANT: Head-graft still applies with declarations
val f[x]: x + 1     # ERROR: Parses as (: (val f [x]) (+ x 1))
                     # f grafted with [x] - not intended!

val f: [x] => x + 1  # Correct: explicit lambda

# Same applies to all declaration types
async fetch[url]: ...  # ERROR: Head-graft applies
async val fetch: [url] => ... # Correct: explicit declaration
```

### 20.6 Typed Bindings

Type tags can be combined with all declaration types:

```glisp
val x::Int: 10                    # Typed immutable
↦ (: (:: (val x) Int) 10)

var count::Int: 0                 # Typed mutable
↦ (: (:: (var count) Int) 0)

async val fetch::Url => Response: [url] => <- http_get(url)
↦ (: (:: (async (val fetch)) (=> Url Response)) (=> [url] (<- (http_get url))))
```

## §21 Abstract Structs as Type Foundation

### 21.1 Core Principle

Types in GLISP are abstract structs - incomplete structural templates that must be completed before instantiation. This unifies all type system features under a single mechanism.

### 21.2 Type Templates

```glisp
# Type as abstract struct template
Point: [x::Float:0 y::Float:0]        # Template with defaults and constraints

# Using types to create instances via head-graft
p: Point[x:10 y:20]               # Constructor syntax
↦ (Point [x:10 y:20])

# Abstract type requiring implementation
Shape: [
  area: ?                         # Must be implemented
  perimeter: ?                    # Must be implemented
  center: [0 0]                   # Has default
]

# Concrete implementation
Circle: [
  ...Shape
  radius: 0
  area: [self] => π * self.radius^2
  perimeter: [self] => 2 * π * self.radius
]
```

### 21.3 Handler Types

```glisp
# Handler as abstract struct
IOHandler: [
  *read: ?                        # Abstract read handler
  *write: ?                       # Abstract write handler
  buffer_size: 4096               # Concrete configuration
]

# Concrete handler implementation
FileIO: [
  ...IOHandler
  *read: [file] => read_file_contents(file)
  *write: [file data] => write_file_contents(file, data)
]

# Handler type constraints
process_files: [files::IO where IO has *read and IO has *write] => ...
```

### 21.4 Reactive Types

```glisp
# Stream as abstract struct
Stream: [
  subscribe: ?                    # Abstract subscription
  transform: ?                    # Abstract transformation
  buffer: []                      # Concrete buffer
]

# Concrete stream implementation
ClickStream: [
  ...Stream
  subscribe: [self listener] => add_click_listener(listener)
  transform: [self f] => ClickStream[
    subscribe: [listener] => self.subscribe([event] => listener(f(event)))
  ]
]
```

### 21.5 Async Types

```glisp
# Future as abstract struct
Future: [
  then: ?                         # Abstract continuation
  catch: ?                        # Abstract error handler
  state: "pending"                # Concrete state
]

# Promise implementation
Promise: [
  ...Future
  then: [self callback] => schedule_continuation(self, callback)
  catch: [self handler] => schedule_error_handler(self, handler)
]
```

## §22 Pattern Matching with All Types

### 22.1 Syntax

Pattern matching uses sections and supports all type patterns:
```glisp
match expr :
  Some[x] : process(x)
  None : default
  
# With guards on all types
match stream :
  ClickStream[active:true] where stream.subscribers > 0 : process_clicks
  Stream[buffer:buf] where buf.length > 100 : flush_buffer
  * : default_handling

# With handler patterns
match operation :
  IOHandler[*read:r *write:w] : setup_io(r, w)
  LogHandler[*log:l] : setup_logging(l)
  * : error("Unknown handler")

# With async patterns
match future_result :
  Promise[state:"resolved" value:v] : use_value(v)
  Promise[state:"rejected" error:e] : handle_error(e)
  Future[state:"pending"] : wait_more()
```

### 22.2 Pattern Guards

Guards extend pattern matching with conditional logic:
```glisp
# Guards on patterns
match value :
  [x y] where x > y : "descending"
  [x y] where x < y : "ascending"
  [x y] : "equal"
  
  # Guards on constructors with all types
  Some[x] where x > 0 : "positive"
  Some[x] where x < 0 : "negative"  
  Some[0] : "zero"
  None : "empty"
  
  # Guards with handlers
  IOResult[*success:s] where s.bytes > 1000 : "large_read"
  IOResult[*error:e] where e.recoverable : retry_operation
  
  # Guards with async
  AsyncResult[future:f] where f.timeout < 1000 : fast_result
  AsyncResult[future:f] where f.retries < 3 : retry_async
```

### 22.3 Binary Pattern Matching

Pattern matching extends to bit-level patterns:

```glisp
# Match on bit patterns
match packet :
  <[0x45 len:16 rest:binary]> : 
    parse_ipv4(len, rest)
    
  <[0x60 flow:20 len:16 rest:binary]> : 
    parse_ipv6(flow, len, rest)
    
  <[flags:8 seq:32 ack:32 data:binary]> where flags has 0x02 : 
    parse_tcp_syn(seq, ack, data)

# Binary construction
make_header: [version ihl tos length] =>
  <[version:4, ihl:4, tos:8, length:16]>

# With handlers in binary context
match network_operation :
  IOHandler[*packet:<[version:4 rest:binary]>] where version == 4 :
    handle_ipv4_io
```

---

# Part V: Advanced Evaluation

## §23 Handler System

### 23.1 Core Concepts

GLISP's handler system provides algebraic handlers that integrate seamlessly with all other language features. Handlers are first-class values that can be composed, passed as arguments, and handled dynamically.

### 23.2 Handler Definition

Handlers are defined as abstract structs with handler holes:

```glisp
# Core handler definition using abstract struct
Handler: [
  *perform: ?                   # Abstract performance handler
  name: ""                      # Handler name
  state: nil                    # Handler state
]

# Specific handlers
Logger: [
  ...Handler
  *perform: [operation args] =>
    match operation :
      "log" : append_to_log(args.0)
      "error" : log_error(args.0)
      * : error(s"Unknown log operation: ${operation}")
]

State: [
  ...Handler
  current_value: nil
  *perform: [operation args] =>
    match operation :
      "get" : self.current_value
      "set" : self.current_value = args.0
      * : error(s"Unknown state operation: ${operation}")
]
```

### 23.3 Handler Installation

Handler installation uses the `with_handler` structure:

```glisp
# Installing handlers
with_handler [
  *log: [msg] => append_to_buffer(msg)
  *read: [file] => cached_read(file)
] :
  process()  # All operations captured and handled

# Compose multiple handlers
with_handler Logger[buffer: []] :
  with_handler State[value: 0] :
    computation()  # Both handlers available
```

### 23.4 Handler Operations

Handlers are invoked using the `perform` function:

```glisp
# Basic handler performance
perform("log", "Starting operation")
perform("set", new_value)
val current: perform("get")

# Handler pipelines
data !> log_handler !> validate_handler !> save_handler

# Async handlers
async val result: <- perform("async_fetch", url)
```

### 23.5 Handler Composition

Handlers compose naturally through handler chaining:

```glisp
# Layered handler handling
with_handler network_handlers :
  with_handler logging_handlers :
    with_handler state_handlers :
      complex_operation()

# Handler combination
combined_handler: [
  *fetch: [url] => with_retry(3, [] => http_get(url))
  *cache: [key value] => redis_set(key, value, ttl:3600)
  *log: [level msg] => structured_log(level, msg, context)
]
```

## §24 Reactive Streams

### 24.1 Core Concepts

GLISP's reactive streams extend the channel system with time-based transformations and event-driven programming patterns.

### 24.2 Stream Creation

Streams are created using stream constructors:

```glisp
# Create reactive streams
val counter: stream(0)
val clicks: stream()
val timer: interval(1000)  # Emits every second

# Derived streams from transformations
val doubled: counter~>map([x] => x * 2)
val filtered: clicks~>filter([e] => e.button == 'left)
val combined: merge(stream1, stream2)
```

### 24.3 Stream Transformations

Stream operations use the `~>` transformer:

```glisp
# Reactive pipelines
clicks ~>
  throttle(300) ~>
  map([e] => e.position) ~>
  distinctUntilChanged() ~>
  sink([pos] => updateUI(pos))

# Combining streams with handlers
input_stream ~>
  perform("validate") ~>
  perform("log") ~>
  map(process_data) ~>
  output_stream

# Async stream operations
data_stream ~>
  async_map([item] => <- fetch_details(item)) ~>
  buffer(10) ~>
  batch_process
```

### 24.4 Stream Composition

Streams compose through multiple patterns:

```glisp
# Stream merging
user_actions: merge([
  keyboard_events~>map(parse_key),
  mouse_events~>map(parse_mouse),
  network_events~>map(parse_network)
])

# Stream splitting
[valid_data, invalid_data]: data_stream~>partition(is_valid)

# Stream joining
combined: join([
  temperature_stream,
  humidity_stream,
  pressure_stream
], weather_combiner)
```

### 24.5 Reactive Bindings

Automatic dependency tracking for reactive updates:

```glisp
# Reactive values
val a: reactive(10)
val b: reactive(20)
val sum: computed([] => a.value + b.value)

a.value = 15  # sum automatically updates to 35

# Two-way bindings with lenses
val model: reactive([name:"" age:0])
bind(input_field, model@lens(.name))  # Updates flow both ways

# Stream-reactive integration
user_input~>debounce(300)~>bind_to(model@lens(.search_term))
```

## §25 Asynchronous Operations

### 25.1 Async Functions

Async operations are marked with the `async` keyword and use monadic extraction:

```glisp
# Async function definition
async val fetch_user_data: [id] =>
  val user: <- fetch_user(id)
  val posts: <- fetch_posts(user.id)
  val friends: <- fetch_friends(user.id)
  [user:user posts:posts friends:friends]

# Async operations in pipelines
data !> async_validate !> async_process !> async_save
```

### 25.2 Async Control Flow

Async operations support all control structures:

```glisp
# Async error handling
async val safe_fetch: [url] =>
  try :
    val response: <- fetch(url)
    <- response.json()
  catch e :
    log_error(e)
    default_value

# Async iteration
async val process_stream: [stream] =>
  for await [item <- stream] :
    <- process_item(item)

# Async handlers
async with_handler [*db: async_db_handler] :
  val users: <- perform("query", "SELECT * FROM users")
  for [user <- users] :
    <- perform("update", user.id, process_user(user))
```

### 25.3 Parallel Execution

Parallel operations use the `!!>` transformer:

```glisp
# Parallel async operations
async val parallel_fetch: [urls] =>
  val futures: urls~>map([url] => async val result: <- fetch(url))
  <- await_all(futures)

# Parallel pipeline
data !!> transform1 !!> transform2 !!> combine_results

# Parallel with handlers
operations !!>
  perform("log_start") !!>
  async_process !!>
  perform("log_complete")
```

## §26 Memory Management

### 26.1 Core Concept

Memory marks provide ownership and resource management integrated with all language features:

```glisp
# Ownership transfer
&data |> process |> &result

# Reference counting
share&expensive_obj |> multiple_consumers

# Temporary borrowing
borrow&resource |> read_only_operation

# Manual management
acquire&buffer(1024) |> process |> release&buffer
```

### 26.2 Arena Memory Context

Arena provides scoped memory allocation:

```glisp
# Arena for temporary allocations
arena frame_memory(megabyte * 10) :
  temp_data: alloc(size)      # Arena allocation
  result: process(temp_data)
  copy&result                  # Must copy to escape arena

# Arena with handlers
arena request_arena(kilobyte * 500) :
  with_handler [*alloc: arena_alloc] :
    process_request()
```

### 26.3 Memory Operations with Other Features

```glisp
# Memory with async
async val fetch_and_process: [&url] =>
  val response: <- fetch(&url)
  val data: <- response.json()
  share&processed <- expensive_async_transform(data)
  
  # Parallel operations sharing data
  async parallel [
    cache.store("key", share&processed)
    analytics.track(borrow&processed)
    notifications.send(copy&processed)
  ]
  
  &processed  # Return ownership

# Memory with streams
data_stream~>
  validate~>
  &validated~>              # Zero-copy through stream
  expensive_transform~>
  share&result~>            # Share result across stream branches
  [data] => {
    cache_branch: share&data~>cache.store
    ui_branch: borrow&data~>ui.update
    log_branch: copy&data~>analytics.log
    data  # Original continues down main stream
  }
```

## §27 Transducers

### 27.1 Core Concept

Transducers enable efficient transformation composition without intermediate collections, processing data in a single pass.

### 27.2 Basic Usage

```glisp
# Define a transducer pipeline
transduce: mapping(f) >> filtering(p) >> taking(10)

# Apply with |>> transformer
data |>> transduce              # Single pass, no intermediates

# Compare with regular pipeline
data |> map(f) |> filter(p) |> take(10)  # Creates intermediate collections
```

### 27.3 Transducer Definition

Transducers are reducing function transformers:

```glisp
# Core transducer patterns
mapping: [f] => [rf] => [acc x] => rf(acc, f(x))
filtering: [p] => [rf] => [acc x] => if p(x) : rf(acc, x) else : acc
taking: [n] => [rf] => 
  var count: 0
  [acc x] => if count < n : { count = count + 1; rf(acc, x) } else : acc

# Composition via >>
[t1 >> t2] => [rf] => t1(t2(rf))

# Async transducers
async_mapping: [f] => [rf] => [acc x] => <- rf(acc, <- f(x))

# Handler transducers
handler_filtering: [handler_pred] => [rf] => [acc x] =>
  if perform(handler_pred, x) : rf(acc, x) else : acc
```

### 27.4 Complex Transformations

```glisp
# Stateful transducers
dedupe: [] => [rf] =>
  var last: nil
  [acc x] => 
    if x != last : { last = x; rf(acc, x) }
    else : acc

# Applied to any reducible collection
numbers |>> 
  mapping([x] => x * 2) >> 
  dedupe() >> 
  taking(5) >>
  handler_logging("processed")
```

## §28 Lenses and Optics

### 28.1 Core Concept

Lenses provide composable, immutable access and updates to nested struct fields.

### 28.2 Lens Creation and Usage

```glisp
# Create lenses via lens constructor
addressLens: lens(.address)
streetLens: lens(.street)
numberLens: lens(.number)

# Compose lenses
fullLens: addressLens >> streetLens >> numberLens

# Lens operations using @ suffix transformer
view: [lens obj] => lens@obj          # Get value
set: [lens val obj] => lens@obj = val # Set value  
over: [lens f obj] => lens@obj = f(lens@obj) # Modify value
```

### 28.3 Practical Examples

```glisp
# Deep updates without mutation
person
  |> set(lens(.address.city), "NYC")
  |> over(lens(.age), inc)
  |> over(lens(.address.street.number), [n] => n + 100)

# Lens-based traversals with handlers
users~>
  map(over(lens(.address.country), normalize_country))~>
  filter([u] => lens(.age)@u >= 18)~>
  perform("log_processed")
```

### 28.4 Reactive Lenses

Lenses integrate with reactive streams:

```glisp
# Reactive lens bindings
val model: reactive([user:[name:"" profile:[age:0]]])
val name_lens: lens(.user.name)
val age_lens: lens(.user.profile.age)

# Two-way reactive binding
name_input~>bind_to(model, name_lens)
age_input~>bind_to(model, age_lens)

# React to deep changes
model~>
  changes_at(name_lens)~>
  debounce(300)~>
  async_validate_name
```

## §29 Contracts and Refinements

### 29.1 Core Concept

Contracts extend GLISP's type system with runtime-checked refinements, bridging static types and dynamic validation.

### 29.2 Basic Contracts

```glisp
# Contracts as type refinements
sqrt: [x::Float where x >= 0] => ::Float where result >= 0

# Multi-condition contracts
divide: [x::Number y::Number where y != 0] => ::Number

# Dependent contracts
resize: [array size::Int where size > 0] => ::Array where result.len == size

# Async contracts
async val fetch_user: [id::String where id.length > 0] => ::User where user.id == id
```

### 29.3 Contract Tags

Contracts can be specified through meta sections:

```glisp
# Via meta sections
@contract(
  pre: [x] => x >= 0
  post: [x result] => result * result ≈ x
  invariant: [self] => self.balance >= 0
)
sqrt: [x] => ...

# Contract inheritance
@contract(extends: Comparable)
SortedList: [...]

# Handler contracts
@contract(
  handlers: [*read *write]
  pre: [file] => file_exists(file)
  post: [file result] => result.size == file_size(file)
)
async val process_file: [file] => ...
```

### 29.4 Contract Enforcement

```glisp
# Development mode - full checking
val result: sqrt(-1)  # CONTRACT VIOLATION: Precondition failed: x >= 0

# Production mode - contracts compiled out for performance
@compile_mode(production)
val result: sqrt(x)  # No runtime check

# Handler contract checking
async with_handler [*file: checked_file_handler] :
  <- process_file("data.txt")  # Contracts enforced through handlers
```

## §30 Comprehensions

### 30.1 Core Concept

Comprehensions provide expressive syntax for building collections through macro expansion.

### 30.2 List Comprehensions

```glisp
# Basic list comprehension
for [x <- range(10)] : x * 2
# => [0 2 4 6 8 10 12 14 16 18]

# Multiple generators
for [
  x <- range(3)
  y <- range(3)
] : [x, y]

# With filtering and handlers
for [
  x <- range(10)
  where: x % 2 == 0
  perform: perform("log", s"Processing ${x}")
] : x * x
```

### 30.3 Async Comprehensions

```glisp
# Async comprehensions
async for [url <- urls] : <- fetch(url)

# Parallel async comprehensions
async for parallel [
  user <- users
  where: user.active
] : <- fetch_user_details(user.id)

# Streaming comprehensions
for stream [
  event <- event_stream
  where: event.type == "click"
] : process_click(event)
```

### 30.4 Handler Comprehensions

```glisp
# Comprehensions with handlers
for [
  file <- files
  perform: perform("validate", file)
  where: perform("check_access", file)
] : process_file(file)

# Reactive comprehensions
for reactive [
  item <- reactive_list
  where: item.value > threshold.value
] : item.transform()
```

## §31 Pattern Guards and Advanced Matching

### 31.1 Guard Syntax

Pattern matching extends with guard conditions for precise control:

```glisp
match value :
  # Patterns with where guards
  [x y] where x > y : "descending"
  [x y] where x < y : "ascending"
  [x y] : "equal"
  
  # Guards on constructors with all computational contexts
  Some[x] where x > 0 : "positive"
  Some[x] where x < 0 : "negative"
  Some[0] : "zero"
  None : "empty"
  
  # Guards with handlers
  IOResult[*success:s] where s.bytes > 1000 : "large_operation"
  IOResult[*error:e] where e.recoverable : retry_with_handler
  
  # Guards with async patterns
  AsyncResult[future:f] where f.completed : extract_value(f)
  AsyncResult[future:f] where f.timeout > 5000 : cancel_operation(f)
```

### 31.2 Complex Guards

```glisp
# Guards can reference bound variables across contexts
match complex_operation :
  # Traditional pattern with guard
  Node[left:l right:r] where height(l) > height(r) + 1 : 
    rebalance_left(tree)
  
  # Handler pattern with guard
  Handler[*perform:p *handle:h] where p.arity == h.arity :
    link_handlers(p, h)
  
  # Stream pattern with guard
  StreamProcessor[source:s transform:t] where s.rate > t.capacity :
    add_backpressure(s, t)
  
  # Async pattern with guard
  AsyncOperation[future:f] where f.dependencies.all(completed) :
    execute_async(f)
```

## §32 Computed Struct Fields

### 32.1 Dynamic Field Names

Structs support computed field names and conditional inclusion:

```glisp
# Computed field names in struct literals
val key: "dynamic"
val obj: [
  (key): "value"                # Field name from variable
  (compute_name()): 42          # Field name from expression
  ...existing_struct            # Spread existing fields
]

# With handlers and async
async val dynamic_config: [
  (await get_env("CONFIG_KEY")): await fetch_config_value()
  ...base_config
]

# Build structs programmatically
make_struct: [fields] =>
  fields~>reduce([acc [k v]] => [...acc (k):v], [])
```

### 32.2 Conditional Fields

```glisp
# Include fields conditionally
[
  name: "Alice"
  if include_age : age: 30
  if is_admin : role: "admin"
  ...optional_fields
  if async_mode : async_config: await fetch_async_config()
]

# Pattern for optional fields with handlers
maybe_field: [condition name value] =>
  if condition : [(name):value] else : []

[
  ...maybe_field(include_email, "email", user.email)
  ...maybe_field(include_phone, "phone", user.phone)
  ...maybe_field(has_handlers, "handlers", handler_list)
]
```

## §33 Tail Call Optimization

### 33.1 Basic TCO

GLISP provides explicit tail call optimization through the `recur` marker:

```glisp
# recur guarantees tail call optimization
factorial: [n acc:1] =>
  if n <= 1 : acc
  else : recur(n - 1, n * acc)

# Sum with accumulator
sum: [list acc:0] =>
  match list :
    [] : acc
    [h *t] : recur(t, acc + h)

# Async tail recursion
async val process_stream: [stream acc:[]] =>
  match <- stream.next() :
    Done[result] : acc
    Item[value] : 
      val processed: <- process_item(value)
      recur(stream, [...acc processed])
```

### 33.2 Mutual Recursion

```glisp
# recur_to for mutual tail recursion
is_even: [n] =>
  if n == 0 : true
  else : recur_to(is_odd, n - 1)

is_odd: [n] =>
  if n == 0 : false
  else : recur_to(is_even, n - 1)

# State machines via mutual recursion with handlers
state_a: [input] =>
  perform("log", "In state A")
  match input :
    0 : recur_to(state_b, read())
    1 : recur_to(state_c, read())
    _ : 'rejected

# Async mutual recursion
async val producer: [queue] =>
  val item: <- generate_item()
  queue~>push(item)
  recur_to(consumer, queue)

async val consumer: [queue] =>
  val item: <- queue~>pop()
  <- process_item(item)
  recur_to(producer, queue)
```

---

# Part VI: Meta-Programming and Modules

## §34 Macro System

GLISP provides two forms of code generation:
- **Macros** (AST-based) - exemplified by spread `...`
- **Templates** (text-based) - exemplified by string interpolation `s""`

Both maintain full source tracing through all synthetic code generation and work across all computational contexts.

### 34.1 Macros (AST Manipulation)

Macros operate on AST before evaluation:

```glisp
macro when: [cond body] =>
  ''(if ~cond : ~body else : [])

when [x > 0] : print("positive")
↦ (if (> x 0) : {(print "positive")} else : {[]})

# Async macros
macro async_when: [cond async_body] =>
  ''(if ~cond : async ~async_body else : async [])

# Handler macros  
macro with_logging: [body] =>
  ''(with_handler [*log: console_log] : ~body)

# Stream macros
macro debounced: [ms body] =>
  ''(stream_input~>debounce(~ms)~>map([_] => ~body))
```

**Spread as exemplar macro:**
```glisp
...data  # Operates on AST, generates synthetic bindings
# Each binding maintains full trace: spread-site ↦ AST-transform ↦ original
```

### 34.2 Template System

Templates process text into AST with context awareness:

```glisp
# String interpolation as exemplar template
s"Hello ${name}"  
# Parses text, expands ${expr}, maintains traces

# Context-aware templates
async_s"Fetch from ${url} with timeout ${timeout}"
stream_s"Process ${event_type} events"
```

String prefixes invoke template processors:
- Parser: `s"text"` ↦ `(s "text")`
- Evaluator: recognizes `s` as template processor

### 34.3 Synthetic Code Tracing

**All macros and templates MUST maintain complete source tracing through expansion.**

Components of trace chain:
1. **Invocation site** - Where macro/template was called
2. **Expansion transforms** - Intermediate processing steps  
3. **Original source** - Ultimate definition origins

**Macro tracing (AST to AST):**
```glisp
val core: [x:10 async_config:[timeout:5000]]
val extended: [...core y:20]
# extended.x traces: spread@line2 ↦ AST(core.x) ↦ definition@line1
# extended.async_config traces: spread@line2 ↦ AST(core.async_config) ↦ definition@line1
```

**Template tracing (text to AST):**
```glisp
val name: "Alice"
val config: [async_mode: true]
s"Hello ${name}, async: ${config.async_mode}"
# ${name} traces: template@line3 ↦ parse(name) ↦ binding@line1
# ${config.async_mode} traces: template@line3 ↦ parse(config.async_mode) ↦ binding@line2
```

### 34.4 Scope and Macro Responsibility

**Blocks create scope** - This is mechanical fact enforced by evaluator.

Macros are **free to manipulate AST** but **responsible for scope consistency**:

```glisp
# Macro receives AST and environment context
macro let: [bindings body] =>
  ''({~bindings ~body})  # Creates block scope

# Different strategies for different problems:

# Strategy A: Isolated scope
macro transaction: [body] =>
  ''({
    val __tx: begin_transaction()
    try : ~body
    finally : __tx.close()
  })

# Strategy B: Current scope injection  
macro debug: [expr] =>
  ''(do
    print(s"Debug: ${~expr}")
    ~expr)

# Strategy C: Deliberate capture
macro with_timing: [name body] =>
  ''({
    val start: now()
    ~body
    val ~name: now() - start
  })

# Strategy D: Context-aware expansion
macro async_let: [bindings body] =>
  ''(async {~bindings <- ~body})
```

**No "Hygiene" Morality:**

GLISP explicitly rejects the traditional "macro hygiene" debate. There is no moral superiority in macro design - no "clean" or "dirty" approaches, no "hygienic" or "unhygienic" distinctions. These terms introduce unnecessary moral judgment into engineering decisions.

The language provides guardrails:
- Blocks mechanically create scopes
- Macros receive environment context
- Name generation utilities available
- Full tracing maintains provenance

These are **tools**, not judgments. Different problems need different solutions.

## §35 Module and Resource System

### 35.1 Core Concepts

* **Module** — A project or package with identity
* **Resource** — Any importable unit (file, URL, module)
* **Export boundary** — Explicit declaration of public bindings

### 35.2 Import and Include

Resources can be imported as values or included into scope:

```glisp
# import - returns resource as value binding
val math: import("math_utils")
math.sin(x)

# include - spreads exported bindings into current scope
include("math_utils")
sin(x)  # Directly available

# Async imports
async val remote_config: <- import("https://config.service.com/app.json")

# Handler-aware imports
with_handler [*io: secure_io] :
  val secure_data: import("encrypted://data.enc")

# Stream imports
val live_data: import_stream("ws://live.data.feed")
```

### 35.3 Export Mechanism

Resources explicitly declare exports:

```glisp
# Internal implementation (not exported)
val _helper: [x] => x * 2
val _internal_logger: [*log: console.log]

# Explicit exports across all paradigms
export val process: [data] =>
  _helper(data.value)

export async val fetch_and_process: [url] =>
  val data: <- fetch(url)
  process(data)

export val Logger: [*log: ? level:"info"]

export val data_stream: input~>throttle(100)~>process

# Bulk export
export [process fetch_and_process Logger data_stream map filter]
```

### 35.4 Module Composition

```glisp
# Compose modules with all computational patterns
export val web_service: [
  ...http_server
  ...database_client
  ...logging_system
  ...stream_processor
  
  # Override specific behaviors
  *handle_request: [req] => 
    async with_handler [*db: database_client.*query] :
      <- process_request(req)
      
  start: [self] =>
    async val result: {
      logging_system.start()
      stream_processor.start()
      database_client.connect()
      http_server.listen(self.port)
    }
]
```

---

# Part VII: Surface Style Philosophy

## §36 The Natural Progression

The sequence of surface styles follows the natural order of computational thinking and GLISP's philosophy of fluidity, enhanced by the full spectrum of computational patterns.

### 36.1 Foundational Progression

The first styles represent stages of computational maturity:

1. **Command-line** — Direct action, minimal syntax friction
   ```glisp
   fetch user
   validate permissions
   save database
   ```
   Writing is fluid: thought ↦ action with no ceremony. This is how we sketch solutions.

2. **Functional-first (FFP)** — Data flow emerges from command sequences
   ```glisp
   user_id |> fetch_user |> validate_permissions |> grant_access
   ```
   Natural evolution: commands reveal their data flow, pipes make it explicit.

3. **Data-oriented (DOP)** — Structure becomes necessary
   ```glisp
   [id:42 name:"Alice" permissions:[read write]]
   ```
   Transparent data shapes that flow through transformations.

4. **Object-oriented (OOP)** — Encapsulation when boundaries matter
   ```glisp
   account.withdraw(50)  # When invariants need protection
   ```
   Complexity introduced only when necessary.

### 36.2 Advanced Computational Patterns

The progression continues with sophisticated computational needs:

5. **Memory-aware** — Managing computational resources explicitly
   ```glisp
   &data |> process |> &result  # Zero-copy ownership transfer
   share&expensive |> consumers  # Reference-counted sharing
   ```
   When performance and resource management matter.

6. **Reactive** — Time-based and event-driven computation
   ```glisp
   user_clicks~>debounce(300)~>validate~>update_ui
   ```
   When systems need to respond to streams of events over time.

7. **Async-native** — Concurrent and parallel computation
   ```glisp
   async parallel [url <- urls] : <- fetch_and_process(url)
   ```
   When operations must coordinate across time and threads.

### 36.3 Familiarity Bridges

The remaining styles welcome different programming communities while preserving the same mechanical core:
- **ML-like** — Pattern matching and type inference culture
- **Python-like** — Indentation and comprehension culture  
- **Nim-like** — UFCS and macro culture
- **C-like** — Braces and semicolon culture
- **Lisp-like** — The underlying reality revealed

### 36.4 Fluidity Principle

GLISP follows nature's compositional hierarchy - simple things compose better until complexity is necessary. The progression enables:
- **Writing fluidity** — Start simple, add precision without restructuring
- **Cognitive fluidity** — Each stage builds on previous understanding
- **Refactoring fluidity** — Transition between styles without rewriting
- **Computational fluidity** — Same syntax for all computational contexts

The parser doesn't punish starting simple. There's no migration cost from prototype to production - only gradual precision refinement.

## §37 The Fundamental Unification

GLISP recognizes a profound truth: **everything is struct transformation** - functions receive input structs and produce output structs, across all computational contexts.

### 37.1 Core Principle

Everything in GLISP reduces to structured data undergoing transformation:
- **Values** are structs (may be optimized to scalars)
- **Functions** are struct ↦ struct transformers
- **Types** are struct templates with defaults/constraints
- **Handlers** are struct transformers of execution context
- **Streams** are struct transformers over time
- **Async operations** are struct transformers across time

```glisp
# A struct template
Point: [x:0 y:0]

# Using as constructor
p: Point[x:10 y:20]

# Function as struct transformer
distance: [p q] => sqrt((p.x - q.x)^2 + (p.y - q.y)^2)

# Handler as context transformer
Logger: [*log: ? level:"info"]

# Stream as time-ordered transformer
ClickStream: [subscribe:? transform:?]

# Async as time-distributed transformer
async val fetch_user: [id] => <- api.get(s"/users/${id}")
```

### 37.2 Struct Model

All structures are:
1. **Positions** (0, 1, 2, ...) - the universal substrate
2. **Names** (optional aliases to positions)
3. **Values** (content at positions, may be abstract with `?`)
4. **Context** (execution, memory, reactive, async)

```glisp
[10 20 30]           # Pure positional
[x:10 y:20 z:30]     # Positions with names
[10 y:20 30]         # Mixed (position 0=10, position 1 named 'y'=20, position 2=30)
[draw:? bounds:[0 0]] # Abstract (position 0 requires implementation)
[*handle:? state:nil] # Handler (position 0 handles operations)
```

### 37.3 Access Unification

Every structure supports both positional and named access across all contexts:
```glisp
data: [x:10 y:20]
data.0 == data.x      # Both access position 0
data.1 == data.y      # Both access position 1
data(0) == data(x)    # Dynamic access to same position

# Works with all computational contexts
async_data: [result:? error:?]
stream_data: [source:? sink:?]
handler_data: [*perform:? state:?]
```

### 37.4 Object-Oriented Patterns as Natural Emergence

GLISP's OO patterns **emerge naturally** from struct transformation across all computational contexts:

**Classes Emerge from Struct Templates**
```glisp
Person: [
  name:"" 
  age:0 
  greet:[self] => s"Hi, I'm ${self.name}"
  async val fetch_details:[self] => <- api.get(s"/person/${self.id}")
]
alice: Person[name:"Alice" age:30]
```

**Handlers Emerge from Handler Templates**
```glisp
IOService: [
  *read:?
  *write:? 
  buffer_size:4096
  async val process:[self file] => 
    val data: <- perform("read", file)
    <- perform("write", s"processed_${file}", process_data(data))
]
```

**Streams Emerge from Reactive Templates**
```glisp
DataProcessor: [
  source:?
  transform:?
  process:[self] => self.source~>self.transform~>output_sink
]
```

---

# Part VIII: Examples

[Examples ...]

---

# Part IX: Implementation Guidance

[Implementation guidance ...]

---

# Part X: Future Considerations

[Future considerations ...]

---

*End of GLISP – Language Description*
