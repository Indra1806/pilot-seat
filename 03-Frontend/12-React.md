> **Mode:** Book
> **Pilot-Seat Standard**

# Introduction
React is a popular, open-source JavaScript library developed by Meta (formerly Facebook) for building user interfaces. It is a tool that allows developers to write **declarative** code using reusable building blocks called **Components**. Instead of writing complex instructions to manually edit the webpage's HTML elements, developers simply describe what the page should look like based on the current data state, and React handles updating the screen automatically.

# Why It Exists
Before React was created in 2013, developers built interactive pages using plain JavaScript DOM manipulation (like `document.getElementById` and `element.appendChild`). In a large application, this was extremely difficult to manage. For example, if a user clicked a button, the developer had to write code to find the shopping cart span, read the number, add 1, write the text back, find the checkout button, and change its color to green. As pages grew larger and had hundreds of moving parts, code became a tangled web of instructions. If one minor detail changed, the whole page layout could break. Facebook engineers created React to let developers structure their code around data, letting React handle the messy details of drawing it.

# Problem It Solves
React solves the UI-data synchronization and manual DOM update bottleneck.

### Before React (Imperative DOM updates):
- Developers had to write detailed step-by-step instructions to select, edit, and append elements on the screen.
- State (data) and UI (graphics) frequently fell out of sync, showing users incorrect totals or broken screens.
- Manual DOM updates are slow; updating dozens of elements individually caused page stutter.

### After React (Declarative Components):
- Developers describe *what* the UI should look like for a given data state; React draws it.
- State is the single source of truth; when data changes, the UI updates instantly and automatically.
- React calculates changes in memory first, making updates incredibly fast.

# Core Concepts
To write React applications, you must master three fundamental concepts:

1. **Components:** Self-contained, reusable building blocks of UI that contain their own structure (HTML-like syntax called JSX), styling, and logic.
2. **State & Props:** 
   - **State** is a component's private, internal memory (like a checkbox being checked or unchecked). When state changes, the component immediately redraws itself (re-renders).
   - **Props** (short for properties) are configuration inputs passed down from a parent component to a child component (like passing a custom label to a button).
3. **The Virtual DOM:** A lightweight copy of the real HTML DOM stored in the computer's memory. React uses this copy to figure out exactly what changed before touching the real browser screen.

# Architecture / Components
React manages updates using a reconciliation pipeline between the Virtual DOM and the real browser DOM.

```text
  [ User Action ] ──> [ Updates State ]
                             │
                             ▼
                    [ Re-render Component ]
                             │
                             ▼
                 [ New Virtual DOM Tree ]
                             │
            (Diffing: Compare old tree with new tree)
                             ▼
                 [ Calculate Minimal Changes ]
                             │
                             ▼
               [ Update Real Browser DOM ]  (Batch updates)
```

- **JSX:** A syntax extension that allows developers to write HTML structure directly inside JavaScript code (e.g. `return <h1>Hello</h1>`).
- **Diffing Algorithm:** The engine that compares the old Virtual DOM tree with the new Virtual DOM tree to find the differences.
- **Reconciliation:** The process of taking the calculated differences and applying only the necessary changes to the real browser DOM.

# Workflow
The React rendering lifecycle:

```text
Step 1: The application loads; React builds a Virtual DOM tree from your components.
                             ↓
Step 2: React paints the real browser DOM matching the Virtual DOM.
                             ↓
Step 3: A user triggers an action (e.g. clicks a button), updating a State variable.
                             ↓
Step 4: React generates a new Virtual DOM tree representing the updated state.
                             ↓
Step 5: The Diffing engine compares the old and new Virtual DOM trees.
                             ↓
Step 6: React updates only the specific changed nodes in the real browser DOM.
```

# Real World Examples
Think of React as a **Lego building kit** and traditional JavaScript as **modeling clay**.
- Writing plain JavaScript is like building a house out of wet clay. If you decide to change a window size, you have to scrape away the clay, remold the window, stick it back in, and smooth out the surrounding walls. It is messy and easy to ruin the whole sculpture.
- React is like building a house out of Legos. The house is made of individual, snaps-together brick modules: `<Door>`, `<Window>`, `<Roof>`. If you want to change a window, you swap that specific Lego piece. The base Lego board (the Virtual DOM) tracks exactly which brick changed and snaps the new brick in place instantly without affecting the rest of the structure.

# Implementation
Here is how you write a simple counter component in React using JSX and Hooks:

```jsx
import React, { useState } from 'react';

// 1. Define a reusable Functional Component
function ClickCounter(props) {
  // 2. Declare a state variable called "count", initialized to 0
  const [count, setCount] = useState(0);

  // 3. Define the function to run when the button is clicked
  const handleIncrement = () => {
    setCount(count + 1); // Updates state, triggering a re-render
  };

  // 4. Return the visual structure using JSX
  return (
    <div className="counter-card">
      <h2>{props.title}</h2>
      <p>You have clicked the button {count} times.</p>
      <button onClick={handleIncrement}>Click Me</button>
    </div>
  );
}

export default ClickCounter;
```

# Best Practices
- **Keep Components Small and Single-Purpose:** Don't write a single component that holds your header, list, footer, and sidebar. Break them into smaller components like `<Header>`, `<ListItem>`, and `<Footer>`.
- **Never Modify State Directly:** Never write `count = count + 1`. Always use the state setter function provided by the Hook (like `setCount(newCount)`). Modifying variables directly bypasses React, meaning the screen will not update.
- **Lift State Up When Needed:** If two separate components need to share the same data, move that state to their closest common parent component and pass it down as props.

# Industry Standards
React is the most popular frontend tool in the world. It is used by major companies like Netflix, Airbnb, Twitter, and Facebook to build their web interfaces. It has a massive ecosystem of pre-built components, routing libraries, and developer tools.

# Common Mistakes
- **Mutating Objects in State:** Trying to update an object in state by changing a property directly (e.g. `user.name = 'John'`) instead of creating a clean copy of the object (e.g. `setUser({ ...user, name: 'John' })`).
- **Forgetting the `key` Prop in Lists:** When rendering lists of items in React, you must give each item a unique `key` prop (like `key={item.id}`). Without keys, React cannot track which items were added, moved, or deleted, leading to rendering bugs.
- **Creating Infinite Render Loops:** Calling a state setter function directly in the main body of a component, which triggers a render, which calls the setter again, crashing the browser.

# Security & Performance Considerations
- **Automatic XSS Prevention:** By default, React escapes values rendered in JSX before displaying them. If a user tries to inject a `<script>` tag into an input box, React displays it as plain text rather than running it, preventing script injection attacks.
- **Unnecessary Re-renders:** When a parent component updates, all of its children re-render by default. In large applications, this can cause lag. Use optimization hooks like `useMemo` or `useCallback` to cache values and prevent redundant renders.

# Related Technologies
- **Next.js:** A framework built on top of React that adds page routing, server rendering, and speed optimizations.
- **React Native:** A tool that compiles React code into native mobile apps for iOS and Android.
- **Redux / Zustand:** Libraries used to manage global state in very large React applications.

# Summary

## What We Learned
- React is a declarative UI library based on reusable components.
- Components manage internal data via State and receive external configurations via Props.
- React uses a Virtual DOM and a diffing engine to update the real browser screen efficiently.

## Key Takeaways
- Always update state using setter functions to trigger re-renders.
- Break UI layouts into small, single-purpose components for maintainability.

# Keywords
- React
- Component
- JSX
- State
- Props
- Virtual DOM
- Re-render
- Hooks

# Glossary

| Term | Meaning |
|---|---|
| Declarative | A programming style where you describe the desired end result rather than the step-by-step instructions to get there. |
| JSX | JavaScript XML; a syntax extension that lets you write HTML structure directly inside JavaScript files. |
| State | An internal memory object managed inside a component that triggers a UI redraw when modified. |
| Props | Immutable configuration data passed from a parent component down to a child component. |
| Virtual DOM | A lightweight, in-memory representation of the real browser DOM used to calculate UI updates quickly. |
| Hooks | Special built-in functions (starting with `use`, like `useState` or `useEffect`) that let functional components tap into state and lifecycles. |

## Next Recommended Chapters
- 03-JavaScript-Fundamentals.md
- 13-State-Management.md
