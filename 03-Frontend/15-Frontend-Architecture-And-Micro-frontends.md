> **Mode:** Book
> **Pilot-Seat Standard**

# Introduction
Frontend Architecture is the plan and organizational system used to structure a web project's files, components, and data flows to ensure the codebase remains clean and easy to scale. **Micro-frontends** is an architectural pattern that extends this by splitting a massive, single web application (a monolith) into smaller, independent sub-applications that different engineering teams can build, test, and deploy completely separately, before merging them into a single webpage for the user.

# Why It Exists
When a startup begins, a small team of developers can easily write code in a single, simple folder structure. However, as the company grows to hire 100+ developers, they all end up working in the exact same files. Developers constantly overwrite each other's changes, wait in lines to merge code, and accidentally break unrelated features when shipping updates. If a developer makes a typo in the login screen, it can crash the entire shop checkout screen. Engineers created frontend architecture standards and micro-frontends to divide large websites into isolated, self-contained zones so that teams can work in parallel without getting in each other's way.

# Problem It Solves
Frontend architecture and micro-frontends solve team scaling, code collision, and deployment bottleneck problems.

### Before Architecture & Micro-frontends (The Monolith):
- A single massive codebase meant any minor code error could crash the entire website.
- Long build and test times delayed feature releases.
- Different teams had to use the exact same framework version (e.g. React 16), preventing teams from upgrading or experimenting with newer tools.

### After Architecture & Micro-frontends (Decoupled Units):
- Teams own independent sub-projects; a crash in the "Recommendations" team's widget doesn't affect the "Checkout" team's payment page.
- Fast, independent deployments: teams ship features instantly when ready without waiting for other departments.
- Architectural freedom: one team can use React while another uses Vue or Svelte, and they assemble seamlessly on the same screen.

# Core Concepts
To architect clean frontends, you must understand three key principles:

1. **Separation of Concerns:** Dividing code by its responsibility. Keep visual formatting (CSS) separated from structural bones (HTML), which is kept separated from business calculations (JavaScript) and API requests.
2. **Container Application (The Shell):** In micro-frontends, this is the main outer webpage that loads first. It is responsible for displaying the shared header and footer, managing global user login tokens, and loading the appropriate micro-frontend based on the URL path.
3. **Module Federation:** A technology that allows different compiled web applications to dynamically share code and load modules from one another at runtime, making micro-frontends work seamlessly together.

# Architecture / Components
The micro-frontend architecture coordinates multiple sub-apps under a single Container Shell:

```text
                      [ The Container Shell ] (Main Website)
                      (Header, Footer, Routing)
                      ┌───────────┼───────────┐
                      ▼           ▼           ▼
               [ Sub-App A ]  [ Sub-App B ]  [ Sub-App C ]
                (Dashboard)     (Checkout)    (Settings)
                - React team    - Vue team    - Svelte team
                - Server A      - Server B    - Server C
```

- **Container Shell:** The host page that handles navigation, routing, and shared styles.
- **Remote Micro-frontends:** The autonomous apps hosted on separate servers that are injected into the Container Shell dynamically at runtime.
- **Shared Library Layer:** Common utilities (like design system components or API managers) shared across teams to maintain visual consistency.

# Workflow
How a micro-frontend app loads inside a user's browser:

```text
Step 1: The user navigates to `mysite.com/checkout`.
                             ↓
Step 2: The browser downloads the small Container Shell application.
                             ↓
Step 3: The Shell detects the `/checkout` route.
                             ↓
Step 4: The Shell queries a map to find the server address of the Checkout Sub-App.
                             ↓
Step 5: The Shell fetches and runs the Checkout Javascript bundle dynamically in the background.
                             ↓
Step 6: The checkout micro-app mounts on the screen, appearing as a single seamless site to the user.
```

# Real World Examples
Think of micro-frontends as a **large department store** versus a **small boutique shop**.
- A monolith is like a tiny mom-and-pop shop. The owner does everything: sweeps floors, manages inventory, works the cash register, and handles accounting. If the store grows massive and the owner is still trying to handle every task personally, the shop descends into chaos.
- A micro-frontend architecture is like a giant department store (e.g. Macy's). The store is split into independent sections: the shoes section, cosmetics, jewelry, and electronics. Each department has its own manager, its own specialized staff, and its own checkout counter. They can change their display racks (deploy updates) without shutting down the entire department store. The customer walks into one single building (the Container Shell), but behind the scenes, each department runs as an independent business.

# Implementation
Here is how you separate concerns inside a single project by dividing UI layout from database API logic using a **Custom Hook**:

### 1. The Data Fetcher (Logic Layer - `useUserData.js`)
```javascript
import { useState, useEffect } from 'react';

// This hook handles the network request and state logic
export function useUserData(userId) {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    fetch(`https://api.example.com/users/${userId}`)
      .then(res => res.json())
      .then(data => {
        setUser(data);
        setLoading(false);
      });
  }, [userId]);

  return { user, loading };
}
```

### 2. The Visual Element (UI Layer - `UserProfile.jsx`)
```jsx
import React from 'react';
import { useUserData } from './useUserData';

function UserProfile({ userId }) {
  // Extract state logic from the custom hook
  const { user, loading } = useUserData(userId);

  if (loading) return <p>Loading user details...</p>;

  // This component only cares about visual presentation (HTML structure)
  return (
    <div className="profile-card">
      <img src={user.avatarUrl} alt={user.name} />
      <h2>{user.name}</h2>
      <p>Email: {user.email}</p>
    </div>
  );
}
```

# Best Practices
- **Establish a Shared Design System:** To prevent your website from looking like a patchwork of different styles, build a shared UI library (using tools like Storybook) containing standard buttons, input fields, and fonts that all teams must use.
- **Communicate via Custom Browser Events:** Micro-frontends should never access each other's internal state directly. If the Checkout app needs to tell the Header app that an item was added, it should dispatch a standard browser Custom Event (`window.dispatchEvent`) rather than calling functions directly inside the other app.
- **Keep Folders Structured by Feature:** In single projects, organize files by feature (e.g., a `cart/` folder containing cart components, cart styles, and cart hooks) rather than grouping by file type (e.g., putting all components in one massive folder and all styles in another).

# Industry Standards
Large-scale web platforms like Amazon, Spotify, and IKEA use micro-frontend architectures to let hundreds of independent product teams deploy updates daily without coordinating schedules. They typically use **Webpack Module Federation** or **Single-SPA** to orchestrate runtime assembly.

# Common Mistakes
- **Creating Dependency Hell:** Having each micro-frontend load its own copy of React, resulting in the user downloading React 5 different times on a single page. Configure your bundlers to share core libraries.
- **Leaking Styles:** Writing global CSS in a sub-app that accidentally changes the font color of the container header. Always use CSS Modules or scoped CSS-in-JS to isolate styles.
- **Choosing Micro-frontends Too Early:** Implementing micro-frontends for a small 3-person team. Micro-frontends add substantial server and build complexity. Only use them when your organization grows too large to work in a single repository.

# Security & Performance Considerations
- **Version Skew:** If the Container Shell and a Micro-frontend use incompatible versions of shared utilities, the app can crash. Implement strict runtime version checks.
- **Initial Load Latency:** Downloading multiple separate application bundles can increase page load times. Implement aggressive caching on remote bundles.

# Related Technologies
- **Webpack Module Federation:** The standard configuration tool that compiles apps to dynamically share code at runtime.
- **Single-SPA:** A popular Javascript framework used to tie different frontend sub-apps together.
- **Storybook:** A tool used to build and document shared UI design components in isolation.

# Summary

## What We Learned
- Frontend architecture structures files and logic to support long-term scalability.
- Micro-frontends split monoliths into self-contained sub-applications managed by autonomous teams.
- Runtime orchestration (like Module Federation) brings micro-apps together into a single user interface.

## Key Takeaways
- Decouple business logic from display components using custom hooks.
- Standardize UI visuals across micro-frontends with a shared design system.

# Keywords
- Frontend Architecture
- Micro-frontends
- Monolith
- Container Shell
- Module Federation
- Custom Hooks
- Storybook
- Decoupling

# Glossary

| Term | Meaning |
|---|---|
| Monolith | A single, unified software application containing all code, routing, and styles in a single project. |
| Container Shell | The host webpage in a micro-frontend setup that coordinates routing and dynamically loads sub-apps. |
| Custom Hook | A reusable JavaScript function that extracts React state and side-effect logic outside of visual components. |
| Module Federation | A bundler feature allowing compiled web builds to import and export modules dynamically at runtime. |
| Decoupling | The practice of separating software modules so they run independently with minimal direct dependencies. |

## Next Recommended Chapters
- 11-Frontend-Build-Tools.md
- 12-React.md
