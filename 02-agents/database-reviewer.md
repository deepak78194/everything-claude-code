# Agent: database-reviewer

**Category:** Data  
**Model Tier:** `sonnet`  
**Source:** `agents/database-reviewer.md`

---

## Purpose

The database-reviewer agent is a **PostgreSQL and Supabase specialist** for query optimization, schema design, security, and performance. It ensures database code follows best practices, prevents performance issues, and maintains data integrity.

---

## When to Use

Invoke this agent:
- When writing SQL queries
- When designing or modifying database schemas
- When creating database migrations
- When experiencing slow queries
- When implementing Row Level Security (RLS) policies
- When troubleshooting database performance

---

## Responsibilities

1. **Query optimization** — Identify slow queries and missing indexes
2. **Schema design** — Normalize tables, enforce integrity constraints
3. **Security** — RLS policies, injection prevention, privilege review
4. **Migration safety** — Rollback strategies, zero-downtime migrations
5. **Performance** — N+1 prevention, connection pooling, caching

---

## Inputs

- SQL files, migration files, or ORM model code
- Access to codebase (Read/Write/Edit/Bash/Grep/Glob)
- Schema diagrams or existing table definitions

## Outputs

- Optimized queries with explanations
- Schema improvement recommendations
- Index recommendations with EXPLAIN ANALYZE output
- RLS policy implementations

---

## Tools

| Tool | Usage |
|---|---|
| `Read` | Read schema files and migrations |
| `Write` | Create new migration files |
| `Edit` | Optimize existing queries |
| `Bash` | Run `EXPLAIN ANALYZE`, `psql` diagnostics |
| `Grep` | Search for query patterns |
| `Glob` | Find migration and schema files |

---

## Query Optimization Checklist

### Index Analysis
```sql
-- Check query plan
EXPLAIN ANALYZE SELECT * FROM orders WHERE user_id = $1 AND status = 'pending';

-- Add missing composite index
CREATE INDEX CONCURRENTLY idx_orders_user_status 
ON orders(user_id, status) 
WHERE status = 'pending';  -- Partial index for active rows only
```

### N+1 Query Prevention
```sql
-- BAD: N+1 (one query per user)
-- Application fetches users, then for each user fetches orders

-- GOOD: Single JOIN query
SELECT u.id, u.email, json_agg(o.*) as orders
FROM users u
LEFT JOIN orders o ON o.user_id = u.id
GROUP BY u.id, u.email;
```

### Pagination
```sql
-- BAD: OFFSET pagination (slow at high offsets)
SELECT * FROM posts ORDER BY created_at DESC OFFSET 10000 LIMIT 20;

-- GOOD: Cursor-based pagination
SELECT * FROM posts 
WHERE created_at < $cursor 
ORDER BY created_at DESC 
LIMIT 20;
```

---

## Schema Design Standards

### Naming Conventions
- Tables: `snake_case`, plural (`users`, `user_sessions`)
- Columns: `snake_case` (`created_at`, `user_id`)
- Indexes: `idx_<table>_<columns>` (`idx_users_email`)
- Foreign keys: `fk_<table>_<referenced>` (`fk_orders_user_id`)

### Required Columns
```sql
CREATE TABLE resources (
  id          uuid        PRIMARY KEY DEFAULT gen_random_uuid(),
  created_at  timestamptz NOT NULL DEFAULT now(),
  updated_at  timestamptz NOT NULL DEFAULT now()
);
```

### Constraints
```sql
-- Always add appropriate constraints
email TEXT NOT NULL CHECK (email ~* '^[^@]+@[^@]+\.[^@]+$'),
status TEXT NOT NULL CHECK (status IN ('active', 'inactive', 'deleted')),
amount NUMERIC(10, 2) NOT NULL CHECK (amount >= 0)
```

---

## Row Level Security (RLS) — Supabase

```sql
-- Enable RLS on table
ALTER TABLE user_data ENABLE ROW LEVEL SECURITY;

-- Users can only see their own data
CREATE POLICY "users_own_data" ON user_data
  FOR ALL
  USING (auth.uid() = user_id);

-- Service role bypass (for backend operations)
-- Use service role key, not anon key for admin operations
```

---

## Migration Safety

```sql
-- Safe: Adding a column with default
ALTER TABLE users ADD COLUMN preferences jsonb NOT NULL DEFAULT '{}';

-- Unsafe: Adding NOT NULL without default on large table
-- First: add nullable column
ALTER TABLE users ADD COLUMN preferences jsonb;
-- Then: backfill
UPDATE users SET preferences = '{}' WHERE preferences IS NULL;
-- Then: add constraint
ALTER TABLE users ALTER COLUMN preferences SET NOT NULL;
```

---

## Platform Adaptation

### GitHub Copilot
Add database review checklist to `copilot-instructions.md`. Use Supabase MCP server for direct database introspection.

### IntelliJ AI Assistant
Use IntelliJ's database tools (DataGrip integration) alongside this agent. Create an AI Action for "Review SQL" with this agent's system prompt.

### Monorepo Usage
Apply per service that owns database access. For shared databases, designate a single team as owner and route all schema changes through their review process.

---

## Dependencies

- Consults: **security-reviewer** (for RLS policy and injection review)
- Followed by: **tdd-guide** (database changes need migration tests)
- Interacts with: **architect** (database design decisions)
