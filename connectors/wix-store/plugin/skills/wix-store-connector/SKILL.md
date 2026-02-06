---
description: Connect to Wix Store to analyze and generate a dashboard in Squadbase.
---

API Documentation: https://dev.wix.com/api/rest/wix-stores

## Connecting from a dashboard

Use the client file to access Wix Store in `lib/wix-store.ts`.

- If client file is not present, create it.

```typescript lib/wix-store.ts
export const BASE_URL = "https://www.wixapis.com";

export function getSiteId() {
  return process.env.WIX_SITE_ID!;
}

export function getAuthHeaders() {
  return {
    Authorization: process.env.WIX_API_KEY!,
    "wix-site-id": process.env.WIX_SITE_ID!,
  };
}
```

## Explore data

To understand the data source or identify data to use in your dashboard, create and run scripts in `explore/` directory.

```typescript explore/list-products.ts
import { BASE_URL, getAuthHeaders } from "../lib/wix-store";

async function main() {
  const response = await fetch(`${BASE_URL}/stores/v1/products/query`, {
    method: "POST",
    headers: {
      ...getAuthHeaders(),
      "Content-Type": "application/json",
    },
    body: JSON.stringify({
      query: {
        paging: { limit: 10 },
      },
    }),
  });
  const data = await response.json();

  console.log("Products:");
  data.products?.forEach((product: { id: string; name: string; price: { formatted: { price: string } } }) => {
    console.log(`  - [${product.id}] ${product.name} - ${product.price?.formatted?.price}`);
  });
}

main();
```

```typescript explore/list-orders.ts
import { BASE_URL, getAuthHeaders } from "../lib/wix-store";

async function main() {
  const response = await fetch(`${BASE_URL}/stores/v2/orders/query`, {
    method: "POST",
    headers: {
      ...getAuthHeaders(),
      "Content-Type": "application/json",
    },
    body: JSON.stringify({
      query: {
        paging: { limit: 10 },
      },
    }),
  });
  const data = await response.json();

  console.log("Orders:");
  data.orders?.forEach((order: { id: string; number: number; priceSummary: { total: { formattedAmount: string } } }) => {
    console.log(`  - [${order.number}] ${order.id} - ${order.priceSummary?.total?.formattedAmount}`);
  });
}

main();
```

- Create files in the `explore/` directory.
- Run scripts with `tsx` (e.g., `npx tsx explore/list-products.ts`).
- Only output the minimum information needed for exploration and understanding the data. Limiting output helps you understand the data more efficiently. For example, when checking specific metrics or data summaries, avoid printing all rows.
- Leave `explore/` scripts in the codebase for future reference.

## General guidelines for connectors and connections

- Connector: A plugin that connects to a specific data source (e.g., Wix Store, Shopify, etc.)
- Connection: An instance of a connector with specific configuration (e.g., credentials, site, etc.)
- Each connector supports multiple connections to allow users to connect to different data sources or accounts. For the second and subsequent connections to the same data source, use the following environment variables. The third and subsequent connections follow the same naming convention.
  - `WIX_ACCOUNT_ID_2`
  - `WIX_SITE_ID_2`
  - `WIX_API_KEY_2`
    Create a separate client file for each connection, such as `lib/wix-store-2.ts`.
