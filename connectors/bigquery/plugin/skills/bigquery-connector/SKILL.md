---
description: Connect to BigQuery to analyze and generate a dashboard in Squadbase.
---

## Connecting from a dashboard

Use the client file to access BigQuery in `lib/bigquery.ts`.

- If `@google-cloud/bigquery` is not installed, install it with `npm install @google-cloud/bigquery`.
- If client file is not present, create it.

```typescript lib/bigquery.ts
import { BigQuery } from "@google-cloud/bigquery";

export function createBigQueryClient() {
  const base64Credentials = process.env.BIGQUERY_SERVICE_ACCOUNT_JSON_BASE64;

  if (!base64Credentials) {
    throw new Error(
      "BIGQUERY_SERVICE_ACCOUNT_JSON_BASE64 environment variable is not set",
    );
  }

  const credentials = JSON.parse(
    Buffer.from(base64Credentials, "base64").toString("utf-8"),
  );

  return new BigQuery({
    projectId: process.env.BIGQUERY_PROJECT_ID,
    credentials,
  });
}
```

## Explore data

To understand the data source or identify data to use in your dashboard, create and run scripts in `explore/` directory.

```typescript explore/list-datasets.ts
import { createBigQueryClient } from "../lib/bigquery";

async function main() {
  const bigquery = createBigQueryClient();

  const [datasets] = await bigquery.getDatasets();

  console.log("Datasets:");
  datasets.forEach((dataset) => {
    console.log(`  - ${dataset.id}`);
  });
}

main();
```

- Create files in the `explore/` directory.
- Run scripts with `tsx` (e.g., `npx tsx explore/list-datasets.ts`).
- Only output the minimum information needed for exploration and understanding the data. Limiting output helps you understand the data more efficiently. For example, when checking specific metrics or data summaries, avoid printing all rows.
- Leave `explore/` scripts in the codebase for future reference.

## General guidelines for connectors and connections

- Connector: A plugin that connects to a specific data source (e.g., BigQuery, Snowflake, etc.)
- Connection: An instance of a connector with specific configuration (e.g., credentials, project, etc.)
- Each connector supports multiple connections to allow users to connect to different data sources or accounts. For the second and subsequent connections to the same data source, use the following environment variables. The third and subsequent connections follow the same naming convention.
  - `BIGQUERY_SERVICE_ACCOUNT_JSON_BASE64_2`
  - `BIGQUERY_PROJECT_ID_2`
    Create a separate client file for each connection, such as `lib/bigquery-2.ts`.
