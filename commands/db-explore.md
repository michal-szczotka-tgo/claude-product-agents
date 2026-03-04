---
name: db-explore
description: Understand the Redshift database schema (tables, columns, relationships) using exported DDL files in the database-context directory. Use when the user says "show me the database", "what tables", "db schema", "explore database", or needs to understand data models before making changes.
---

# Database Exploration

TransferGo uses Amazon Redshift (accessed via DataGrip with SSO). Schema exports live in the `database-context/` directory.

## How to Use

1. Read schema files from `database-context/` in the product workspace
2. Parse CREATE TABLE statements to understand structure
3. Identify relationships via foreign keys and naming conventions
4. Present findings in business-friendly language

## Output Format

```
Table: <table_name>
Purpose: <inferred purpose>
Key columns:
  - id (uuid, PK)
  - user_id (uuid, FK -> users)
  - status (varchar) - values: active, inactive, suspended
  - created_at (timestamp)
Relationships:
  - belongs_to: users
  - has_many: transactions
```

## If Schema Is Missing or Outdated

Tell the user:
1. Open DataGrip and connect to DWH Production
2. Right-click the `dms` database -> SQL Scripts -> Generate DDL to Clipboard
3. Save output to `database-context/schema.sql`

## Rules

- Schema files show structure only, not live data
- For live queries (counts, recent records, analysis), direct the user to DataGrip
- Focus on business meaning, not technical details -- the user is a PM
