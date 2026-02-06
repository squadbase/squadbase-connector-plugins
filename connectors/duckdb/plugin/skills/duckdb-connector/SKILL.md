---
description: Analyze Excel and CSV files using DuckDB to generate a dashboard in Squadbase.
---

## Connecting from a dashboard

Use the client file to access DuckDB in `lib/duckdb.ts`.

- If `@duckdb/node-api` is not installed, install it with `npm install @duckdb/node-api`.
- If client file is not present, create it.

```typescript lib/duckdb.ts
import { DuckDBInstance } from "@duckdb/node-api";

let instance: DuckDBInstance | null = null;

export async function createDuckDBConnection() {
  if (!instance) {
    instance = await DuckDBInstance.create();
  }
  return await instance.connect();
}
```

## Explore data

To understand the data source or identify data to use in your dashboard, create and run scripts in `explore/` directory.

```typescript explore/read-csv.ts
import { createDuckDBConnection } from "../lib/duckdb";

async function main() {
  const connection = await createDuckDBConnection();

  // Read a CSV file
  const result = await connection.run(`
    SELECT *
    FROM read_csv_auto('data/sample.csv')
    LIMIT 10
  `);

  const rows = result.getRows();

  console.log("Sample data:");
  rows.forEach((row) => {
    console.log(row);
  });
}

main();
```

```typescript explore/read-excel.ts
import { createDuckDBConnection } from "../lib/duckdb";

async function main() {
  const connection = await createDuckDBConnection();

  // Install and load spatial extension for Excel support
  await connection.run("INSTALL spatial");
  await connection.run("LOAD spatial");

  // Read an Excel file
  const result = await connection.run(`
    SELECT *
    FROM st_read('data/sample.xlsx')
    LIMIT 10
  `);

  const rows = result.getRows();

  console.log("Sample data:");
  rows.forEach((row) => {
    console.log(row);
  });
}

main();
```

- Create files in the `explore/` directory.
- Run scripts with `tsx` (e.g., `npx tsx explore/read-csv.ts`).
- Only output the minimum information needed for exploration and understanding the data. Limiting output helps you understand the data more efficiently. For example, when checking specific metrics or data summaries, avoid printing all rows.
- Leave `explore/` scripts in the codebase for future reference.

## General guidelines for connectors and connections

- Connector: A plugin that connects to a specific data source (e.g., DuckDB for Excel/CSV files)
- Connection: An instance of a connector with specific configuration
- DuckDB runs locally and does not require external credentials. Multiple connections share the same in-memory instance by default.
