> **Mode:** Book
> **Pilot-Seat Standard**

# Introduction
HTML, short for **HyperText Markup Language**, is the foundation of every single webpage on the internet. It is not a programming language that performs logic (like math or decision-making). Instead, it is a **markup language** that organizes text, images, and buttons so your computer browser knows how to arrange them.

# Why It Exists
Before HTML was invented in the early 1990s, sharing documents between computers was messy. If an engineer wanted to share a paper with diagrams, tables, and cross-references, there was no standard way to display it. Different computers would format it in different ways, often turning it into unreadable gibberish. Engineers needed a single, universally accepted language that could tell any computer screen: *"This is a main heading, this is a paragraph, and this is a link to another document."*

# Problem It Solves
Without HTML, a browser would view a webpage's content as a single, endless blob of text.

### Before HTML (What the browser sees without structure):
```text
Welcome to my shop. We sell apples. Apples are delicious. Click here to buy.
```
There is no visual hierarchy, no way to distinguish a button from a paragraph, and no way to click a link.

### After HTML (What HTML does):
```html
<h1>Welcome to my shop</h1>
<p>We sell apples. Apples are delicious.</p>
<a href="/buy">Click here to buy</a>
```
The browser now knows to make "Welcome to my shop" big and bold, treat the description as a normal paragraph, and turn "Click here to buy" into a clickable link.

# Core Concepts
To master HTML, you only need to understand three basic concepts:

1. **Tags (The Labels):** Every piece of content is wrapped in labels called tags. Tags are written with angle brackets. Most tags come in pairs: an opening tag (like `<p>`) and a closing tag (like `</p>`).
2. **Elements (The Objects):** An element is the complete package of the opening tag, the content inside, and the closing tag. For example: `<p>Hello World</p>` is a paragraph element.
3. **Attributes (The Details):** Attributes provide extra details about an element. They are written inside the opening tag. For example, in `<a href="https://google.com">Click</a>`, the `href` attribute tells the browser exactly where the link should go.

# Architecture / Components
HTML uses a tree-like architecture. A main outer tag contains inner tags, which contain even smaller tags. This is called nesting.

```text
       [html] (Root)
       /    \
  [head]    [body]
  /         /    \
[title]   [h1]   [p]
```

### The Standard Document Structure
Every correct HTML page contains these basic structural components:
- `<!DOCTYPE html>`: Tells the browser that this is a modern HTML5 document.
- `<html>`: The outer container for everything on the page.
- `<head>`: The brain of the page. It holds hidden information like the page title, search keywords, and links to stylesheets.
- `<body>`: The body of the page. This holds all the content that users actually see on their screens.

# Workflow
When you visit a website, HTML follows this path:

```text
Step 1: Your browser downloads the raw HTML code file from a server.
                  ↓
Step 2: The browser reads the file line-by-line from top to bottom.
                  ↓
Step 3: The browser builds a structural map of the page (called the DOM Tree).
                  ↓
Step 4: The browser draws the elements on your screen based on the map.
```

# Real World Examples
Think of HTML as the **structural blueprint of a house**. 
- The HTML file is the blueprint.
- The tags define what each room is: `<kitchen>`, `<bedroom>`, `<doorway>`.
- The attributes are the specifications: `<doorway width="3feet" lock="yes">`.
- At this stage, there is no paint on the walls (CSS) and the light switches do not do anything yet (JavaScript). It is just the bare-bones structural frame.

# Implementation
Here is how a developer writes a simple HTML page:

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>My Simple Webpage</title>
</head>
<body>
    <h1>Welcome to My Website</h1>
    <p>This is a paragraph containing some text and a <strong>bold word</strong>.</p>
    
    <h2>My Favorite Fruits</h2>
    <ul>
        <li>Apples</li>
        <li>Bananas</li>
        <li>Cherries</li>
    </ul>

    <a href="https://example.com">Visit Example Website</a>
</body>
</html>
```

# Best Practices
- **Use Semantic Tags:** Always use tags that match their actual meaning. Use `<header>` for the top navigation, `<article>` for posts, and `<button>` for clickable buttons. Do not just use `<div>` (generic boxes) for everything.
- **Always Include Image Descriptions:** Use the `alt` attribute on image tags (e.g., `<img src="dog.jpg" alt="A golden retriever playing with a ball">`) so screen readers can describe the image to visually impaired users.
- **Keep Nested Tags Clean:** Properly indent your code so it is clear which elements sit inside other elements.

# Industry Standards
Modern web applications use **HTML5**, the latest version of the HTML standard. In large companies, HTML is often generated dynamically using frontend frameworks (like React or Next.js), but it ultimately outputs standard HTML to the browser because that is the only language web browsers understand.

# Common Mistakes
- **Forgetting to Close Tags:** Leaving out a closing tag (like forgetting `</p>`) can cause the browser to format the entire rest of the page incorrectly.
- **Using the Wrong Tags for Actions:** Using a link (`<a>`) when you should be using a `<button>` (or vice-versa). Links should navigate to a new page; buttons should trigger an action (like opening a menu or submitting a form).
- **Skipping Page Structure:** Creating a webpage without `<!DOCTYPE html>`, `<html>`, or `<body>` tags. Browsers might still show it, but it will break on different devices.

# Security & Performance Considerations
- **Content Injection (XSS):** If you display user-written text directly on your webpage without cleaning it, attackers can inject malicious HTML tags (like `<script>`) that run harmful code in other users' browsers.
- **Too Many Nested Elements:** Having thousands of nested tags on a single page makes it difficult for the browser to build its map, slowing down how fast the page loads and responds to user clicks.

# Related Technologies
- **CSS:** Used to style, color, and arrange the HTML structure.
- **JavaScript:** Used to add interactive behavior and logic to the HTML structure.
- **DOM (Document Object Model):** The internal representation of the HTML document created by the browser.

# Summary

## What We Learned
- HTML is the structured markup language used to build the skeleton of all webpages.
- Pages are built using nesting elements, labels called tags, and descriptive attributes.
- Semantic HTML is crucial for accessibility, browser understanding, and search engines.

## Key Takeaways
- Always write semantic, accessible HTML to ensure anyone on any device can use your site.
- Keep structural markup separated from visual presentation (which belongs in CSS).

# Keywords
- HTML
- Tag
- Element
- Attribute
- Nesting
- Semantic HTML
- Accessibility
- DOM Tree

# Glossary

| Term | Meaning |
|---|---|
| Tag | A character-based label enclosed in angle brackets (like `<p>`) that tells the browser how to format text. |
| Element | The combination of an opening tag, the content, and a closing tag. |
| Attribute | A property specified inside an opening tag that provides extra details about the element (like `href` or `src`). |
| Semantic HTML | Writing HTML using elements that describe their meaning and purpose (like `<header>`, `<main>`, `<article>`). |
| DOM | Document Object Model; the tree-like structure the browser builds to represent the page. |

## Next Recommended Chapters
- 02-CSS-Fundamentals.md
- 06-Accessibility.md
