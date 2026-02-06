---
description: Connect to Databricks to analyze and generate a dashboard in Squadbase.
---

## Connecting from a dashboard

Use the client file to access Databricks SQL Warehouse in `lib/databricks.ts`.

- If `@databricks/sql` is not installed, install it with `npm install @databricks/sql`.
- If client file is not present, create it.

```typescript lib/databricks.ts
import { DBSQLClient } from "@databricks/sql";

export function createDatabricksClient() {
  const client = new DBSQLClient();

  return client.connect({
    host: process.env.DATABRICKS_HOST!,
    path: process.env.DATABRICKS_HTTP_PATH!,
    token: process.env.DATABRICKS_TOKEN!,
  });
}
```

## Explore data

To understand the data source or identify data to use in your dashboard, create and run scripts in `explore/` directory.

```typescript explore/list-tables.ts
import { createDatabricksClient } from "../lib/databricks";

async function main() {
  const connection = await createDatabricksClient();
  const session = await connection.openSession();

  const result = await session.executeStatement("SHOW TABLES", {
    runAsync: true,
  });

  const rows = await result.fetchAll();

  console.log("Tables:");
  rows.slice(0, 20).forEach((row: Record<string, unknown>) => {
    console.log(`  - ${row.tableName}`);
  });

  await result.close();
  await session.close();
  await connection.close();
}

main();
```

- Create files in the `explore/` directory.
- Run scripts with `tsx` (e.g., `npx tsx explore/list-tables.ts`).
- Only output the minimum information needed for exploration and understanding the data. Limiting output helps you understand the data more efficiently. For example, when checking specific metrics or data summaries, avoid printing all rows.
- Leave `explore/` scripts in the codebase for future reference.

## General guidelines for connectors and connections

- Connector: A plugin that connects to a specific data source (e.g., Databricks, Snowflake, etc.)
- Connection: An instance of a connector with specific configuration (e.g., credentials, workspace, etc.)
- Each connector supports multiple connections to allow users to connect to different data sources or accounts. For the second and subsequent connections to the same data source, use the following environment variables. The third and subsequent connections follow the same naming convention.
  - `DATABRICKS_HOST_2`
  - `DATABRICKS_TOKEN_2`
  - `DATABRICKS_HTTP_PATH_2`
    Create a separate client file for each connection, such as `lib/databricks-2.ts`.
