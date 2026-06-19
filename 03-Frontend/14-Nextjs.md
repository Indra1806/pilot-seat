> **Mode:** Book
> **Pilot-Seat Standard**

# Introduction
Next.js is a web development framework built on top of React. While React provides the UI building blocks (components), Next.js provides the **infrastructure and structure** needed to run a production-ready application. It introduces features like automatic page routing, image optimization, search-engine friendliness (SEO), and options to render webpages on a backend server instead of forcing the user's browser to do all the work.

# Why It Exists
Standard React applications use **Client-Side Rendering (CSR)**. When a user visits a React site, the server sends a completely blank HTML file along with a massive JavaScript file. The user's browser must download, parse, and execute this JS file before it can draw anything on the screen. This causes two major problems:
1. **Slow First Load:** Users sit staring at a blank white screen while their device downloads and runs the JS code (especially bad on slow mobile networks).
2. **Terrible SEO:** Search engine bots (like Google's crawler) scan webpages to rank them. When they scan a standard React site, they see a blank HTML file and move on before the JavaScript runs, making it nearly impossible for the site to show up in search results.
Vercel engineers created Next.js in 2016 to solve these problems by pre-rendering webpages into fully formed HTML on a server *before* sending them to the browser.

# Problem It Solves
Next.js solves page speed bottlenecks, complex client routing configuration, and SEO search visibility challenges in React.

### Before Next.js (Standard React CSR):
- The browser received a blank page and had to build the UI from scratch, delaying visual loading.
- Search engines could not index page content, destroying search rankings.
- Developers had to configure complex, fragile routing packages (like React Router) manually.

### After Next.js (Server Pre-rendering):
- The browser receives a pre-built HTML page filled with text and layout, displaying it instantly.
- Search engines read fully formed text immediately, maximizing SEO optimization.
- Routing is automatic: creating a file inside the `app/` folder automatically creates a corresponding page route URL.

# Core Concepts
Next.js is famous for giving developers three rendering choices for every page:

1. **Client-Side Rendering (CSR):** The standard React way. The browser builds the page.
2. **Server-Side Rendering (SSR):** The server builds the page HTML dynamically *every single time* a user requests it (great for pages displaying live data, like a personalized dashboard).
3. **Static Site Generation (SSG):** The server pre-builds all page HTML files once when the developer compiles the code (build time). When a user visits, they download the pre-built file instantly (great for blogs, documentation, or product pages).

# Architecture / Components
Next.js splits execution between the server environment and the client browser:

```text
  [ User visits URL ] ──> [ Next.js Server ]
                                 │
                 (Determines page rendering type)
                 ┌───────────────┴───────────────┐
                 ▼                               ▼
            [ SSR / SSG ]                      [ CSR ]
         - Fetches data on server          - Sends blank HTML
         - Renders complete HTML           - Sends JS bundle
                 │                               │
                 ▼                               ▼
        [ Sends Fully Formed HTML ]     [ Sends Blank Page ]
                 │                               │
                 ▼                               ▼
       [ Screen displays instantly ]    [ Browser compiles page ]
                 │
                 ▼
       [ Hydration: JS loads and ]
       [ makes buttons clickable ]
```

- **Next.js Server:** The Node.js environment that processes requests, fetches data, and pre-renders pages.
- **Client Hydration:** The process where the browser takes the static HTML page sent by the server and attaches React JavaScript event listeners to make it interactive.
- **File-System Routing:** A directory structure where folder names map directly to URL paths (e.g. `app/about/page.js` becomes `mysite.com/about`).

# Workflow
How a Next.js Server-Side Rendered page loads:

```text
Step 1: The user enters the URL `mysite.com/profile`.
                             ↓
Step 2: The Next.js backend server receives the request.
                             ↓
Step 3: The server fetches user data from a database.
                             ↓
Step 4: The server renders the React profile component into static HTML text containing the fetched data.
                             ↓
Step 5: The server sends the complete HTML page back to the browser.
                             ↓
Step 6: The user sees the completed page instantly, and JavaScript "hydrates" it to make buttons clickable.
```

# Real World Examples
Think of page rendering strategies as **different ways to serve a meal in a restaurant**.
- **Client-Side Rendering (CSR)** is like a **meal kit service** (e.g. Blue Apron). The restaurant sends you a box of raw, chopped vegetables, raw meat, and a recipe card (the JS bundle). You (the browser) have to cook it all yourself on your stove before you can eat. It takes time and effort.
- **Server-Side Rendering (SSR)** is like a **traditional restaurant**. You order your food, the chef in the kitchen cooks it immediately, and the waiter serves it to your table hot and completed (fully formed HTML with database data). You eat immediately.
- **Static Site Generation (SSG)** is like a **bakery**. The baker bakes 100 loaves of bread early in the morning before the shop opens (build time). When you walk in to buy bread, they hand you a completed loaf instantly over the counter. You don't have to wait for the oven to heat up.

# Implementation
Here is how you write a Server Component in Next.js that fetches data on the server and renders it statically or dynamically:

```jsx
// 1. Next.js App Router uses Server Components by default
// This function runs ENTIRELY on the server, not in the browser!
async function BlogPostPage({ params }) {
  // 2. Fetch data directly from a database or API on the server
  const response = await fetch(`https://api.example.com/posts/${params.id}`);
  const post = await response.json();

  // 3. Return the UI. Next.js turns this into HTML before sending it
  return (
    <article className="blog-post">
      <h1>{post.title}</h1>
      <p className="meta">Published on {post.date}</p>
      <div className="content">
        {post.body}
      </div>
    </article>
  );
}

export default BlogPostPage;
```

# Best Practices
- **Use Server Components by Default:** In Next.js (App Router), components are Server Components by default. Keep them as Server Components to reduce the amount of JavaScript sent to the browser. Only add `"use client"` at the top of a file if you need browser-only features (like `useState`, event listeners, or browser APIs).
- **Statically Pre-render as Much as Possible:** If a page does not contain personalized user data (like a homepage or blog list), use Static Site Generation (SSG). Next.js will cache the static HTML files, making page loads incredibly fast.
- **Leverage Built-in `<Image>` Components:** Never use the standard HTML `<img>` tag in Next.js. Use the Next.js `<Image>` component, which automatically resizes, compresses, and lazy-loads images based on the user's screen size.

# Industry Standards
Next.js is the leading enterprise React framework. It is used by major digital platforms like TikTok, Twitch, Hulu, Nike, and Notion to run their main web portals.

# Common Mistakes
- **Using `"use client"` Everywhere:** Treating Next.js like a traditional React app by slapping `"use client"` at the top of every file. This destroys the server-rendering benefits and sends massive JS files to the browser.
- **Executing Server Code on the Client:** Trying to run server-only Node.js libraries (like file system readers `fs` or database connectors) inside Client Components. This will crash the browser compiler.

# Security & Performance Considerations
- **Environment Variable Protection:** Next.js hides sensitive environment variables (like API keys) on the server by default. Only variables prefixed with `NEXT_PUBLIC_` are sent to the browser, protecting your database credentials.
- **Automatic Code Splitting:** Next.js automatically splits your JavaScript code. If a user visits the `/about` page, they only download the code needed for that specific page. Code for the `/contact` or `/dashboard` pages is not loaded, drastically speeding up initial download times.

# Related Technologies
- **Vercel:** The cloud hosting platform created by the authors of Next.js, optimized to deploy Next.js apps with zero configuration.
- **React Server Components (RSC):** The underlying React feature that Next.js uses to run components on the server.
- **Remix:** A popular competing React framework focusing on web standards and dynamic data fetches.

# Summary

## What We Learned
- Next.js is a React framework that supports server rendering (SSR/SSG) and dynamic routing.
- It solves React's slow initial load and SEO problems by compiling components into HTML on the server.
- File-system routing removes the need for manual router packages.

## Key Takeaways
- Use SSG (Static Generation) for public informational pages, and SSR (Server Rendering) for private dynamic pages.
- Use Next.js built-in optimization components (Image, Link) to ensure optimal performance.

# Keywords
- Next.js
- Server-Side Rendering
- SSR
- Static Site Generation
- SSG
- Hydration
- Server Components
- File-System Routing

# Glossary

| Term | Meaning |
|---|---|
| Client-Side Rendering | Rendering a webpage completely inside the user's browser using JavaScript. |
| Server-Side Rendering | Generating a webpage's HTML dynamically on a backend server for every incoming request. |
| Static Site Generation | Generating static HTML files for webpages once during the code compilation (build) phase. |
| Hydration | The process where the browser loads React JavaScript and attaches event listeners to pre-rendered server HTML. |
| Server Component | A React component that executes only on the backend server, outputting HTML to the client without sending its JS code. |

## Next Recommended Chapters
- 12-React.md
- 16-Web-Performance-Optimization.md
