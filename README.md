# On-Chain NFT Game

A real-time multiplayer NFT card flip game. Players flip cards to find rare NFTs — winners get real tokens transferred on-chain, and burns destroy tokens permanently.

Built live on stream with 1,000+ viewers on MegaETH.

## How It Works

- Players connect wallets and join rooms
- Each room has a grid of face-down cards
- Flip a card: find a **rare** (NFT transferred to you) or get **toasted** (burned on-chain)
- 30-second rounds with automatic player rotation
- Admin controls for starting, pausing, and managing rooms

## Architecture

```
Frontend (Next.js / React)
    ↕ WebSocket
Game Server (Node.js + Express + SQLite)
    ↕ ethers.js
EVM Chain (ERC-721 contract)
```

### Game Server
- WebSocket-based multiplayer (up to 10 players per room + lobby queue)
- SQLite (WAL mode) for crash-resilient state
- Serial transaction queue to prevent nonce collisions
- Dual RPC: separate read/write endpoints
- Room system: holder-only, whitelist, or public rooms
- 30-second round timer with automatic player rotation

### On-Chain
- **Prizes**: `transferFrom(signer, winner, tokenId)` — real NFT transfer
- **Burns**: `burn(tokenId)` — permanent on-chain destruction
- Contract owner can burn any token in the collection

### Frontend
- React page with real-time card grid and flip animations
- Live countdown timer, cursor tracking, notification feed
- Admin panel for room management
- 8-bit sound effects

## Setup

### Prerequisites
- Node.js 18+
- An EVM wallet with ERC-721 tokens
- The wallet must be the contract owner (for burns)

### Environment Variables

```bash
# Required
BREADIO_PRIVATE_KEY=your_signer_private_key
SIGNER_ADDRESS=0x...your_signer_wallet
TREASURY_ADDRESS=0x...wallet_holding_burn_tokens
CONTRACT_ADDRESS=0x...your_erc721_contract
ADMIN_WALLETS=0xadmin1,0xadmin2

# Optional
WHITELIST_WALLETS=0xwl1,0xwl2,0xwl3
WRITE_RPC=https://your-chain-rpc
READ_RPC=https://your-chain-rpc-for-reads
```

### Install & Run

```bash
npm install

# Rebuild room data (scans chain for token ownership via Transfer events)
node rebuild-rooms.js [prizesPerHolderRoom] [prizesPerPublicRoom] [realBurns] [cardsPerRoom]

# Start server
node server-v2.js

# Or with PM2
pm2 start server-v2.js --name nft-game
```

### Frontend

The game UI is in `frontend-page.tsx` — a self-contained React component. Drop it into any Next.js app:

```bash
# Set these in your Next.js .env.local
NEXT_PUBLIC_GAME_SERVER=wss://your-game-server.com
NEXT_PUBLIC_GAME_API=https://your-game-server.com
NEXT_PUBLIC_ADMIN_WALLETS=0xadmin1,0xadmin2
```

## Configuration

Edit `server-v2.js` to configure:
- `ROOM_CONFIG` — room names, holder requirements, cooldowns, max players
- `ROUND_DURATION` — seconds per round (default 30)

## Room Types

| Type | Access | Use Case |
|------|--------|----------|
| Holder | Must own an NFT from the collection | Reward holders |
| Whitelist | Specific wallet addresses | VIP / invite-only |
| Public | Anyone | Open to all |

## Admin

### Browser Panel
Admin wallets see an ADMIN button. Sign to authenticate, then start/pause/kick per room.

### CLI (from server)
```bash
# Start all rooms (30s timer)
curl -X POST http://localhost:3001/api/game/admin/startall

# Start single room
curl -X POST "http://localhost:3001/api/game/admin/start?room=breadio"

# Pause all
curl -X POST http://localhost:3001/api/game/admin/pauseall

# Check status
curl http://localhost:3001/api/game/admin/status
```

## Utility Scripts

| Script | Purpose |
|--------|---------|
| `rebuild-rooms.js` | Generate room data from on-chain token ownership |
| `send-prizes.js` | Manually send prizes to winners |
| `burn-tokens.js` | Bulk burn tokens from treasury |
| `verify-burns.js` | Verify burns confirmed on-chain |
| `deep-audit.js` | Cross-reference flips, players, and cards tables |
| `check-db.js` | Quick DB inspection |

## Tech Stack

- **Server**: Node.js, Express, WebSocket (ws), better-sqlite3
- **Chain**: ethers.js v6 (works with any EVM chain)
- **Frontend**: React / Next.js, TypeScript
- **Deploy**: PM2 + Caddy (server), Vercel (frontend)

## License

MIT

## Credits

Built by [@Tuteth_](https://x.com/Tuteth_)
