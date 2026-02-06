---
description: Connect to Google Analytics to analyze and generate a dashboard in Squadbase.
---

## Connecting from a dashboard

Use the client file to access Google Analytics Data API in `lib/google-analytics.ts`.

- If `@google-analytics/data` is not installed, install it with `npm install @google-analytics/data`.
- If client file is not present, create it.

```typescript lib/google-analytics.ts
import { BetaAnalyticsDataClient } from "@google-analytics/data";

export function createGoogleAnalyticsClient() {
  const credentials = JSON.parse(
    Buffer.from(
      process.env.GA_SERVICE_ACCOUNT_JSON_BASE64!,
      "base64",
    ).toString("utf-8"),
  );
  return new BetaAnalyticsDataClient({
    credentials,
  });
}

export function getGAPropertyId() {
  return process.env.GA_PROPERTY_ID!;
}
```

## Explore data

To understand the data source or identify data to use in your dashboard, create and run scripts in `explore/` directory.

```typescript explore/get-active-users.ts
import {
  createGoogleAnalyticsClient,
  getGAPropertyId,
} from "../lib/google-analytics";

async function main() {
  const client = createGoogleAnalyticsClient();
  const propertyId = getGAPropertyId();

  const [response] = await client.runReport({
    property: `properties/${propertyId}`,
    dateRanges: [
      {
        startDate: "7daysAgo",
        endDate: "today",
      },
    ],
    dimensions: [
      {
        name: "date",
      },
    ],
    metrics: [
      {
        name: "activeUsers",
      },
    ],
  });

  console.log("Active Users (Last 7 Days):");
  response.rows?.forEach((row) => {
    console.log(`  ${row.dimensionValues?.[0]?.value}: ${row.metricValues?.[0]?.value}`);
  });
}

main();
```

- Create files in the `explore/` directory.
- Run scripts with `tsx` (e.g., `npx tsx explore/get-active-users.ts`).
- Only output the minimum information needed for exploration and understanding the data. Limiting output helps you understand the data more efficiently. For example, when checking specific metrics or data summaries, avoid printing all rows.
- Leave `explore/` scripts in the codebase for future reference.

## General guidelines for connectors and connections

- Connector: A plugin that connects to a specific data source (e.g., Google Analytics, BigQuery, etc.)
- Connection: An instance of a connector with specific configuration (e.g., credentials, property, etc.)
- Each connector supports multiple connections to allow users to connect to different data sources or accounts. For the second and subsequent connections to the same data source, use the following environment variables. The third and subsequent connections follow the same naming convention.
  - `GA_SERVICE_ACCOUNT_JSON_BASE64_2`
  - `GA_PROPERTY_ID_2`
    Create a separate client file for each connection, such as `lib/google-analytics-2.ts`.
