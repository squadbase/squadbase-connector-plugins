---
description: Connect to Redshift to analyze and generate a dashboard in Squadbase.
---

## Connecting from a dashboard

Use the client file to access Redshift in `lib/redshift.ts`.

- If `@aws-sdk/client-redshift-data` is not installed, install it with `npm install @aws-sdk/client-redshift-data`.
- If client file is not present, create it.

```typescript lib/redshift.ts
import {
  RedshiftDataClient,
  ExecuteStatementCommand,
  ExecuteStatementCommandInput,
} from "@aws-sdk/client-redshift-data";

const client = new RedshiftDataClient({
  region: process.env.REDSHIFT_AWS_REGION!,
  credentials: {
    accessKeyId: process.env.REDSHIFT_AWS_ACCESS_KEY_ID!,
    secretAccessKey: process.env.REDSHIFT_AWS_SECRET_ACCESS_KEY!,
  },
});

export async function executeQuery(sql: string) {
  const params: ExecuteStatementCommandInput = {
    Database: process.env.REDSHIFT_DATABASE!,
    Sql: sql,
  };

  // Use either Provisioned Cluster or Serverless
  if (process.env.REDSHIFT_CLUSTER_IDENTIFIER) {
    params.ClusterIdentifier = process.env.REDSHIFT_CLUSTER_IDENTIFIER;
  } else if (process.env.REDSHIFT_WORKGROUP_NAME) {
    params.WorkgroupName = process.env.REDSHIFT_WORKGROUP_NAME;
  }

  // Authentication: use SecretArn or DbUser
  if (process.env.REDSHIFT_SECRET_ARN) {
    params.SecretArn = process.env.REDSHIFT_SECRET_ARN;
  } else if (process.env.REDSHIFT_DB_USER) {
    params.DbUser = process.env.REDSHIFT_DB_USER;
  }

  const command = new ExecuteStatementCommand(params);
  return client.send(command);
}
```

## Explore data

To understand the data source or identify data to use in your dashboard, create and run scripts in `explore/` directory.

```typescript explore/list-tables.ts
import {
  RedshiftDataClient,
  DescribeStatementCommand,
  GetStatementResultCommand,
} from "@aws-sdk/client-redshift-data";
import { executeQuery } from "../lib/redshift";

async function main() {
  const client = new RedshiftDataClient({
    region: process.env.REDSHIFT_AWS_REGION!,
    credentials: {
      accessKeyId: process.env.REDSHIFT_AWS_ACCESS_KEY_ID!,
      secretAccessKey: process.env.REDSHIFT_AWS_SECRET_ACCESS_KEY!,
    },
  });

  const response = await executeQuery(`
    SELECT tablename
    FROM pg_tables
    WHERE schemaname = 'public'
    ORDER BY tablename
    LIMIT 20
  `);

  // Wait for query to complete
  let status = "SUBMITTED";
  while (status === "SUBMITTED" || status === "PICKED" || status === "STARTED") {
    await new Promise((resolve) => setTimeout(resolve, 1000));
    const describeResponse = await client.send(
      new DescribeStatementCommand({ Id: response.Id }),
    );
    status = describeResponse.Status || "FAILED";
  }

  if (status === "FINISHED") {
    const resultResponse = await client.send(
      new GetStatementResultCommand({ Id: response.Id }),
    );

    console.log("Tables:");
    resultResponse.Records?.forEach((record) => {
      console.log(`  - ${record[0]?.stringValue}`);
    });
  } else {
    console.error("Query failed with status:", status);
  }
}

main();
```

- Create files in the `explore/` directory.
- Run scripts with `tsx` (e.g., `npx tsx explore/list-tables.ts`).
- Only output the minimum information needed for exploration and understanding the data. Limiting output helps you understand the data more efficiently. For example, when checking specific metrics or data summaries, avoid printing all rows.
- Leave `explore/` scripts in the codebase for future reference.

## General guidelines for connectors and connections

- Connector: A plugin that connects to a specific data source (e.g., Redshift, BigQuery, etc.)
- Connection: An instance of a connector with specific configuration (e.g., credentials, database, etc.)
- Each connector supports multiple connections to allow users to connect to different data sources or accounts. For the second and subsequent connections to the same data source, use the following environment variables. The third and subsequent connections follow the same naming convention.
  - `REDSHIFT_AWS_ACCESS_KEY_ID_2`
  - `REDSHIFT_AWS_SECRET_ACCESS_KEY_2`
  - `REDSHIFT_AWS_REGION_2`
  - `REDSHIFT_DATABASE_2`
  - `REDSHIFT_CLUSTER_IDENTIFIER_2` (optional)
  - `REDSHIFT_WORKGROUP_NAME_2` (optional)
  - `REDSHIFT_SECRET_ARN_2` (optional)
  - `REDSHIFT_DB_USER_2` (optional)
    Create a separate client file for each connection, such as `lib/redshift-2.ts`.
