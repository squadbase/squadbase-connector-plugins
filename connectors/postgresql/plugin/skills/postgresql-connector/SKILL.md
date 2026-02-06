---
description: Connect to PostgreSQL to analyze and generate a dashboard in Squadbase.
---

## Connecting from a dashboard

Use the client file to access PostgreSQL in `lib/postgresql.ts`.

- If `pg` is not installed, install it with `npm install pg && npm install --save-dev @types/pg`.
- If client file is not present, create it.

```typescript lib/postgresql.ts
import { Pool } from "pg";

let pool: Pool | undefined;

export function createPostgresClient() {
  if (!pool) {
    pool = new Pool({
      connectionString: process.env.POSTGRES_URL!,
      ssl: { rejectUnauthorized: false },
    });
  }
  return pool;
}
```

## Explore data

To understand the data source or identify data to use in your dashboard, create and run scripts in `explore/` directory.

```typescript explore/list-tables.ts
import { createPostgresClient } from "../lib/postgresql";

interface TableInfo {
  table_name: string;
}

async function main() {
  const client = createPostgresClient();

  const result = await client.query<TableInfo>(`
    SELECT table_name
    FROM information_schema.tables
    WHERE table_schema = 'public'
    ORDER BY table_name
    LIMIT 20
  `);

  console.log("Tables:");
  result.rows.forEach((row) => {
    console.log(`  - ${row.table_name}`);
  });

  await client.end();
}

main();
```

- Create files in the `explore/` directory.
- Run scripts with `tsx` (e.g., `npx tsx explore/list-tables.ts`).
- Only output the minimum information needed for exploration and understanding the data. Limiting output helps you understand the data more efficiently. For example, when checking specific metrics or data summaries, avoid printing all rows.
- Leave `explore/` scripts in the codebase for future reference.

## General guidelines for connectors and connections

- Connector: A plugin that connects to a specific data source (e.g., PostgreSQL, MySQL, etc.)
- Connection: An instance of a connector with specific configuration (e.g., credentials, database, etc.)
- Each connector supports multiple connections to allow users to connect to different data sources or accounts. For the second and subsequent connections to the same data source, use the following environment variables. The third and subsequent connections follow the same naming convention.
  - `POSTGRES_URL_2`
    Create a separate client file for each connection, such as `lib/postgresql-2.ts`.
