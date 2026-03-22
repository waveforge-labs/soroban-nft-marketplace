# soroban-nft-marketplace


> A fully on-chain NFT marketplace for Stellar Soroban — mint, list,
> auction, and collect NFTs via Rust smart contracts with a polished
> Next.js storefront and a real-time activity indexer.

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Built with Soroban](https://img.shields.io/badge/Soroban-Rust-orange)](https://soroban.stellar.org)
[![Drips Wave](https://img.shields.io/badge/Drips-Wave%20Program-blue)](https://drips.network)

---

## Overview

`soroban-nft-marketplace` is a fully modular, open-source NFT infrastructure
stack for Stellar — designed so any team can fork it and launch their own
NFT collection or marketplace in days:

- 🎨 **NFT Contract** — mint, transfer, burn, and store on-chain metadata
  URI per token ID; supports per-collection royalty configuration
- 🏪 **Marketplace Contract** — list tokens at fixed price; escrows funds
  trustlessly; distributes royalties to creators on sale
- 🔨 **Auction Contract** — English auctions with reserve price, minimum
  bid increment, and automatic settlement at deadline
- 🗂️ **Activity Indexer** — real-time ownership and listing state for fast
  frontend rendering without slow on-chain lookups
- 🖥️ **Next.js Storefront** — explore collections, view token details,
  place bids, manage profile listings

---

## Technical Architecture

### Contract Architecture
```
NFTContract (per collection)
├── mint(to, token_id, metadata_uri)
├── transfer(from, to, token_id)
├── burn(token_id)
├── owner_of(token_id) → Address
├── metadata_uri(token_id) → String
└── royalty_info() → { recipient, bps }

MarketplaceContract
├── list(token_id, price, currency)
├── buy(listing_id)              ← escrow + royalty split
├── cancel_listing(listing_id)
└── get_listing(listing_id) → Listing

AuctionContract
├── create_auction(token_id, reserve, increment, deadline)
├── place_bid(auction_id, amount)
├── settle(auction_id)           ← auto-triggers at deadline
└── get_auction(auction_id) → Auction
```

### Frontend Pages

| Route | Content |
|---|---|
| `/explore` | All collections, filter by floor price / volume |
| `/collection/[id]` | Grid of NFTs, stats, activity feed |
| `/token/[collection]/[id]` | Token detail: owner, bids, history |
| `/profile/[address]` | Owned NFTs, active listings, bid history |

### Indexer Event Handlers
```ts
// Handles Soroban events emitted by the contracts
onEvent("nft:Transfer", updateOwnership);
onEvent("marketplace:Listed", addListing);
onEvent("marketplace:Sold", settleListing);
onEvent("auction:BidPlaced", updateAuction);
onEvent("auction:Settled", closeAuction);
```

---

## 🌊 Drips Wave Program

This repository is an **active participant in the
[Drips Wave Program](https://drips.network)** — contributors earn
USDC/DAI rewards for solving clearly scoped GitHub issues.

### How to Participate

1. **Browse Open Issues**
   Visit the [Issues tab](../../issues) and filter by label:
   - 🟢 `drips:trivial` — Fix NFT card loading skeleton, improve
     auction timer display, add ARIA labels to modals (~1–2 hrs)
   - 🟡 `drips:medium` — Implement royalty distribution logic in
     Marketplace contract, add collection stats to indexer,
     build `<ActivityFeed />` component (~4–8 hrs)
   - 🔴 `drips:high` — Full auction contract with settlement,
     IPFS metadata pinning service, collection deploy wizard
     (~1–3 days)

2. **Register on Drips**
   - Visit [drips.network](https://drips.network)
   - Connect your Ethereum-compatible wallet
   - Search for **`soroban-nft-marketplace`** under "Explore"

3. **Claim an Issue**
   - Comment: `"Claiming via Drips Wave — ETA [X days]"`
   - Maintainer assigns you and registers the bounty on Drips

4. **Submit Your PR**
   - Contract changes require passing `cargo test` + a test case
     covering the new logic
   - Frontend changes require no TypeScript errors and a screenshot
   - Indexer changes must include a migration file

5. **Get Paid**
   Bounty amounts (USDC/DAI) listed per issue. Released on merge.

---

## Project Structure
```
soroban-nft-marketplace/
├── contracts/
│   ├── nft/                  # NFT ownership + metadata contract
│   ├── marketplace/          # Fixed-price listing + escrow contract
│   ├── auction/              # English auction contract
│   └── tests/                # Rust unit + integration tests
├── frontend/
│   ├── src/
│   │   ├── app/
│   │   │   ├── explore/      # Collection browser
│   │   │   ├── collection/   # Per-collection NFT grid
│   │   │   ├── token/        # Token detail + bid/buy UI
│   │   │   └── profile/      # User portfolio + listings
│   │   ├── components/
│   │   │   ├── NFTCard/      # Reusable NFT thumbnail card
│   │   │   ├── ListingModal/ # Fixed-price listing flow
│   │   │   ├── AuctionTimer/ # Countdown + current bid display
│   │   │   ├── BidPanel/     # Place / increase / cancel bid
│   │   │   ├── CollectionHeader/ # Banner, stats, description
│   │   │   └── ActivityFeed/ # Real-time sales + transfer events
│   │   ├── hooks/            # useNFT, useListing, useAuction
│   │   └── lib/              # Contract clients, IPFS loader
│   └── public/
├── indexer/
│   ├── src/
│   │   ├── handlers/         # Per-event state updaters
│   │   ├── models/           # NFT, Listing, Auction, Bid DB models
│   │   └── utils/            # Event parsing, logging
│   └── migrations/           # PostgreSQL schema migrations
├── tests/
│   ├── unit/                 # Frontend hooks + indexer handlers
│   ├── integration/          # Testnet contract interaction tests
│   └── e2e/                  # Full buy/auction flow tests
├── scripts/                  # Deploy, mint sample collection
├── docs/                     # Contract spec, indexer data model
├── docker-compose.yml
└── .env.example
```

---

## Quick Start
```bash
# 1. Deploy contracts
cd contracts
cargo build --target wasm32-unknown-unknown --release
cd ../scripts && npx ts-node deploy.ts --network testnet

# 2. Start indexer
cd ../indexer && npm install && npm run start

# 3. Start storefront
cd ../frontend && npm install && npm run dev
# Visit http://localhost:3000/explore
```

---

## License

MIT © waveforge-labs contributors
