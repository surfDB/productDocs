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
curl -L -o surf https://bafybeiglsoezey24a6eu6jbkvs4qdcjcz6asq7ds224c6dukhiigyvtdo4.ipfs.gateway.valist.io/ipfs/bafybeiglsoezey24a6eu6jbkvs4qdcjcz6asq7ds224c6dukhiigyvtdo4/surf-linux
chmod +x surf
sudo cp ./surf /usr/local/bin
```
{% endtab %}

{% tab title="MacOS" %}
```bash
curl -L -o surf https://bafybeiglsoezey24a6eu6jbkvs4qdcjcz6asq7ds224c6dukhiigyvtdo4.ipfs.gateway.valist.io/ipfs/bafybeiglsoezey24a6eu6jbkvs4qdcjcz6asq7ds224c6dukhiigyvtdo4/surf-linux
```
{% endtab %}
{% endtabs %}

{% hint style="warning" %}
The binary has not been tested on macOS.
{% endhint %}

## Nextjs & Rainbow-kit starter app

You can scaffold a new [next.js](https://nextjs.org/) + [Rainbow Kit](https://www.rainbowkit.com/docs/installation) + [wagmi](https://wagmi.sh/) + [surfDB](./#install-the-binary) using just a single command

```bash
npx create-next-surf-app@latest
```

Then cd into the app folder and get the mongo and redis database up&#x20;

```bash
docker-compose up -d
```

Run the binary

```bash
surf
# you can provide the following env variables to change the default values
# MONGO_URL="mongodb://root:example@localhost:27017/"
# REDIS_HOST="localhost"
# SURF_PORT="4200"
```

You should see the following output

<figure><img src="../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

Now we can run the app&#x20;

```bash
yarn dev
```

<figure><img src="../.gitbook/assets/image (2).png" alt=""><figcaption><p>Homepage</p></figcaption></figure>
