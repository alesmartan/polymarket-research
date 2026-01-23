# Glossary

A comprehensive glossary of terms used in prediction market platforms, blockchain technology, trading systems, and regulatory frameworks.

---

## Table of Contents

- [Prediction Market Terms](#prediction-market-terms)
- [Blockchain & Crypto Terms](#blockchain--crypto-terms)
- [Trading Terms](#trading-terms)
- [Technical Terms](#technical-terms)
- [Regulatory & Legal Terms](#regulatory--legal-terms)
- [Financial Terms](#financial-terms)

---

## Prediction Market Terms

### A

**AMM (Automated Market Maker)**
A smart contract that provides liquidity and enables trading without a traditional order book. Uses mathematical formulas (like constant product) to determine prices. Alternative to order book-based trading.

### B

**Binary Market**
A prediction market with exactly two possible outcomes: "Yes" or "No." The most common type of prediction market. Share prices for both outcomes typically sum to $1.00.

### C

**Categorical Market**
A prediction market with multiple mutually exclusive outcomes (more than two). Example: "Who will win the election?" with candidates A, B, C, D. Share prices across all outcomes sum to $1.00.

**Condition**
In the Conditional Token Framework, a condition represents a question with multiple possible outcomes. Each condition has a unique identifier (conditionId) and is associated with an oracle for resolution.

**Conditional Token**
A tokenized representation of a claim on a specific outcome. When the outcome occurs, the token can be redeemed for collateral. Implemented using ERC-1155 standard.

**Conditional Token Framework (CTF)**
A protocol developed by Gnosis for creating and trading conditional tokens. The foundational smart contract system used by Polymarket and similar platforms.

### E

**Event Contract**
A financial contract whose value is determined by the outcome of a real-world event. The CFTC's classification for prediction market contracts.

### I

**Information Aggregation**
The process by which prediction markets collect and synthesize information from many participants into a single probability estimate (the market price).

### L

**Liquidity Provider (LP)**
A market participant who provides capital to enable trading. Can be through market making (posting limit orders) or providing liquidity to an AMM pool.

### M

**Market**
A tradeable question about a future event. Contains one or more outcomes that users can trade.

**Market Creator**
The entity or individual who creates a new market by defining the question, outcomes, and resolution criteria.

**Market Maker**
A trader who provides liquidity by simultaneously posting buy and sell orders. Earns the bid-ask spread in exchange for the risk of holding positions.

**Market Resolution**
The process of determining and reporting the final outcome of a market. Handled by an oracle system.

### O

**Oracle**
A system that reports real-world outcomes to the blockchain for market resolution. Can be centralized (single authority), decentralized (voting), or dispute-based (UMA model).

**Outcome**
One possible result of a market. In a binary market, outcomes are "Yes" and "No."

**Outcome Share**
A tradeable token representing a claim on a specific outcome. Pays $1.00 if the outcome occurs, $0.00 if it doesn't.

### P

**Payout**
The amount received when redeeming shares after market resolution. Winning shares pay $1.00 per share.

**Position**
A user's holdings in a particular market. Can be long (bought shares) or short (sold shares).

**Price Discovery**
The process by which markets determine fair prices through supply and demand. In prediction markets, prices represent collective probability estimates.

**Probability**
In prediction markets, the share price directly represents the market's implied probability. A price of $0.65 implies a 65% probability.

### R

**Resolution**
See Market Resolution.

**Resolution Criteria**
The objective rules that determine which outcome wins when a market closes. Clear criteria are essential for trust.

**Resolution Source**
The authoritative source used to determine market outcomes. Examples: official election results, weather data, sports scores.

### S

**Scalar Market**
A prediction market where the outcome is a number within a range. Example: "What will be the price of BTC on Dec 31?" with range $0-$100,000.

**Share**
A unit of ownership in a market outcome. Priced between $0.01 and $1.00.

**Skin in the Game**
The concept that prediction market participants have financial incentive to trade on accurate information, unlike pollsters or pundits.

### W

**Winning Payout Fee**
A fee charged only on successful (winning) trades. Polymarket's primary revenue model.

**Wisdom of the Crowd**
The phenomenon where aggregated predictions from many independent individuals often outperform expert forecasts.

---

## Blockchain & Crypto Terms

### A

**Address**
A unique identifier (usually 42 characters starting with "0x") that represents a wallet or smart contract on a blockchain.

**ABI (Application Binary Interface)**
The interface definition for interacting with smart contracts. Specifies function signatures and data types.

### B

**Block**
A bundle of transactions grouped together and added to the blockchain. Each block references the previous block, forming a chain.

**Block Explorer**
A web tool for viewing blockchain data. Examples: Etherscan, Polygonscan.

**Bridge**
A protocol for transferring assets between different blockchains. Example: Moving USDC from Ethereum to Polygon.

### C

**Chain ID**
A unique identifier for a blockchain network. Polygon mainnet is 137; Mumbai testnet is 80001.

**Collateral**
The underlying asset locked to create conditional tokens. Typically a stablecoin like USDC.

**Consensus**
The mechanism by which blockchain nodes agree on the state of the ledger. Polygon uses Proof of Stake.

### D

**dApp (Decentralized Application)**
An application built on blockchain with smart contracts for backend logic.

**DeFi (Decentralized Finance)**
Financial services built on blockchain without traditional intermediaries.

### E

**ERC-20**
The standard interface for fungible tokens on Ethereum-compatible chains. Used for USDC and most cryptocurrencies.

**ERC-1155**
The multi-token standard supporting both fungible and non-fungible tokens in a single contract. Used by the Conditional Token Framework.

**EVM (Ethereum Virtual Machine)**
The runtime environment for executing smart contracts on Ethereum and compatible chains (Polygon, Arbitrum, etc.).

### G

**Gas**
The unit measuring computational effort required to execute transactions on Ethereum-compatible networks.

**Gas Price**
The amount of native currency (ETH, MATIC) paid per unit of gas. Higher prices result in faster transaction inclusion.

**Gas Limit**
The maximum amount of gas a transaction is allowed to consume.

### L

**L1 (Layer 1)**
The base blockchain layer. Ethereum is an L1.

**L2 (Layer 2)**
A scaling solution built on top of an L1. Polygon, Arbitrum, and Optimism are Ethereum L2s.

### M

**Mainnet**
The production blockchain network where real value is transacted. Opposite of testnet.

**MATIC**
The native token of Polygon network. Used to pay for gas fees.

**MetaMask**
The most popular browser wallet for interacting with Ethereum-compatible dApps.

**Minting**
Creating new tokens. In CTF, minting creates conditional tokens from collateral.

### N

**Node**
A computer running blockchain software that validates transactions and maintains the ledger.

**Non-Custodial**
A system where users maintain control of their own private keys and assets. No third party holds funds.

### P

**Polygon**
An Ethereum Layer 2 scaling solution. Polymarket is built on Polygon.

**Private Key**
A secret cryptographic key that controls access to a blockchain address. Never share.

**Public Key**
A cryptographic key derived from the private key. Used to generate addresses.

### R

**RPC (Remote Procedure Call)**
The interface for communicating with blockchain nodes. RPC providers like Alchemy and Infura offer reliable access.

### S

**Smart Contract**
Self-executing code deployed on a blockchain. Handles logic like trading, resolution, and payouts.

**Stablecoin**
A cryptocurrency pegged to a stable asset like USD. USDC is the primary stablecoin for prediction markets.

### T

**Testnet**
A blockchain network for testing with no real value. Mumbai is Polygon's testnet.

**Transaction**
An operation recorded on the blockchain. Includes transfers, contract interactions, and deployments.

**TVL (Total Value Locked)**
The total value of assets deposited in a protocol. A key metric for DeFi platforms.

### U

**UMA (Universal Market Access)**
A decentralized oracle protocol used by Polymarket for dispute resolution and outcome reporting.

**USDC (USD Coin)**
A stablecoin pegged 1:1 to the US dollar. The primary collateral for Polymarket and similar platforms.

### W

**Wallet**
Software for managing blockchain addresses and signing transactions. Examples: MetaMask, Rainbow, Coinbase Wallet.

**Web3**
The paradigm of decentralized internet applications built on blockchain technology.

---

## Trading Terms

### A

**Ask**
The lowest price at which someone is willing to sell. Also called "offer."

**Average Price**
The weighted average price at which an order was filled, accounting for multiple price levels.

### B

**Bid**
The highest price at which someone is willing to buy.

**Bid-Ask Spread**
The difference between the best bid and best ask. Tighter spreads indicate more liquidity.

### C

**CLOB (Central Limit Order Book)**
An order book system where orders are matched based on price-time priority. Polymarket uses a hybrid CLOB.

### D

**Depth**
The volume of orders at various price levels. Deeper markets can handle larger trades without significant price impact.

### F

**Fill**
When an order is matched and executed, it is "filled."

**FOK (Fill or Kill)**
An order that must be completely filled immediately or canceled entirely.

### G

**GTC (Good Till Cancelled)**
An order that remains active until filled or manually canceled.

**GTD (Good Till Date)**
An order that expires at a specified date/time if not filled.

### I

**IOC (Immediate or Cancel)**
An order that fills whatever is immediately available and cancels the remainder.

### L

**Limit Order**
An order to buy or sell at a specified price or better. Provides price certainty but not execution certainty.

**Liquidity**
The ease with which assets can be bought or sold without significantly affecting the price. More liquidity = tighter spreads.

**Long Position**
Owning shares and profiting if the price increases (or if the outcome occurs).

### M

**Maker**
A trader who adds liquidity by placing limit orders that don't immediately execute. Often receives fee rebates.

**Market Order**
An order to buy or sell immediately at the best available price. Provides execution certainty but not price certainty.

**Matching Engine**
The system that pairs buy and sell orders. Typically uses price-time priority.

### O

**Order Book**
The list of all open buy and sell orders for a market, organized by price level.

### P

**Price-Time Priority**
The matching algorithm where orders are filled first by best price, then by earliest submission time.

### S

**Short Position**
Owing shares (having sold) and profiting if the price decreases (or if the outcome doesn't occur).

**Slippage**
The difference between expected price and actual execution price, usually due to market movement or low liquidity.

### T

**Taker**
A trader who removes liquidity by placing orders that immediately execute against existing orders. Usually pays fees.

### V

**Volume**
The total value or number of shares traded over a time period.

---

## Technical Terms

### A

**API (Application Programming Interface)**
A set of protocols for building and interacting with software applications. REST and WebSocket are common API types.

### C

**CDN (Content Delivery Network)**
A distributed network of servers that delivers content quickly to users globally.

**CI/CD (Continuous Integration/Continuous Deployment)**
Automated processes for testing and deploying code changes.

### D

**Database Schema**
The structure defining how data is organized in a database.

### E

**E2E (End-to-End) Testing**
Testing that simulates real user scenarios across the entire application.

### I

**Indexer**
A service that processes blockchain events and stores them in a queryable database for efficient retrieval.

### M

**Microservices**
An architectural style where an application is composed of loosely coupled, independently deployable services.

### R

**REST (Representational State Transfer)**
An architectural style for designing networked applications using standard HTTP methods.

### S

**SDK (Software Development Kit)**
A collection of tools and libraries for building applications on a specific platform.

**SSR (Server-Side Rendering)**
Generating HTML on the server for faster initial page loads and better SEO.

### W

**WebSocket**
A protocol providing full-duplex communication channels over a single TCP connection. Used for real-time data feeds.

---

## Regulatory & Legal Terms

### A

**AML (Anti-Money Laundering)**
Regulations requiring financial institutions to prevent, detect, and report money laundering activities.

**AMLD (Anti-Money Laundering Directive)**
EU directives setting AML requirements. Currently AMLD6.

### C

**CFTC (Commodity Futures Trading Commission)**
US federal agency regulating derivatives markets. Classifies prediction markets as event contracts (swaps).

**CEA (Commodity Exchange Act)**
The primary US law governing commodity futures and options. Administered by the CFTC.

### D

**DCM (Designated Contract Market)**
A CFTC-registered exchange authorized to list futures and options contracts. Kalshi is a DCM.

**DLT Framework**
Distributed Ledger Technology regulatory framework. Gibraltar pioneered DLT licensing.

### F

**FinCEN (Financial Crimes Enforcement Network)**
US Treasury bureau administering the Bank Secrecy Act and combating financial crimes.

### G

**GDPR (General Data Protection Regulation)**
EU regulation on data protection and privacy. Requires consent, data minimization, and user rights.

### K

**KYC (Know Your Customer)**
Identity verification procedures required by financial regulations. Typically involves ID verification and address proof.

### M

**MiCA (Markets in Crypto-Assets)**
EU regulation creating a framework for crypto-asset service providers. Effective 2024-2025.

**Money Transmitter License (MTL)**
US state-level license required for businesses that transfer money. Requirements vary by state.

### N

**No-Action Letter**
A letter from a regulator stating they will not take enforcement action against specific conduct. PredictIt operates under a CFTC no-action letter.

### S

**SEC (Securities and Exchange Commission)**
US federal agency regulating securities markets. May have jurisdiction if tokens are deemed securities.

**SEF (Swap Execution Facility)**
A CFTC-regulated trading platform for swaps.

### T

**ToS (Terms of Service)**
The legal agreement between a platform and its users defining rights and responsibilities.

---

## Financial Terms

### A

**AUM (Assets Under Management)**
The total value of assets managed by a fund or platform.

### B

**Breakeven**
The price at which a trade neither gains nor loses money.

### C

**CAC (Customer Acquisition Cost)**
The cost to acquire a single customer, including marketing and sales expenses.

### E

**Expected Value (EV)**
The probability-weighted average of all possible outcomes. Positive EV trades are profitable long-term.

### L

**LTV (Lifetime Value)**
The total revenue expected from a customer over their entire relationship with the platform.

### M

**MAU (Monthly Active Users)**
The number of unique users who engage with the platform in a month.

### P

**P&L (Profit and Loss)**
The financial gain or loss from trading activities.

### R

**ROI (Return on Investment)**
The ratio of profit or loss relative to the original investment.

### U

**Unit Economics**
The direct revenues and costs associated with a single unit (user, trade, etc.). Positive unit economics are required for sustainable growth.

---

## Quick Reference

### Common Abbreviations

| Abbreviation | Full Term |
|--------------|-----------|
| AMM | Automated Market Maker |
| API | Application Programming Interface |
| AML | Anti-Money Laundering |
| CEA | Commodity Exchange Act |
| CFTC | Commodity Futures Trading Commission |
| CLOB | Central Limit Order Book |
| CTF | Conditional Token Framework |
| DCM | Designated Contract Market |
| EVM | Ethereum Virtual Machine |
| GDPR | General Data Protection Regulation |
| KYC | Know Your Customer |
| L1/L2 | Layer 1/Layer 2 |
| LP | Liquidity Provider |
| MiCA | Markets in Crypto-Assets |
| MTL | Money Transmitter License |
| RPC | Remote Procedure Call |
| SEF | Swap Execution Facility |
| TVL | Total Value Locked |
| UMA | Universal Market Access |
| USDC | USD Coin |

---

*Glossary Version: 1.0*
*Last Updated: January 2025*
