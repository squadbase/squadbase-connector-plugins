---
description: Connect to AWS Athena to analyze and generate a dashboard in Squadbase.
---

## Connecting from a dashboard

Use the client file to access AWS Athena in `lib/aws-athena.ts`.

- If `@aws-sdk/client-athena` is not installed, install it with `npm install @aws-sdk/client-athena`.
- If client file is not present, create it.

```typescript lib/aws-athena.ts
import {
  AthenaClient,
  StartQueryExecutionCommand,
  StartQueryExecutionCommandInput,
  GetQueryExecutionCommand,
  GetQueryResultsCommand,
} from "@aws-sdk/client-athena";

const client = new AthenaClient({
  region: process.env.ATHENA_AWS_REGION!,
  credentials: {
    accessKeyId: process.env.ATHENA_AWS_ACCESS_KEY_ID!,
    secretAccessKey: process.env.ATHENA_AWS_SECRET_ACCESS_KEY!,
  },
});

export async function executeQuery(sql: string) {
  const params: StartQueryExecutionCommandInput = {
    QueryString: sql,
  };

  // Use either WorkGroup or OutputLocation (at least one is required)
  if (process.env.ATHENA_WORKGROUP) {
    params.WorkGroup = process.env.ATHENA_WORKGROUP;
  }
  if (process.env.ATHENA_OUTPUT_LOCATION) {
    params.ResultConfiguration = {
      OutputLocation: process.env.ATHENA_OUTPUT_LOCATION,
    };
  }

  const startCommand = new StartQueryExecutionCommand(params);
  const { QueryExecutionId } = await client.send(startCommand);

  // Wait for query completion
  let status = "RUNNING";
  while (status === "RUNNING" || status === "QUEUED") {
    const statusCommand = new GetQueryExecutionCommand({ QueryExecutionId });
    const response = await client.send(statusCommand);
    status = response.QueryExecution?.Status?.State || "FAILED";
    if (status === "RUNNING" || status === "QUEUED") {
      await new Promise((resolve) => setTimeout(resolve, 1000));
    }
  }

  if (status !== "SUCCEEDED") {
    throw new Error(`Query failed with status: ${status}`);
  }

  const resultsCommand = new GetQueryResultsCommand({ QueryExecutionId });
  return client.send(resultsCommand);
}
```

## Explore data

To understand the data source or identify data to use in your dashboard, create and run scripts in `explore/` directory.

```typescript explore/list-tables.ts
import { executeQuery } from "../lib/aws-athena";

async function main() {
  const result = await executeQuery("SHOW TABLES");

  console.log("Tables:");
  result.ResultSet?.Rows?.slice(1).forEach((row) => {
    console.log(`  - ${row.Data?.[0]?.VarCharValue}`);
  });
}

main();
```

- Create files in the `explore/` directory.
- Run scripts with `tsx` (e.g., `npx tsx explore/list-tables.ts`).
- Only output the minimum information needed for exploration and understanding the data. Limiting output helps you understand the data more efficiently. For example, when checking specific metrics or data summaries, avoid printing all rows.
- Leave `explore/` scripts in the codebase for future reference.

## General guidelines for connectors and connections

- Connector: A plugin that connects to a specific data source (e.g., AWS Athena, BigQuery, etc.)
- Connection: An instance of a connector with specific configuration (e.g., credentials, region, etc.)
- Each connector supports multiple connections to allow users to connect to different data sources or accounts. For the second and subsequent connections to the same data source, use the following environment variables. The third and subsequent connections follow the same naming convention.
  - `ATHENA_AWS_ACCESS_KEY_ID_2`
  - `ATHENA_AWS_SECRET_ACCESS_KEY_2`
  - `ATHENA_AWS_REGION_2`
  - `ATHENA_WORKGROUP_2` (optional)
  - `ATHENA_OUTPUT_LOCATION_2` (optional)
    Create a separate client file for each connection, such as `lib/aws-athena-2.ts`.
