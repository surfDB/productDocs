---
description: Get started with creating,reading and updating data
---

# Surf Client

{% hint style="info" %}
Surf client sdk can only be used server side for now
{% endhint %}

## Create your first data

Need to ensure we are authenticated before we call any of the write functions

```javascript
// req: API request 
// res: API response
await surfClient.create(req, res, {
  schema: "test",
  data: req.body,
});
```

## Read it

```typescript
// get a particular data by id and schema name
await surfClient.get(req, res, {
  schema:"test",
  id:"..."
});

// get all rows in a schema
await surfClient.getAll(req, res, {
  schema: "test",
});

```

## Update it

```javascript
await surfClient.create(req, res, {
  schema: "test",
  id:"...",
  data: req.body.data
});
```

## Realtime

```javascript
const realtime = new SurfRealtime({
  client: "http://localhost:4200",
});

// get updates whenver a row is updated
realtime.onUpdate((update) => {
  console.log({ update });
});

// get updates whenever a new row is created
realtime.onCreate((create) => {
  console.log({ create });
});

```

## Next.js example

```typescript
// pages/api/data/index.ts
import { NextApiRequest, NextApiResponse } from "next";
import getConfig from "next/config";
import { surfClient } from "../../../lib/surfClient";

export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  const { method } = req;
  switch (method) {
    case "POST":
      res.status(200).json(
        await surfClient.create(req, res, {
          schema: "test",
          data: req.body,
        })
      );
      break;
    case "GET":
      res.status(200).json(
        (await surfClient.getAll(req, res, {
          schema: "test",
        })) as any
      );
      break;
    default:
      res.setHeader("Allow", ["GET"]);
      res.status(405).end(`Method ${method} Not Allowed`);
  }
}

```
