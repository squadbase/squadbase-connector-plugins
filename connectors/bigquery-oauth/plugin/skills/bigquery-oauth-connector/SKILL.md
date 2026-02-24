---
description: Connect to BigQuery using OAuth to analyze and generate a dashboard in Squadbase.
---

## BigQuery REST API Reference

Refer to the BigQuery REST API Discovery Document for API specifications.

URL: https://bigquery.googleapis.com/$discovery/rest?version=v2

This document is very large. Do NOT read the file directly. Download it locally and use `jq` to read only the parts you need.

```bash
# Download
curl -s 'https://bigquery.googleapis.com/$discovery/rest?version=v2' -o /tmp/bigquery-discovery.json

# List available resources
jq '.resources | keys' /tmp/bigquery-discovery.json

# List methods for a specific resource
jq '.resources.jobs.methods | keys' /tmp/bigquery-discovery.json

# View details of a specific method
jq '.resources.jobs.methods.query' /tmp/bigquery-discovery.json
```

## Connecting from a dashboard

Use the client file to access BigQuery via OAuth proxy in `lib/bigquery.ts`.

- If `@squadbase/nextjs` is not installed, install it with `npm install @squadbase/nextjs`.
- If client file is not present, create it.

```typescript lib/bigquery.ts
import { createConnectionClient } from "@squadbase/nextjs";

export const BIGQUERY_API_BASE = "https://bigquery.googleapis.com/bigquery/v2";

export function createBigQueryClient() {
  return createConnectionClient({ connectionId: "<connection-id>" });
}

export function getProjectId() {
  return process.env.BIGQUERY_OAUTH_PROJECT_ID!;
}
```

## Explore data

To understand the data source or identify data to use in your dashboard, create and run scripts in `explore/` directory.

```typescript explore/list-datasets.ts
import {
  createBigQueryClient,
  BIGQUERY_API_BASE,
  getProjectId,
} from "../lib/bigquery";

async function main() {
  const client = createBigQueryClient();
  const projectId = getProjectId();

  const res = await client.fetch(
    `${BIGQUERY_API_BASE}/projects/${projectId}/datasets`,
  );
  const data = res.body as {
    datasets?: { datasetReference: { datasetId: string } }[];
  };

  console.log("Datasets:");
  data.datasets?.forEach((ds) => {
    console.log(`  - ${ds.datasetReference.datasetId}`);
  });
}

main();
```

```typescript explore/run-query.ts
import {
  createBigQueryClient,
  BIGQUERY_API_BASE,
  getProjectId,
} from "../lib/bigquery";

async function main() {
  const client = createBigQueryClient("YOUR_CONNECTION_ID");
  const projectId = getProjectId();

  const res = await client.fetch(
    `${BIGQUERY_API_BASE}/projects/${projectId}/queries`,
    {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: {
        query: "SELECT 1 AS test",
        useLegacySql: false,
      },
    },
  );
  console.log("Query result:", JSON.stringify(res.body, null, 2));
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
  - `BIGQUERY_OAUTH_PROJECT_ID_2`
    Create a separate client file for each connection, such as `lib/bigquery-2.ts`.
