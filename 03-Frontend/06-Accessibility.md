> **Mode:** Book
> **Pilot-Seat Standard**

# Introduction
Web Accessibility (often shortened to **a11y** because there are 11 letters between the 'A' and the 'Y') is the practice of designing and building websites so that everyone can use them. This includes people with visual, auditory, physical, speech, cognitive, or neurological disabilities, as well as elderly users or anyone browsing on a slow internet connection.

# Why It Exists
The internet has become the primary way we access education, bank accounts, healthcare, government services, and jobs. However, early websites were designed purely for users who could see a screen clearly, hold a mouse with steady hands, and process complex pages quickly. If you are blind and use a screen reader to read the text on screen, or if you have tremors and cannot use a mouse, a poorly built website is an impassable brick wall. Engineers and advocates created accessibility standards to ensure the internet remains an open, equal-opportunity space for everyone.

# Problem It Solves
Web accessibility solves the digital exclusion problem.

### Before Accessibility (Exclusive design):
- Blind users would hear "button" repeated continuously because the images had no descriptions.
- Users unable to use a mouse were trapped on a page because they could not highlight or click links using only a keyboard.
- Low-contrast text (like light grey text on a white background) made pages impossible to read for elderly users or anyone sitting in bright sunlight.

### After Accessibility (Inclusive design):
- Screen readers read descriptive label tags aloud, allowing blind users to navigate sites easily.
- A user can press the `Tab` key to jump sequentially through links and press `Enter` to click.
- Clear color contrast and adjustable text sizes ensure content is readable for all eyes.

# Core Concepts
To build accessible websites, you must understand four foundational ideas:

1. **Semantic HTML:** Using HTML tags for their intended purpose. A `<button>` tag tells the browser it is clickable, while a heading tag (`<h1>` to `<h6>`) defines the page's structure.
2. **Keyboard Navigation:** Ensuring that every link, input field, and button can be highlighted and activated using only the keyboard (usually the `Tab`, `Arrow`, and `Enter` keys).
3. **Contrast and Color:** Making sure the text color stands out sharply against the background, and never using color as the *only* way to convey information (e.g., instead of just turning a box red to show an error, add a text label that says "Error").
4. **ARIA (Accessible Rich Internet Applications):** A set of special HTML attributes (like `aria-label` or `aria-live`) that provide extra context to screen readers when standard HTML tags are not enough.

# Architecture / Components
The browser constructs a hidden structural layer parallel to the DOM, called the **Accessibility Tree**.

```text
               [ HTML Code ]
                     ↓
             [ Browser Engine ]
             /                \
     [ DOM Tree ]          [ Accessibility Tree ]
     (For styling &        (For Screen Readers)
      JS interaction)      - Name: "Submit Order"
                           - Role: Button
                           - State: Disabled
```

- **Accessibility Tree:** A filtered version of the DOM containing only information relevant to screen readers (Role, Name, State).
- **Screen Reader:** Assistive software (like NVDA, JAWS, or VoiceOver) that reads aloud the text and structure in the Accessibility Tree.
- **WCAG Guidelines:** The Web Content Accessibility Guidelines, which are the official global rules defining standard compliance levels (A, AA, AAA).

# Workflow
How a screen reader interacts with a page:

```text
Step 1: The browser loads the HTML and builds the DOM.
                  ↓
Step 2: The browser generates the Accessibility Tree from the DOM.
                  ↓
Step 3: The screen reader software hooks into the Accessibility Tree.
                  ↓
Step 4: The user presses keyboard shortcuts to read the page.
                  ↓
Step 5: The screen reader announces the elements (e.g. "Heading Level 1: Welcome").
```

# Real World Examples
Think of web accessibility as a **modern public library**.
- A library built only for people who can walk up steep stairs and read printed paper excludes a large part of the community.
- An accessible library adds ramps and automatic doors (Keyboard Navigation), books in Braille and audiobooks (Screen Readers), clear directional signs with high-contrast text (Visual Sizing), and quiet study rooms (Cognitive support).
- Designing a website is like building that library: you ensure there are ramps and audio formats built in from day one.

# Implementation
Here is how you write HTML that is accessible to screen readers and keyboard users:

### 1. Inaccessible HTML (Bad Practice)
```html
<!-- Broken: Blind users hear "click image", keyboard users cannot select this -->
<div onclick="submitForm()">
  <img src="arrow.png">
</div>
```

### 2. Accessible HTML (Best Practice)
```html
<!-- Correct: Screen readers read "Submit Form", and keyboard users can Tab to it -->
<button onclick="submitForm()" aria-label="Submit Form">
  <img src="arrow.png" alt="">
</button>
```

### 3. CSS Focus Styles (Do not remove the outline!)
```css
/* Ensure users navigating with a keyboard can see where they are on the page */
button:focus, a:focus {
  outline: 3px solid #3498db;
  outline-offset: 2px;
}
```

# Best Practices
- **Never Remove Focus Outlines:** Do not write `outline: none;` in your CSS unless you are replacing it with a custom, high-visibility focus indicator. Without outlines, keyboard users cannot see which button they have currently selected.
- **Use `alt` Attributes Correctly:** Every `<img>` tag needs an `alt` attribute. For decorative images (like background shapes), write `alt=""` so the screen reader knows to skip it. For informative images, write a brief description.
- **Maintain a Logical Heading Order:** Always start with `<h1>` for the main title, followed by `<h2>` for major sections, and `<h3>` for subsections. Never skip from `<h1>` directly to `<h4>` just because you like the font size.

# Industry Standards
Most countries legally require public websites to meet **WCAG 2.1 AA** compliance standards. Failing to meet these standards can result in expensive lawsuits, especially for e-commerce, banking, and government portals.

# Common Mistakes
- **Using Generic Tags for Buttons:** Using a `<div>` or `<span>` as a button. Even if you make it look like a button with CSS, a keyboard user cannot tab to it, and a screen reader will treat it as static text.
- **Low Contrast Ratios:** Using yellow text on a white background, or dark grey text on a black background.
- **Empty Link Labels:** Having a link that only contains an icon (like a trash can icon for deleting). A screen reader will read "link", leaving the user with no idea what clicking it will do. Add `aria-label="Delete Item"` to the link.

# Security & Performance Considerations
- **No Performance Cost:** Writing semantic, accessible HTML costs zero performance overhead. In fact, it often makes your page files smaller because you don't need heavy JavaScript workarounds to make generic elements keyboard-navigable.
- **Better Search Engine Optimization (SEO):** Google's search crawlers act like blind users: they cannot "see" images or layout shapes. They read the Accessibility Tree. Accessible sites score much higher on search engine rankings.

# Related Technologies
- **Screen Readers:** Software NVDA (Windows), VoiceOver (macOS/iOS), TalkBack (Android).
- **Lighthouse:** A built-in tool in Google Chrome that automatically audits your page's accessibility score.
- **WAI-ARIA:** The technical specification that adds extra accessibility attributes to HTML.

# Summary

## What We Learned
- Accessibility ensures websites are usable by everyone, regardless of physical or cognitive ability.
- The browser builds an Accessibility Tree used by assistive software like screen readers.
- Keyboard navigation, color contrast, and semantic tags are the core requirements of web accessibility.

## Key Takeaways
- Always use semantic tags like `<button>` and `<main>` instead of generic `<div>` wrappers.
- Never remove focus outlines without providing a visible alternative.

# Keywords
- Accessibility
- a11y
- Screen Reader
- Accessibility Tree
- Semantic HTML
- WCAG
- ARIA
- Focus Outline

# Glossary

| Term | Meaning |
|---|---|
| a11y | Abbreviation for accessibility (A + 11 letters + Y). |
| Accessibility Tree | The tree structure built by the browser that feeds names, roles, and states of elements to screen readers. |
| WCAG | Web Content Accessibility Guidelines; the global standard rules for web accessibility. |
| ARIA | Accessible Rich Internet Applications; special HTML attributes that describe complex components to screen readers. |
| Focus Outline | The visual boundary line drawn around an element when it is selected via keyboard navigation. |

## Next Recommended Chapters
- 01-HTML-Fundamentals.md
- 05-Responsive-Design.md
