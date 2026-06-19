> **Mode:** Book
> **Pilot-Seat Standard**

# Introduction
TypeScript is a programming language that is a **strict superset of JavaScript**. This means that any valid JavaScript code is also valid TypeScript code. However, TypeScript adds a powerful extra feature: **Static Type Checking**. It forces developers to specify what *type* of data (like numbers, text, or lists) variables and functions are allowed to hold, catching bugs before the code ever runs in a browser.

# Why It Exists
JavaScript is a dynamically-typed language. This means variables can hold any type of data and change types on the fly. 
For example, a variable can start as a number `5`, and later be changed to text `"hello"`. While this flexibility sounds nice, it is a major source of bugs in large applications. If a developer accidentally passes text into a function that expects a number to perform math, the application won't crash when saving the code; it will crash on a user's computer when they try to check out their shopping cart. Engineers created TypeScript in 2012 to catch these type mismatch bugs early during development.

# Problem It Solves
TypeScript solves the runtime error and lack of code predictability problem in JavaScript.

### Before TypeScript (Plain JavaScript):
- You only find out there is a typo or type mismatch error when you run the code in a browser and try the feature.
- Large teams struggle to know what shape of data (objects) functions expect without reading pages of outdated documentation.

### After TypeScript (Static Types):
- Your code editor checks your types in real-time, pointing out errors with red squiggly lines as you type.
- Developers get rich autocomplete suggestions explaining exactly what properties exist on a data object.

*JavaScript (Will run, but outputs math gibberish at runtime):*
```javascript
function addTax(price) {
  return price * 1.1;
}
addTax("ten dollars"); // Returns NaN (Not a Number) in the browser
```

*TypeScript (Catches the error immediately during editing):*
```typescript
function addTax(price: number): number {
  return price * 1.1;
}
addTax("ten dollars"); // Error: Argument of type 'string' is not assignable to parameter of type 'number'.
```

# Core Concepts
To write TypeScript, you must understand three core concepts:

1. **Type Annotations:** Writing a colon followed by the data type next to variables and function inputs. Basic types include `number`, `string`, `boolean`, and `any` (disables type checking).
2. **Interfaces & Custom Types:** Creating structures that define the exact shape of an object.
   ```typescript
   interface User {
     id: number;
     name: string;
     email: string;
     isAdmin?: boolean; // The '?' means this property is optional
   }
   ```
3. **Compilation:** Web browsers do not understand TypeScript code. TypeScript files (`.ts`) must be run through the TypeScript Compiler (`tsc`), which checks for errors and compiles it down to standard JavaScript (`.js`) files that browsers can read.

# Architecture / Components
The TypeScript system relies on two main components:

```text
               [ TypeScript Code (.ts) ]
                          ↓
      +---------------------------------------+
      |        TypeScript Compiler (tsc)      |
      |                                       |
      |  1. Type Checker  -> Catches errors   |
      |  2. Transpiler    -> Strips type tags |
      +-------------------|-------------------+
                          ↓
        [ Standard JavaScript Code (.js) ]
```

- **Type Checker:** The static analysis engine that reads your code structure and validates that variables follow your defined rules.
- **Transpiler:** The code converter that strips away all TypeScript annotations, leaving behind clean, standard JavaScript.
- **TSConfig File (`tsconfig.json`):** The configuration file containing compiler settings (e.g. how strict the type-checking should be).

# Workflow
How a developer works with TypeScript:

```text
Step 1: The developer writes code in a `.ts` file.
                  ↓
Step 2: The code editor highlights any type errors in real-time.
                  ↓
Step 3: The developer runs the compiler tool (`tsc`).
                  ↓
Step 4: If type errors exist, compilation fails, forcing the developer to fix them.
                  ↓
Step 5: Once error-free, the compiler outputs a clean `.js` JavaScript file.
                  ↓
Step 6: The browser loads the compiled `.js` file and runs it normally.
```

# Real World Examples
Think of TypeScript as a **plumbing safety checklist**.
- Writing JavaScript is like connecting plumbing pipes in a dark basement without looking. You can screw a 2-inch pipe onto a 3-inch socket, and you only find out it doesn't fit when you turn on the main water valve (Runtime) and flood the house.
- TypeScript is like a smart blueprint specification tool. Before you connect any pipes, the tool inspects your selections. If you try to connect a square pipe to a round socket, the tool flashes red, locked, and says: *"This size mismatch will leak. Fix it before we turn on the water."*

# Implementation
Here is how you define custom types and enforce them in a TypeScript application:

```typescript
// 1. Define the shape of a Product object
interface Product {
  id: number;
  title: string;
  price: number;
  inStock: boolean;
}

// 2. Enforce the type on a variable
const laptop: Product = {
  id: 101,
  title: "Ultrabook 15",
  price: 999,
  inStock: true
};

// 3. Enforce types on function arguments and returns
function getProductDiscount(item: Product, discountRate: number): number {
  if (item.price > 500) {
    return item.price * discountRate;
  }
  return 0;
}

// 4. Run the function safely
const savings = getProductDiscount(laptop, 0.1);
console.log(`Saved: $${savings}`);
```

# Best Practices
- **Enable `strict` Mode:** Always set `"strict": true` in your `tsconfig.json`. This turns on the strongest type-safety features, helping you catch the maximum number of potential bugs.
- **Avoid Using `any`:** The `any` type tells TypeScript to stop checking that variable. Using `any` everywhere turns TypeScript back into plain JavaScript, defeating the purpose of using it. Use `unknown` if you genuinely do not know the type of incoming data.
- **Let TypeScript Infer Simple Types:** You do not need to write types for everything. If you write `const isLoggedIn = true;`, TypeScript automatically knows it is a `boolean`. You do not need to write `const isLoggedIn: boolean = true;`.

# Industry Standards
TypeScript is the default language for modern professional frontend web development. Over 90% of new React projects, Angular applications, and NestJS backends are built using TypeScript to ensure code stability and maintainability in large team codebases.

# Common Mistakes
- **Confusing Type-Safety with Runtime Checking:** TypeScript types only exist *during development*. Once compiled to JavaScript, all type annotations are deleted. If your backend API returns bad data at runtime, TypeScript cannot prevent it. You still need runtime validation (using libraries like Zod).
- **Overcomplicating Types:** Creating massive, highly complex nested type structures that are hard for other developers to read and understand. Keep type interfaces simple and flat.

# Security & Performance Considerations
- **No Runtime Performance Overhead:** Since TypeScript compiles down to standard JavaScript and removes all type annotations, running TypeScript does not slow down your webpage in the browser at all.
- **Slower Build Times:** Compiling TypeScript adds an extra step to your build process. In very large codebases, this compiling step can take seconds or minutes, which is why developers use modern, fast build tools like ESBuild or SWC to speed up compiles.

# Related Technologies
- **tsconfig.json:** The configuration settings file for the compiler.
- **Zod:** A runtime schema validation library used to verify data coming from APIs matches your TypeScript interfaces.
- **ESLint:** A tool that checks your code for stylistic mistakes and bad habits, working alongside TypeScript.

# Summary

## What We Learned
- TypeScript is a superset of JavaScript that adds static type safety.
- It catches bugs in your editor before the code is executed.
- TypeScript code compiles down to clean JavaScript for browser execution.

## Key Takeaways
- Use static typing for data contracts (API payloads, component props).
- Keep compiler configurations strict to get the maximum benefit of type checking.

# Keywords
- TypeScript
- Superset
- Static Typing
- Compiler
- Type Annotation
- Interface
- tsconfig.json
- Transpiler

# Glossary

| Term | Meaning |
|---|---|
| Superset | A language built on top of another that contains all features of the original language plus new additions. |
| Static Typing | Checking the data types of variables and functions at compile-time (before execution). |
| Dynamic Typing | Checking and assigning data types to variables at runtime (during execution). |
| Interface | A TypeScript declaration that defines the expected structure and properties of an object. |
| Transpiling | The process of translating code from one source language (TypeScript) into another (JavaScript). |

## Next Recommended Chapters
- 03-JavaScript-Fundamentals.md
- 11-Frontend-Build-Tools.md
