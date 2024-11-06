## Profiling Tools 🍴

| Tool | Concept | Pros | Cons | Use Cases |
| --- | --- | --- | --- | --- |
| **`perf`** | Linux-based profiling tool that uses hardware counters and sampling for system-level profiling. | Very flexible, supports many event types (CPU, memory, cache).  System-wide profiling, low-level control. | Complex setup; requires Linux knowledge.  May have high overhead with detailed profiling. | System-level profiling, low-level hardware analysis, profiling OS or kernel code. |
| **`samply`** | A sampling profiler for generating `flamegraph`s with low overhead, particularly useful with Rust. | Simple to set up, integrates well with Rust.  Low overhead due to sampling. | Limited to CPU usage and `flamegraph` generation.  Less detailed than tools like `perf`. | Quick CPU profiling, Rust application `flamegraph`s, identifying CPU-bound sections. |
| **`flamegraph`** | Visualization technique that shows CPU usage over time in a stack structure, highlighting hotspots. | Intuitive, visual representation of function calls and hot spots.  Shows cumulative time per function. | Requires sampling data (e.g., from `perf`, `samply`).  Doesn’t capture memory, I/O, or network usage. | Visualize CPU-bound code sections, analyze CPU hotspots, optimize function call structure. |
| **`valgrind`** | Tool suite for memory and threading error detection, profiling, and debugging. | Excellent memory and error analysis (e.g., leaks, invalid accesses).  Versatile with multiple tools. | High overhead; can slow down programs significantly.  Limited to Linux; some tools (e.g., `DHAT`) are for heap memory only. | Memory leak detection, debugging threading issues, profiling memory usage, memory allocation patterns. |
| **`tokio-console`** | Observability tool for `async` Rust applications using the `tokio` runtime. | Real-time view of `async` tasks.  Insights into task lifetimes, resource usage, and task dependencies. | Limited to applications using `tokio`.  Limited analysis features compared to general profilers. | Profiling `async` tasks, monitoring task concurrency, debugging `async` Rust applications using `tokio`. |
| **`criterion.rs`** | Benchmarking library for Rust with statistical analysis to track performance changes. | Accurate statistical benchmarking, configurable options.  Generates visualizations for easy comparison. | Not for system profiling or low-level metrics.  Limited to function-level benchmarks in Rust. | Micro-benchmarking functions, regression testing, comparing optimization changes in Rust functions. |
| **coz** | Causal profiler that estimates the performance impact of potential optimizations by simulating code speedups. | Unique causal profiling approach; shows which optimizations would have the most impact.  Easy to use. | Requires code annotations for best results.  Limited to causal profiling; lacks detailed breakdown of memory and I/O. | Estimating potential optimizations, pinpointing high-impact code sections, profiling parallel code. |

## Use Case Categories 🍪

| Use cases | Tools | Purpose |
| --- | --- | --- |
| Sampling Profiling | `Flamegraph`, `Samply` | Provide `flamegraph` visualizations that focus on CPU-bound functions |
| `Async` Task Monitoring | `Tokio-Console` |  Tailored for `async` Rust applications, offering a view into task management and concurrency |
| Benchmarking | `Criterion.rs` | Ideal for precise, function-level benchmarks in Rust, focusing on tracking performance changes over time |
| Low-Level Profiling | `Perf`, `Valgrind`, `Coz` | Offer in-depth profiling for memory, CPU, and low-level system details |

### 1. **Perf**

**Concept**: `perf` is a powerful Linux-based profiling tool that leverages hardware counters and sampling to collect low-level data on CPU, memory, and cache usage. It’s a versatile tool widely used for system-level profiling and is highly customizable for various profiling scenarios, from basic CPU usage to detailed cache performance.

**How It Works**: `perf` uses hardware counters to track events on the CPU and memory. These events can be general, like CPU cycles or task scheduling, or hardware-specific, such as cache hits, branch misses, and page faults. `perf` periodically samples these events, building a statistical profile of where time is being spent in the program.

**Setup**:

1. Install `perf` on Linux:
    
    ```bash
    sudo apt install linux-tools-common linux-tools-generic linux-tools-$(uname -r)
    ```
    
2. To profile a program, run `perf` with sampling:
    
    ```bash
    perf record -g -- ./your_program
    ```
    
    This captures the program's performance data, storing it in a `perf.data` file.
    
3. To view the recorded data in a human-readable report:
    
    ```bash
    perf report
    ```
    
4. To generate flamegraphs from `perf` data, use:
    
    ```bash
    perf script | FlameGraph/flamegraph.pl > flamegraph.svg
    ```
    
    (You’ll need the [FlameGraph](https://github.com/brendangregg/FlameGraph) tool for this step).
    

**Use Cases**: `perf` is ideal for system-level profiling and low-level hardware analysis. It’s particularly useful for identifying bottlenecks at the OS level, profiling CPU and memory usage, and gaining insights into hardware-specific events, like cache usage and branch mispredictions.

---

### 2. **Samply**

**Concept**: Samply is a sampling-based profiler designed for generating flamegraphs in Rust applications. It captures stack traces at regular intervals to generate flamegraphs, providing a straightforward view of where CPU time is concentrated in the code. Its low overhead makes it ideal for profiling without impacting performance heavily.

**How It Works**: Samply takes snapshots of the application’s stack trace at fixed intervals, building a statistical model of which functions are most CPU-intensive. It doesn’t instrument each function call, so it incurs minimal runtime overhead.

**Setup**:

1. Install Samply using Cargo:
    
    ```bash
    cargo install samply
    ```
    
2. Run your Rust program with Samply:
    
    ```bash
    samply record -- cargo run --release
    ```
    
    This command starts profiling and outputs a flamegraph in SVG format by default.
    
3. You can also specify a filename for the flamegraph:
    
    ```bash
    samply record --output flamegraph.svg -- cargo run --release
    ```
    

**Use Cases**: Samply is suitable for quick CPU profiling and generating flamegraphs for Rust applications. It’s a good choice for identifying CPU-bound sections, analyzing function call structure, and understanding stack depth without deep profiling of other resources.

---

### 3. **Flamegraph**

**Concept**: Flamegraph is a visualization technique for profiling data, showing CPU usage in a stacked format with functions arranged by the call hierarchy. The width of each function block indicates the cumulative time spent in that function, allowing easy identification of hotspots.

**How It Works**: Flamegraphs are generated from sampled stack traces, such as those captured by `perf` or Samply. Each function call is represented by a block, with the function name at the bottom. The X-axis shows different call paths, while the Y-axis shows the depth of the call stack.

**Setup**:

1. Clone the [FlameGraph](https://github.com/brendangregg/FlameGraph) repository:
    
    ```bash
    git clone https://github.com/brendangregg/FlameGraph
    ```
    
2. Generate a flamegraph from `perf` data:
    
    ```bash
    perf script | FlameGraph/flamegraph.pl > flamegraph.svg
    ```
    
3. Open the SVG file in a browser to explore the flamegraph interactively.

**Use Cases**: Flamegraphs are excellent for visualizing CPU usage, identifying performance bottlenecks, analyzing function call structures, and understanding which parts of the code consume the most CPU time.

---

### 4. **Valgrind**

**Concept**: Valgrind is a suite of tools for profiling memory, threading, and CPU usage. It’s best known for its `Memcheck` tool, which detects memory errors, but it also includes `Callgrind` for call graphs, `Cachegrind` for cache performance, and `DHAT` for heap memory usage.

**How It Works**: Valgrind runs the target program in a virtualized environment that intercepts memory operations. This enables it to detect issues like memory leaks, invalid accesses, and other bugs. Tools like `Callgrind` and `Cachegrind` gather data on function call costs and cache efficiency by monitoring memory access patterns.

**Setup**:

1. Install Valgrind:
    
    ```bash
    sudo apt install valgrind
    ```
    
2. Run a memory check with `Memcheck`:
    
    ```bash
    valgrind --tool=memcheck ./your_program
    ```
    
3. Use `Callgrind` for call graph profiling:
    
    ```bash
    valgrind --tool=callgrind ./your_program
    ```
    
4. Analyze call graphs with KCachegrind:
    
    ```bash
    kcachegrind callgrind.out.<pid>
    ```
    
5. Use `Cachegrind` for cache usage analysis:
    
    ```bash
    valgrind --tool=cachegrind ./your_program
    ```
    

**Use Cases**: Valgrind is excellent for memory leak detection, threading error analysis, profiling memory usage, and understanding cache performance. It’s also helpful in analyzing the performance of specific functions with `Callgrind`.

---

### 5. **tokio-console**

**Concept**: `tokio-console` is an observability tool designed for async Rust applications using the `tokio` runtime. It provides real-time insights into async task execution, including task lifetimes, resource usage, and task dependencies, helping you monitor concurrency in async code.

**How It Works**: `tokio-console` hooks into `tokio` tasks to track async execution in real time. It shows task start times, durations, and dependencies, making it easier to debug and optimize async applications.

**Setup**:

1. Add `tokio-console` as a dependency in `Cargo.toml`:
    
    ```toml
    [dependencies]
    tokio = { version = "1", features = ["full"] }
    tokio-console = "0.1"
    ```
    
2. Run your program with `tokio-console` enabled:
    
    ```bash
    RUST_LOG=console=trace cargo run --release
    ```
    
3. Access the `tokio-console` view to monitor async tasks, concurrency, and task dependencies in real time.

**Use Cases**: `tokio-console` is ideal for profiling async tasks in Rust, monitoring task concurrency, and debugging task dependencies and resource usage in applications that rely heavily on async programming with `tokio`.

---

### 6. **Criterion.rs**

**Concept**: Criterion.rs is a benchmarking library for Rust that provides statistical insights and visualization for performance testing. It’s designed to produce reliable results by applying statistical analysis to function execution times, helping track performance improvements or regressions.

**How It Works**: Criterion.rs repeatedly executes a function and measures its execution time, using statistical methods to filter out noise and cold-start effects. It generates graphs and reports to visualize performance trends over time.

**Setup**:

1. Add Criterion.rs to `dev-dependencies` in `Cargo.toml`:
    
    ```toml
    [dev-dependencies]
    criterion = "0.4"
    ```
    
2. Write benchmarks in the `benches` folder:
    
    ```rust
    use criterion::{black_box, criterion_group, criterion_main, Criterion};
    
    fn fibonacci(n: u64) -> u64 { /* ... */ }
    
    fn benchmark(c: &mut Criterion) {
        c.bench_function("fibonacci 20", |b| b.iter(|| fibonacci(black_box(20))));
    }
    
    criterion_group!(benches, benchmark);
    criterion_main!(benches);
    ```
    
3. Run benchmarks:
    
    ```bash
    cargo bench
    ```
    
4. View HTML reports in `target/criterion`.

**Use Cases**: Criterion.rs is used for benchmarking specific functions, regression testing, tracking performance improvements over time, and providing reliable, statistical insights into Rust function performance.

---

### 7. **Coz**

**Concept**: Coz is a causal profiler that simulates potential performance improvements by selectively slowing down code. This allows it to measure the performance impact of potential optimizations and identify sections of code where optimizations would yield the greatest performance gains.

**How It Works**: Coz runs the program and introduces artificial pauses in various parts of the code. By simulating speedups elsewhere, Coz creates a causal relationship between code speed and overall performance, helping identify high-impact code sections.

**Setup**:

1. Install Coz:
    
    ```bash
    git clone https://github.com/plasma-umass/coz.git
    cd coz
    make
    sudo make install
    ```
    
2. Add progress points in code:
    
    ```rust
    #[cfg(feature = "coz")]
    macro_rules! progress_point {
        ($name:expr) => {
            unsafe { coz::progress($name); }
        };
    }
    for item in items {
        progress_point!("ProcessingItem");
        // Code to profile
    }
    ```
    
3. Run the program with Coz:
    
    ```bash
    coz run --- ./your_program
    ```
    

**Use Cases**: Coz is useful for estimating the impact of optimizations, identifying high-impact code sections, and prioritizing optimization efforts, especially in parallel or multi-threaded applications.
