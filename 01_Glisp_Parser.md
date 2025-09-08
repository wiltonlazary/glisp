# GLISP – Parser Description

> **Authority:** This document is the authoritative technical description. No abstractions or proposals beyond what is written here.

---

# Part I: Foundation

## Philosophy

GLISP parser emerged from recognizing that different programming paradigms (command-line, functional, object-oriented, Lisp) aren't fundamentally different - they're variations on structured data transformation wearing different syntactic clothes. The mechanical transformation from `(print x)` to `print(x)` reveals that we can bridge different syntactic styles through consistent, local transformations. This insight, combined with careful whitespace handling, enables GLISP to unify all programming styles without exceptions.

The language follows thermodynamic principles - simple, local rules that compose to create complex behaviors. Just as thermodynamics provides fundamental laws governing energy transformations without dictating specific configurations, GLISP provides core parsing rules that enable all surface syntaxes without prescribing one style. The parser applies mechanical transformations based on local context, never needing to understand the semantics of what it's parsing.

GLISP's design recognizes that everything in computation is structured data undergoing transformation. Functions receive input structs and produce output structs. Types are struct templates with defaults and constraints. Effects are structured transformations of execution context. Reactive streams are structured transformations of time-ordered data. All emerge from the same mechanical principles.

## Overview

GLISP (Grafted Lisp) is a homoiconic, multi-style surface language with a Lisp-like core. The parser operates as a stream processor that transforms source text into abstract syntax trees through mechanical pattern recognition. These patterns, configured through directives, enable the same core language to express command-line scripts, functional pipelines, object-oriented systems, effect handlers, reactive streams, and traditional Lisp - all within the same program.

Key properties include streaming parse capability (the parser never needs to see the whole program), deterministic transformations (the same input always produces the same AST), preserved source locations (every element tracks where it came from), configurable syntax (directives customize parsing behavior), and complete structural expressiveness (all computational patterns emerge from the same rules).

## Core Concepts

The parser works with fundamental data types. **AtomDatum** represents contiguous non-whitespace characters like identifiers, numbers, transformers, or abstract markers. **ListDatum** represents delimited sequences that preserve their delimiter type - parentheses for application, brackets for structs, braces for blocks, angle brackets for binary patterns.

The parser is always inside some collection context, starting with the TOP section (the source body). It maintains a buffer of atoms being collected and closes this buffer when it hits boundary markers. The mechanical process of collecting, detecting patterns, and closing buffers creates all of GLISP's syntactic richness - from simple commands through complex effect handlers and reactive systems.

## Transformer Spacing Rules

**Spaced:** Requires whitespace on BOTH sides
- Section trigger: `x : y` (spaced colon)
- Infix: `a + b` (spaced plus)
- Associative: `x = y` (spaced equals)
- Effect: `x ! y` (spaced exclamation)

**Glued:** NO whitespace on specified side(s)
- Prefix glue: `'x`, `--x`, `&data` (glued left)
- Suffix glue: `x:y`, `x.field`, `x~>y` (glued right)
- LHS-glued: `x: y`, `x:\n  y` (no left space, any right)

---

# Part II: Parser Mechanics

## The Fundamental Algorithm

The GLISP parser operates through two mutually recursive functions that form the heart of the system. When `parse_list` encounters non-delimiter characters, it calls `parse_atom`. When `parse_atom` completes and finds certain patterns (like an opening delimiter immediately following), it calls `parse_list`. This mutual recursion naturally handles arbitrary nesting without exceptions or complex state machines.

```
# The conceptual structure (pseudocode, not GLISP implementation)
function parse_atom(context):
  # Collect characters into buffer
  # Check for glue patterns (attached transformers)
  # Check for head-graft (delimiter touching)
  # Return atom or recurse into parse_list

function parse_list(context):
  # Process items until closing condition
  # For each non-delimiter, call parse_atom
  # For each delimiter, recurse with parse_list
  # Return completed list when termination reached
```

## Whitespace Intelligence

When the parser consumes whitespace, it doesn't just skip over it - it gathers critical intelligence that shapes parsing decisions. The whitespace consumer tracks how many newlines were encountered (which determines line boundaries), the current indentation level (which determines block structure), and any effect markers (which determine execution context). This information guides whether newlines act as statement boundaries or mere spacing, whether indentation indicates a nested block, whether zero whitespace enables special patterns like grafting, and whether effect contexts are active.

```
# Conceptual pseudocode
function consume_whitespace():
  newline_count = 0
  indent = 0
  effect_markers = []
  
  while is_whitespace(current_char()):
    if current_char() == '\n':
      newline_count = newline_count + 1
      indent = 0
    elif current_char() == ' ':
      indent = indent + 1
    elif current_char() == '\t':
      if indent == 0: error("No tabs in leading indentation")
    elif current_char() == '#' and peek() == ' ':
      # Comment detected - skip to end of line
      skip_to_newline()
      newline_count = newline_count + 1
      indent = 0
    advance()
  
  return {newlines: newline_count, indent: indent, effects: effect_markers}
```

## Collection Contexts (Sections)

The parser is always inside some collection context, which we call a section. The TOP level of your source file is a section. When you write an if statement, its body is a section. When you open parentheses, you're in a delimited section. Each section knows how it will terminate - the TOP section ends at EOF, delimited sections end at their closing delimiter, indented sections end when the indentation decreases.

```
# Conceptual pseudocode
function create_section(parent, termination_condition, indent_level):
  return {
    items: []                    # Collected items
    parent: parent               # Enclosing section
    termination: termination_condition
    indent: indent_level
    
    add_item: function(item) { items.append(item) }
    
    materialize: function() {    # Called when section completes
      if items.length == 0:
        error("Empty section")
      elif items.length == 1 and is_list(items[0]):
        return items[0]          # Single list collapses
      elif items.length == 1:
        return wrap_with_delimiters(items[0])
      else:
        return wrap_with_delimiters(items)
    }
  }
```

## Structure Collection Contexts

Parallel to sections which collect for execution, structures collect for data. The parser enters structure collection mode when encountering the structure trigger `·` (middle dot) with surrounding whitespace. Structures follow the same mechanical collection rules as sections but materialize with different delimiters. Abstract structures use `?` markers to indicate incomplete fields requiring implementation.

```glisp
# Section (execution collection)
if condition : expr1 ; expr2    ↦ (if condition {expr1 expr2})

# Structure (data collection)  
point · 10 20 30                ↦ (point [10 20 30])

# Abstract structure (incomplete collection)
Drawable · draw:? bounds:[x:0 y:0]  ↦ (Drawable [draw:? bounds:[x:0 y:0]])
```

The parser is always in either a section context (default), structure context (after `·`), or delimited context (inside brackets). Structure contexts terminate at the same boundaries as sections: dedent, closing delimiters, or explicit terminators.

## Boundary Detection and Item Creation

Boundaries determine when the parser closes its current collection buffer and what happens next. When parsing undelimited sequences like `print hello world`, the parser collects these atoms into a buffer. When it hits a boundary (like a newline in an undelimited context), it must decide what to do with the collected atoms.

If the buffer contains multiple undelimited atoms, they represent a function application. The parser wraps them with application delimiters (parentheses by default), creating `(print hello world)`. This mechanical rule is what enables command-line syntax, async operations, and reactive streams - the parser doesn't recognize these semantic distinctions, it just wraps undelimited sequences that terminate at boundaries.

```
# Conceptual pseudocode
function handle_boundary(boundary_char):
  buffer_contents = close_current_buffer()
  
  # Wrap undelimited sequences as applications
  if buffer_contents.is_undelimited_sequence():
    item = wrap_with(application_delimiters, buffer_contents)
  else:
    item = buffer_contents  # Already structured (from glue, etc.)
  
  if boundary_char == ';':  # Semicolon: item boundary, continue same level
    current_section.add_item(item)
    start_new_buffer()
    
  elif boundary_char == '\n':  # Newline: context-dependent
    if in_delimited_context():
      continue_collecting()  # Just whitespace in lists
    else:
      current_section.add_item(item)
      start_new_buffer()
      
  elif boundary_char == ':' and was_spaced():  # Section trigger
    current_section.add_item(item)
    child = create_section(current_section, 'dedent', next_indent())
    parse_into(child)
    current_section.add_item(child.materialize())

  elif boundary_char == '·' and was_spaced():  # Structure trigger
    current_section.add_item(item)
    child = create_structure(current_section, 'dedent', next_indent())
    parse_into(child)
    current_section.add_item(child.materialize())

  elif boundary_char == '!' and was_spaced():  # Effect trigger
    current_section.add_item(item)
    child = create_effect_section(current_section, 'dedent', next_indent())
    parse_into(child)
    current_section.add_item(child.materialize())
```

## Glue Transformers (Attached Transformers)

Glue transformers are characters that attach directly to atoms without intervening whitespace. When the parser completes an atom and finds a configured glue character immediately following, it creates a synthetic list structure. This mechanical transformation turns `x:42` into `(: x 42)`, `data~>stream` into `(~> data stream)`, and `&data` into `(& data)` - the parser creates a list with the glue character as the transformer.

```
# Conceptual pseudocode
function complete_atom_parsing(atom_buffer):
  next_char = current_char()
  
  # Check for suffix glue (like : or :: or ~>)
  if next_char in configured_suffix_glue:
    advance()  # Consume the glue character
    
    # Create synthetic list with glue as transformer
    synthetic_list = create_list(
      delimiter: application_delimiters,
      elements: [
        next_char,              # The glue character becomes the transformer
        create_atom(atom_buffer),  # The left operand
        parse_next_element()    # Parse the right operand
      ]
    )
    return synthetic_list
  
  # Check for prefix glue (like &, share&, etc.)
  if atom_buffer in configured_prefix_glue:
    # Create synthetic list with atom as transformer
    synthetic_list = create_list(
      delimiter: application_delimiters,
      elements: [
        create_atom(atom_buffer),  # The prefix becomes the transformer
        parse_next_element()       # Parse the operand
      ]
    )
    return synthetic_list
  
  # No glue - just return the atom
  return create_atom(atom_buffer)
```

## Graft Transformers (Zero-Whitespace Connections)

Grafts occur when structures touch with zero intervening whitespace. Head-graft happens when an atom directly touches an opening delimiter, transforming `print(x)` into `(print x)` and `async{body}` into `(async {body})`. The atom becomes the head of the list that follows. Tail-graft happens when a closing delimiter directly touches another opening delimiter, transforming `f(x){y}` into `(f x {y})` and `handler[handlers]{body}` into `(handler [handlers] {body})`. The second list gets appended to the first.

```
# Conceptual pseudocode
function check_for_head_graft(atom):
  # After parsing atom, check what immediately follows
  if current_char() in opening_delimiters:
    # Head-graft detected!
    list = parse_list()
    list.prepend(atom)  # Atom becomes head
    return list
  
  return atom  # No graft

function check_for_tail_graft(list):
  # After closing delimiter, check what follows
  advance_past_closer()
  
  if current_char() in opening_delimiters:
    # Tail-graft detected!
    tail = parse_list()
    list.append(tail)
    return list
  
  return list  # No graft
```

## Spaced Transformers (Infix and Associative)

Unlike glue transformers that attach directly, spaced transformers are recognized after collecting multiple atoms in a buffer. When the parser reaches a boundary and examines its buffer, it checks for configured transformers and rearranges the contents accordingly.

For infix transformers (like arithmetic), the parser folds left without precedence. Finding `a + b * c` in the buffer, it creates `(+ a b)` first, then `(* (+ a b) c)`. For associative transformers (like pipes), the parser considers associativity direction and may create sections for multi-line constructs.

```
# Conceptual pseudocode
function process_buffer_with_transformers(atoms):
  # Scan for transformers
  transformers = atoms.filter(a => is_configured_transformer(a))
  
  if transformers.is_empty():
    # No transformers - simple application
    return wrap_as_application(atoms)
  
  # Process infix transformers left to right
  result = atoms[0]
  i = 1
  
  while i < atoms.length:
    if atoms[i] in infix_transformers and i + 1 < atoms.length:
      # Create (transformer left right)
      result = create_list(
        delimiter: application_delimiters,
        elements: [atoms[i], result, atoms[i + 1]]
      )
      i = i + 2
    elif atoms[i] in associative_transformers:
      # Handle associative patterns (pipes, etc.)
      result = handle_associative_pattern(result, atoms[i], atoms[i+1:])
      break
    else:
      i = i + 1
  
  return result
```

## Advanced Mechanics: Sibling Grouping

The parser uses limited lookahead for sibling grouping - an advanced mechanical rule for control structures. When the parser encounters a configured sibling keyword (like `if`, `try`, `with_handler`) followed by a section trigger, it **uses near-by short-range forces** to watch for related keywords (`elseif`, `else`, `catch`, `finally`) at the same indentation level.

```
# Conceptual pseudocode
function handle_sibling_group(first_keyword):
  group = find_sibling_configuration(first_keyword)
  if group == null: return null
  
  base_indent = current_indent()
  collected = [(first_keyword, parse_section())]
  
  # Use near-by short-range forces to detect siblings
  while peek_keyword_at_indent(base_indent) in group:
    sibling = advance_to_keyword()
    collected.append((sibling, parse_section()))
  
  return create_grouped_structure(collected)
```

This uses local context - checking for keywords at the same indentation level immediately following. The mechanics remain deterministic and configuration-driven, using near-by pattern detection.

## Pattern Recognition

After parsing certain structures, the parser checks for patterns that have semantic significance. For example, after parsing a list with struct delimiters (brackets by default), it checks if all items follow struct patterns. If so, it marks the structure as a struct, allowing the evaluator to treat it specially. Abstract structs are recognized when they contain `?` abstract markers.

```
# Conceptual pseudocode
function check_struct_pattern(list):
  if list.delimiter != struct_delimiter: return false
  
  # Structs can contain any mix of:
  # - Positional items
  # - Pairs (: name value)
  # - Parameters (-- name)
  # - Rest collectors (* name)
  # - Abstract markers (?)
  return true  # All [] lists are structs
```

The parser only mechanically identifies these patterns. It marks structures for the evaluator's semantic interpretation but assigns no meaning itself.

## Section Type Tagging

When materializing a section, the parser mechanically checks if the last item is `::Type` where `::` has no left operand (standalone tag transformer). This follows simple positional rules:

```
# Conceptual pseudocode
function materialize_section(items):
  # Check if last item is standalone type tag
  if items.length > 0 and is_standalone_tag(items.last()):
    if items.length == 1:
      # Single item - section collapses to it
      return items[0]
    else:
      # Last item tags the section
      type_tag = items.last()
      section_items = items[0...-1]  # All except last
      return create_list(
        delimiter: application_delimiters,
        elements: ["::", wrap_items(section_items), type_tag[1]]
      )
  else:
    # Normal materialization
    return apply_standard_section_rules(items)
```

This enables type annotations for all constructs:
```glisp
# Tags entire section
[x] =>
  val doubled: x * 2
  doubled
  ::Int
# ↦ (=> [x] (:: {(: (val doubled) (* x 2)) doubled} Int))
```

---

# Part III: Configuration System

## Default Configuration

GLISP's behavior is controlled through configuration directives. Each directive controls a specific aspect of parsing, from which characters act as delimiters to which keywords trigger special grouping behavior. The configuration is designed to support all programming paradigms while maintaining internal consistency.

```glisp
#directive="#"                        # Directive prefix
#comment="#"                          # Comment prefix (requires space)
#string={"escaped":"\"\"" "raw":"``"} # String delimiters
#separator=[","]                      # Comma acts like whitespace
#sequence=[";"]                       # Semicolon creates item boundaries

#glue={
  "prefix": {
    "quote": "'",       # 'x quotes expressions
    "quasiquote": "''", # ''x creates templates
    "unquote": "~",     # ~x unquotes in templates
    "splice": "~*",     # ~*x splices in templates
    "meta": "@",        # @x adds metadata
    "spread": "...",    # ...x spreads collections
    "param": "--",      # --x creates parameters
    "lens": "lens",     # lens x creates lens
    # Memory marks
    "own": "&",         # &x transfers ownership
    "share": "share&",  # share&x reference counts
    "borrow": "borrow&", # borrow&x temporary access
    "copy": "copy&",    # copy&x explicit copy
    "acquire": "acquire&", # acquire&x manual alloc
    "release": "release&", # release&x manual free
    "pin": "pin&",      # pin&x locks in memory
    "unpin": "unpin&",  # unpin&x unlocks memory
  },
  "suffix": {
    "nav": ".",         # x.y navigates fields
    "cascade": "..",    # x..y cascades operations
    "pair": ":",        # x:y creates pairs
    "tag": "::",        # x::T tags types
    "stream": "~>",     # x~>y creates stream
    "channel": "<~>",   # x<~>y creates channel
    "bind": "<-",       # x<-y monadic bind
    "send": "->",       # x->y message send
  }
}

#section={"trigger":":" "materialize":"{}"}    # Section configuration
#structure={"trigger":"·" "materialize":"[]"}  # Structure configuration
#effect={"trigger":"!" "materialize":"{}"}     # Effect configuration
#bracket=["()" "[]" "{}" "⟅⟆" "<[]>"]           # Recognized brackets
#apply=["()"]       # Application delimiters
#array=["[||]"]     # Array delimiters
#block=["{}"]       # Block delimiters
#struct=["[]"]      # Struct delimiters
#binary=["<[]>"]    # Binary pattern delimiters

#infix=[            # Left-associative, no precedence
  "+" "-" "*" "/"   # Arithmetic
  "==" "!=" "<=" ">=" "<" ">"  # Comparison
  "&&" "||"         # Logical
  "in" "is" "as"    # Type operations
  "or" "and"        # Type/logical combinations
  "has" "where"     # Constraint operations
]

#associative={      # Directional folding
  "left": [
    "="             # Mutation
    "<-"            # Monadic extraction
    "=>"            # Lambda
    "|>" "|" ">>"   # Pipes
    "!>" "&>" "!!>" # Async/lazy/parallel
    "~>" "<~>"      # Channels/streams
    "with" "handle" # Operations
  ],
  "right": [
    "<|" "|<" "<<"  # Reverse pipes
    "<~"            # Channel receive
    "throws" "raises" # Error propagation
  ]
}

#sibling=[          # Control structure grouping
  ["try" "catch" "finally"],
  ["if" "elseif" "else"],
  ["with_handler" "with_effect" "with_context"],
  ["match" "case" "default"],
  ["for" "while" "until"],
  ["async" "await" "yield"]
]

#declaration=["val" "var" "rec" "type" "export" "async" "arena"]
```

**Evaluator conventions (NOT parser configuration):**
* Atoms starting with `*` may be varargs/handlers/properties - evaluator interprets
* The `*` prefix in `*rest`, `*get`, `*age`, `*handle` is just part of the atom name
* Parser sees these as regular atoms, evaluator assigns meaning
* Abstract marker `?` is just an atom to parser, evaluator interprets as hole

**Collection triggers:**
* `#section` — Configures execution collection (trigger `:`, materialize `{}`)
* `#structure` — Configures data collection (trigger `·`, materialize `[]`)
* `#effect` — Configures effect collection (trigger `!`, materialize `{}`)

All triggers must be spaced (whitespace on both sides) to activate collection mode.

**Note on synthesized lists:** When the parser creates lists during transformations, it uses appropriate delimiters based on context: transformations that create function applications (e.g., `x: y` ↦ `(: x y)`, `a + b` ↦ `(+ a b)`, `data~>stream` ↦ `(~> data stream)`) use delimiters from `#apply`, while materialized sections use delimiters from `#section.materialize`, materialized structures use delimiters from `#structure.materialize`, and materialized effects use delimiters from `#effect.materialize`.

## How Configuration Affects Parsing

The configuration directives directly control the parser's mechanical behavior. When the parser encounters a character, it consults the configuration to determine whether it's a delimiter, transformer, glue transformer, or plain character. This lookup drives all parsing decisions.

For example, when the parser sees `:` with no preceding whitespace, it checks if it's configured as a suffix glue transformer. If so, it triggers the glue transformation mechanism. When it sees `~>` with surrounding whitespace, it checks if it's an associative transformer and later processes it during buffer analysis. When it sees `?` as an atom, it marks it as an abstract marker for the evaluator.

## Character Processing

### Shebang

If first two characters are `#!`, entire first line is treated as comment. Not configurable, applies regardless of directive/comment settings.

### Comments

**Start:** `<C> ` where `<C>` is configured prefix followed by single space
**Range:** from start to newline (exclusive)
**Effect:** discarded before parsing

Comments do not break glue operations:
```glisp
x:: # comment
  Type
↦ (:: x Type)

&data # comment
  |> process
↦ (|> (& data) process)
```

### Whitespace and Separators

* **Whitespace** — spaces, tabs, newlines separate datums
* **Tabs** — forbidden in leading indentation
* **Separators** — characters in `#separator` (default: `,`) act like whitespace, terminating values and constructs
* **Sequence markers** — characters in `#sequence` (default: `;`) act as statement boundaries, completing the current item and starting the next item at the same level in the parent section

### Strings

Three string types with configurable delimiters:

**Escaped strings** — Process escape sequences (`\n`, `\t`, `\\`). Default: `"..."`

**Raw strings** — Literal content, escape delimiter by doubling. Default: `` `...` ``

**Binary strings** — Bit-level patterns. Default: `<[...]>`

**Prefixed strings** — Atom glued to string opening becomes processor:
```glisp
f"Hello ${name}"    ↦ (f "Hello ${name}")
sql`SELECT * FROM`  ↦ (sql "SELECT * FROM")
s"User: ${id}"      ↦ (s "User: ${id}")  # String interpolation template
regex"[a-z]+"       ↦ (regex "[a-z]+")
```

### Directives

Form: `<D>datum` where `<D>` is directive prefix (default `#`)
Prefix glued to following datum. May alter parsing behavior by scope.

### Abstract Markers

The `?` character is recognized as an abstract marker when appearing as a standalone atom in struct contexts:

```glisp
Drawable: [
  draw: ?           # Abstract field - must be implemented
  bounds: [0 0 0 0] # Concrete field - has default
]
```

### Binary Patterns

Binary patterns use `<[...]>` delimiters for bit-level matching and construction:

```glisp
<[version:4 ihl:4 tos:8 length:16]>  # Bit pattern with named fields
<[1:1 0:3 data:4]>                   # Mixed literal/field pattern
```

### TOP

The source body (TOP) is a section. Follows standard section mechanics.

### System Atom Patterns

System patterns for file paths and URLs take precedence over all configured transformers. These patterns are always active and cannot be disabled.

**Pattern Recognition:**

When the parser encounters these character sequences, it enters system atom mode:
* `./` — Current directory prefix
* `../` — Parent directory prefix  
* `:/` — Protocol separator (includes `://` for URLs)
* `~/` — Home directory prefix

**Collection Rule:**

Once a system pattern is detected, the parser:
1. Collects all characters until whitespace (space, tab, newline)
2. Ignores all configured transformers during collection
3. Returns the complete sequence as a single atom

**Examples:**
```glisp
./tmp/file.txt          # Single atom: "./tmp/file.txt"
../../../parent.json    # Single atom: "../../../parent.json"
https://api.com/v1      # Single atom: "https://api.com/v1"
file:///absolute/path   # Single atom: "file:///absolute/path"
~/Documents/data        # Single atom: "~/Documents/data"

# Without system pattern - normal transformer rules apply
obj.field              # Navigation: (. obj field)
obj..method            # Cascade: (.. obj method)
key:value              # Pair: (: key value)
data~>stream           # Stream: (~> data stream)
```

**Precedence:**

System patterns are recognized before:
1. Glue transformers
2. Configured transformers  
3. Normal atom collection

This ensures filesystem and network paths remain intact regardless of configuration.

---

# Part IV: Syntactic Patterns

## Command-Line Style

Command-line syntax emerges naturally from the parser's boundary and wrapping rules. Each line becomes an item, each item gets wrapped as an application. No exceptional command recognition is needed.

```glisp
ls -la              # Buffer: [ls, -la]
                    # Boundary: newline
                    # Wrapped: (ls -la)

grep error | sort   # Buffer: [grep, error, |, sort]
                    # Process: | is associative
                    # Result: (| (grep error) sort)

cd /tmp; pwd        # First: (cd /tmp)
                    # Semicolon: item boundary
                    # Second: (pwd)
                    # Result: {(cd /tmp) (pwd)}
```

## Functional Style

Functional programming patterns emerge from associative transformers and lambda syntax. The same mechanical rules that handle commands also handle function composition and pipelines.

```glisp
# Lambda with arrow (struct parameters only)
[x y] => x + y
# Buffer processing creates: (=> [x y] (+ x y))

# Pipeline with sections
data |>
  validate
  log("processing")
|> save
# Section creation: (|> (|> data {validate (log "processing")}) save)

# Function composition
h <| g <| f <| x
# Right-associative: (<| h (<| g (<| f x)))
```

## Object-Oriented Style

Method chaining and field access emerge from the navigation glue transformer and head-graft patterns. The parser doesn't understand objects or methods - it just applies mechanical transformations.

```glisp
# Navigation chains
obj.field.method()
# Glue creates: (. obj field)
# Then: (. (. obj field) method)
# Head-graft: ((. (. obj field) method))

# Cascade for mutations
widget..width: 100..height: 200
# Creates: (.. (.. widget (: width 100)) (: height 200))
# Returns original widget after mutations
```

## Structure Collection Style

Structure collection with `·` enables bracket-free data definition, parallel to how `:` enables brace-free execution:

```glisp
# Traditional (requires brackets)
val point: [10 20 30]
matrix: [[1 2] [3 4]]

# With structure fluidity
val point: · 10 20 30
matrix · · 1 2
        · 3 4

# Mixed with other patterns
config ·
  server · host:"localhost" port:8080
  database · url:"postgres://..." pool:10

# Combines with pipes
data · 1 2 3 |> map square |> sum
```

The mechanical rules are identical to section collection:
1. Trigger creates collection context
2. Items collected until boundary
3. Singleton rules apply
4. Materializes with configured delimiters

## Memory Management Style

Memory marks emerge from prefix glue transformers:

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

## Reactive Stream Style

Reactive programming emerges from stream transformers and channel operations:

```glisp
# Stream transformations
clicks ~> throttle(300) ~> map(get_position) ~> update_ui

# Channel operations
input_channel <~> output_channel

# Async operations with sections
async val fetch_data: [id] =>
  val user: <- fetch_user(id)
  val posts: <- fetch_posts(user.id)
  [user:user posts:posts]
```

## Structs and Data Structures

Structs emerge from lists with bracket delimiters. The parser mechanically recognizes all `[]` lists as structs.

```glisp
# Struct with pairs
[name:"Alice" age:30]
# Parsed: [(: name "Alice") (: age 30)]
# Marked as struct

# Parameters with punning
val host: "localhost"
[--host port:8080]
# Parsed: [(-- host) (: port 8080)]
# Parameter (-- host) resolved by evaluator

# Abstract structs with holes
Drawable: [
  draw: ?           # Abstract field
  bounds: [0 0 0 0] # Concrete field
]
# Parsed: [(: draw ?) (: bounds [0 0 0 0])]

# Rest collectors
[x y *rest]
# Parsed: [x y *rest]
# *rest is just an atom, evaluator interprets as rest collector
```

## Control Structures

Control structures use sibling grouping and section triggers to create indentation-based syntax without alternative parsing modes.

```glisp
# If-else with sibling grouping
if x > 0 :
  print("positive")
elseif x < 0 :
  print("negative")
else :
  print("zero")

# Try-catch-finally
try :
  val result: risky_operation()
  process(result)
catch e :
  log_error(e)
  default_value
finally :
  cleanup()

# Mechanical process:
# 1. "if"/"try" triggers sibling collection
# 2. Section after : becomes condition/body
# 3. Parser uses near-by short-range forces for siblings
# 4. Groups into structured control flow
```

## Pattern Matching with Guards

Pattern matching supports guard conditions and binary patterns:

```glisp
# Guards on patterns
match value :
  [x y] where x > y : "descending"
  Some[x] where x > 0 : "positive"
  Person[age:a] where a >= 18 : "adult"

# Binary pattern matching
match packet :
  <[0x45 len:16 rest:binary]> : parse_ipv4(len, rest)
  <[version:4 data:binary]> where version == 6 : parse_ipv6(data)
```

## Declaration Keywords and Parsing Context

Declaration keywords (`val`, `var`, `rec`, `type`, `export`, `async`, `arena`) do not create special parsing contexts or disable mechanical rules like head-graft:

```glisp
val f[x]: x + 1      # Head-graft still applies
↦ (: (val f [x]) (+ x 1))  # ERROR: f grafted with [x]

val f: [x] => x + 1  # Correct: explicit lambda
↦ (: (val f) (=> [x] (+ x 1)))

# Async declarations (async before val)
async val fetch: [url] => <- http_get(url)
↦ (: (async (val fetch)) (=> [url] (<- (http_get url))))

# Sugar form: async alone implies val
async fetch: [url] => <- http_get(url)
↦ (: (async (val fetch)) (=> [url] (<- (http_get url))))
```

The parser treats declaration keywords as regular atoms that happen to appear in binding patterns. All mechanical rules continue to apply.

---

# Part V: Semantic Interpretation

## Evaluation Model

The evaluator interprets the AST produced by the parser. Where the parser provides structure through mechanical transformations, the evaluator provides meaning through interpretation. The parser doesn't know that `+` means addition, that `if` is conditional execution, that `~>` creates streams, or that `?` marks abstract fields - it just creates the structures. The evaluator assigns these meanings.

Evaluation proceeds left-to-right and depth-first by default. Special forms may alter evaluation order - for instance, `if` evaluates its condition first, then only one branch. The `&&` transformer short-circuits, not evaluating its right operand if the left is false. Async operations may evaluate out of order.

## Function Application

When the evaluator encounters a list with application delimiters (parentheses by default), it treats it as a function call. The first element is the function, the remaining elements are arguments. This uniform treatment means command-line syntax, function calls, transformer applications, and stream transformations all flow through the same evaluation mechanism.

```glisp
(print "hello")     # Function call
(+ 1 2)            # Transformer application
(ls -la)           # Command execution
(~> data transform) # Stream operation
(& data)           # Memory mark operation
# All evaluated as: function in head position, arguments follow
```

## Special Forms

Certain patterns in the AST trigger special evaluation rules. These special forms provide control flow, binding, and other fundamental behaviors that can't be implemented as regular functions.

```glisp
# Conditional - evaluates only one branch
(if condition then-expr else-expr)

# Binding - creates new bindings in scope
(: name value)      # Creates binding (context-dependent)
(: (var name) value)  # Mutable binding

# Lambda - creates closure
(=> [params] body)

# Monadic extraction - extracts from monad
(<- name m-expr)    # Extract and bind/mutate

# Stream operations - reactive transformations
(~> source transform sink)
(<~> channel1 channel2)

# Memory operations - ownership management
(& data)            # Transfer ownership
(share& data)       # Reference count
(borrow& data)      # Temporary access

# Pattern matching - tests patterns sequentially
(match value
  (pattern1 result1)
  (pattern2 result2))
```

## Type System

GLISP supports gradual typing, from fully dynamic to fully static. Type annotations are hints that can be ignored for scripting or enforced for systems programming. The type system supports structural typing (compatibility by shape), abstract structs (incomplete types requiring implementation), and nominal typing (compatibility by name).

```glisp
# Type annotations
x::Int: 42
f::[x::Int y::Int] => ::Int: [a b] => a + b

# Structural typing
Point: [x::Int y::Int]
Coord: [x::Int y::Int]  # Compatible with Point

# Abstract structs - incomplete types
Drawable: [
  draw: ?             # Must be implemented
  bounds: [0 0 0 0]   # Has default
]

# Union and intersection
StringOrInt: String or Int
Named: [name::String]
Aged: [age::Int]
Person: Named and Aged  # Has both fields
```

## Module System

Modules provide namespace management and explicit boundaries. Each module declares its exports explicitly, providing security and stability when importing from untrusted sources.

```glisp
# In math_utils.glisp
export val add: [a b] => a + b
export val multiply: [a b] => a * b
val helper: [x] => x * 2  # Private, not exported

# In main.glisp
val math: import("./math_utils")
math.add(1, 2)  # Access exported function

# Or spread into scope
include("./math_utils")
add(1, 2)  # Directly available
```

---

# Part VI: Examples

## Complete Parsing Trace

Let's trace through a complete example to see how all the mechanical pieces work together:

```glisp
async val fetch_user: [id] =>
  val user: <- api.get(s"/users/${id}")
  if user.active :
    user
  else :
    error("User inactive")
```

The parser begins in the TOP section, consuming no initial whitespace. It collects `async` and `val` as declaration keywords, then `fetch_user`. The `:` glue transformer triggers, creating `(: (async (val fetch_user)) ...)`.

The `[id]` struct is parsed normally. The `=>` associative transformer creates a lambda, so the parser expects a body section.

In the lambda body, the parser collects `val` and `user`. The `:` glue transformer triggers, creating `(: (val user) ...)`. Then `<-` as an associative transformer combines with the navigation pattern `api.get(...)` where the head-graft makes `s"/users/${id}"` an argument.

The `if` triggers sibling grouping, creating conditional sections. Each section follows normal boundary rules.

The final AST demonstrates how async operations, monadic extraction, navigation, string interpolation, and control flow all emerge from the same mechanical parsing rules.

## Multi-Paradigm Program

GLISP's mechanical parsing rules allow different paradigms to coexist naturally in the same program:

```glisp
# Command-line style file processing
val files: ls "*.txt"

# Functional pipeline with memory optimization
val processed: &files 
  |> map(f => read_file(f))
  |> filter(content => content.contains("TODO"))
  |> &filtered_data
  |> map(content => extract_todos(content))

# Object-oriented manipulation
val report: Report.new()
  ..title: "TODO Items"
  ..add_all(processed)
  ..format()

# Handler for I/O operations
with_handler [
  *write: [file data] => safe_write(file, data)
  *log: [msg] => append_log(msg)
] :
  save_report(report)

# Reactive stream for monitoring
file_changes ~> 
  debounce(500) ~> 
  filter(is_text_file) ~> 
  trigger_reprocess

# Abstract struct for extensibility
Processor: [
  validate: ?
  transform: ?
  output: ?
]

# Pattern matching with guards
match save_result :
  Ok[path] where file_exists(path) : 
    log(s"Saved to: ${path}")
  Err[e] where is_io_error(e) : 
    retry_save()
  * : 
    fatal_error("Unknown save result")

# Declarative pattern matching
summary ·
  files: files.length
  todos: processed.flatten().length
  saved: path.or("failed")

# Arena for temporary allocations
arena request_processing(megabyte) :
  temp_buffer: alloc(large_size)
  result: process_with_buffer(temp_buffer, &processed)
  copy&result  # Copy before arena cleanup
```

Each style emerges from the same mechanical rules - no exceptional modes or complex state machines. The parser consistently applies its pattern detection and transformation rules, creating a uniform AST that the evaluator can interpret with appropriate semantics for each construct type.

---

# Part VII: Implementation Guidance

## Building Your Own Parser

To implement a GLISP parser, start with the two recursive functions at the core. Your `parse_atom` function should collect characters until hitting a boundary, check for glue patterns (including memory marks), check for head-graft, and return an atom or recurse into list parsing. Your `parse_list` function should process items until reaching its termination condition, calling `parse_atom` for non-delimiters and recursing for nested lists.

The whitespace consumer should track newlines, indentation, and any context markers, returning this information to guide parsing decisions. This information determines whether newlines are boundaries or just spacing, whether you're in an indented block, whether zero-whitespace patterns like grafts can occur, and whether special contexts are active.

Boundary handling should follow the uniform pattern: close the current buffer, wrap undelimited sequences as applications, and then follow the boundary's specific continuation rule. Semicolons continue at the same level, newlines depend on context, section triggers open child sections, structure triggers open child structures, and closing delimiters return to the parent context.

## Testing Your Implementation

Test your parser incrementally, starting with simple cases and building complexity. Begin with basic atoms and lists, verifying that delimiters are preserved. Add glue transformers, checking that `x:42` becomes `(: x 42)`, `data~>stream` becomes `(~> data stream)`, and `&data` becomes `(& data)`. Implement grafts, ensuring `f[x]` becomes `(f [x])` and `async{body}` becomes `(async {body})`. Add transformers, verifying left-folding for infix and proper associativity for directional transformers.

Test boundary handling carefully. Verify that newlines create item boundaries in sections but act as whitespace in delimited lists. Check that semicolons consistently create item boundaries regardless of context. Ensure section triggers properly create and materialize child sections. Verify that structure triggers properly create and materialize their respective contexts.

Edge cases to verify include EOF after glue characters (should error), mixing transformers with different precedence rules, deeply nested structures with mixed delimiters, abstract struct parsing with `?` markers, binary pattern parsing with `<[...]>` delimiters, memory mark parsing with all prefix forms, async declaration ordering, and recovery from parse errors at section boundaries.

## Performance Considerations

The streaming nature of GLISP parsing enables efficient implementation. You never need to hold the entire source in memory - just the current section's contents. The local decision-making means you can parse in a single pass without backtracking.

For optimization, consider caching configuration lookups since they're consulted frequently. Use efficient string building for atom collection rather than concatenation. Implement section, structure, and materialization lazily when possible. Consider parallel parsing of independent sections in large files. Handle abstract markers, binary patterns, and memory marks efficiently by recognizing them early in the atom parsing phase.

---

# Part VIII: Language Evolution

## Design Principles

GLISP's design follows several key principles that guide its evolution. **Mechanical simplicity** means the parser should remain a simple stream processor applying local transformations. New features should emerge from configuration rather than parser complexity. **Syntactic flexibility** allows multiple programming styles to coexist through the same mechanical rules. **Semantic uniformity** ensures that similar-looking constructs behave similarly regardless of syntactic style. **Structural completeness** means all computational patterns can be expressed through struct transformation.

## Future Directions

Several areas remain open for exploration and standardization. The memory management system could be extended with more sophisticated strategies. Reactive streams could support more complex temporal operations. The abstract struct system could be enhanced with dependent constraints. The binary pattern system could support more protocol types.

Other collection modes might emerge following the same pattern as sections and structures. The configuration system could be extended while maintaining backward compatibility. Advanced optimization strategies could be developed while preserving semantic clarity.

## Stability Guarantees

The core parsing algorithm is stable and won't change. The mechanical rules for boundaries, glue, grafts, sections, structures, and patterns are fixed. The default configuration may evolve, but existing configurations will continue to work. The AST structure is stable - existing tools can rely on its shape.

The abstract struct mechanism provides a stable foundation for future structural contract development. The memory mark system provides a stable foundation for ownership and resource management. The stream transformer system provides a stable foundation for reactive programming patterns.

---

# Appendix A: Core Syntax Reference

## §9 Sections

Sections are undelimited lists with specific mechanics. Each section collects items and materializes based on simple rules. Singleton sections collapse, multi-element sections wrap with materialize delimiters.

## §10 Structures

Structures are undelimited value collections triggered by `·` (spaced). They follow identical mechanics to sections but materialize with `[]` delimiters.

```glisp
point · 10 20 30                ↦ (point [10 20 30])
matrix · · 1 2                  ↦ (matrix [[1 2] [3 4]])
         · 3 4
```

## §11 Glue Transformers

### 11.1 Configuration

Glue transformers attach directly to atoms without spaces. Prefix glue includes quotes, meta markers, rest collectors, memory marks. Suffix glue includes navigation, cascade, pairs, tags, streams, channels.

### 11.2 No Symbol Collision

The same character can serve different syntactic roles based on whitespace context without ambiguity:

**Rest collector `*` (glued prefix):**
```glisp
[x *rest]     # * touches rest ↦ *rest is just an atom
```

**Multiplication `*` (spaced infix):**
```glisp
x * y         # * has spaces ↦ (* x y)
```

**Handler `*` (in context):**
```glisp
[*handle: ?]  # * in handler definition
```

The mechanical distinction:
- During atom parsing: `*atom` ↦ atom named `*atom` (evaluator interprets)
- During buffer processing: `atom * atom` ↦ infix transformation
- During struct parsing: `*field:?` ↦ handler field

### 11.3 Memory Marks

Memory marks are prefix glue transformers:
```glisp
&data              ↦ (& data)
share&expensive    ↦ (share& expensive)
borrow&resource    ↦ (borrow& resource)
acquire&buffer     ↦ (acquire& buffer)
```

### 11.4 Stream Suffix

The `~>` suffix enables stream transformations:
```glisp
data~>transform    ↦ (~> data transform)
```

### 11.5 Channel Suffix

The `<~>` suffix creates bidirectional channels:
```glisp
input<~>output     ↦ (<~> input output)
```

## §12 Grafts

### 12.1 Head-Graft (HG) - Universal for All Brackets

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

### 12.2 Tail-Graft

Grafts chain when closing delimiter touches opening delimiter:
```glisp
f[x]{y}      ↦ (f [x] {y})    # Tail-graft
handler[handlers]{body}  ↦ (handler [handlers] {body})
```

## §13 Abstract Structs

Lists with delimiter `#struct` (default `[]`) containing `?` markers are abstract structs:

```glisp
Drawable: [
  draw: ?           # Abstract field - must be implemented
  bounds: [0 0 0 0] # Concrete field - has default
]

# Cannot instantiate abstract structs
d: Drawable[]       # ERROR: Cannot instantiate abstract struct

# Must complete abstract fields
ConcreteDrawable: [
  ...Drawable
  draw: [self ctx] => ctx.render(self)
]

# Now can instantiate
d: ConcreteDrawable[]  # OK - all holes filled
```

## §14 Binary Patterns

Lists with delimiter `#binary` (default `<[]>`) are binary patterns:

```glisp
# Binary pattern matching
<[version:4 ihl:4 tos:8 length:16]>  # Named bit fields
<[1:1 0:3 data:4]>                   # Literal bits + field

# Binary construction
header: <[4:4 5:4 0:8 1500:16]>      # IPv4 header fragment
```

## §15 Infix Transformers

### 15.1 Configuration

Transformers listed in `#infix` are left-associative. Spacing required (whitespace on both sides).

### 15.2 Left-Fold Rule

All infix transformers fold left without precedence:
```glisp
a + b * c    ↦ (* (+ a b) c)
```

### 15.3 Lispian Neutrality

If infix transformer appears as list head, no transformation:
```glisp
(+ 1 2)      ↦ (+ 1 2)
```

## §16 Lambda Syntax

The `=>` associative transformer creates lambda expressions with struct patterns only:

```glisp
[x y] => x + y                      ↦ (=> [x y] (+ x y))
[x *rest] => process(x, rest)       ↦ (=> [x *rest] (process x rest))
```

## §17 Async Declarations

The `async` keyword must appear before other declaration keywords:

```glisp
async val fetch: [url] => <- http_get(url)
↦ (: (async (val fetch)) (=> [url] (<- (http_get url))))

# Sugar form: async alone implies val
async fetch: [url] => <- http_get(url)
↦ (: (async (val fetch)) (=> [url] (<- (http_get url))))
```

## §18 Arena Memory Context

Arena creates a memory scope for temporary allocations:

```glisp
arena frame_memory(size) :
  temp_data: alloc(size)
  result: process(temp_data)
  copy&result  # Must copy to escape arena

↦ (arena frame_memory size {body})
```

## §19 Stream Operations

Stream operations use stream transformers:

```glisp
# Stream transformation
data ~> throttle(100) ~> process   ↦ Stream pipeline
input <~> output                   ↦ Bidirectional channel
clicks ~> map(get_pos) ~> update   ↦ Reactive stream
```

## §20 Memory Operations

Memory operations use prefix glue transformers:

```glisp
&data |> process |> &result        ↦ Ownership transfer
share&expensive |> consumers       ↦ Reference counting
borrow&resource |> read_only       ↦ Temporary access
acquire&buffer(1024)               ↦ Manual allocation
release&buffer                     ↦ Manual deallocation
```

---

# Appendix B: Quick Reference

## Parsing Transformations

```glisp
# Glue transformers (attached)
x:42          ↦ (: x 42)
obj.field     ↦ (. obj field)
data~>stream  ↦ (~> data stream)
&data         ↦ (& data)
share&obj     ↦ (share& obj)
[x *rest]     ↦ [x *rest]         # *rest is atom
...data       ↦ (... data)        # Spread macro

# Grafts (zero whitespace)
f[x]          ↦ (f [x])          # Head-graft
f[x]{y}       ↦ (f [x] {y})      # Tail-graft
Type[x:10]    ↦ (Type [x:10])    # Constructor
async{body}   ↦ (async {body})   # Async block

# Transformers (spaced)
x = value     ↦ (= x value)      # Mutation
x <- monad    ↦ (<- x monad)     # Monadic extraction
a + b * c     ↦ (* (+ a b) c)    # Left-fold
x |> f |> g   ↦ (|> (|> x f) g)  # Left-associative
data ~> func  ↦ (~> data func)   # Stream transform

# Sections and structures
if p : body   ↦ (if p {body})    # Section
point · 10 20 ↦ (point [10 20])  # Structure
x; y          ↦ {x y}

# Abstract structs
[draw:? bounds:[0 0]]  ↦ Abstract struct with hole

# Binary patterns
<[version:4 data:binary]>  ↦ Binary pattern with fields

# String interpolation (template)
s"Hi ${name}" ↦ (s "Hi ${name}")  # Template processor

# Memory marks
&input |> process      ↦ (|> (& input) process)
share&data |> use      ↦ (|> (share& data) use)

# Arena context
arena temp(size) : body ↦ (arena temp size {body})
```

---
*End of GLISP – Parser Description*
