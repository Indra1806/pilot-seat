> **Mode:** Book
> **Pilot-Seat Standard**

# Introduction
WebAssembly (often shortened to **Wasm**) is a low-level binary code format that runs with near-native speed in modern web browsers. It is not a language you write by hand; instead, it is a compilation target. Developers write high-performance programs in languages like C, C++, Rust, or Go, and compile them into Wasm files. The browser can then run these files alongside JavaScript to handle intensive workloads like 3D gaming, video editing, or heavy math calculations.

# Why It Exists
JavaScript was created in 1995 to handle simple tasks like button clicks and form checks. Over the years, JavaScript engines became incredibly fast, but JavaScript is still a dynamically-typed, interpreted language. This means the browser must spend valuable CPU cycles figuring out types and translating code at runtime. When developers tried to build intensive browser applications—like 3D games, CAD design tools, or video editors—JavaScript's execution overhead caused frames to drop, lag, and freeze. Engineers created WebAssembly in 2017 as a lightweight, pre-compiled binary format that executes instructions directly on the computer's CPU with almost no translation overhead.

# Problem It Solves
WebAssembly solves the browser performance bottleneck for resource-intensive tasks.

### Before WebAssembly:
- Complex calculations (like image compression, physics engines, or video effects) had to be sent to a backend server to process, which was slow and expensive.
- Running massive C++ desktop programs in a web browser was impossible without rewriting the entire program in JavaScript.

### After WebAssembly:
- Developers compile existing desktop programs (written in C++ or Rust) directly into Wasm, and they run inside a webpage immediately.
- Heavy computational work happens directly on the user's computer inside the browser, saving server costs and removing network delay.

# Core Concepts
To understand WebAssembly, you need to understand three core ideas:

1. **Compilation Target:** You do not write Wasm code. You write code in a language like Rust or C++, and use a compiler tool (like Emscripten or `wasm-pack`) to translate your code into a `.wasm` file.
2. **The Stack Machine:** WebAssembly runs as a virtual stack machine. It processes instructions by pushing values onto a stack and popping them off to perform operations, which is incredibly fast for CPU chips to process.
3. **Linear Memory:** WebAssembly does not use JavaScript's garbage collector. It stores data in a single, raw array of bytes called "linear memory". Wasm reads and writes to this memory space directly, which is extremely fast but requires careful management.

# Architecture / Components
WebAssembly runs in the browser inside the same security sandbox as JavaScript.

```text
       [ Browser Engine Sandbox ]
       ┌────────────────────────┐
       │  [ JavaScript Engine ] │ <─── Shared Web APIs
       │           ▲            │      (DOM, Console, Fetch)
       │           │ (Imports/  │
       │           ▼  Exports)  │
       │  [ WebAssembly VM ]    │
       │  (Runs compiled .wasm) │
       │           ▲            │
       │           ▼            │
       │    [ Linear Memory ]   │ (Raw bytes array)
       └────────────────────────┘
```

- **WebAssembly Virtual Machine:** The secure execution environment inside the browser that runs compiled Wasm instructions.
- **Linear Memory:** An ArrayBuffer containing the raw bytes accessible by the Wasm program.
- **JavaScript Wrapper:** JavaScript is used to load, compile, and run the `.wasm` file, and pass data back and forth to it.

# Workflow
How WebAssembly goes from source code to browser execution:

```text
Step 1: Write a high-performance program in C++, Rust, or Go.
                             ↓
Step 2: Run the compiler to generate a binary `.wasm` file.
                             ↓
Step 3: In your JavaScript file, use `fetch` to download the `.wasm` file.
                             ↓
Step 4: Use `WebAssembly.instantiate` to compile and load it in the browser VM.
                             ↓
Step 5: Call the Wasm functions from JavaScript just like normal JS functions.
```

# Real World Examples
Think of WebAssembly as a **Formula 1 racing car** and JavaScript as a **spacious family SUV**.
- JavaScript is like a family SUV. It is versatile, easy to drive, has room for groceries, and is perfect for running everyday errands (rendering forms, handling buttons, fetching standard API data).
- WebAssembly is like a Formula 1 racing car. It has a tiny, cramped cockpit, has no space for grocery bags, is extremely difficult to build, but when you put it on a racetrack (heavy mathematics or graphics rendering), it performs at mind-blowing, native speeds.
- You do not replace your SUV with a racing car; you use the SUV for daily driving and bring in the racing car to win the speed race.

# Implementation
Here is a high-level view of how you compile a simple Rust function and run it in the browser using JavaScript:

### 1. Write the code in Rust (`lib.rs`)
```rust
// A Rust function compiled to Wasm
#[no_mangle]
pub extern "C" fn calculate_fibonacci(n: i32) -> i32 {
    if n <= 1 { return n; }
    return calculate_fibonacci(n - 1) + calculate_fibonacci(n - 2);
}
```

### 2. Compile it to Wasm (Outputs `math.wasm`)
*(Run compiler tool: `cargo build --target wasm32-unknown-unknown`)*

### 3. Load and execute the Wasm file in JavaScript
```javascript
// Fetch and load the compiled binary file
async function loadWasm() {
  const response = await fetch('math.wasm');
  const bytes = await response.arrayBuffer();
  
  // Instantiate the Wasm module
  const wasmInstance = await WebAssembly.instantiate(bytes);
  
  // Extract our exported Rust function
  const fibonacci = wasmInstance.instance.exports.calculate_fibonacci;
  
  // Run the high-speed function directly!
  const result = fibonacci(40);
  console.log("Fibonacci result: ", result);
}

loadWasm();
```

# Best Practices
- **Use the Right Tool for the Job:** Do not compile your entire website into WebAssembly. Use Wasm *only* for performance-critical bottlenecks (like calculations, video encoding, or game loops). Use JavaScript and HTML for normal layout and UI.
- **Minimize JS-to-Wasm Boundary Crossings:** Passing data back and forth between JavaScript and WebAssembly requires copying data into Wasm's linear memory. This crossing is relatively slow. Pass data in large batches rather than constantly crossing the boundary.
- **Use Streaming Compilation:** Use `WebAssembly.instantiateStreaming` to compile the Wasm binary while it is still downloading over the network, ensuring the fastest possible page load times.

# Industry Standards
WebAssembly is widely adopted by major web applications. Figma uses WebAssembly to run its C++ interface engine at 60fps in the browser. Adobe uses Wasm to run Photoshop in the browser. Zoom uses Wasm to process real-time video backgrounds.

# Common Mistakes
- **Assuming Wasm has DOM Access:** WebAssembly cannot access the HTML DOM directly. If a Wasm function wants to change the text of a paragraph, it must tell JavaScript to do it.
- **Compiling Garbage-Collected Languages Unoptimized:** Compiling languages like Go or Java requires bundling their entire garbage collection engine inside the `.wasm` file, making the file download size massive. Prefer languages like Rust or C++ which do not require garbage collection.

# Security & Performance Considerations
- **Sandbox Security:** WebAssembly runs inside the same secure browser sandbox as JavaScript. It cannot access the user's hard drive or local files directly; it only has access to the memory space explicitly given to it by JavaScript.
- **Memory Management Risks:** Because Wasm uses linear memory without automatic garbage collection, bugs in C++ compiled code (like buffer overflows) can corrupt Wasm memory, though they cannot break out of the browser sandbox to infect the host computer.

# Related Technologies
- **Rust:** The most popular modern programming language compiled to WebAssembly due to its lack of runtime and memory safety guarantees.
- **Emscripten:** A compiler toolchain used to compile C and C++ projects into WebAssembly.
- **AssemblyScript:** A language with TypeScript-like syntax that compiles directly to WebAssembly.

# Summary

## What We Learned
- WebAssembly is a binary bytecode format designed to run intensive tasks in the browser at native speed.
- It acts as a compilation target for high-performance languages like Rust, C++, and Go.
- Wasm runs alongside JavaScript in a secure sandbox, sharing Web APIs and data through linear memory buffers.

## Key Takeaways
- Use WebAssembly for heavy CPU workloads (gaming, image/video editing, complex math).
- Keep communication across the JS-to-Wasm boundary minimal to avoid overhead.

# Keywords
- WebAssembly
- Wasm
- Bytecode
- Compilation Target
- Virtual Machine
- Linear Memory
- Rust
- Performance

# Glossary

| Term | Meaning |
|---|---|
| Bytecode | Low-level instruction set designed for efficient execution by software interpreters or VMs. |
| Compilation Target | A format that compiler tools translate higher-level code into, rather than writing it directly. |
| Linear Memory | A flat, contiguous array of raw bytes used by WebAssembly to store data. |
| Sandboxing | A security practice that isolates running programs, preventing them from accessing system resources outside their allocated space. |
| Emscripten | A toolchain compiler used to translate C/C++ code into WebAssembly. |

## Next Recommended Chapters
- 08-Browser-Internals.md
- 11-Frontend-Build-Tools.md
