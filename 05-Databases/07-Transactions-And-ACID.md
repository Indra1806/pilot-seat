> **Mode:** Book
> **Pilot-Seat Standard**

# Introduction
A **Database Transaction** is a single, logical unit of work that groups multiple database actions (like inserting rows or updating balances) into one unbreakable bundle. Relational databases enforce a set of safety rules known as **ACID** to guarantee that every transaction is processed reliably and safely, preventing data loss, half-saved updates, and record corruption.

# Why It Exists
In early software development, updating data involved executing separate database operations one after another. If a program needed to update three different tables, and the server crashed or lost power during the second update, the first update remained saved, but the third was never executed. This left the database in a broken, half-finished state. Engineers invented transactions and the ACID properties to ensure that a group of updates either completes successfully as a single unit or fails completely, leaving the database clean and untouched.

# Problem It Solves
Transactions and ACID properties solve data inconsistency, half-finished updates, and concurrent access conflicts.

### Before Transactions (Unprotected Operations):
- A server crash mid-operation left databases corrupted (e.g., deducting money from one bank account without adding it to the receiving account).
- Multiple users editing the same record simultaneously overwrote each other's changes, leading to lost data.
- System crashes immediately after a save operation could result in data failing to write to the hard drive, disappearing forever.

### After Transactions (ACID Guarded):
- If any action inside a transaction fails, the database automatically performs a **Rollback**, reversing all preceding actions in that transaction.
- Concurrent transactions are isolated in their own "bubbles," preventing users from seeing or disrupting other users' half-saved data.
- Once a transaction is committed, the changes are written to non-volatile disk logs, ensuring they survive power outages and crashes.

# Core Concepts
The safety guarantees of database transactions are represented by the acronym **ACID**:

1. **Atomicity (All-or-Nothing):** A transaction cannot be broken down into smaller pieces. Either every query inside the transaction succeeds, or the entire transaction fails and is completely undone.
2. **Consistency (Valid State):** The database guarantees that any transaction will bring the database from one valid state to another, strictly enforcing all structural rules, data types, and constraint checks.
3. **Isolation (Bubble Separation):** Even if thousands of transactions run at the same millisecond, the database engine executes them in a way that prevents them from interfering with each other.
4. **Durability (Permanent Save):** Once a transaction completes successfully (referred to as a **Commit**), the database writes the changes to physical disk storage, guaranteeing that the data will not be lost even if the system crashes a microsecond later.

# Architecture / Components
Relational database engines use two primary internal subsystems to manage transaction safety:

```text
  [ Application Queries ] ──> [ Transaction Manager ]
                                      │
               ┌──────────────────────┴──────────────────────┐
               ▼                                             ▼
      [ Write-Ahead Log (WAL) ]                       [ Lock Manager ]
    - Writes transaction intent to disk             - Restricts concurrent edits
    - Restores data during system reboot            - Enforces Isolation levels
               │                                             │
               └──────────────────────┬──────────────────────┘
                                      ▼
                             [ Physical Database ]
```

- **Write-Ahead Log (WAL):** A highly optimized, append-only log file on disk. Before any changes are made to the actual database tables, the database engine writes the intended changes to the WAL. If the system crashes, the database reads the WAL during reboot to either finish the changes (Roll forward) or clean them up (Rollback).
- **Lock Manager:** An internal coordinator that places temporary locks on rows or tables being edited, preventing other transactions from modifying the same data until the current transaction completes.

# Workflow
The lifecycle of a database transaction:

```text
Step 1: The application opens a transaction block with the command: BEGIN;
                             ↓
Step 2: The application executes several SQL statements (e.g., subtract balance, log history).
                             ↓
Step 3: If all statements execute without errors, the application sends: COMMIT;
        - The engine writes the changes to the WAL and then updates the physical tables.
                             ↓
Step 4: If any error occurs (e.g., balance drops below zero, network drops), the application sends: ROLLBACK;
        - The database undoes all changes made since step 1, leaving the tables unchanged.
```

# Real World Examples
Think of a database transaction as a **banking wire transfer** between two accounts.
- Suppose you want to transfer $100 from Alice's account to Bob's account. This requires two distinct steps:
  1. Deduct $100 from Alice's balance.
  2. Add $100 to Bob's balance.
- Without a transaction: If the bank's power cuts out after Step 1 but before Step 2, Alice's money disappears into thin air.
- With a transaction: Both steps are wrapped in a transaction block. If the power cuts out after Alice's money is deducted, the database reboots, detects that the transaction was never committed, and automatically executes a **Rollback**—refunding Alice her $100.
- **Atomicity:** You can't have a half-transfer. Either both Alice and Bob see their balances update, or neither does.
- **Consistency:** The bank has a rule that balances cannot be negative. If Alice only has $50, the transaction is rejected because it breaks the rule.
- **Isolation:** If Alice tries to transfer money to Bob while Charlie is simultaneously transferring money to Alice, they don't corrupt each other's calculations. They are handled sequentially in isolation.
- **Durability:** Once the receipt is shown on screen, the bank's server can lose power, but the balances remain updated.

# Implementation
Here is how to write database transactions in SQL and Javascript:

### 1. Raw SQL Transaction Block
```sql
-- Start the transaction
BEGIN;

-- 1. Deduct money from Alice
UPDATE accounts 
SET balance = balance - 100 
WHERE owner = 'Alice' AND balance >= 100;

-- 2. Add money to Bob
UPDATE accounts 
SET balance = balance + 100 
WHERE owner = 'Bob';

-- Save and finalize all changes
COMMIT;
```
*Note: If the first UPDATE fails to find a balance >= 100, the application can issue a `ROLLBACK;` instead of `COMMIT;` to abort.*

### 2. Node.js Database Transaction Implementation
```javascript
const { Pool } = require('pg');
const pool = new Pool();

async function transferMoney(fromUser, toUser, amount) {
  const client = await pool.connect();
  
  try {
    // Start transaction block
    await client.query('BEGIN');
    
    // Step 1: Deduct from sender
    const deductResult = await client.query(
      'UPDATE accounts SET balance = balance - $1 WHERE owner = $2 AND balance >= $1',
      [amount, fromUser]
    );
    
    if (deductResult.rowCount === 0) {
      throw new Error("Insufficient funds or user not found!");
    }
    
    // Step 2: Add to receiver
    await client.query(
      'UPDATE accounts SET balance = balance + $1 WHERE owner = $2',
      [amount, toUser]
    );
    
    // Commit transaction if both succeeded
    await client.query('COMMIT');
    console.log("Transaction committed successfully!");
  } catch (error) {
    // Roll back everything to prevent half-finished changes
    await client.query('ROLLBACK');
    console.error("Transaction rolled back due to error:", error.message);
  } finally {
    client.release();
  }
}
```

# Best Practices
- **Keep Transactions Short:** Keep the time between `BEGIN` and `COMMIT` as short as possible. While a transaction is open, it holds locks on database rows, which can block other operations and slow down the entire system.
- **Never Run API Calls Inside Transactions:** Do not perform external network operations (like calling a payment gateway or sending an email) inside a database transaction block. If the API call takes 5 seconds, those database locks will remain held for 5 seconds.
- **Choose the Right Isolation Level:** By default, database engines use balanced isolation settings (like `Read Committed`). Only use strict isolation settings (like `Serializable`) when absolute mathematical precision is required (e.g. financial ledgers), as strict isolation significantly decreases query speed.

# Industry Standards
Most relational databases (PostgreSQL, MySQL, SQLite) support ACID transactions out of the box. NoSQL databases (like MongoDB) traditionally traded transactions for scaling speed, but modern versions of major NoSQL engines have added transaction support to meet corporate enterprise requirements.

# Common Mistakes
- **Nested Transactions:** Attempting to start a transaction inside an already open transaction. SQL does not support nested transaction commands natively; developers must use **Savepoints** instead.
- **Forgetting to Handle Rollbacks:** Catching errors in application code but forgetting to send the `ROLLBACK` command to the database. This leaves the database transaction open, keeping database locks active indefinitely and eventually crashing the server.
- **Deadlocks:** Occurs when Transaction A holds Lock 1 and waits for Lock 2, while Transaction B holds Lock 2 and waits for Lock 1. Both wait forever. Database engines automatically detect and kill one of these transactions, meaning application code must be built to retry failed transactions.

# Security & Performance Considerations
- **Concurrency Bottlenecks:** Under high traffic, too many transactions updating the same row (like a viral post's view counter) will queue up at the Lock Manager, grinding database performance to a halt.
- **Read Phenomenon Vulnerabilities:** Depending on the transaction isolation level, databases can suffer from anomalies like **Dirty Reads** (reading uncommitted, temporary data that might get rolled back) or **Phantom Reads** (queries returning different sets of rows when repeated within the same transaction).

# Related Technologies
- **Write-Ahead Logging (WAL):** The storage protocol used by PostgreSQL, MySQL, and SQLite to guarantee durability.
- **Two-Phase Commit (2PC):** A protocol used to coordinate transactions across multiple different databases (distributed databases).

# Summary

## What We Learned
- Transactions bundle multiple database statements into a single, indivisible unit of work.
- ACID principles guarantee that transactions are Atomic (all-or-nothing), Consistent (rules-abiding), Isolated (bubble-contained), and Durable (permanently written).
- Database engines ensure safety using Write-Ahead Logs (WAL) for durability and Lock Managers for isolation.

## Key Takeaways
- Always wrap multi-table modifications (like transfers or double-entry ledgers) in a single transaction block.
- Keep transaction execution windows brief to avoid resource lockouts and database slowdowns.
- Always include a robust `ROLLBACK` action inside your database error handling blocks.

# Keywords
- Transaction
- ACID
- Atomicity
- Consistency
- Isolation
- Durability
- Rollback
- Commit
- Lock Manager
- Write-Ahead Log

# Glossary

| Term | Meaning |
|---|---|
| Transaction | A group of database queries executed together as a single indivisible unit. |
| Commit | The final command in a transaction that permanently saves all changes to the database. |
| Rollback | The command that aborts a transaction and undoes all changes made since the transaction started. |
| Deadlock | A state where two transactions are blocked because each is waiting for the other to release a database lock. |
| Write-Ahead Log (WAL) | A database log file on disk where transaction details are written before being applied to the tables, ensuring durability. |
| Dirty Read | An isolation issue where a transaction reads changes that have not yet been committed by another running transaction. |

## Next Recommended Chapters
- 08-Normalization.md
