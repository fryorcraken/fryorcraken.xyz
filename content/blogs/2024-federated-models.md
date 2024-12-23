---
title: "Waku vs Federated Social Networks"
date: 2024-12-23T00:00:00+00:00
author: "fryorcraken"
tags:
  - waku
# image: 
# summary: "List of my talks at Devcon 7 and side events"
toc: false
---

Christine Lemmer-Webber wrote an interesting [article](https://dustycloud.org/blog/how-decentralized-is-bluesky/) about Bluesky and her vision of how true decentralization can be achieved for ActivityPub.

She proposed an interesting framework to model federated social networks by defining two categories: _message passing_ and _shared heap_. She also addressed the challenges of each model in terms of both scalability and message reliability in a decentralization context. Both topics are core properties that Waku [aims to solve](https://blog.waku.org/explanation-series-a-unified-stack-for-scalable-and-reliable-p2p-communication/).

Hence, I thought it would be an interesting exercise to see how Waku fits in this framework and how it attempts to solve the reliability and scalability challenges raised in the original post.

A first disclaimer is that Waku doesn't attempt to be a solution for public data social networks such as those mentioned in the article (Bluesky, Nostr, Mastodon, Scuttlebutt). Waku was designed to be the message routing layer for the Status chat application. Core differences in these use cases are twofold:

- While the data on such social networks are meant to be public, Waku focuses on enabling private communication
- Often, it is assumed that data in a social feed is permanent; Waku is a solution for ephemeral messages, with caching to support mostly offline devices.

Despite the difference in context, I believe the points made around reliability and scalability for both models make it relevant to Waku.

## Models Overview

Let's first review and paraphrase the definitions:

**Message passing** assumes a way to address message recipients and route messages towards them. _Towards_ meaning that some server may be the routing target of those messages, which are then delivered to the final recipient via client-server architecture. Email and Matrix are message passing systems. Usernames include a domain name (e.g., [john@gmail.com](mailto:john@gmail.com)) that is used for routing the message to the right server. The client then retrieves their messages by connecting to this server and requesting messages for themselves (e.g., John). In ActivityPub, it also works for outgoing messages, where @[alice@mastodon.social](mailto:alice@mastodon.social) will post her public messages in her outbox on @mastodon.social.

One of the [challenges](https://github.com/mastodon/mastodon/discussions/22608) in this model is that if Bob posts in his outbox at me.dm and Alice replies to it in her mastodon.social outbox, then there is no guarantee that David will see the reply unless he follows Alice.

Nostr and Bluesky fit in the **shared heap** model. Messages are posted in **relays**, and clients need to connect to those relays to get messages. As there is no addressing at the routing layer, it means that clients have to filter through all messages on the relay to display what the user is interested in. In Nostr, the clients select preferred relays where they post and need to learn their friends' preferred relays to see their content.

The [Nostr documentation](https://nostr.com/relays) recommends selecting a few relays when posting content and announcing the relays they use on all known relays (broadcast). Here is a great [animation](https://how-nostr-works.pages.dev/#/outbox) that explains how this model helps with censorship resistance (we'll talk about it later).

## Waku and the models.

We need to preface that in all examples above, the routing and application protocols are well intertwined. Waku purposefully separates application and routing layers to enable stronger privacy properties.

A Waku message contains:

- A payload: binary data encoded by the application
- A content topic: a string set by the application for content addressing purposes
- Other data such as timestamp and proof for rate limiting

Waku messages are sent on a gossipsub network. It does follow a pub/sub model. However, the topics of subscriptions (pubsub topics) in Waku are not dynamic or recipient-related. Instead, Waku node operators agree on a number of shards to support (e.g., 8), each shard having a unique pubsub topic.

So one could say that Waku is more of a shared heap, where all the messages are sent and aggregated on a pubsub topic. Instead of users or applications selecting a list of relays to push their messages to, they select a shard.

Note that while developers are free to select a shard, protocols such as auto-sharding remove the burden from developers who only need to worry about setting a content topic.

Similar to connecting to a Nostr relay, users do have to filter through the messages to find the ones relevant to them. This is done via content topics.

## Message reliability

In the case of **shared heap**, message reliability issues stem from the fact that if a client does not retrieve data from the right relay, or server instance, messages from specific publishers may be missing.

The original article mentioned how Bluesky uses centralization to solve this, ensuring that users do not have to look for messages on several relays or ActivityPub servers. In the case of Waku, application reliability is handled by specific data sync protocols. These protocols are enabled by the fact that messages are published in groups with **active participants**. This is a fundamental difference from the workings of public social networks, where messages (tweets, posts, toots) are generally published on feed or timeline. Because chat groups have active participants, it is possible to have heuristics in place to understand whether said recipients received a message. This can be based on [acknowledgements](https://rfc.vac.dev/vac/2/mvds), [causal history, or bloom filters](https://rfc.vac.dev/vac/raw/sds/). If it seems that no one has received a message, it can be rebroadcast.

Having said that, I can see a way where the feed of one poster, Alice, could be seen as a large chat group, where her posts and the comments from her followers would be treated as the messages in this chat room. A protocol such as Scalable Data Sync ([SDS](https://rfc.vac.dev/vac/raw/sds/)) might help her and her followers understand if any original post or reply is missing.
Take this with a pinch of salt, as we would need to properly review such a use case (but you could [try it yourself](https://github.com/waku-org/nim-sds).

## Scalability

The second property to address is scalability. In the case of Bluesky with centralized relay, it was noted that it currently consumes at least 5TB of storage.

In the case of Nostr, each relay stores the messages they want via deciding their retention but, more importantly, who can post on the relay. It also opens an incentivization route where users can pay to have their messages stored on a relay.

The path to scalability seems fairly straightforward. A given relay can decide to stop accepting new users or modify post retention based on fees. As demand increases, new relay can pop-up to accommodate new users.

For Waku, we need to remember that it aims to solve ephemeral message transmission. While indeed store nodes cache messages for the network, they are only here to accommodate offline devices.

Such a use case should apply fewer constraints on message retention and longevity, meaning Waku is not designed to store all messages forever. Decentralized storage with strong durability guarantees, such as [Codex](https://codex.storage/), can be used to retain messages that matter.

While the data may not need to be kept by Waku, there are still scalability constraints as Waku onboards thousands or millions of users. This is solved in several ways.

### Rate limit

Rate and size limits are applied to message publishers. This allows capping of the bandwidth and storage being used.

### Sharding

The traffic is sharded across different pubsub topics in a deterministic manner.

This takes from both the **message passing** and **shared heap** models. Similar to the shared heap model, it means that not all "relays" (here pubsub topics) have all the messages. However, it is done in a way that the application knows which shard to subscribe to to receive specific messages (similar to a Mastodon outbox). This means that messages do not need to be duplicated over shards, contrary to Nostr relays.

### Filter protocol

The _filter protocol_ enables server-side filtering of messages in a shard by content topic. While it does not reduce the data transmitted on a shard, it enables nodes with fewer resources (browser, mobile) to only process a subset of the data. This could be seen as similar to Bluesky's AppView component. However, it is implemented in a decentralized manner as any service node in the network can provide filter service.

## Censorship-resistance

Decentralization is a tool to an end. In the case of Waku and Nostr, censorship resistance is part of the core properties the protocols care about.

The Nostr animation [shared earlier](https://how-nostr-works.pages.dev/#/outbox) explains how it can resist censorship by countering whack-a-mole takedowns. BitTorrent trackers have demonstrated such a technique works. However, it is not efficient due to the cost of setting up a BitTorrent relay or Nostr tracker. It is an added effort that often requires basic technical skills.

Waku's approach to censorship resistance is similar to the BitTorrent DHT: a decentralized protocol that all clients participate in, moving away from a client-server model to a peer-to-peer one.

Waku nodes running in your desktop app fully participate in Waku and can serve less resource-able nodes, such as mobile and browsers ([we call them edge nodes](https://blog.waku.org/explanation-series-light-protocols-and-edge-nodes/) as they are at the edge of the network). This means that the barrier to support the network is lowered. One does not need to deploy a BitTorrent tracker or Nostr relay; they simply need to run the app on their laptop.

## Privacy

An obvious downside to the message passing model is the loss of privacy. Indeed, for a message to be routed to the right recipient, it means the routing layer has a way to identify users. Waku separates routing and application layers on purpose. If someone can be identified at the routing layer, they can then easily be identified in the physical world.

In the case of Nostr, Bluesky, and Mastodon, [unless](https://github.com/0xtrr/onion-service-nostr-relays) [Tor](https://docs.joinmastodon.org/admin/optional/tor/) is used, posters reveal their IP to the relays they use.

The same criticism can be made about Waku, which only applies weak anonymity on the routing layer (libp2p-gossipsub with `StrictNoSign` policy), at least until mixnet becomes an [integral part of the protocol](https://forum.vac.dev/t/introducing-the-mix-protocol-enhancing-privacy-across-libp2p-networks/348/8).

## Conclusion

The proposed model is a fair framework to describe today's federated systems. The Nostr protocol pushes social networks one step further towards decentralization than its federated cousin Mastodon. However, I am not convinced that a non-peer-to-peer system can give us enough, in all scenarios that matter.

While Waku's promises are not yet fully delivered, especially in terms of incentivization and routing anonymity, its peer-to-peer properties lay a more robust foundation to build private and censorship-resistant applications such as [Status](https://status.app/) and [RAILGUN](https://railgun.org/).



