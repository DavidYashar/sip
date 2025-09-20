SIP: 2
Title: BTNFT - Bitcoin Token Non-Fungible Extension for Spark
Author: DavidYashar
Status: Draft
Type: Standard
Category: Core
Created: 2025-08-18
Discussions-to: TBD

## Summary

BTNFT (Bitcoin Token Non-Fungible) brings native NFT functionality to the Spark Bitcoin Layer 2 network by extending the proven BTKN protocol. Rather than building a separate system, BTNFT seamlessly integrates with Spark's existing TTXO (Token Transaction Output) infrastructure, enabling NFT collections, individual token minting, and instant transfers while maintaining complete compatibility with Spark's statechain technology and FROST signing mechanisms.

## Motivation

The NFT market presents a massive opportunity for Bitcoin Layer 2 adoption, with over $7.9 billion in market cap and approximately $2.4 billion in trading volume across all blockchains. However, most NFT activity remains trapped on expensive, slow networks that charge high gas fees and suffer from poor user experience.

Spark's unique architecture solves these fundamental problems by providing the ideal foundation for Bitcoin-native NFTs. Users can enjoy instant transfers through proven statechain technology that delivers sub-second NFT movements without any gas fees, while maintaining complete self-custody through Bitcoin's security guarantees. Perhaps most importantly, BTNFT enables native Lightning Network compatibility, allowing NFT micropayments and streaming capabilities that simply don't exist on other platforms.

By building on the battle-tested BTKN and LRC-20 foundation, BTNFT positions Spark as the premier platform for Bitcoin NFTs while demonstrating how Bitcoin can evolve beyond simple value transfer to support sophisticated digital assets and applications.

## Technical Architecture

BTNFT extends the proven BTKN protocol by adding NFT-specific transaction types to the existing TokenTransaction model. This approach ensures seamless compatibility with existing wallets, services, and Lightning integration rather than fragmenting the ecosystem with separate protocols.

The core design follows five fundamental principles that make BTNFT both powerful and practical. First, NFTs are treated as specialized BTKN tokens with unique metadata, ensuring they inherit all existing security and functionality. Second, NFT data integrates directly within the proven Token Transaction Output structure, maintaining consistency across the system. Third, the same Start/Sign/Finalize transaction flow that users know from regular BTKN tokens applies to NFTs, eliminating learning curves. Fourth, NFTs flow naturally through Lightning channels using existing infrastructure, enabling unprecedented payment and streaming capabilities. Finally, users maintain complete self-custody through the same proven cryptographic mechanisms that secure all Spark assets.

### Metadata Strategy: The Side-Car Approach Explained

BTNFT implements a carefully designed side-car approach to metadata handling that prioritizes security and compatibility above all else. This isn't just a technical decision - it's a strategic choice that allows us to deliver full NFT functionality today while keeping the proven BTKN foundation completely untouched and rock-solid.

The side-car approach works like this: instead of modifying the core `TokenTransaction` or `TokenOutput` structures that have been battle-tested in production, we keep NFT metadata in a separate but linked system. Think of it like having a secure vault (the BTKN TokenOutput) that holds the actual ownership rights, while keeping a detailed catalog (the NFT metadata) that describes what's in the vault. The catalog and vault are connected, but the vault's security doesn't depend on the catalog being perfect.

This means ownership remains purely within the base TTXO structure using all the same cryptographic proofs and security mechanisms that protect regular BTKN tokens. Meanwhile, rich metadata like names, descriptions, images, and attributes live in an associated index that's keyed by collection and token identifiers. The beauty of this approach is that even if something happened to the metadata system, the ownership rights would remain completely secure and recoverable.

Our side-car implementation uses the `NftOutputRecord` structure that elegantly combines both worlds. Each record contains a standard `TokenOutput` that handles all the Bitcoin-secured ownership mechanics, plus `NftMetadata` that provides all the rich information that makes each NFT unique and interesting. A transaction hash links these together, creating a tamper-evident connection between ownership and metadata.

All metadata fields include strict validation limits that prevent denial-of-service attacks while supporting real-world usage patterns. Collection IDs are limited to 50 characters, token IDs to 20 characters, descriptions to 1000 characters, and image URLs to 500 characters. Each NFT can have up to 50 attributes, and batch minting operations are capped at 50 NFTs per transaction. These limits aren't arbitrary - they're carefully chosen based on actual NFT usage patterns and system performance requirements.

The side-car approach also provides a clear path for future evolution. A future phase may embed an optional `nft_metadata` field directly within `TokenOutput`, but only after extensive hashing determinism reviews and comprehensive security audits. This conservative approach means we can deliver NFT functionality immediately while keeping all our options open for even deeper integration when the time is right.

### NFT Service Architecture: Extending What Works

BTNFT introduces a dedicated `SparkNftService` that provides all NFT functionality through clean, purpose-built endpoints. This isn't a replacement for existing services - it's an extension that works alongside everything users already know and trust. The service includes two core transaction operations (`start_nft_transaction` and `commit_nft_transaction`) that follow the exact same two-phase pattern that BTKN users are familiar with, plus four specialized query operations for discovering and managing NFT data.

The transaction operations work identically to existing BTKN flows. When you want to create a collection, mint an NFT, or transfer ownership, you start by calling `start_nft_transaction` with your NFT intent and authorization signatures. Spark Operators validate your request, coordinate among themselves, and return a fully prepared transaction along with keyshare information. Then you finalize everything with `commit_nft_transaction`, providing the final signatures from all operators to make your NFT operation permanent.

The query operations provide comprehensive NFT discovery and management capabilities. `query_nft_collections` helps you find collections by creator, identifier, or network, perfect for marketplace browsing and creator portfolio displays. `query_nft_tokens` finds individual NFTs across collections, supporting portfolio tracking and marketplace search functionality. `query_nft_outputs` gives you access to the underlying Bitcoin-secured ownership layer, essential for wallet management and ownership verification. Finally, `query_nft_metadata` provides pure NFT information without ownership complexity, optimized for gallery displays and attribute browsing.

What makes this architecture brilliant is how the side-car approach enables clean separation of concerns. The transaction operations handle ownership changes through proven BTKN mechanisms, while query operations can access either the ownership layer, the metadata layer, or both combined. This flexibility means different applications can interact with exactly the data they need without unnecessary complexity.

### Lightning Network Integration: Revolutionary NFT Capabilities

BTNFT's Lightning Network integration creates entirely new possibilities for NFT usage that don't exist on other platforms. Unlike systems where NFTs are large, immobile assets, BTNFT NFTs can participate in conditional payment workflows through Lightning channels. The ownership outputs, rather than bulky metadata, flow through payment channels, enabling rapid state changes and micropayment patterns.

This architecture enables fascinating use cases that showcase Bitcoin's potential beyond simple value transfer. Streaming access becomes possible where users pay satoshis per minute for premium content NFTs, automatically unlocking access as payments flow. Gaming assets can be transferred instantly between players with Lightning payments, creating fluid in-game economies. Royalty distribution happens automatically on Lightning-enabled marketplaces, ensuring creators receive payments in real-time as their works are traded.

Perhaps most importantly, Lightning integration brings privacy benefits to NFT trading. Payment paths remain obscured through onion routing, and off-chain transfers reduce on-chain surveillance compared to Ethereum NFTs where every transaction is permanently visible. This combination of speed, low cost, and privacy creates a compelling alternative to existing NFT platforms.

## Infrastructure Changes: What Each Component Needs

Implementing BTNFT requires coordinated updates across Spark's infrastructure, but the beauty of the side-car approach is that these changes are purely additive - nothing existing gets broken or replaced. Each component in the ecosystem gets enhanced with new NFT capabilities while continuing to handle regular BTKN operations exactly as before.

### Spark Core Repository Changes

The core Spark repository needs to be extended with the new NFT service definitions and validation logic, but these additions work alongside existing functionality rather than replacing anything. The `SparkNftService` gets added as a separate service with its own endpoints, while existing BTKN services continue operating unchanged. This separation ensures that NFT functionality can be developed, tested, and deployed without any risk to existing operations.

The core changes include adding the NFT protobuf definitions that we've carefully designed, implementing the service handlers that process NFT requests, and extending the validation logic to handle NFT-specific rules like collection uniqueness and supply limits. All of this builds on top of the existing FROST signing infrastructure, transaction processing pipeline, and security mechanisms that users already trust.

In general, the side-car approach means the core `TokenTransaction` and `TokenOutput` structures remain completely unchanged in Phase 1. This eliminates any risk of introducing bugs or compatibility issues in the most critical parts of the system. NFT functionality gets layered on top through clean interfaces rather than modifying proven code paths.

### Spark Operator Updates: The Consensus Layer

Spark Operators need the most significant updates because they handle validation and consensus for all NFT operations. However, these updates follow the same patterns that SOs already use for BTKN validation, so they're extending familiar functionality rather than implementing something completely new.

Each Spark Operator needs to add NFT-specific validation functions that check collection uniqueness, verify supply limits, validate token ID uniqueness within collections, and enforce royalty and mutability rules. These validation functions run during the standard transaction processing flow, so they integrate seamlessly with existing FROST signing and consensus mechanisms.

The operators also need database schema updates to track NFT collections and tokens locally. These databases serve as operational caches that improve query performance and enable the discovery functionality that marketplaces and wallets need. Importantly, these databases don't affect consensus or security - they're purely for operational efficiency, with the real source of truth remaining in the signed transactions on the statechain.

All Spark Operators must deploy these updates in a coordinated fashion because NFT validation requires consensus across the operator network. If only some operators understood NFT transactions, the network couldn't reach agreement on their validity. This coordinated deployment is standard practice for Spark protocol upgrades and ensures the network maintains consistency.

### LRC-20 Node Enhancements: Gossip and Data Availability

LRC-20 nodes need updates to handle NFT metadata in the gossip network, but these changes follow the exact same patterns used for regular BTKN transactions. The nodes already know how to gossip transaction data, validate signatures, and maintain data availability - they just need to recognize and process the additional NFT metadata that gets attached to transactions.

The gossip protocol extensions enable NFT metadata to flow through the network alongside transaction data. When an LRC-20 node receives a transaction with NFT metadata, it validates the metadata format, stores it for availability, and forwards it to other nodes just like any other transaction data. This ensures that NFT information remains accessible even if individual operators go offline.

These nodes also provide the data availability layer that enables the query operations in the NFT service. When a wallet wants to display a user's NFT collection or a marketplace wants to show available NFTs, they can query the LRC-20 network to get comprehensive metadata without depending on any single operator. This distributed approach ensures robust data availability without creating central points of failure.

### State Chain Protocol: No Changes Needed

Here's where the side-car approach really shines - the core state chain protocol needs absolutely no changes. The statechain already handles TTXO ownership transfers, key rotation, revocation mechanisms, and all the cryptographic operations that make Spark secure. NFTs use these exact same mechanisms for ownership, so there's nothing new for the statechain to learn.

When an NFT gets transferred, what actually happens under the hood is a standard TTXO ownership change using the same key tweaking and threshold signing that protects regular BTKN tokens. The NFT metadata travels alongside this ownership change through the side-car system, but the statechain itself only sees and processes a normal ownership transfer.

This means all the battle-tested security properties of the statechain automatically extend to NFTs. The same revocation keys protect against operator misbehavior, the same watchtower infrastructure guards against attacks, and the same cryptographic proofs ensure ownership validity. NFT users get enterprise-grade security without any new attack surfaces or experimental protocols.

## Development Timeline and Deployment Strategy

The BTNFT implementation follows a carefully planned four-week timeline that ensures thorough testing and smooth deployment across the entire Spark ecosystem. This timeline reflects the complexity of coordinating updates across multiple systems while maintaining the high security and reliability standards that Spark users expect.

### Phase 1: Protocol Foundation (Week 1)

Week one gets the core protocol extensions right by implementing fundamental NFT operations without rushing user-facing features. We start by extending protobuf definitions with our NFT message types, then build service handlers that process these transactions within existing Spark infrastructure.

Spark Operators receive validation logic for collection uniqueness, supply limits, and token ID validation - the consensus layer that keeps everything secure. LRC-20 nodes get gossip protocol updates to handle NFT metadata alongside transaction data, ensuring information flows properly even when nodes go offline.

The week wraps with comprehensive unit testing that proves every NFT transaction type works flawlessly with existing BTKN functionality, confirming our side-car approach adds capabilities without breaking anything.

### Phase 2: Integration and SDK Development (Week 2)

Week two brings BTNFT to developers through SDK updates and API extensions. The TypeScript SDK gets complete NFT support with helper functions for collections, minting, transfers, and queries - following familiar BTKN patterns that make development natural.

Spark wallets receive updates to display and manage NFT TTXOs through intuitive interfaces users already trust, making NFT management feel like a seamless extension rather than separate functionality. GraphQL API extensions provide efficient marketplace integration with browsing, filtering, and discovery capabilities designed for high-traffic applications.

Lightning integration testing validates our innovative conditional transfer capabilities, ensuring streaming and micropayment use cases actually deliver the revolutionary functionality we've designed.

### Phase 3: Production Deployment (Weeks 3-4)

The final weeks handle critical mainnet deployment tasks. Security audits ensure every aspect meets Spark's standards, covering both NFT-specific code and integration points to prevent any weakening of existing systems.

Performance testing optimizes high-volume operations through stress testing collections, batch minting, concurrent transfers, and marketplace query patterns. We ensure BTNFT handles real-world usage without degrading BTKN performance.

Documentation completion provides developer guides and API references, while community review incorporates feedback from developers, operators, and users. The coordinated Spark Operator network deployment requires careful synchronization since all operators must update simultaneously for consensus, with rollback planning ensuring quick issue resolution.

### Database and Storage: Performance Without Compromising Security

Spark Operators implement local NFT indexing for speed, but here's the key - protocol security never depends on these databases. This separation means operators get lightning-fast queries while maintaining bulletproof security through statechain cryptographic proofs.

The PostgreSQL schema reflects real-world usage with collections storing creator keys, metadata, and royalty info, while tokens link through foreign keys with flexible JSON attributes. These databases serve as operational caches, not sources of truth - wallets get instant portfolio displays and marketplaces get millisecond queries, but ownership validation relies on tamper-proof signed transactions.

Protocol validity stays anchored in TokenTransactions and the statechain, so database corruption affects performance, not security. The beauty? If an operator's database gets wiped, they rebuild it entirely from transaction history without losing data or compromising security - it's just a reconstructible performance layer over the distributed secure infrastructure.

## Security Model and Analysis

BTNFT builds entirely on Spark's proven foundation, inheriting FROST threshold signing, statechain ownership transfers, and watchtower protection without introducing new attack surfaces. Collection creators control minting through cryptographic signatures, while Spark Operators enforce supply limits and uniqueness through consensus validation.

The layered security approach keeps essential ownership within TTXO cryptographic proofs while metadata lives in side-car indices. Off-chain content uses IPFS or HTTP with optional hash references, accepting standard NFT external dependency risks while keeping core ownership bulletproof.

Protocol design prevents collection hijacking through creator validation, supply manipulation through consensus enforcement, and double-spending through existing revocation infrastructure. Standard market risks like royalty bypass and wash trading represent business challenges rather than protocol vulnerabilities - collections remain functional even if creators disappear, protecting user investments.

## Lightning Capabilities and Use Cases

The Lightning Network integration creates unprecedented capabilities for Bitcoin NFTs that showcase the technology's potential beyond simple payments. Unlike other NFT platforms where assets remain static until explicitly traded, BTNFT enables dynamic, programmable interactions through Lightning's conditional payment mechanisms.

Streaming access represents one of the most innovative use cases, where premium content NFTs unlock automatically as users stream micropayments. This enables new business models for content creators who can monetize their work directly without platform intermediaries, receiving payments in real-time as users consume content. The granular payment capability means creators can charge per minute, per view, or per interaction, creating flexible monetization strategies impossible on traditional platforms.

Gaming applications become particularly compelling when items can be transferred instantly with Lightning payments. Players can rent powerful items for specific battles, lending them back automatically when rental periods expire. Cross-game asset interoperability becomes practical when transfers happen instantly and cheaply, allowing players to move valuable items between different gaming ecosystems. Tournament organizers can implement automatic prize distribution where winning NFTs transfer to victors simultaneously with Lightning prize payments.

Utility tokens gain new functionality when combined with Lightning micropayments. Event tickets can unlock access automatically upon payment, membership tokens can enable recurring subscriptions through streaming payments, and certificate systems can validate credentials while receiving micropayments for verification services. These applications showcase how NFTs can become active, programmable assets rather than static collectibles.

## Success Metrics and Timeline

The success of BTNFT will be measured through both technical performance and ecosystem adoption metrics that demonstrate real-world utility. Technical targets include maintaining transaction throughput above 1,000 NFT operations per second, matching existing BTKN performance while keeping transfer latency below 100 milliseconds for excellent user experience. Storage efficiency remains crucial, with minimal TTXO size increases even with embedded metadata, and 100% compatibility with existing Lightning infrastructure ensuring seamless integration.

Adoption metrics for the first six months include ambitious but achievable targets that would establish BTNFT as a significant player in the NFT space. The goal of 500 unique NFT collections would demonstrate creator adoption across diverse use cases, while 50,000 individual tokens would show meaningful user engagement. Active user metrics targeting 2,000 unique addresses performing NFT operations would indicate healthy ecosystem growth, and 10,000 NFT transfers via Lightning channels would prove the unique capabilities are being utilized.

Ecosystem integration targets focus on building the infrastructure necessary for long-term success. Support from three major Spark wallets with full NFT functionality would ensure users have quality tools available, while integration with two NFT marketplaces would provide liquidity and discovery mechanisms. Developer adoption through 10 projects building on BTNFT infrastructure would create momentum for continued growth, and community engagement targeting 1,000 Discord and forum members would establish the social foundation for a thriving ecosystem.

## Conclusion

BTNFT demonstrates Bitcoin's evolution beyond simple payments to sophisticated digital assets while maintaining trustlessness and self-custody. By extending proven BTKN protocol rather than fragmenting the ecosystem, it inherits battle-tested security, existing infrastructure, and established user bases.

Lightning integration creates genuinely unique capabilities - instant micropayments, streaming access, automatic royalties, and privacy-preserving transfers - that position BTNFT as a foundation for innovative applications impossible on other platforms. The conservative side-car approach enables safe deployment while preserving future integration options.

BTNFT positions Spark to capture significant NFT market share through superior user experience: instant transfers, unique Lightning capabilities, and true Bitcoin nativity. This creates a foundation for gaming, streaming, and utility applications that could drive broader Bitcoin adoption while proving the network's evolution into a platform for innovative financial applications.

## References

1. **Spark Documentation**: https://docs.spark.money
2. **Spark GitHub Repository**: https://github.com/buildonspark/spark  
3. **BTKN Token Protocol**: https://docs.spark.money/lrc20/hello-btkn
4. **Bitcoin Statechains**: https://github.com/RubenSomsen/rubensomsen.github.io/blob/master/img/statechains.pdf
5. **FROST Threshold Signatures**: https://eprint.iacr.org/2020/852.pdf
6. **Lightning Network Specification**: https://github.com/lightning/bolts
7. **ERC-721 NFT Standard**: https://eips.ethereum.org/EIPS/eip-721 (for comparison)
8. **Bitcoin Improvement Proposals**: https://github.com/bitcoin/bips
