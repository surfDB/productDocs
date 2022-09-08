---
description: Get started with the surfDB
---

# Quick Start

{% hint style="warning" %}
**Caution:** We are still in very early stage so things can break and change. The docs will be updated frequently.
{% endhint %}

## Install the binary

We need to first install the surf server binary, currently only supported on linux and macos

{% tabs %}
{% tab title="Linux" %}
```bash
curl -L -o surf https://bafybeibohpwkbx4r57aauyh5unaubwvjhofpfitg7czsz3jck4js3hnbwe.ipfs.gateway.valist.io/ipfs/bafybeibohpwkbx4r57aauyh5unaubwvjhofpfitg7czsz3jck4js3hnbwe/surf-linux
chmod +x surf
sudo cp ./surf /usr/local/bin
```
{% endtab %}

{% tab title="MacOS" %}
```bash
curl -L -o surf https://bafybeibohpwkbx4r57aauyh5unaubwvjhofpfitg7czsz3jck4js3hnbwe.ipfs.gateway.valist.io/ipfs/bafybeibohpwkbx4r57aauyh5unaubwvjhofpfitg7czsz3jck4js3hnbwe/surf-macos
```
{% endtab %}
{% endtabs %}

{% hint style="warning" %}
The binary has not been tested on macOS.
{% endhint %}

## Run the databases

Make sure you have a postgres and a redis instance running. You can copy this docker compose file to get it up quickly

```yaml
version: "3.3"
services:
  postgres:
    image: postgres:latest
    hostname: postgres
    ports:
      - 5432:5432
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: postgres
      POSTGRES_DB: Todo
    volumes:
      - postgres-data:/var/lib/postgresql/data
    restart: unless-stopped

  redis:
    image: redis:alpine
    ports:
      - 6379:6379

volumes:
  postgres-data:

```

```bash
docker-compose up
```

## Run the binary

```bash
DATABASE_URL=postgresql://postgres:postgres@localhost:5432/Todo?schema=public REDIS_HOST=localhost surf
```

Now you should see the following output

<figure><img src=".gitbook/assets/image.png" alt=""><figcaption><p>server listening on port 3000</p></figcaption></figure>

## Install the Surf Client SDK

{% tabs %}
{% tab title="yarn" %}
```bash
yarn add @surfdb/client-sdk
```
{% endtab %}

{% tab title="pnpm" %}
```bash
pnpm add @surfdb/client-sdk
```
{% endtab %}

{% tab title="npm" %}
```bash
npm i @surfdb/client-sdk
```
{% endtab %}
{% endtabs %}

{% hint style="info" %}
We will be using next.js api to setup the surfclient, you can use the same in other backend frameworks as well
{% endhint %}

## Configure next.js

Add this in the next.config.js&#x20;

```javascript
/** @type {import('next').NextConfig} */
const { SurfClient, SurfRealtime } = require("@surfdb/client-sdk");

const surfClient = new SurfClient({
  client: "http://localhost:3000",
});

const nextConfig = {
  reactStrictMode: true,
  serverRuntimeConfig: {
    surfClient,
  },
};

module.exports = nextConfig;
```

This is done to ensure we have a single instance of the surfclient througout all the backend apis

## Authenticate

First we need to get the nonce

```typescript
// pages/api/auth/nonce.ts
import getConfig from "next/config";
import { NextApiRequest, NextApiResponse } from "next";
import { SurfClient } from "@surfdb/client-sdk";

export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  const { method } = req;
  switch (method) {
    case "GET":
      const { serverRuntimeConfig } = getConfig();
      const { surfClient }: { surfClient: SurfClient } = serverRuntimeConfig;
      const nonce = await surfClient.getAuthNonce();
      console.log({ nonce });
      return res.status(200).json(nonce);
    default:
      res.setHeader("Allow", ["GET"]);
      res.status(405).end(`Method ${method} Not Allowed`);
  }
}

```

Next we need to login

```typescript
// pages/api/auth/login.ts
import { SurfClient } from "@surfdb/client-sdk";
import { NextApiRequest, NextApiResponse } from "next";
import getConfig from "next/config";

export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  const { method } = req;
  switch (method) {
    case "POST":
      const { publicRuntimeConfig } = getConfig();
      const { surfClient }: { surfClient: SurfClient } = publicRuntimeConfig;
      const resp = await surfClient.authenticate(req.body.authsig);
      res.status(200).json("ok");
      break;
    default:
      res.setHeader("Allow", ["GET"]);
      res.status(405).end(`Method ${method} Not Allowed`);
  }
}

```

This is how you can use the apis in the frontend to complete the authentication

{% hint style="info" %}
We use [SIWE](https://www.npmjs.com/package/siwe) for authentication. You need to have that installed
{% endhint %}

```typescript
const res = await fetch("/api/auth/nonce", {
  credentials: "include",
});
const nonce = await res.json();
const message = new SiweMessage({
  domain: window.location.host,
  address: address,
  statement: "Sign in with Ethereum to the app.",
  uri: window.location.origin,
  version: "1",
  chainId: activeChain?.id,
  nonce,
});
const signature = await signMessageAsync({
  message: message.prepareMessage(),
});
const loginres = await fetch("/api/auth/login", {
  method: "POST",
  credentials: "include",
  body: JSON.stringify({
    authsig: {
      message,
      signature,
      signedMessage: message.prepareMessage(),
    },
  }),
});
```

## Create your first data

```javascript
await surfClient.create("schemaName", {
  name: "test",
  description: "this is just a description",
});
```

## Read it

```typescript
// get a particular data by id and schema name
await surfClient.get("schemaName", 1);

// get all rows in a schema
await surfClient.getAll("schemaName");
```

## Update it

```javascript
await surfClient.update("schemaName", 1, {
  name: "updated title",
  description: "this description is updated",
});
```

## Realtime

```javascript
const realtime = new SurfRealtime({
  client: "http://localhost:3000",
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

