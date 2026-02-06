---
description: Connect to kintone to analyze and generate a dashboard in Squadbase.
---

## Connecting from a dashboard

Use the client file to access kintone in `lib/kintone.ts`.

- If `@kintone/rest-api-client` is not installed, install it with `npm install @kintone/rest-api-client`.
- If client file is not present, create it.

```typescript lib/kintone.ts
import { KintoneRestAPIClient } from "@kintone/rest-api-client";

let client: KintoneRestAPIClient | undefined;

export function createKintoneClient() {
  if (!client) {
    const baseUrl = process.env.KINTONE_BASE_URL;
    const username = process.env.KINTONE_USERNAME;
    const password = process.env.KINTONE_PASSWORD;

    if (!baseUrl) {
      throw new Error("KINTONE_BASE_URL environment variable is not set");
    }

    if (!username) {
      throw new Error("KINTONE_USERNAME environment variable is not set");
    }

    if (!password) {
      throw new Error("KINTONE_PASSWORD environment variable is not set");
    }

    client = new KintoneRestAPIClient({
      baseUrl,
      auth: {
        username,
        password,
      },
    });
  }

  return client;
}
```

## Explore data

To understand the data source or identify data to use in your dashboard, create and run scripts in `explore/` directory.

```typescript explore/list-apps.ts
import { createKintoneClient } from "../lib/kintone";

async function main() {
  const client = createKintoneClient();

  const apps = await client.app.getApps({});

  console.log("Apps:");
  apps.apps.slice(0, 20).forEach((app) => {
    console.log(`  - [${app.appId}] ${app.name}`);
  });
}

main();
```

```typescript explore/get-records.ts
import { createKintoneClient } from "../lib/kintone";

async function main() {
  const client = createKintoneClient();

  // Replace with your app ID
  const appId = 1;

  const records = await client.record.getRecords({
    app: appId,
    query: "order by $id asc limit 10",
  });

  console.log(`Records from App ${appId}:`);
  records.records.forEach((record) => {
    console.log(record);
  });
}

main();
```

- Create files in the `explore/` directory.
- Run scripts with `tsx` (e.g., `npx tsx explore/list-apps.ts`).
- Only output the minimum information needed for exploration and understanding the data. Limiting output helps you understand the data more efficiently. For example, when checking specific metrics or data summaries, avoid printing all rows.
- Leave `explore/` scripts in the codebase for future reference.

## General guidelines for connectors and connections

- Connector: A plugin that connects to a specific data source (e.g., kintone, Salesforce, etc.)
- Connection: An instance of a connector with specific configuration (e.g., credentials, environment, etc.)
- Each connector supports multiple connections to allow users to connect to different data sources or accounts. For the second and subsequent connections to the same data source, use the following environment variables. The third and subsequent connections follow the same naming convention.
  - `KINTONE_BASE_URL_2`
  - `KINTONE_USERNAME_2`
  - `KINTONE_PASSWORD_2`
    Create a separate client file for each connection, such as `lib/kintone-2.ts`.
