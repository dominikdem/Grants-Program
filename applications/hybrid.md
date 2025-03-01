# Hybrid Block Explorer

- **Team Name:** Jonathan Brown
- **Payment Address:** 0x36a7401F269555916a0243E586234D3Bbf5A0c36 (DAI)
- **[Level](https://github.com/w3f/Grants-Program/tree/master#level_slider-levels):** 1


## Project Overview :page_facing_up:

This application is in response to two RFPs:
- Multi-chain Block Explorer https://github.com/w3f/Grants-Program/blob/master/docs/RFPs/Under%20Development/multi-chain-block-explorer.md
- ink! smart contract block explorer. https://github.com/w3f/Grants-Program/blob/master/docs/RFPs/Under%20Development/ink_smart_contract_block_explorer.md


### Overview

Hybrid takes a unique, partially decentralized approach that improves two major problems with current open source Substrate block explorers: centralization and huge hosting requirements.

A fully centralized block explorer typically populates an SQL database with the entirety of an archive node and stores additional data to index everything. Operating such a database reliably requires huge system resources and expense.

When querying block information, or the chain state at any block height, the Hybrid dapp will use the [Substrate Connect](https://substrate.io/developers/substrate-connect/) light client from within the browser. Alternatively, these queries can be made directly to an archive node via WSS.

For event search functionality, the Hybrid indexer efficiently indexes events in all blocks so they can be found with a simple WSS query. For example, to find all events connected with a specific `AccountId`.

This architecture has three main advantages:
- state queries are fully decentralized - you don't have to trust an RPC provider not to lie to you
- 100% availability - the light client doesn't depend on any centralized service that may not always be available
- the Hybrid indexer has significantly lower system requirements - it doesn't need to store all chain history

Eventually, Hybrid will use this centralized / decentralized approach as the basis for an ink! contract explorer.

Because Substrate is a federated platform, it will be possible browse multiple chains from the Hybrid dapp.

### Project Details

Hybrid will be a Substrate block explorer dapp. By default it will connect to major Substrate blockchains. Additionally it can be configured to connect to any Substrate chain.

There are two types of queries that the explorer can perform:
- Decentralized - ideally using the Substrate Connect light client, or alternatively connecting to an archival node via WSS. For example, querying blocks or state storage.
- Centralized - the Hybrid indexer for the chain will be queried via WSS. This is for event searching and watching, such as finding all events across all chains that relate to a specific `AccountId`.

![Hybrid Architecture](https://raw.githubusercontent.com/ethernomad/hybrid-diagram/main/hybrid.png)

#### Indexer

The Hybrid indexer will be written in Rust. It can be configured to connect to any Substrate chain.

It will read events in all blocks using [subxt](https://github.com/paritytech/subxt) and index these events in a Key-value database using the [sled](http://sled.rs/) library. This is considerably more efficient than storing the index in an SQL database.

All events in all pallets that have identifying parameters will be indexed. For example the Transfer event in the Balances pallet is identifiable by the `AccountId` of both `from` and `to`.

Other examples of identifying event parameters are `assetId` in the Assets pallet, `code_hash` in the contracts pallet, `CollectionId` and `ItemId` in the NFTs pallet, and `MultiLocation` in the XCM pallet.

Reading blocks and writing indexes will occur in separate threads with a queue from the read thread to the write thread. Enabling reading and writing simultaneously will maximize indexing throughput.

WSS queries will be handled via the highly scalable [tokio_tungstenite](https://github.com/snapview/tokio-tungstenite) Rust library.

In addition to the identifier being searched for, queries will be able to include start block, offset, and limit to control which events are returned.

Consumers will be able to subscribe for new events that match a query. This will be handles by sled's powerful [watch_prefix](https://docs.rs/sled/latest/sled/struct.Tree.html#method.watch_prefix) functionality.

The database keys will be constructed in such a way so that events can be found using iterators starting at a specific block number. For example, for for the AccountId keyspace:

`AccountId/BlockNumber/EventIndex`


#### Dapp

The Hybrid dapp will be a [Vue](https://vuejs.org/) dapp, using the [Vuetify](https://vuetifyjs.com) framework for the user interface. [pnpm](https://pnpm.io/) and [Vite](https://vitejs.dev/) will be used for the build.

It will use [@polkadot/api](https://github.com/polkadot-js/api) to retrieve data from the chain, either via the Substrate Connect light client, or via an RPC connection to a full archival node. The Hybrid indexer will be queried via WSS.

This grant will only include a basic dapp including:
- block browsing, showing transactions and events
- event searching, e.g. searching for all events connected to a specific `AccountId`.

A subsequent grant application will be made to develop a richer dapp experience.


### Ecosystem Fit

A major issue holding back the Substrate and Polkadot ecosystem is a high quality block explorer comparable to Etherscan. We need such a block explorer to be:
- non-proprietary
- as decentralized as possible
- not requiring massive system resources
- feature-rich

The target audience is blockchain enthusiasts and developers. Eventually end-users should not need to know about block explorers, but this depends on dapps improving their user experience.

The indexing component has value far beyond the Hybrid block explorer. Many Substrate applications will find a centralized event indexer extremely useful.

#### Comparison to other Substrate explorers / indexers

- [DotScan](https://dotscan.com/) - proprietary
- [Blockchair](https://blockchair.com/polkadot) - proprietary
- [Subscan](https://www.subscan.io/) - proprietary. Open source version is very basic with fully centralized database
- [Polkadot Apps](https://polkadot.js.org/apps/) - open source and fully decentralized, but non-indexing
- [Polkascan](https://explorer.polkascan.io/) - fully centralized database.
- [Blockcoders ink! Explorer](https://ink-explorer.blockcoders.io/) - fully centralized database, focused on contracts
- [Sirato ink! Explorer](https://github.com/web3labs/epirus-substrate) (previously Epirus) - fully centralized database, focused on contracts
- [SubSquid](https://www.subsquid.io/) - fully centralized database
- [SubQuery](https://subquery.network/) - fully centralized database
- [Substrate Archive](https://github.com/paritytech/substrate-archive) - fully centralized database, unmaintained

There are no known block explorers in other ecosystems that are indexing, and either fully decentralized or have a hybrid design like this one.


## Team :busts_in_silhouette:

### Team members

- Jonathan Brown

### Contact

- **Contact Name:** Jonathan Brown
- **Contact Email:** jbrown@acuity.social
- **Website:** https://acuity.social


### Legal Structure

- **Registered Address:** n/a
- **Registered Legal Entity:** n/a


### Team's experience

Jonathan Brown has extensive relevant experience to build this software.

He built a proof-of-concept centralized indexer for a cross-chain DEX that has a very similar architecture to the Hybrid indexer. It is written in Rust, indexes data from Substrate and Ethereum blockchains, writes it to RocksDB, and enables WSS queries via tokio_tungstenite.

https://github.com/acuity-social/acuity-atomic-swap-offchain

He is also very experienced building dapps with polakdot.js such as the [Acuity DEX](https://github.com/acuity-social/acuity-dex-app).

Jonathan has also made some videos about Substrate development: https://www.youtube.com/watch?v=FMr2bNSmnfY


### Team Code Repos

- https://github.com/ethernomad
- https://github.com/acuity-social

### Team LinkedIn Profiles (if available)

- n/a


## Development Status :open_book:

Development has not started on the project, however the codebase will largely follow that of the [Acuity DEX offchain indexer](https://github.com/acuity-social/acuity-atomic-swap-offchain).

## Development Roadmap :nut_and_bolt:

### Overview

- **Total Estimated Duration:** 8 weeks
- **Full-Time Equivalent (FTE):**  1
- **Total Costs:** 10,000 USD

### Milestone 1 — Event Indexing component

- **Estimated duration:** 4 weeks
- **FTE:**  1
- **Costs:** 5,000 USD

| Number | Deliverable | Specification |
| -----: | ----------- | ------------- |
| **0a.** | License | Apache 2.0 |
| **0b.** | Documentation | We will provide both **inline documentation** of the code and a basic **tutorial** that explains how a user can index a Substrate node and query events. |
| **0c.** | Testing and Testing Guide | Core functions will be fully covered by comprehensive unit tests to ensure functionality and robustness. In the guide, we will describe how to run these tests. |
| **0d.** | Docker | We will provide a Dockerfile(s) that can be used to test all the functionality delivered with this milestone. |
| 1. | Connect to Substrate chains | The indexer will be written in Rust and configurable to connect to any Substrate chain using the subxt library. |
| 2. | Block syncing | As new blocks are produced, the indexer reads all events. Additionally, it will read events from existing blocks, starting at block 0. |
| 3. | Index writing | All identifying parameters in events will be indexed in the database using the sled library. |
| 4. | Status querying | It will be possible to query the current status of the indexer via WSS. This will include information about which chain is being indexed, indexing progress and last know block. Queries will be handled via  tokio_tungstenite. |
| 5. | Index querying | It will be possible to search via WSS for events with an identifier. |
| 6. | Dapp | A rudimentary web interface will be developed to expose this functionality. This will be built using pnpm, vite, vue, vuetify & polkdadot.js .|

### Milestone 2 — Event Subscribing

- **Estimated duration:** 2 weeks
- **FTE:**  1
- **Costs:** 2,500 USD

| Number | Deliverable | Specification |
| -----: | ----------- | ------------- |
| **0a.** | License | Apache 2.0 |
| **0b.** | Documentation | We will provide both **inline documentation** of the code and a basic **tutorial** that explains how a user can witness live updates to event search results. |
| **0c.** | Testing and Testing Guide | Core functions will be fully covered by comprehensive unit tests to ensure functionality and robustness. In the guide, we will describe how to run these tests. |
| **0d.** | Docker | We will provide a Dockerfile(s) that can be used to test all the functionality delivered with this milestone. |
| 1. | Event subscription API | The indexer will be updated to service subscription requests via WSS. |
| 2. | Live dapp | The dapp will be updated so that pages displaying results from event queries will be updated as soon as a new event appears on the chain. |


### Milestone 3 — Decentralized Component

- **Estimated Duration:** 2 weeks
- **FTE:**  1
- **Costs:** 2,500 USD

| Number | Deliverable | Specification |
| -----: | ----------- | ------------- |
| **0a.** | License | Apache 2.0 |
| **0b.** | Documentation | We will provide both **inline documentation** of the code and a basic **tutorial** that explains how a user can use the rudimentary explorer dapp. |
| **0c.** | Testing and Testing Guide | Core functions will be fully covered by comprehensive unit tests to ensure functionality and robustness. In the guide, we will describe how to run these tests. |
| **0d.** | Docker | We will provide a Dockerfile(s) that can be used to test all the functionality delivered with this milestone. |
| 0e. | Video | We will publish a video that explains and demonstrates all aspects of the explorer. |
| 1. | Dapp | The dapp will be extended to query block and state via either the Substrate Connect light client, or via direct connect to a full node via WSS. |
| 2. | Cross-chain UI | When a Tx includes a XCM, it will be easy and intuitive to open the relevant block from the other chain(s). |


## Future Plans

- hosting - The project needs to host indexes for all major Substrate chains. The frontend can be hosted as a traditional website and on IPFS.

- improve dapp - explore how the event index can be used to better display the richness of the Polkadot ecosystem.

- support for tokens and nfts

- add support for ink! smart contracts with decentralized source code publishing

- maintain Hybrid as the most decentralized Substrate block explorer

- marketing

## Additional Information :heavy_plus_sign:

**How did you hear about the Grants Program?** Twitter

* What work has been done so far? Development has not started yet.

* Are there are any teams who have already contributed (financially) to the project?
No
* Have you applied for other grants so far?
No
