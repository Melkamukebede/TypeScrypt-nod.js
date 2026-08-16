Incoming Request
      │
      ▼
  Middleware 1 (rate limiter — is this user sending too many requests?)
      │
      ▼
  Middleware 2 (auth — is this user logged in?)
      │
      ▼
  Middleware 3 (validation — is the request body correct?)
      │
      ▼
  Route Handler (controller — process the request)
      │
      ▼
  Service (business logic — talk to database)
      │
      ▼
  Database (Supabase — fetch or save data)
      │
      ▼
  Response sent back to client
