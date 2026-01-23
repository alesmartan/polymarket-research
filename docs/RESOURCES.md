# Resources

A curated collection of external resources, tools, documentation, and learning materials for building prediction market platforms.

---

## Table of Contents

1. [Polymarket Official Resources](#polymarket-official-resources)
2. [Competitor Resources](#competitor-resources)
3. [Blockchain Documentation](#blockchain-documentation)
4. [Smart Contract References](#smart-contract-references)
5. [Development Tools](#development-tools)
6. [Design Inspiration](#design-inspiration)
7. [Infrastructure & Services](#infrastructure--services)
8. [Communities & Forums](#communities--forums)
9. [Learning Resources](#learning-resources)
10. [News & Research](#news--research)

---

## Polymarket Official Resources

### Documentation & Guides

| Resource | URL | Description |
|----------|-----|-------------|
| **Polymarket Docs** | https://docs.polymarket.com | Official documentation including API, CLOB, and trading guides |
| **Learn Polymarket** | https://learn.polymarket.com | Educational content for new users |
| **Polymarket FAQ** | https://polymarket.com/faq | Frequently asked questions |

### Technical Resources

| Resource | URL | Description |
|----------|-----|-------------|
| **Polymarket GitHub** | https://github.com/Polymarket | Official repositories including contracts and SDKs |
| **CLOB Client SDK** | https://github.com/Polymarket/py-clob-client | Python SDK for interacting with Polymarket's order book |
| **Order Book Contracts** | https://github.com/Polymarket/ctf-exchange | Smart contracts for the CTF Exchange |

### APIs & Data

| Resource | URL | Description |
|----------|-----|-------------|
| **CLOB API** | https://docs.polymarket.com/#clob-api | Central Limit Order Book API documentation |
| **Gamma API** | https://docs.polymarket.com/#gamma-api | Market data and analytics API |
| **REST Endpoints** | https://clob.polymarket.com | Base URL for REST API |
| **WebSocket** | wss://ws-subscriptions-clob.polymarket.com | Real-time data feed |

### Contract Addresses (Polygon Mainnet)

| Contract | Address | Description |
|----------|---------|-------------|
| CTF Exchange | `0x4bFb41d5B3570DeFd03C39a9A4D8dE6Bd8B8982E` | Main exchange contract |
| Conditional Tokens | `0x4D97DCd97eC945f40cF65F87097ACe5EA0476045` | ERC-1155 outcome tokens |
| USDC | `0x2791Bca1f2de4661ED88A30C99A7a9449Aa84174` | Collateral token |
| NegRisk Exchange | `0xC5d563A36AE78145C45a50134d48A1215220f80a` | Negated risk exchange |

---

## Competitor Resources

### Kalshi (CFTC-Regulated)

| Resource | URL | Description |
|----------|-----|-------------|
| **Website** | https://kalshi.com | US-regulated prediction market |
| **API Docs** | https://trading-api.readme.io | Trading API documentation |
| **Blog** | https://kalshi.com/blog | Market analysis and updates |

### Metaculus

| Resource | URL | Description |
|----------|-----|-------------|
| **Website** | https://www.metaculus.com | Forecasting platform (reputation-based, not financial) |
| **API** | https://www.metaculus.com/api2 | Public API for questions and predictions |
| **Track Record** | https://www.metaculus.com/questions/track-record | Historical accuracy data |

### Augur

| Resource | URL | Description |
|----------|-----|-------------|
| **Website** | https://augur.net | Decentralized prediction market protocol |
| **GitHub** | https://github.com/AugurProject | Open source contracts and clients |
| **Docs** | https://docs.augur.net | Technical documentation |

### Gnosis / Omen

| Resource | URL | Description |
|----------|-----|-------------|
| **Gnosis Protocol** | https://docs.gnosis.io | Protocol documentation |
| **Conditional Tokens Docs** | https://docs.gnosis.io/conditionaltokens | CTF documentation (foundation for Polymarket) |
| **GitHub** | https://github.com/gnosis | Open source repositories |

### PredictIt

| Resource | URL | Description |
|----------|-----|-------------|
| **Website** | https://www.predictit.org | Academic research prediction market |
| **API** | https://www.predictit.org/api | Public market data API |

### Manifold Markets

| Resource | URL | Description |
|----------|-----|-------------|
| **Website** | https://manifold.markets | Play-money prediction market |
| **GitHub** | https://github.com/manifoldmarkets | Open source codebase |
| **API** | https://docs.manifold.markets | API documentation |

---

## Blockchain Documentation

### Polygon (Primary Network)

| Resource | URL | Description |
|----------|-----|-------------|
| **Polygon Docs** | https://docs.polygon.technology | Official documentation |
| **PolygonScan** | https://polygonscan.com | Block explorer |
| **Mumbai Testnet** | https://mumbai.polygonscan.com | Testnet explorer |
| **Faucet** | https://faucet.polygon.technology | Test MATIC for Mumbai |
| **Bridge** | https://portal.polygon.technology | Official Polygon bridge |

### Ethereum (L1)

| Resource | URL | Description |
|----------|-----|-------------|
| **Ethereum Docs** | https://ethereum.org/developers | Official developer documentation |
| **Etherscan** | https://etherscan.io | Ethereum block explorer |
| **EIPs** | https://eips.ethereum.org | Ethereum Improvement Proposals |

### Alternative L2s

| Network | Documentation | Explorer |
|---------|---------------|----------|
| **Arbitrum** | https://docs.arbitrum.io | https://arbiscan.io |
| **Base** | https://docs.base.org | https://basescan.org |
| **Optimism** | https://docs.optimism.io | https://optimistic.etherscan.io |

### Solana (Alternative Chain)

| Resource | URL | Description |
|----------|-----|-------------|
| **Solana Docs** | https://docs.solana.com | Official documentation |
| **Solscan** | https://solscan.io | Block explorer |
| **Anchor** | https://www.anchor-lang.com | Smart contract framework |

---

## Smart Contract References

### Token Standards

| Standard | EIP | Description | Use Case |
|----------|-----|-------------|----------|
| **ERC-20** | https://eips.ethereum.org/EIPS/eip-20 | Fungible token standard | USDC, utility tokens |
| **ERC-1155** | https://eips.ethereum.org/EIPS/eip-1155 | Multi-token standard | Conditional tokens |
| **ERC-721** | https://eips.ethereum.org/EIPS/eip-721 | NFT standard | Unique assets |

### Oracle Systems

| Oracle | Documentation | Description |
|--------|---------------|-------------|
| **UMA** | https://docs.uma.xyz | Optimistic oracle used by Polymarket |
| **Chainlink** | https://docs.chain.link | Industry-leading decentralized oracle |
| **API3** | https://docs.api3.org | First-party oracle network |

### Reference Implementations

| Repository | URL | Description |
|------------|-----|-------------|
| **Gnosis Conditional Tokens** | https://github.com/gnosis/conditional-tokens-contracts | Original CTF implementation |
| **Gnosis CTF Market Makers** | https://github.com/gnosis/conditional-tokens-market-makers | AMM for conditional tokens |
| **OpenZeppelin** | https://github.com/OpenZeppelin/openzeppelin-contracts | Audited contract library |
| **Solmate** | https://github.com/transmissions11/solmate | Gas-optimized contract library |

### Security Audits

| Firm | Website | Description |
|------|---------|-------------|
| **Trail of Bits** | https://www.trailofbits.com | Top-tier security firm |
| **OpenZeppelin** | https://www.openzeppelin.com/security-audits | Audits and security services |
| **Consensys Diligence** | https://consensys.io/diligence | Smart contract audits |
| **Certik** | https://www.certik.com | Security audits and monitoring |
| **Quantstamp** | https://quantstamp.com | Blockchain security |

---

## Development Tools

### Smart Contract Development

| Tool | URL | Description |
|------|-----|-------------|
| **Foundry** | https://book.getfoundry.sh | Modern Solidity development toolkit |
| **Hardhat** | https://hardhat.org | Popular development environment |
| **Remix** | https://remix.ethereum.org | Browser-based IDE |
| **Tenderly** | https://tenderly.co | Debugging and simulation |

### Frontend Development

| Tool | URL | Description |
|------|-----|-------------|
| **wagmi** | https://wagmi.sh | React hooks for Ethereum |
| **viem** | https://viem.sh | TypeScript Ethereum library |
| **RainbowKit** | https://www.rainbowkit.com | Wallet connection UI |
| **ConnectKit** | https://docs.family.co/connectkit | Alternative wallet UI |
| **Web3Modal** | https://web3modal.com | WalletConnect's modal |

### Data & Indexing

| Tool | URL | Description |
|------|-----|-------------|
| **The Graph** | https://thegraph.com | Decentralized indexing protocol |
| **Goldsky** | https://goldsky.com | Real-time blockchain data |
| **Alchemy** | https://www.alchemy.com | Node infrastructure + data APIs |
| **QuickNode** | https://www.quicknode.com | Blockchain infrastructure |

### Testing & Monitoring

| Tool | URL | Description |
|------|-----|-------------|
| **Vitest** | https://vitest.dev | Fast unit test framework |
| **Playwright** | https://playwright.dev | E2E testing |
| **Cypress** | https://www.cypress.io | E2E testing alternative |
| **Sentry** | https://sentry.io | Error monitoring |
| **DataDog** | https://www.datadoghq.com | Infrastructure monitoring |

---

## Design Inspiration

### Prediction Market UIs

| Platform | URL | What to Study |
|----------|-----|---------------|
| **Polymarket** | https://polymarket.com | Market cards, trading interface, mobile design |
| **Kalshi** | https://kalshi.com | Clean fintech aesthetic, order form |
| **Betfair Exchange** | https://www.betfair.com/exchange | Traditional exchange UI, depth charts |
| **Robinhood** | https://robinhood.com | Simple trading UX, onboarding |

### Design Systems & Components

| Resource | URL | Description |
|----------|-----|-------------|
| **Radix UI** | https://www.radix-ui.com | Accessible headless components |
| **shadcn/ui** | https://ui.shadcn.com | Re-usable component collection |
| **Tailwind UI** | https://tailwindui.com | Premium Tailwind components |
| **Headless UI** | https://headlessui.com | Unstyled accessible components |

### Charting Libraries

| Library | URL | Best For |
|---------|-----|----------|
| **Lightweight Charts** | https://tradingview.github.io/lightweight-charts | Financial charts (TradingView) |
| **Recharts** | https://recharts.org | General React charts |
| **D3.js** | https://d3js.org | Custom visualizations |
| **Visx** | https://airbnb.io/visx | Low-level visualization primitives |

### Design Resources

| Resource | URL | Description |
|----------|-----|-------------|
| **Figma** | https://www.figma.com | Collaborative design tool |
| **Dribbble** | https://dribbble.com/search/trading-app | Design inspiration |
| **Mobbin** | https://mobbin.com | Mobile app design patterns |
| **Land-book** | https://land-book.com | Landing page inspiration |

---

## Infrastructure & Services

### Cloud Providers

| Provider | URL | Best For |
|----------|-----|----------|
| **AWS** | https://aws.amazon.com | Full-featured, enterprise |
| **Google Cloud** | https://cloud.google.com | Data/ML capabilities |
| **Vercel** | https://vercel.com | Frontend deployment (Next.js) |
| **Railway** | https://railway.app | Simple backend deployment |
| **Render** | https://render.com | Full-stack platform |

### RPC Providers

| Provider | URL | Description |
|----------|-----|-------------|
| **Alchemy** | https://www.alchemy.com | Enterprise-grade nodes + APIs |
| **Infura** | https://www.infura.io | Reliable RPC infrastructure |
| **QuickNode** | https://www.quicknode.com | Multi-chain support |
| **Ankr** | https://www.ankr.com | Decentralized RPC |

### KYC/Identity Providers

| Provider | URL | Description |
|----------|-----|-------------|
| **Jumio** | https://www.jumio.com | ID verification |
| **Onfido** | https://onfido.com | Identity verification |
| **Persona** | https://withpersona.com | Identity infrastructure |
| **Veriff** | https://www.veriff.com | AI-powered verification |

### Fiat On-Ramps

| Provider | URL | Description |
|----------|-----|-------------|
| **MoonPay** | https://www.moonpay.com | Popular crypto on-ramp |
| **Transak** | https://transak.com | Global coverage |
| **Sardine** | https://www.sardine.ai | Fraud-focused on-ramp |
| **Ramp** | https://ramp.network | European focus |

### Communication & Support

| Tool | URL | Description |
|------|-----|-------------|
| **Intercom** | https://www.intercom.com | Customer messaging platform |
| **Zendesk** | https://www.zendesk.com | Customer support software |
| **Discord** | https://discord.com | Community management |
| **Telegram** | https://telegram.org | Community communication |

---

## Communities & Forums

### Crypto & Web3 Communities

| Community | URL | Description |
|-----------|-----|-------------|
| **Ethereum Magicians** | https://ethereum-magicians.org | Technical discussions |
| **ETHResearch** | https://ethresear.ch | Research forum |
| **r/ethereum** | https://reddit.com/r/ethereum | Reddit community |
| **r/CryptoMarkets** | https://reddit.com/r/CryptoMarkets | Trading discussions |

### Developer Communities

| Community | URL | Description |
|-----------|-----|-------------|
| **Ethereum Stack Exchange** | https://ethereum.stackexchange.com | Q&A for developers |
| **Polygon Discord** | https://discord.gg/polygon | Official Polygon community |
| **Foundry Telegram** | https://t.me/foundry_support | Foundry development help |

### Prediction Market Specific

| Community | URL | Description |
|-----------|-----|-------------|
| **Polymarket Discord** | https://discord.gg/polymarket | Official Polymarket community |
| **Metaculus Discord** | https://discord.gg/metaculus | Forecasting community |
| **Prediction Market Twitter** | Follow #predictionmarkets | Active Twitter community |

### News & Discussion

| Platform | URL | Description |
|----------|-----|-------------|
| **Crypto Twitter/X** | https://twitter.com | Real-time news and discussion |
| **Farcaster** | https://warpcast.com | Web3-native social network |
| **Lens Protocol** | https://lens.xyz | Decentralized social |

---

## Learning Resources

### Blockchain Development Courses

| Course | Platform | Description |
|--------|----------|-------------|
| **CryptoZombies** | https://cryptozombies.io | Interactive Solidity tutorial |
| **Ethereum Dev Bootcamp** | https://www.alchemy.com/university | Free Alchemy University course |
| **Buildspace** | https://buildspace.so | Project-based Web3 learning |
| **LearnWeb3** | https://learnweb3.io | Web3 developer curriculum |

### Prediction Market Theory

| Resource | URL | Description |
|----------|-----|-------------|
| **Wisdom of Crowds** (Book) | ISBN: 978-0385721707 | James Surowiecki's foundational book |
| **Prediction Markets (Wolfers/Zitzewitz)** | https://www.nber.org/papers/w10504 | Academic paper on PM theory |
| **Hanson's Prediction Markets** | https://mason.gmu.edu/~rhanson/ideafutures.html | Robin Hanson's research |

### Technical Documentation

| Resource | URL | Description |
|----------|-----|-------------|
| **Solidity Docs** | https://docs.soliditylang.org | Official Solidity documentation |
| **EVM Deep Dives** | https://noxx.substack.com | Technical EVM articles |
| **Blockchain Data Engineering** | https://ethereum-etl.readthedocs.io | Data extraction tools |

### Books

| Title | Author | Topic |
|-------|--------|-------|
| **Mastering Ethereum** | A. Antonopoulos & G. Wood | Ethereum deep dive |
| **How to DeFi** | CoinGecko | DeFi protocols and concepts |
| **The Art of Prediction** | Various | Forecasting techniques |

### YouTube Channels

| Channel | URL | Description |
|---------|-----|-------------|
| **Finematics** | https://youtube.com/c/Finematics | DeFi explainers |
| **Whiteboard Crypto** | https://youtube.com/c/WhiteboardCrypto | Crypto concepts |
| **Patrick Collins** | https://youtube.com/c/PatrickCollins | Smart contract tutorials |
| **Smart Contract Programmer** | https://youtube.com/@smartcontractprogrammer | Solidity tutorials |

---

## News & Research

### Crypto News

| Source | URL | Description |
|--------|-----|-------------|
| **The Block** | https://www.theblock.co | Crypto journalism |
| **CoinDesk** | https://www.coindesk.com | Industry news |
| **Decrypt** | https://decrypt.co | Crypto and Web3 news |
| **Blockworks** | https://blockworks.co | Institutional crypto |

### Research & Analysis

| Source | URL | Description |
|--------|-----|-------------|
| **Messari** | https://messari.io | Crypto research and data |
| **Delphi Digital** | https://delphidigital.io | Research reports |
| **Coin Metrics** | https://coinmetrics.io | On-chain analytics |
| **Nansen** | https://www.nansen.ai | Blockchain analytics |

### Prediction Market Coverage

| Source | URL | Description |
|--------|-----|-------------|
| **Polymarket Blog** | https://polymarket.com/blog | Official updates |
| **Prediction Markets Twitter** | https://twitter.com/pmresearch | PM research account |
| **Star Spangled Gamblers** | https://starspangledgamblers.com | PM analysis |

### Regulatory Updates

| Source | URL | Description |
|--------|-----|-------------|
| **CFTC** | https://www.cftc.gov | US regulatory updates |
| **SEC** | https://www.sec.gov | Securities regulation |
| **MiCA Updates** | https://www.esma.europa.eu | EU crypto regulation |

### Newsletters

| Newsletter | URL | Description |
|------------|-----|-------------|
| **Week in Ethereum** | https://weekinethereumnews.com | Weekly dev updates |
| **Bankless** | https://banklesshq.com | Crypto strategy |
| **The Daily Gwei** | https://thedailygwei.substack.com | Ethereum news |
| **Not Boring** | https://www.notboring.co | Tech/crypto analysis |

---

## Quick Links

### Essential Bookmarks

```
Development:
- Polygon Docs: https://docs.polygon.technology
- Foundry Book: https://book.getfoundry.sh
- wagmi Docs: https://wagmi.sh
- Polymarket API: https://docs.polymarket.com

Reference:
- ERC-1155: https://eips.ethereum.org/EIPS/eip-1155
- Gnosis CTF: https://docs.gnosis.io/conditionaltokens
- UMA Docs: https://docs.uma.xyz

Tools:
- PolygonScan: https://polygonscan.com
- Remix: https://remix.ethereum.org
- Tenderly: https://tenderly.co

Community:
- Polymarket Discord: https://discord.gg/polymarket
- Polygon Discord: https://discord.gg/polygon
```

---

*Resources Version: 1.0*
*Last Updated: January 2025*
*Note: URLs may change over time. Please verify links are current.*
