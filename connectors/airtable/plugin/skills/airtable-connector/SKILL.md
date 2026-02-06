---
description: Connect to Airtable to analyze and generate a dashboard in Squadbase.
---

API Documentation: https://airtable.com/developers/web/api/introduction

## Connecting from a dashboard

Use the client file to access Airtable in `lib/airtable.ts`.

- If client file is not present, create it.

```typescript lib/airtable.ts
export const BASE_URL = "https://api.airtable.com/v0";

export function getBaseId() {
  return process.env.AIRTABLE_BASE_ID!;
}

export function getAuthHeaders() {
  return {
    Authorization: `Bearer ${process.env.AIRTABLE_API_KEY}`,
  };
}
```

## Explore data

To understand the data source or identify data to use in your dashboard, create and run scripts in `explore/` directory.

```typescript explore/get-tables.ts
import { BASE_URL, getBaseId, getAuthHeaders } from "../lib/airtable";

async function main() {
  const response = await fetch(
    `${BASE_URL}/meta/bases/${getBaseId()}/tables`,
    { headers: getAuthHeaders() },
  );
  const data = await response.json();

  console.log("Tables:");
  data.tables?.forEach((table: { id: string; name: string }) => {
    console.log(`  - [${table.id}] ${table.name}`);
  });
}

main();
```

```typescript explore/list-records.ts
import { BASE_URL, getBaseId, getAuthHeaders } from "../lib/airtable";

async function main() {
  // Replace "Table Name" with your actual table name
  const tableName = "Table Name";
  const response = await fetch(
    `${BASE_URL}/${getBaseId()}/${encodeURIComponent(tableName)}?maxRecords=10`,
    { headers: getAuthHeaders() },
  );
  const data = await response.json();

  console.log("Records:");
  data.records?.forEach((record: { id: string; fields: Record<string, unknown> }) => {
    console.log(`  - [${record.id}] ${JSON.stringify(record.fields)}`);
  });
}

main();
```

- Create files in the `explore/` directory.
- Run scripts with `tsx` (e.g., `npx tsx explore/get-tables.ts`).
- Only output the minimum information needed for exploration and understanding the data. Limiting output helps you understand the data more efficiently. For example, when checking specific metrics or data summaries, avoid printing all rows.
- Leave `explore/` scripts in the codebase for future reference.

## General guidelines for connectors and connections

- Connector: A plugin that connects to a specific data source (e.g., Airtable, Google Sheets, etc.)
- Connection: An instance of a connector with specific configuration (e.g., credentials, base, etc.)
- Each connector supports multiple connections to allow users to connect to different data sources or accounts. For the second and subsequent connections to the same data source, use the following environment variables. The third and subsequent connections follow the same naming convention.
  - `AIRTABLE_BASE_ID_2`
  - `AIRTABLE_API_KEY_2`
    Create a separate client file for each connection, such as `lib/airtable-2.ts`.
