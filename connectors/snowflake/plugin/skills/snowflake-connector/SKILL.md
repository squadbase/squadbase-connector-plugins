---
description: Connect to Snowflake to analyze and generate a dashboard in Squadbase.
---

## Connecting from a dashboard

Use the client file to access Snowflake in `lib/snowflake.ts`:

```typescript lib/snowflake.ts
import snowflake from "snowflake-sdk";

// Disable Snowflake SDK logging
snowflake.configure({ logLevel: "OFF" });

export function createSnowflakeConnection() {
  return snowflake.createConnection({
    account: process.env.SNOWFLAKE_ACCOUNT!,
    username: process.env.SNOWFLAKE_USER!,
    role: process.env.SNOWFLAKE_ROLE!,
    warehouse: process.env.SNOWFLAKE_WAREHOUSE!,
    authenticator: "SNOWFLAKE_JWT",
    privateKey: Buffer.from(
      process.env.SNOWFLAKE_PRIVATE_KEY_BASE64 ?? "",
      "base64",
    ).toString("utf-8"),
  });
}
```

- If client file is not present, create it.
- If `snowflake-sdk` is not installed, install it with `npm install snowflake-sdk`.

## Explore data

To understand the data source or identify data to use in your dashboard, create and run temporary scripts.

Use the Snowflake client above to explore datasets. For example:

Example exploration script:

```typescript explore/user-activity.ts
import { createSnowflakeConnection } from "../lib/snowflake";

const connection = createSnowflakeConnection();

await new Promise<void>((resolve, reject) => {
  connection.connect((err) => {
    if (err) reject(err);
    else resolve();
  });
});

interface UserActivity {
  USER_ID: string;
  ACTIVITY_TYPE: string;
  ACTIVITY_TIMESTAMP: Date;
}

const rows = await new Promise<UserActivity[]>((resolve, reject) => {
  connection.execute({
    sqlText: `
      SELECT
        user_id,
        activity_type,
        activity_timestamp
      FROM
        your_database.your_schema.user_activity
      LIMIT 10
    `,
    complete: (err, _stmt, rows) => {
      if (err) reject(err);
      else resolve((rows ?? []) as UserActivity[]);
    },
  });
});

console.log("User Activity:");

rows.forEach((row) => {
  console.log(
    `${row.USER_ID}: ${row.ACTIVITY_TYPE} at ${row.ACTIVITY_TIMESTAMP}`,
  );
});

connection.destroy(() => {});
```

- Create files in the `explore/` directory.
- Run scripts with `tsx` (e.g., `npx tsx explore/user-activity.ts`).
- Only output the minimum information needed for exploration and understanding the data. Limiting output helps you understand the data more efficiently. For example, when checking specific metrics or data summaries, avoid printing all rows.
- Leave `explore/` scripts in the codebase for future reference.

## General guidelines for connectors and connections

- Connector: A plugin that connects to a specific data source (e.g., Snowflake, Postgres, etc.)
- Connection: An instance of a connector with specific configuration (e.g., credentials, database, etc.)
- Each connector supports multiple connections to allow users to connect to different data sources or accounts. For the second and subsequent connections to the same data source, use the following environment variables. The third and subsequent connections follow the same naming convention.
  - `SNOWFLAKE_ACCOUNT_2`
  - `SNOWFLAKE_USER_2`
  - `SNOWFLAKE_ROLE_2`
  - `SNOWFLAKE_WAREHOUSE_2`
  - `SNOWFLAKE_PRIVATE_KEY_BASE64_2`
    Create a separate client file for each connection, such as `lib/snowflake-2.ts`.
