---
description: Database for Dapps
cover: .gitbook/assets/Color logo with background.png
coverY: -6.574642126789364
---

# Surf DB

## Introduction

SurfDB aims to be the database which dapps actually need.

Currently decentralized databases like [ceramic](https://ceramic.network/), [IPFS](https://web3.storage/), [arweave](https://bundlr.network/), [Filecoin](https://filecoin.io/) etc. cannot be used as a database for dapps. These databases are more meant to store arbitrary data like files or metadata.

The developers have to resort to using mongoDB or SQL as their databases which as we all know isn't really web3 native.

We wanted to make something which provides developers the ease of use of using web2 databases while also maintaining the decentralization aspect, without compromising the end user experience.

Using a decentralized database also allows the users to own their data instead of the apps owning the data which we see in a lot of dapps these days.

## Features

Surf DB has the following features:

* Decentralized database built on top of **ceramic** and cached using redis for **fast response times**
* Ability to have **contract level access control condition**, ex. We can have a condtion which allows only members of multi sig to update the row (in our case, the ceramic stream)
* Can be decentralized by **hosting mutliple nodes**: the data itself is stored on ceramic which is decentralized but the caching and the ceramic indexer itself also can be decentralized
* It supports **realtime updates**:get updated whenever there is a change in the database
* **Upload files**: it supports uploading files either to arweave or a dedicated s3 type object storage (coming soon)
* You can **eliminate data islands by forking an existing surfDB node**: if you want to create a frontend on someone else's data you can just **fork their node and then use your node to build out your frontend**, You don't have to start from 0, and the main data node owner can even monetize this. (coming soon)
* Instantly deploy it in just a few clicks (coming soon)
* Supports private data (coming soon)

## Let's jump right in?

Jump in to the quick start docs and get started:

{% content-ref url="quick-start/" %}
[quick-start](quick-start/)
{% endcontent-ref %}
