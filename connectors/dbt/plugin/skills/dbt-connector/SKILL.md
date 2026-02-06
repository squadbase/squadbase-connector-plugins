---
description: Connect to dbt Cloud to explore model metadata and lineage for dashboard creation in Squadbase.
---

API Documentation: https://docs.getdbt.com/docs/dbt-cloud-apis/discovery-api

## Overview

This connector provides access to dbt Cloud Discovery API for understanding your dbt project structure before querying the underlying data warehouse (Snowflake, BigQuery, etc.).

Use this to:
- List models and their schemas
- Understand source definitions
- Explore lineage (upstream/downstream dependencies)

## Setup

Create a client file in `lib/dbt.ts`.

```typescript lib/dbt.ts
// Discovery API endpoint varies by region:
// - North America: https://metadata.cloud.getdbt.com/graphql
// - EMEA: https://metadata.emea.dbt.com/graphql
// - APAC: https://metadata.au.dbt.com/graphql
export function getDiscoveryApiUrl() {
  const host = process.env.DBT_HOST!;
  if (host.includes("emea")) {
    return "https://metadata.emea.dbt.com/graphql";
  } else if (host.includes("au")) {
    return "https://metadata.au.dbt.com/graphql";
  }
  return "https://metadata.cloud.getdbt.com/graphql";
}

export function getEnvironmentId() {
  return process.env.DBT_PROD_ENV_ID!;
}

export function getAuthHeaders() {
  return {
    Authorization: `Bearer ${process.env.DBT_TOKEN}`,
  };
}
```

## Explore model metadata

Use these scripts to understand your dbt project structure before writing queries against the data warehouse.

```typescript explore/list-models.ts
import { getDiscoveryApiUrl, getEnvironmentId, getAuthHeaders } from "../lib/dbt";

async function main() {
  const query = `
    query ($environmentId: BigInt!, $first: Int!) {
      environment(id: $environmentId) {
        applied {
          models(first: $first) {
            edges {
              node {
                uniqueId
                name
                description
                database
                schema
                materializedType
              }
            }
          }
        }
      }
    }
  `;

  const response = await fetch(getDiscoveryApiUrl(), {
    method: "POST",
    headers: {
      ...getAuthHeaders(),
      "Content-Type": "application/json",
    },
    body: JSON.stringify({
      query,
      variables: {
        environmentId: parseInt(getEnvironmentId()),
        first: 50,
      },
    }),
  });
  const data = await response.json();

  console.log("Models:");
  data.data?.environment?.applied?.models?.edges?.forEach(
    (edge: { node: { uniqueId: string; name: string; schema: string; materializedType: string } }) => {
      const model = edge.node;
      console.log(`  - [${model.schema}] ${model.name} (${model.materializedType})`);
    },
  );
}

main();
```

```typescript explore/list-sources.ts
import { getDiscoveryApiUrl, getEnvironmentId, getAuthHeaders } from "../lib/dbt";

async function main() {
  const query = `
    query ($environmentId: BigInt!, $first: Int!) {
      environment(id: $environmentId) {
        applied {
          sources(first: $first) {
            edges {
              node {
                uniqueId
                name
                sourceName
                database
                schema
                identifier
              }
            }
          }
        }
      }
    }
  `;

  const response = await fetch(getDiscoveryApiUrl(), {
    method: "POST",
    headers: {
      ...getAuthHeaders(),
      "Content-Type": "application/json",
    },
    body: JSON.stringify({
      query,
      variables: {
        environmentId: parseInt(getEnvironmentId()),
        first: 50,
      },
    }),
  });
  const data = await response.json();

  console.log("Sources:");
  data.data?.environment?.applied?.sources?.edges?.forEach(
    (edge: { node: { sourceName: string; name: string; schema: string; identifier: string } }) => {
      const source = edge.node;
      console.log(`  - [${source.sourceName}] ${source.name} -> ${source.schema}.${source.identifier}`);
    },
  );
}

main();
```

```typescript explore/get-lineage.ts
import { getDiscoveryApiUrl, getEnvironmentId, getAuthHeaders } from "../lib/dbt";

async function main() {
  // Replace with your model's uniqueId
  const modelUniqueId = "model.your_project.your_model";

  const query = `
    query ($environmentId: BigInt!, $first: Int!, $uniqueIds: [String!]) {
      environment(id: $environmentId) {
        applied {
          models(first: $first, filter: { uniqueIds: $uniqueIds }) {
            edges {
              node {
                name
                ancestors(types: [Model, Source, Seed, Snapshot]) {
                  ... on ModelAppliedStateNestedNode { uniqueId name resourceType }
                  ... on SourceAppliedStateNestedNode { uniqueId name resourceType }
                  ... on SeedAppliedStateNestedNode { uniqueId name resourceType }
                  ... on SnapshotAppliedStateNestedNode { uniqueId name resourceType }
                }
                children {
                  ... on ModelAppliedStateNestedNode { uniqueId name resourceType }
                }
              }
            }
          }
        }
      }
    }
  `;

  const response = await fetch(getDiscoveryApiUrl(), {
    method: "POST",
    headers: {
      ...getAuthHeaders(),
      "Content-Type": "application/json",
    },
    body: JSON.stringify({
      query,
      variables: {
        environmentId: parseInt(getEnvironmentId()),
        first: 1,
        uniqueIds: [modelUniqueId],
      },
    }),
  });
  const data = await response.json();

  const model = data.data?.environment?.applied?.models?.edges?.[0]?.node;
  if (model) {
    console.log(`Lineage for ${model.name}:`);
    console.log("\nUpstream (ancestors):");
    model.ancestors?.forEach((ancestor: { uniqueId: string; name: string; resourceType: string }) => {
      console.log(`  - [${ancestor.resourceType}] ${ancestor.name}`);
    });
    console.log("\nDownstream (children):");
    model.children?.forEach((child: { uniqueId: string; name: string; resourceType: string }) => {
      console.log(`  - [${child.resourceType}] ${child.name}`);
    });
  }
}

main();
```

- Run scripts with `tsx` (e.g., `npx tsx explore/list-models.ts`).
- Use the model/source information to write queries against your data warehouse (Snowflake, BigQuery, etc.).

## General guidelines for connectors and connections

- Connector: A plugin that connects to a specific data source (e.g., dbt Cloud, Snowflake, etc.)
- Connection: An instance of a connector with specific configuration (e.g., credentials, environment, etc.)
- Each connector supports multiple connections to allow users to connect to different data sources or accounts. For the second and subsequent connections to the same data source, use the following environment variables. The third and subsequent connections follow the same naming convention.
  - `DBT_HOST_2`
  - `DBT_ACCOUNT_ID_2`
  - `DBT_PROD_ENV_ID_2`
  - `DBT_TOKEN_2`
    Create a separate client file for each connection, such as `lib/dbt-2.ts`.
