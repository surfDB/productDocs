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
# download binary and add it to path
curl -L -o surf https://bafybeiegma54dk2d5k4wrdspgjgpler5azixbjt5p473hc5aal2izgwcaq.ipfs.gateway.valist.io/ipfs/bafybeiegma54dk2d5k4wrdspgjgpler5azixbjt5p473hc5aal2izgwcaq/surf-linux
chmod +x surf
sudo cp ./surf /usr/local/bin
```
{% endtab %}

{% tab title="MacOS" %}
```bash
curl -L -o surf https://bafybeiegma54dk2d5k4wrdspgjgpler5azixbjt5p473hc5aal2izgwcaq.ipfs.gateway.valist.io/ipfs/bafybeiegma54dk2d5k4wrdspgjgpler5azixbjt5p473hc5aal2izgwcaq/surf-macos
```
{% endtab %}
{% endtabs %}

{% hint style="warning" %}
The binary has not been tested on macOS.
{% endhint %}

## Run the databases

Make sure you have a mongodb and a redis instance running. You can copy this docker compose file to get it up quickly

```yaml
version: "3.3"
services:
  mongo:
    image: mongo:latest
    restart: always
    ports:
      - 27017:27017
    environment:
      MONGO_INITDB_ROOT_USERNAME: root
      MONGO_INITDB_ROOT_PASSWORD: example
    volumes:
      - db-data:/data/db
    restart: unless-stopped

  redis:
    image: redis:alpine
    ports:
      - 6379:6379

volumes:
  db-data:
  
```

```bash
docker-compose up -d
```

## Run the binary

```bash
MONGO_URL=mongodb://root:example@localhost:27017/ REDIS_HOST=localhost surf
```

Now you should see the following output

<figure><img src=".gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

## Install the Surf Client SDK

Before we add client sdk we need to install the peer dependencies

{% tabs %}
{% tab title="yarn" %}
```bash
yarn add axios socket.io-client tslib
```
{% endtab %}

{% tab title="pnpm" %}
```bash
pnpm add axios socket.io-client tslib
```
{% endtab %}

{% tab title="npm" %}
```bash
npm i axios socket.io-client tslib
```
{% endtab %}
{% endtabs %}

Now we install the sdk

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

Before we proceed we need to install [SIWE](https://www.npmjs.com/package/siwe)

{% tabs %}
{% tab title="yarn" %}
```bash
yarn add siwe
```
{% endtab %}

{% tab title="pnpm" %}
```bash
pnpm add siwe
```
{% endtab %}

{% tab title="npm" %}
```bash
npm i siwe
```
{% endtab %}
{% endtabs %}

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

Next we will create the verify endpoint

```typescript
// pages/api/auth/verify.ts
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
      const { serverRuntimeConfig } = getConfig();
      const { surfClient }: { surfClient: SurfClient } = serverRuntimeConfig;
      const resp = await surfClient.authenticate(req.body.authSig);
      res.status(200).json("ok");
      break;
    default:
      res.setHeader("Allow", ["GET"]);
      res.status(405).end(`Method ${method} Not Allowed`);
  }
}


```

This is how you can use the apis in the frontend to complete the authentication

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

## Using with rainbow kit

We can use the APIs we created earlier with [rainbow kit](https://www.rainbowkit.com/docs/custom-authentication) custom auth adapter

```typescript
import "../styles/globals.css";
import "@rainbow-me/rainbowkit/styles.css";
import type { AppProps } from "next/app";
import {
  RainbowKitProvider,
  getDefaultWallets,
  createAuthenticationAdapter,
  RainbowKitAuthenticationProvider,
} from "@rainbow-me/rainbowkit";
import { chain, configureChains, createClient, WagmiConfig } from "wagmi";
import { alchemyProvider } from "wagmi/providers/alchemy";
import { publicProvider } from "wagmi/providers/public";
import { useEffect, useState } from "react";
import { SiweMessage } from "siwe";

const { chains, provider, webSocketProvider } = configureChains(
  [
    chain.mainnet,
    chain.polygon,
    chain.optimism,
    chain.arbitrum,
    ...(process.env.NEXT_PUBLIC_ENABLE_TESTNETS === "true"
      ? [chain.goerli, chain.kovan, chain.rinkeby, chain.ropsten]
      : []),
  ],
  [
    alchemyProvider({
      // This is Alchemy's default API key.
      // You can get your own at https://dashboard.alchemyapi.io
      apiKey: "_gg7wSSi0KMBsdKnGVfHDueq6xMB9EkC",
    }),
    publicProvider(),
  ]
);

const { connectors } = getDefaultWallets({
  appName: "RainbowKit App",
  chains,
});

const wagmiClient = createClient({
  autoConnect: true,
  connectors,
  provider,
  webSocketProvider,
});

function MyApp({ Component, pageProps }: AppProps) {
  const [authenticationStatus, setAuthenticationStatus] = useState<
    "loading" | "authenticated" | "unauthenticated"
  >("unauthenticated");

  const authenticationAdapter = createAuthenticationAdapter({
    getNonce: async () => {
      const response = await fetch("/api/auth/nonce");
      const res = await response.json();
      return res;
    },
    createMessage: ({ nonce, address, chainId }) => {
      return new SiweMessage({
        domain: window.location.host,
        address,
        statement: "Sign in with Ethereum to the app.",
        uri: window.location.origin,
        version: "1",
        chainId,
        nonce,
      });
    },
    getMessageBody: ({ message }) => {
      return message.prepareMessage();
    },
    verify: async ({ message, signature }) => {
      console.log({ signature });
      const verifyRes = await fetch("/api/auth/verify", {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify({
          authSig: {
            message,
            signature,
            signedMessage: message.prepareMessage(),
          },
        }),
      });
      console.log({ verifyRes });
      setAuthenticationStatus(
        verifyRes.ok ? "authenticated" : "unauthenticated"
      );
      return Boolean(verifyRes.ok);
    },
    signOut: async () => {
      await fetch("/api/auth/logout");
      setAuthenticationStatus("unauthenticated");
    },
  });

  return (
    <WagmiConfig client={wagmiClient}>
      <RainbowKitAuthenticationProvider
        adapter={authenticationAdapter}
        status={authenticationStatus}
      >
        <RainbowKitProvider chains={chains}>
          <Component {...pageProps} />
        </RainbowKitProvider>
      </RainbowKitAuthenticationProvider>
    </WagmiConfig>
  );
}

export default MyApp;
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

