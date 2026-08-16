# Backend Development Roadmap
> Life Planner Project — Deep Concepts Reference
> Read this before writing any backend code.

---

## The One Question Behind Everything

> "Why does this exist?"

Every tool, pattern, and concept in backend development exists to solve one of these 5 problems:

| Problem | Solution |
|---------|----------|
| Code breaks at runtime with no warning | TypeScript |
| Secrets exposed in code | Environment variables |
| Wrong data enters the system | Validation (Zod) |
| Users access each other's data | Auth + RLS |
| Notifications lost on server restart | Job queue (BullMQ) |

If you understand WHY something exists, you will never forget HOW to use it.

---

## Phase 1 — Foundation

### Why Node.js?
JavaScript runs in browsers. Node.js lets JavaScript run on a server (your computer or Railway). Without Node.js, your TypeScript code has nowhere to run.

### Why TypeScript?
JavaScript lets you write anything and only crashes when the app is already running — in production, in front of users. TypeScript catches the same mistakes before you even run the code. It is JavaScript with a safety net.

**The rule:** Every object must have an interface. Every function must declare what it returns.

### Why .env files?
Your code will be pushed to GitHub (public). Your database password must never be in your code. The .env file lives only on your machine (and Railway's servers) — never in Git.

**The rule:** If it's a secret or changes between environments → it goes in .env, never in code.

---

## Phase 2 — Core Backend

### Why Fastify?
Your server needs something that listens for HTTP requests and routes them to the right function. Fastify does this and is the fastest Node.js server framework available. It also gives you built-in TypeScript support, JSON handling, and plugin architecture.

### Why Middleware?
Without middleware, you would write the same auth check in every single route — 50 routes means 50 copies of the same code. Middleware runs once before every request, protecting everything automatically.

**Mental model:** Middleware is a security guard at the building entrance. Every visitor passes through once. No guard = check ID at every room.

### Why the 4-layer module structure?
Each layer has one job. When a bug happens, you know exactly where to look:
- Wrong URL called? → routes.ts
- Wrong data returned? → controller.ts
- Wrong business logic? → service.ts
- Wrong data in DB? → migration SQL

**The rule:** Route → Controller → Service → Database. Each layer only talks to the one below it. Never skip layers.

### Why HTTP status codes matter?
The frontend, mobile app, and Telegram bot all make decisions based on your status code. A 401 means "show login screen." A 404 means "show not found." A 500 means "show error page." If you return 200 with an error message inside, every client breaks silently.

---

## Phase 3 — Database

### Why SQL?
Your data has relationships. A task belongs to a user. A user has many goals. SQL is designed specifically to store and query relational data. Supabase runs PostgreSQL — the most powerful open-source SQL database.

### Why Foreign Keys?
Without foreign keys, you can create a task for a user that doesn't exist. The database has no way to know the relationship is broken. Foreign keys enforce that relationships are always valid.

**ON DELETE CASCADE** means: if a user is deleted, their tasks, goals, and notifications delete automatically. Without it, you get ghost data — records pointing to nothing.

### Why Indexes?
Without an index, a query like "find all tasks for user X" scans every single row in the table — even if there are 1 million rows and only 10 belong to user X. An index is like a book's index — it jumps directly to the right page.

**The rule:** Every column you use in a WHERE clause needs an index.

### Why Row Level Security?
Even if your API has a bug that skips the auth check, RLS at the database level still blocks the wrong user from seeing data. It is your last line of defense.

**The rule:** Enable RLS on every table. Write a policy for every table. No exceptions.

### Why Migrations?
If you change the database manually (clicking in the dashboard), your changes are invisible to other developers and impossible to reproduce. Migration files are the history of every change ever made to the database — versioned, ordered, and repeatable.

**The rule:** Never touch the database manually. Every change is a new migration file.

### Why Zod Validation?
Users send wrong data. Attackers send malicious data. A task title should be a string between 1 and 200 characters — without validation, someone could send a 10MB string and crash your server. Zod validates and transforms input before it ever touches your database.

---

## Phase 4 — Auth, Async, Jobs

### Why JWT?
HTTP is stateless — every request is independent, the server remembers nothing. JWT solves this: the user logs in once, gets a token, and sends it with every future request. The server reads the token to know who is making the request — no database lookup needed.

**Mental model:** A wristband at an event. You show your ID once at the entrance, get the band, and flash it at every door inside.

### Why Hash Passwords?
If your database is ever stolen (it happens to everyone), hashed passwords are useless to the attacker. A hash is a one-way transformation — you can never reverse it back to the original password. bcrypt adds a random "salt" so even identical passwords hash differently.

**The rule:** Never store, log, or transmit plain text passwords. Ever.

### Why Async/Await?
Database calls, API calls, and file operations take time. If you don't use await, the next line runs before the operation finishes — data is undefined, bugs are silent and unpredictable.

**The rule:** Every function that touches a database, external API, or file system must be async and use await.

### Why BullMQ?
Your planner needs to send a Telegram reminder at exactly 2pm. Without a job queue, your server would have to stay alive and remember every reminder — one restart loses them all. BullMQ stores jobs in Redis (a fast in-memory database). Even if your server restarts 10 times, the jobs survive and execute at the right time.

**Mental model:** A post office. You drop a letter with a delivery date. The post office stores it. It delivers at the right time regardless of what else is happening.

### Why try/catch?
Every async operation can fail — network timeout, database down, invalid data. Without try/catch, one failure crashes the entire server for every user. With try/catch, you handle the failure gracefully and return a clear error response.

**The rule:** Every async function in a controller is wrapped in try/catch.

---

## The 10 Rules to Always Follow

1. Every object has a TypeScript interface
2. Secrets live in .env — never in code
3. Route → Controller → Service → Database — never skip layers
4. Every async operation uses await
5. Every controller has try/catch
6. Every request body is validated with Zod
7. Every table has RLS enabled with a policy
8. Every filter column has an index
9. Every schema change is a migration file
10. Every response uses the correct HTTP status code

---

## Why These Phases in This Order

```
Phase 1 (Foundation)
  └── Without a working environment, nothing else can be built

Phase 2 (Core Backend)
  └── Without a server and module structure, you have nowhere to put logic

Phase 3 (Database)
  └── Without a secure, indexed, validated database, data is unsafe and slow

Phase 4 (Auth + Jobs)
  └── Without auth, anyone can access anyone's data
      Without jobs, notifications are unreliable
```

Each phase depends on the one before it. This is the only correct order.
