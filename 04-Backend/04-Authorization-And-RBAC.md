> **Mode:** Book
> **Pilot-Seat Standard**

# Introduction
**Authorization** is the process of verifying *what* specific actions a logged-in user is permitted to perform inside an application. **RBAC** (Role-Based Access Control) is an authorization model that simplifies this process by grouping permissions into categories called **Roles** (like Admin, Editor, or Guest) and assigning users to those roles rather than managing permissions for each user individually.

# Why It Exists
In early multi-user systems, system administrators had to manually assign permissions to every single user account. For example, if a company had 500 employees, the administrator had to select each user one-by-one and click checkboxes saying: *"User 1 can read documents," "User 1 can write documents," "User 1 cannot delete documents."* When a new employee was hired, this setup took hours. Even worse, if the company decided to change what "Editors" could do, administrators had to search through all 500 user profiles and manually edit their settings. Engineers created Role-Based Access Control to manage permissions efficiently at scale by using roles as an intermediate coordinator.

# Problem It Solves
Authorization and RBAC solve complex access management, scale administration overhead, and permission leakage (privilege creep) problems.

### Before RBAC (Individual Permission Mapping):
- Administrative overhead was massive; assigning access rights took hours per employee.
- Security vulnerabilities arose easily because administrators frequently forgot to revoke access rights when employees changed departments.
- Auditing permissions (verifying who could edit financial logs) was nearly impossible.

### After RBAC (Grouped Roles):
- Permissions are assigned to a Role, and users are assigned to that Role.
- Changing permissions for a whole department takes 5 seconds: you edit the Role, and every user assigned to it updates instantly.
- Auditing is simple: you check which users are assigned to the "Finance" role.

# Core Concepts
To implement access control, you must understand three core models:

1. **Authentication vs. Authorization:**
   - **Authentication:** Who are you? (Verifying username and password).
   - **Authorization:** What can you do? (Checking if you have permission to delete a post).
2. **RBAC (Role-Based Access Control):** Users are assigned Roles, and Roles have Permissions.
   ```text
   [ User ] ──> Assigned ──> [ Role: Editor ] ──> Grants ──> [ Permission: EDIT_POST ]
   ```
3. **ABAC (Attribute-Based Access Control):** A more advanced authorization model that determines access based on dynamic attributes (variables) rather than static roles—such as the user's location, the time of day, or the device they are using (e.g. *"Only allow access if the user has the 'Manager' role AND is logging in from the corporate office during business hours"*).

# Architecture / Components
The flow of an authorization check inside a backend system:

```text
  [ User Request ] ──> [ Web Server ] ──> [ Authentication Middleware ] (Who are you? -> User 12)
                                                      │
                                                      ▼
                                          [ Authorization Gate ] (Can User 12 write posts?)
                                          - Check database: User 12 has role "Editor"
                                          - Check role definitions: "Editor" has "WRITE_POST" permission
                                                      │
                                           ┌──────────┴──────────┐
                                         Allowed                Denied
                                           ▼                     ▼
                                    [ Execute Action ]    [ Return 403 Forbidden ]
```

- **Authentication Middleware:** The filter that reads the session cookie or JWT to verify the user's identity.
- **Authorization Gate (Middleware):** The code gate that checks if the verified user has the required permission before letting the request pass to the database.
- **Permission Matrix:** The database table or configuration file defining exactly which permissions belong to which roles.

# Workflow
How a backend system authorizes a request:

```text
Step 1: A user hits a button to delete a product: `DELETE /api/products/45`.
                             ↓
Step 2: The server verifies the user's identity (Authentication).
                             ↓
Step 3: The server loads the user's roles from the database (e.g. User has the "Sales" role).
                             ↓
Step 4: The server checks the "Sales" role permissions.
                             ↓
Step 5: The server finds that "Sales" only has `READ_PRODUCT` and `CREATE_PRODUCT` permissions.
                             ↓
Step 6: Access is denied! The server returns HTTP status `403 Forbidden` and cancels the deletion.
```

# Real World Examples
Think of authentication as your **office security badge** and authorization as the **badge scanner doors**.
- Checking in at the lobby front desk and showing your driver's license to get an office badge is **Authentication**.
- The electronic scanners on individual doors inside the building represent **Authorization**.
- **Role-Based Access Control (RBAC)** is like color-coding the badges:
  - The **"Cleaning Crew" role** (yellow badge) opens the utility closets and all office doors.
  - The **"Accountant" role** (blue badge) opens the finance room, but the badge scanner locks them out of the server room.
  - The **"Guest" role** (green badge) only opens the lobby elevator.
- If you hire a new accountant, you don't manually program their badge room-by-room; you simply hand them a blue "Accountant" badge.

# Implementation
Here is how you write an authorization check gate (middleware) in Node.js (using Express):

```javascript
const express = require('express');
const app = express();

// Mock database role-to-permission mapping
const ROLE_PERMISSIONS = {
  admin: ['read_post', 'write_post', 'delete_post'],
  editor: ['read_post', 'write_post'],
  guest: ['read_post']
};

// 1. Create the Authorization Gate Middleware
function requirePermission(requiredPermission) {
  return (req, res, next) => {
    // Note: In real apps, req.user is set by the authentication middleware
    const userRole = req.user ? req.user.role : 'guest';
    const permissions = ROLE_PERMISSIONS[userRole] || [];

    // Check if the user's role contains the required permission
    if (permissions.includes(requiredPermission)) {
      next(); // User has permission! Let them pass to the route logic
    } else {
      res.status(403).json({ error: "Access Denied: 403 Forbidden" });
    }
  };
}

// 2. Apply the gate to specific routes
app.get('/api/posts', requirePermission('read_post'), (req, res) => {
  res.send("Viewing all posts...");
});

app.post('/api/posts', requirePermission('write_post'), (req, res) => {
  res.send("Post created successfully!");
});

app.delete('/api/posts/:id', requirePermission('delete_post'), (req, res) => {
  res.send("Post deleted!");
});
```

# Best Practices
- **Adhere to the Principle of Least Privilege:** Users should only be assigned the absolute minimum permissions they need to do their jobs. Never make everyone an "Admin" just to make coding easier.
- **Check Permissions, Not Roles:** Do not write code like `if (user.role === 'admin')`. If you later decide that "Editors" should also be able to delete posts, you will have to rewrite your code files. Write `if (user.hasPermission('delete_post'))` instead.
- **Fail Closed (Default Deny):** If your authorization code encounters an error (like a database timeout checking user roles), default to locking the user out. Never default to letting them in.

# Industry Standards
Almost all enterprise business software (like Salesforce or Jira) runs on **RBAC** patterns. Larger organizations dealing with complex compliance (like banks or hospital systems) overlay **ABAC** or **PBAC** (Policy-Based Access Control) to enforce geographic or IP address restrictions on data access.

# Common Mistakes
- **Confusing Authentication and Authorization:** Assuming that because a user is safely logged in, they are allowed to perform any action on the server.
- **Skipping Backend Checks:** Relying on the frontend code to enforce security. For example, hiding the "Delete" button from guest users in React, but forgetting to check permissions on the backend API (`DELETE /api/posts/5`). Anyone can open Postman and hit your API directly, bypassing your React UI block.

# Security & Performance Considerations
- **Privilege Escalation:** A vulnerability where a regular user exploits a bug (like modifying their user ID cookie) to trick the server into treating them as an Admin. Always sign user metadata cookies or tokens (using cryptography).
- **Database Query Overhead:** Checking permissions on every API call can slow down your system. Cache user roles in fast memory stores like Redis so you do not query your main relational database repeatedly on every request.

# Related Technologies
- **CASL:** A popular JavaScript library used to write clean permission policies for Node.js and React.
- **OAuth 2.0 Scopes:** The standard authorization tokens used to grant external apps limited permissions (e.g., granting a calendar app read-only access to your email).

# Summary

## What We Learned
- Authorization verifies user permission scopes, whereas authentication verifies user identity.
- RBAC simplifies permission administration by mapping users to roles and roles to specific actions.
- Access checks must always be enforced on the backend server, never solely on the frontend client.

## Key Takeaways
- Check for specific permissions (like `write_article`) in your route code, not generic roles (like `editor`).
- Default to "Access Denied" if an authorization check errors out.

# Keywords
- Authorization
- RBAC
- Role
- Permission
- Privilege
- Middleware
- Forbidden
- ABAC

# Glossary

| Term | Meaning |
|---|---|
| Authorization | The process of checking whether an authenticated user has permission to execute a specific action. |
| Role | A grouped category representing a job description (e.g. Admin) that holds a set of permissions. |
| Permission | A single, granular action allowed in the system (e.g. `delete_user`). |
| 403 Forbidden | The standard HTTP status code indicating the server understands who the user is, but refuses to allow the action. |
| Principle of Least Privilege | The security practice of granting users only the minimum system access necessary to perform their work. |

## Next Recommended Chapters
- 03-Authentication-And-Sessions.md
- 04-Backend/05-Databases-SQL-vs-NoSQL.md
