---
description: Connect to MySQL to analyze and generate a dashboard in Squadbase.
---

## Connecting from a dashboard

Use the client file to access MySQL in `lib/mysql.ts`.

- If `mysql2` is not installed, install it with `npm install mysql2`.
- If client file is not present, create it.

```typescript lib/mysql.ts
import mysql from "mysql2/promise";

let pool: mysql.Pool | undefined;

export function createMySQLClient() {
  if (!pool) {
    pool = mysql.createPool(process.env.MYSQL_URL!);
  }
  return pool;
}
```

## Explore data

To understand the data source or identify data to use in your dashboard, create and run scripts in `explore/` directory.

```typescript explore/list-tables.ts
import { RowDataPacket } from "mysql2/promise";
import { createMySQLClient } from "../lib/mysql";

async function main() {
  const pool = createMySQLClient();

  const [rows] = await pool.query<RowDataPacket[]>(`
    SELECT TABLE_NAME
    FROM information_schema.TABLES
    WHERE TABLE_SCHEMA = DATABASE()
    ORDER BY TABLE_NAME
    LIMIT 20
  `);

  console.log("Tables:");
  rows.forEach((row) => {
    console.log(`  - ${row.TABLE_NAME}`);
  });

  await pool.end();
}

main();
```

- Create files in the `explore/` directory.
- Run scripts with `tsx` (e.g., `npx tsx explore/list-tables.ts`).
- Only output the minimum information needed for exploration and understanding the data. Limiting output helps you understand the data more efficiently. For example, when checking specific metrics or data summaries, avoid printing all rows.
- Leave `explore/` scripts in the codebase for future reference.

## General guidelines for connectors and connections

- Connector: A plugin that connects to a specific data source (e.g., MySQL, PostgreSQL, etc.)
- Connection: An instance of a connector with specific configuration (e.g., credentials, database, etc.)
- Each connector supports multiple connections to allow users to connect to different data sources or accounts. For the second and subsequent connections to the same data source, use the following environment variables. The third and subsequent connections follow the same naming convention.
  - `MYSQL_URL_2`
    Create a separate client file for each connection, such as `lib/mysql-2.ts`.
