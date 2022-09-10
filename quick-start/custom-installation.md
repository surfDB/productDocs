# Custom Installation

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
MONGO_URL=mongodb://root:example@localhost:27017/ REDIS_HOST=localhost SURF_PORT=300 surf
```

Now you should see the following output

<figure><img src="../.gitbook/assets/image (3).png" alt=""><figcaption></figcaption></figure>

## Install the Surf Client SDK

Before we add client sdk we need to install the peer dependencies

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
/* eslint-disable no-case-declarations */
// pages/api/auth/nonce.ts
import { NextApiRequest, NextApiResponse } from "next";
import { surfClient } from "../../../lib/surfClient";

export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  const { method } = req;
  switch (method) {
    case "GET":
      const nonce = await surfClient.getAuthNonce(req, res);
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
import { NextApiRequest, NextApiResponse } from "next";
import { surfClient } from "../../../lib/surfClient";

export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  const { method } = req;
  switch (method) {
    case "POST":
      const authRes = await surfClient.authenticate(req, res, {
        authSig: req.body.authSig,
      });
      res.status(200).json(authRes);
      break;
    default:
      res.setHeader("Allow", ["GET"]);
      res.status(405).end(`Method ${method} Not Allowed`);
  }
}

```

Logout

```typescript
/* eslint-disable no-case-declarations */
// pages/api/auth/logout.ts
import { NextApiRequest, NextApiResponse } from "next";
import { surfClient } from "../../../lib/surfClient";

export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  const { method } = req;
  switch (method) {
    case "POST":
      const resp = await surfClient.logout(req, res);
      return res.status(200).json(resp);
    default:
      res.setHeader("Allow", ["GET"]);
      res.status(405).end(`Method ${method} Not Allowed`);
  }
}
java
```

Me endpoint to check if you have logged in

```javascript
/* eslint-disable no-case-declarations */
// pages/api/auth/me.ts
import { NextApiRequest, NextApiResponse } from "next";
import { surfClient } from "../../../lib/surfClient";

export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  const { method } = req;
  switch (method) {
    case "GET":
      const profile = await surfClient.getProfile(req, res);
      return res.status(200).json(profile);
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
  darkTheme,
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
  >("loading");

  const [user, setUser] = useState(null);

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
      await fetch("/api/auth/logout", {
        method: "POST",
      });
      setAuthenticationStatus("unauthenticated");
    },
  });

  useEffect(() => {
    (async () => {
      const res = await fetch("/api/auth/me");
      const user = await res.json();
      console.log({ user });
      setAuthenticationStatus(
        user.address ? "authenticated" : "unauthenticated"
      );
    })();
  }, []);

  return (
    <WagmiConfig client={wagmiClient}>
      <RainbowKitAuthenticationProvider
        adapter={authenticationAdapter}
        status={authenticationStatus}
      >
        <RainbowKitProvider
          chains={chains}
          theme={darkTheme()}
          modalSize="compact"
        >
          <Component {...pageProps} />
        </RainbowKitProvider>
      </RainbowKitAuthenticationProvider>
    </WagmiConfig>
  );
}

export default MyApp;

```

##
