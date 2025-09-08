# GLISP – Memory Description

> **Authority:** This document is the authoritative technical description. No abstractions or proposals beyond what is written here.

---

## Overview

GLISP's memory model embodies the language's core principle of **progressive disclosure without ceremony**. The system provides three tiers of memory management that allow developers to start simple and add precision only when needed, while maintaining the same source code across wildly different target platforms.

Memory management follows GLISP's "everything is struct transformation" philosophy - memory operations are simply another dimension of data transformation, integrated seamlessly with the language's existing computational contexts (execution, reactive, async).

The memory model recognizes that different computational domains have different memory requirements. A web frontend needs convenience and safety. A game engine needs predictable performance. An embedded system needs explicit control. GLISP's memory marks provide the vocabulary to express these requirements while adapting gracefully to each target platform's capabilities.

---

## Core Memory System

### Three-Tier Architecture

GLISP provides three levels of memory management that form a natural progression:

**Tier 1: Automatic Management (Default)**
```glisp
# No memory marks - platform manages everything automatically
users: fetch_users()
cache: expensive_computation()
process(users, cache)
# Platform chooses optimal automatic strategy
```

**Tier 2: Ownership Marks (&)**
```glisp
# Performance-oriented ownership transfer
&data |> expensive_transform |> &result
```

**Tier 3: Explicit Strategy Marks**
```glisp
# Full memory control vocabulary
acquire&buffer(1024) |> process(share&data, borrow&config) |> release&buffer
```

### Memory Mark Syntax

Memory marks use GLISP's existing prefix glue transformer pattern, ensuring syntactic consistency:

```glisp
# Prefix glue pattern: mark&target
&data                  # Ownership transfer (default)
share&expensive_obj    # Reference-counted sharing
borrow&shared_state    # Temporary access
copy&data             # Explicit copy
acquire&buffer(size)   # Manual allocation
release&buffer         # Manual deallocation
pin&critical_data     # Memory pinning
unpin&data           # Memory unpinning

# Parser transformation
&data       ↦ (& data)
share&obj   ↦ (share& obj)
borrow&ref  ↦ (borrow& ref)
```

---

## Memory Strategies

### Automatic Strategies (Tier 1)

When no memory marks are specified, GLISP delegates to the platform's optimal automatic memory management:

```glisp
# Automatic memory management - zero configuration
data: expensive_computation()
cache.store("key", data)
background_process(data)
# Platform handles sharing, cleanup, and optimization automatically
```

**Platform Adaptation:**
- Each target platform uses its natural memory management approach
- JavaScript/Python: Their native memory management
- Rust/Swift: Their preferred automatic strategies
- C/C++: Platform-chosen automatic management
- Development vs Production: Platform may choose different strategies for debugging vs performance

The key principle: **the platform knows best** what automatic strategy to use.

### Ownership Strategy (Tier 2)

The `&` memory mark indicates ownership transfer, providing zero-copy performance where supported:

```glisp
# Ownership transfer through pipeline
fetch_data() |>
  validate |>
  &validated_data |>     # Transfer ownership
  expensive_transform |>
  &result |>             # Transfer ownership
  save_to_database

# Function signatures with ownership
process_stream: [&input &config] => &result
  # Takes ownership of input and config, returns owned result
```

**Target Adaptation:**
- **Rust/C**: Real move semantics, zero-copy performance
- **JavaScript/Python**: Graceful degradation to platform-appropriate behavior
- **Swift**: Move semantics where possible
- **Go**: Compiler optimizations, escape analysis

### Explicit Strategies (Tier 3)

Rich vocabulary for specific memory management needs:

```glisp
# Reference counting for sharing
expensive_data() |> share&result |> multiple_consumers

# Borrowing for temporary access
large_dataset |> process |> share&result |>
[data] => {
  ui_display(borrow&data)      # UI just needs to read
  log_summary(borrow&data)     # Logging just needs to read
  analytics(copy&data)         # Analytics needs independent copy
  &data                        # Return ownership
}

# Manual allocation for control
acquire&critical_buffer(1024) |>
[buffer] => {
  pin&important_data           # Lock in memory
  process_with_buffer(&data, borrow&buffer)
  unpin&important_data
  release&buffer
}
```

---

## Target-Adaptive Behavior

### Write Once, Run Everywhere

The same GLISP source code adapts to different target platforms' memory management capabilities:

```glisp
# Single source file
high_performance_processor: [&input] =>
  &input |>
  validate |>
  &validated |>
  share&expensive_result |>
  cache.store |>
  copy&final_result

# Each platform interprets memory marks optimally:
# - Rust: Real zero-copy with reference counting for sharing
# - JavaScript: Platform-native memory handling
# - C: Platform-chosen strategy (pointer transfer, reference counting, etc.)
# - Python: Native object model with appropriate behavior
```

### Memory Mark Support Matrix

```glisp
#memory_targets: [
  # Each platform reports its capabilities
  # "auto" means platform chooses optimal fallback for unsupported marks
  
  rust: [
    supports: ["&" "share&" "borrow&" "copy&" "acquire&" "release&" "pin&"]
    auto_strategy: "platform_optimal"
  ]
  
  javascript: [
    supports: []  # Platform handles all through auto strategy
    auto_strategy: "native"
  ]
  
  c: [
    supports: ["&" "share&" "acquire&" "release&" "pin&"]
    auto_strategy: "platform_chosen"
  ]
  
  python: [
    supports: []  # Platform handles all through auto strategy
    auto_strategy: "native"
  ]
]
```

---

## Integration with Computational Contexts

### Async + Memory Marks

```glisp
# Async operations with memory management
async val fetch_and_process: [&url] =>
  response: <- fetch(&url)
  data: <- response.json()
  share&processed: <- expensive_async_transform(data)
  
  # Parallel async operations sharing data
  async parallel [
    cache.store("key", share&processed)
    analytics.track(borrow&processed)
    notifications.send(copy&processed)
  ]
  
  &processed  # Return ownership to caller
```

### Handlers + Memory Marks

```glisp
# Handler implementation with memory management
MemoryHandler: [
  *allocate: [size] => acquire&pool_buffer(size)
  *deallocate: [buffer] => release&buffer
]

with_handler MemoryHandler[] :
  process_with_handlers: [&data] =>
    buffer: perform("allocate", data.size())
    result: process_in_buffer(&data, borrow&buffer)
    perform("deallocate", buffer)
    &result
```

### Reactive Streams + Memory Marks

```glisp
# Stream processing with memory optimization
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

### Arena Context + Memory Marks

Arena provides scoped memory allocation that works with memory marks:

```glisp
# Arena declaration creates scoped memory context
arena frame_memory(megabyte * 10) :
  temp_data: alloc(size)        # Arena allocation
  processed: process(&temp_data) # Use within arena
  copy&processed                 # Must copy to escape arena

# Parser transformation
arena frame_memory(megabyte * 10) : body
↦ (arena frame_memory (* megabyte 10) {body})

# Arena with handlers
arena request_processing(kilobyte * 500) :
  with_handler [*alloc: arena_alloc] :
    &request_data |> process |> copy&result  # Copy before arena cleanup
```

---

## Configuration

### Compilation-Time Memory Mode

```glisp
# Default: automatic platform-optimal strategy
glisp compile app.glisp                  # Platform chooses best strategy
glisp compile --memory=auto app.glisp    # Explicit auto (same as default)

# Development vs production
glisp dev app.glisp                      # Platform chooses debug-friendly strategy
glisp build app.glisp                    # Platform chooses performance strategy

# Force specific strategy if needed (platform-dependent)
glisp compile --memory=force:arc app.glisp   # Request ARC if available
glisp compile --memory=force:gc app.glisp    # Request GC if available
```

### Minimal Source Configuration

```glisp
# Most code needs zero memory configuration
# Memory marks work automatically

# Default is always "auto"
# No configuration needed unless forcing specific behavior

# Optional tuning for edge cases only
#memory: "auto"                  # Explicit auto (default)

# Advanced tuning (rarely needed, platform-specific)
#memory: [
  mode: "auto"
  hints: ["low_latency" "high_throughput"]  # Platform hints
]
```

### Target-Specific Compilation

```glisp
# Same source, different targets with platform-appropriate strategies
glisp build --target=rust app.glisp          # Rust's optimal strategy
glisp build --target=javascript app.glisp    # JavaScript's native management
glisp build --target=c app.glisp             # C platform choice
glisp build --target=wasm app.glisp          # WASM's linear memory model
```

---

## Examples

### Web Service with Multi-Tier Memory

```glisp
# Web service using all three memory tiers
WebService: [
  # Tier 1: Automatic for configuration and routing
  config: [port:8080 host:"localhost"]
  routes: []
  middleware: []
  
  # Tier 2: Ownership for request processing performance
  handle_request: [&request] =>
    &request |>
    parse_headers |>
    &parsed_request |>
    apply_middleware |>
    route_request |>
    &response |>
    add_standard_headers
  
  # Tier 3: Explicit strategies for resource management
  process_upload: [&request] =>
    if request.content_length > megabyte :
      # Large uploads: manual memory management
      buffer: acquire&upload_buffer(request.content_length)
      result: process_large_upload(&request, borrow&buffer)
      release&buffer
      &result
    else :
      # Small uploads: automatic management
      process_small_upload(&request)
]
```

### Data Processing Pipeline

```glisp
# Scientific data processing with memory optimization
DataProcessor: [
  # Different memory strategies for different data types
  process_dataset: [&raw_data] =>
    # Ownership transfer for large transformations
    &raw_data |>
    validate_format |>
    &validated |>
    expensive_normalization |>
    &normalized |>
    
    # Sharing for multiple analysis passes
    [data] => {
      statistical_analysis: share&data |> compute_statistics
      ml_features: share&data |> extract_features  
      visualizations: borrow&data |> generate_plots
      
      # Combine results with copies for independence
      results: [
        stats: copy&statistical_analysis
        features: copy&ml_features  
        plots: copy&visualizations
      ]
      
      &results  # Return ownership of combined results
    }
]
```

### Game Engine Memory Management

```glisp
# Game engine with performance-critical memory handling
GameEngine: [
  # Manual allocation for frame-critical resources
  render_buffer: acquire&gpu_buffer(screen_width * screen_height * 4)
  audio_buffer: acquire&audio_buffer(sample_rate * channels)
  
  # Shared game state
  game_state: share&initial_state()
  
  frame_update: [delta_time] =>
    # Borrow for read-only operations
    entities: borrow&game_state.entities
    camera: borrow&game_state.camera
    
    # Ownership for transformations
    updated_entities: entities |>
      map([entity] => &entity |> update_physics(delta_time) |> &updated)
    
    # Update shared state
    game_state.entities = updated_entities
    
    # Manual buffer management for rendering
    clear_buffer(borrow&render_buffer)
    render_scene(borrow&updated_entities, borrow&camera, borrow&render_buffer)
    present_buffer(&render_buffer)
  
  cleanup: [self] =>
    release&self.render_buffer
    release&self.audio_buffer
]
```

### Cross-Platform Application

```glisp
# Application that runs on multiple platforms with appropriate optimizations
CrossPlatformApp: [
  initialize: [] =>
    # Automatic memory management for UI state
    ui_state: create_ui_state()
    
    # Performance-critical data processing
    process_user_data: [&input] =>
      &input |>
      validate_input |>
      &validated |>
      expensive_computation |>
      share&result |>              # Share result across UI components
      [data] => {
        ui_update(borrow&data)     # UI reads data
        cache.store(share&data)    # Cache shares reference
        analytics(copy&data)       # Analytics gets independent copy
        &data                      # Return ownership
      }
]

# Compilation targets:
# glisp build --target=electron app.glisp        # JavaScript with Node.js
# glisp build --target=tauri app.glisp           # Rust with web frontend  
# glisp build --target=flutter app.glisp         # Dart compilation
# glisp build --target=native app.glisp          # C/Rust for maximum performance

# Same source code, optimal memory management for each platform
```

### Arena-Based Request Processing

```glisp
# Web server with arena for request handling
RequestHandler: [
  process_request: [request] =>
    # Arena for temporary request processing
    arena request_arena(megabyte * 5) :
      # Parse request in arena memory
      parsed: parse_json(request.body)
      
      # Validate with temporary structures
      validation_result: validate_schema(parsed)
      
      # Transform data
      transformed: business_logic(validation_result)
      
      # Must copy result before arena cleanup
      copy&transformed
  
  # Batch processing with arena
  process_batch: [requests] =>
    arena batch_arena(megabyte * 50) :
      # Process all requests in arena
      results: requests |> map([req] =>
        temp_result: process_single(req)
        enriched: enrich_with_metadata(temp_result)
        copy&enriched  # Copy each result
      )
      results  # All copied, safe to return
]
```

---

## Performance Characteristics

### Memory Mark Performance Impact

| Memory Mark | Platform Behavior |
|-------------|-------------------|
| No mark (auto) | Platform optimal automatic management |
| `&data` | Zero-copy where possible, else platform fallback |
| `share&data` | Reference counting where available, else platform fallback |
| `borrow&data` | Borrowing where supported, else platform fallback |
| `copy&data` | Explicit copy using platform mechanism |
| `acquire&buffer` | Manual allocation where supported |
| `release&buffer` | Manual deallocation where supported |
| `pin&data` | Memory pinning where supported |
| `unpin&data` | Memory unpinning where supported |

### Arena Performance

| Operation | Platform Behavior |
|-----------|-------------------|
| Arena creation | Platform-optimal arena implementation |
| Arena alloc | Fast allocation from arena pool |
| Arena reset | Efficient bulk deallocation |
| Arena destroy | Complete cleanup |

### Development vs Production

**Development Mode (--dev):**
- Platform chooses debug-friendly automatic strategy
- Memory marks provide documentation and performance hints
- Enhanced error messages for memory mark misuse
- Memory leak detection and allocation tracking where available

**Production Mode (--build):**
- Platform chooses performance-optimal automatic strategy
- All supported memory marks honored for optimization
- Minimal runtime overhead
- Maximum performance extraction from target platform

---

## Best Practices

### Progressive Adoption

1. **Start with automatic management**: Write code without memory marks
2. **Profile for bottlenecks**: Identify performance-critical paths
3. **Add ownership marks**: Use `&` for zero-copy in hot paths
4. **Refine with explicit strategies**: Use specific memory marks where needed

### Memory Mark Guidelines

```glisp
# Clear ownership transfer in pipelines
&input |> validate |> &validated |> process |> &result

# Explicit sharing when multiple consumers needed
expensive_data() |> share&result |> [data] => {
  consumer1(borrow&data)
  consumer2(borrow&data)
  consumer3(copy&data)    # Independent copy when needed
}

# Arena for batch operations with clear lifecycle
arena request_processing(megabyte) :
  temp_data: alloc(large_size)
  intermediate: process(temp_data)
  copy&final_result       # Copy before arena cleanup

# Arena context for frame-based processing
arena frame_memory(megabyte * 10) :
  entities: load_frame_entities()
  physics: compute_physics(entities)
  render: prepare_render(physics)
  copy&render             # Copy render data before frame end

# Consistent strategy within processing unit
share&config |> process_with_shared_config |> share&result
```

### Arena Usage Patterns

**Frame-Based Processing:**
```glisp
# Game engines, real-time systems
arena frame_arena(megabyte * 20) :
  # Process entire frame in arena
  frame_data: compute_frame()
  copy&frame_output       # Copy results before reset
```

**Request Processing:**
```glisp
# Web servers, API handlers
arena request_arena(kilobyte * 500) :
  # Process request with temporary allocations
  parsed_data: parse_request()
  processed: handle_request(parsed_data)
  copy&response           # Copy response before cleanup
```

**Batch Compilation:**
```glisp
# Compilers, document processors
arena compilation_arena(megabyte * 50) :
  # Process compilation phase
  atoms: lexical_analysis()
  ast: parsing(atoms)
  copy&analyzed_ast       # Copy results to persistent memory
```

**Scientific Computing:**
```glisp
# Numerical simulations, data processing
arena computation_arena(gigabyte) :
  # Large temporary arrays for computation
  matrices: allocate_work_matrices()
  result: compute_simulation(matrices)
  copy&final_data         # Extract final results
```

### Target Considerations

- **Trust the platform's automatic strategy** - It knows its runtime best
- **Use `&` liberally on platforms with move semantics** - Rust, C, Swift benefit
- **Focus on algorithmic optimization for managed platforms** - JavaScript, Python, Java
- **Manual strategies for embedded/real-time** - Use `acquire&`/`release&` for deterministic behavior
- **Profile across targets** - Same code may have different bottlenecks on different platforms

---

## Conclusion

GLISP's memory model demonstrates the language's core philosophy: **simple rules enabling complex behaviors**. The three-tier system provides a natural learning progression while maintaining source code compatibility across radically different target platforms.

Memory marks are not afterthoughts or bolt-on features - they are fundamental vocabulary that integrates seamlessly with GLISP's existing computational contexts. Whether writing a simple web application or a performance-critical game engine, developers can express their memory management intent clearly while letting the compiler adapt to each target platform's capabilities.

The memory model embodies GLISP's commitment to **progressive disclosure without ceremony**: start simple with automatic management, add performance with ownership marks, achieve full control with explicit strategies. All while maintaining the same readable source code that adapts gracefully to any target platform.

The key insight: **platform trust** - GLISP provides the vocabulary for memory intent, but trusts each platform to implement that intent optimally. This aligns perfectly with the language's philosophy of no moral judgments about implementation choices.

---

*End of GLISP – Memory Description*
