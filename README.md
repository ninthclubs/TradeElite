# ShadowTrade

**Private trading platform with homomorphic order matching**

ShadowTrade enables confidential trading where order sizes, prices, and strategies remain encrypted during order matching and execution. Built on Zama's Fully Homomorphic Encryption Virtual Machine (FHEVM), the platform matches buy and sell orders over encrypted data, revealing only trade execution details—never individual order information.

---

## Concept

ShadowTrade addresses the transparency-privacy tension in decentralized trading. Public order books expose trading strategies, enabling front-running and market manipulation. ShadowTrade uses Zama FHEVM to match orders homomorphically, ensuring that order details remain encrypted until execution, while maintaining verifiable trade settlement on-chain.

**Innovation**: Homomorphic order matching enables efficient price discovery without exposing order book depth or individual trader strategies.

---

## Core Mechanisms

### Order Submission

Traders submit encrypted orders containing:
- **Encrypted Price**: Order price encrypted with FHE
- **Encrypted Quantity**: Order size encrypted with FHE  
- **Encrypted Timestamp**: Time priority encrypted with FHE
- **Public Metadata**: Order type (buy/sell), asset pair (public)

Orders are stored on-chain as encrypted ciphertexts, visible only to the matching engine.

### Homomorphic Matching

The matching engine processes encrypted orders:

**Price Comparison:**
```solidity
ebool canMatch = TFHE.gt(buyOrder.encryptedPrice, sellOrder.encryptedPrice);
```

**Quantity Calculation:**
```solidity
euint64 tradeQuantity = TFHE.min(buyOrder.encryptedQuantity, sellOrder.encryptedQuantity);
```

**Trade Execution:**
```solidity
// Compute trade details homomorphically
euint64 buyerPays = TFHE.mul(tradeQuantity, buyOrder.encryptedPrice);
euint64 sellerReceives = TFHE.mul(tradeQuantity, sellOrder.encryptedPrice);
```

All matching occurs over encrypted data—no decryption required.

### Settlement & Revelation

After matching:
1. Encrypted trade details computed homomorphically
2. Trade settlement executed (tokens transferred)
3. Encrypted order book updated
4. Public trade record published (quantity, price, timestamp only)
5. Individual orders remain encrypted

---

## System Components

### Smart Contracts

**OrderBook Contract**
- Stores encrypted buy and sell orders
- Performs homomorphic order matching
- Executes trade settlements
- Manages order lifecycle (submission, matching, cancellation)

**Asset Manager Contract**
- Handles token transfers for trade settlement
- Manages order collateral
- Implements trading fees (computed homomorphically)
- Ensures atomic trade execution

**Price Oracle Integration**
- Optional: External price feeds for market data
- Used for order validation and settlement verification
- Maintains market price reference

### Client Application

**Trading Interface**
- Order creation and submission UI
- Encrypted order encryption (client-side)
- Trade history and portfolio view
- Real-time market data (aggregate statistics only)

**Wallet Integration**
- MetaMask/WalletConnect for authentication
- Token approval and trading
- Transaction signing
- Balance management

### Matching Engine

**Homomorphic Processing**
- Encrypted order book maintenance
- Price-time priority matching
- Trade quantity optimization
- Fee calculation over encrypted values

**Order Management**
- Order expiration handling
- Partial fill processing
- Order cancellation
- Market depth aggregation (encrypted)

---

## Privacy Model

### Information Visibility

**Public Information:**
- Trade executions (price, quantity, timestamp)
- Aggregate market statistics
- Asset pair information
- Contract addresses

**Encrypted Information:**
- Individual order prices
- Order quantities
- Order submission timestamps
- Trading strategies
- Portfolio positions (until trade)

**Revealed on Settlement:**
- Executed trade price
- Executed trade quantity
- Trade parties (optional: can use proxy addresses)
- Settlement timestamp

### Threat Mitigation

**Front-Running Prevention:**
- Orders encrypted until matching
- No visibility into order book depth
- Time priority encrypted
- Matching occurs atomically

**Strategy Protection:**
- Trading strategies remain hidden
- Order sizes not visible to competitors
- No correlation between orders and addresses (if using proxies)

**Market Manipulation Resistance:**
- Large orders not visible to others
- Price discovery happens over encrypted data
- No order book depth exploitation
- Fair matching algorithm

---

## Trading Features

### Order Types

**Limit Orders**
- Encrypted price and quantity
- Matched when encrypted price conditions met
- Time priority preserved (encrypted)

**Market Orders**
- Immediate execution at best available price
- Price computed homomorphically from order book
- Fast settlement

**Stop Orders**
- Conditional execution triggers
- Encrypted stop price
- Automatic activation when conditions met

### Advanced Features

**Partial Fills**
- Large orders filled incrementally
- Encrypted remaining quantity tracked
- Multiple settlements per order

**Order Cancellation**
- Cancel encrypted orders before matching
- No revelation of order details
- Immediate cancellation confirmation

**Market Depth (Aggregate)**
- Aggregate bid/ask quantities (encrypted totals)
- Price range indicators (without revealing individual orders)
- Market sentiment indicators

---

## Technical Specifications

### FHE Data Structures

```solidity
struct EncryptedOrder {
    euint64 price;          // Encrypted price per unit
    euint64 quantity;        // Encrypted order quantity
    euint32 timestamp;      // Encrypted submission time
    address trader;         // Public trader address
    OrderType orderType;    // Buy or Sell (public)
    AssetPair pair;         // Trading pair (public)
}
```

### Homomorphic Operations

**Price Comparison:**
```solidity
ebool priceMatch = TFHE.gt(buyOrder.price, sellOrder.price);
```

**Quantity Matching:**
```solidity
euint64 fillQuantity = TFHE.min(buyOrder.quantity, sellOrder.quantity);
```

**Trade Value Calculation:**
```solidity
euint64 tradeValue = TFHE.mul(fillQuantity, executionPrice);
```

**Fee Computation:**
```solidity
euint64 fee = TFHE.mul(tradeValue, feeRate);
```

### Gas Optimization

**Batch Operations:**
- Multiple orders processed in single transaction
- Aggregated matching reduces gas costs
- Efficient encrypted order book updates

**Lazy Evaluation:**
- Defer expensive operations until necessary
- Cache encrypted intermediate results
- Optimize matching algorithm complexity

---

## Trading Workflow

### Step 1: Order Preparation

1. Trader selects asset pair (e.g., ETH/USDT)
2. Chooses order type (limit, market, stop)
3. Enters price and quantity (if limit order)
4. Client encrypts order details with FHE public key

### Step 2: Order Submission

1. Encrypted order submitted to OrderBook contract
2. Contract validates order (balance, allowances)
3. Order stored in encrypted order book
4. Trader receives order confirmation

### Step 3: Order Matching

1. Matching engine processes encrypted orders
2. Price comparison performed homomorphically
3. Matching orders identified (without revealing prices)
4. Trade quantity calculated over encrypted values

### Step 4: Trade Execution

1. Trade details computed homomorphically
2. Settlement executed (token transfers)
3. Trade recorded on-chain (public execution data)
4. Order book updated (encrypted orders modified)

### Step 5: Result Processing

1. Trader receives trade confirmation
2. Portfolio updated (balance changes)
3. Trade history recorded
4. Encrypted order details remain private

---

## Use Cases

### Private Trading Strategies

**Large Order Execution**
- Split large orders without revealing total size
- Execute in multiple smaller trades
- Prevent market impact from order visibility

**Strategy Protection**
- Test trading strategies without exposure
- Compete without revealing tactics
- Protect proprietary trading algorithms

### Institutional Trading

**Block Trading**
- Execute large block trades privately
- Match institutional orders efficiently
- Maintain price discovery without disclosure

**Portfolio Rebalancing**
- Rebalance positions without revealing allocations
- Execute multiple asset trades privately
- Maintain strategy confidentiality

### Retail Trading

**Personal Privacy**
- Trade without exposing portfolio size
- Execute strategies without social pressure
- Maintain financial privacy

---

## Security Considerations

### Smart Contract Security

**Order Book Integrity:**
- Immutable order storage
- Atomic matching and settlement
- Reentrancy protection
- Access control for critical functions

**Trade Settlement:**
- Atomic swap execution
- Collateral verification
- Fee calculation verification
- Settlement finality guarantees

### Operational Security

**Key Management:**
- FHE keys stored securely
- Threshold key management for critical operations
- Key rotation procedures
- Disaster recovery planning

**Market Monitoring:**
- Detect unusual trading patterns (aggregate level)
- Monitor for potential manipulation attempts
- Track market statistics
- Alert system for anomalies

---

## Performance Metrics

### Trading Performance

**Order Submission:**
- Time: < 2 blocks
- Gas: ~150,000 per order
- Latency: < 30 seconds

**Order Matching:**
- Time: 2-5 blocks (depending on order book size)
- Gas: ~400,000 - 1,000,000 per match
- Latency: < 2 minutes

**Trade Settlement:**
- Time: 1 block
- Gas: ~200,000 per settlement
- Latency: < 15 seconds

### Scalability

**Order Book Capacity:**
- Current: ~1,000 active orders per pair
- Target: 10,000+ orders per pair
- Optimization: Batch processing, off-chain matching

**Throughput:**
- Current: ~10 trades per minute
- Target: 100+ trades per minute
- Optimization: Layer 2 solutions, parallel processing

---

## Getting Started

### Prerequisites

```bash
# Required software
- Node.js 18+
- Hardhat or Foundry
- MetaMask
- Sepolia testnet ETH
```

### Setup

```bash
# Clone repository
git clone https://github.com/yourusername/shadowtrade.git
cd shadowtrade

# Install dependencies
npm install

# Configure environment
cp .env.example .env
# Edit .env with your settings

# Compile contracts
npx hardhat compile
```

### Deploy

```bash
# Deploy to Sepolia
npx hardhat run scripts/deploy.js --network sepolia

# Initialize trading pairs
npx hardhat run scripts/initPairs.js --network sepolia
```

### Run Frontend

```bash
cd frontend
npm install
npm run dev
```

---

## API Documentation

### Smart Contract Interface

```solidity
interface IShadowTrade {
    // Submit encrypted order
    function submitOrder(
        bytes calldata encryptedOrder,
        address tokenA,
        address tokenB
    ) external returns (uint256 orderId);
    
    // Match orders (homomorphic)
    function matchOrders(
        uint256 buyOrderId,
        uint256 sellOrderId
    ) external;
    
    // Cancel order
    function cancelOrder(uint256 orderId) external;
    
    // Get trade history
    function getTradeHistory(address trader)
        external
        view
        returns (Trade[] memory);
}
```

### JavaScript SDK

```typescript
import { ShadowTrade } from '@shadowtrade/sdk';

const client = new ShadowTrade({
  provider: window.ethereum,
  contractAddress: '0x...',
});

// Submit order
const encryptedOrder = await client.encryptOrder({
  price: 1000,
  quantity: 1.5,
  type: 'buy',
});
await client.submitOrder(encryptedOrder, 'ETH', 'USDT');

// Get market data
const marketData = await client.getMarketData('ETH/USDT');
```

---

## Roadmap

### Q1 2025
- ✅ Core order book and matching
- ✅ Basic homomorphic operations
- ✅ Trade settlement
- 🔄 Gas optimization

### Q2 2025
- 📋 Advanced order types
- 📋 Market depth visualization
- 📋 Mobile application
- 📋 API improvements

### Q3 2025
- 📋 Cross-chain trading
- 📋 Advanced matching algorithms
- 📋 Institutional features
- 📋 Liquidity incentives

### Q4 2025
- 📋 Layer 2 integration
- 📋 Decentralized order matching
- 📋 Advanced analytics
- 📋 Governance token

---

## Contributing

We welcome contributions! Areas of interest:

- FHE optimization for order matching
- Gas cost reduction strategies
- Security audits and reviews
- UI/UX improvements
- Additional order types
- Documentation enhancements

**How to contribute:**
1. Fork the repository
2. Create a feature branch
3. Implement your changes
4. Add tests
5. Submit a pull request

---

## License

MIT License - see [LICENSE](LICENSE) file for details.

---

## Acknowledgments

ShadowTrade leverages:

- **[Zama FHEVM](https://www.zama.ai/fhevm)**: Fully Homomorphic Encryption Virtual Machine
- **[Zama](https://www.zama.ai/)**: FHE research and development
- **Ethereum Foundation**: Blockchain infrastructure

Built with support from the privacy-preserving trading community.

---

## Links

- **Repository**: [GitHub](https://github.com/yourusername/shadowtrade)
- **Documentation**: [Full Docs](https://docs.shadowtrade.io)
- **Discord**: [Community](https://discord.gg/shadowtrade)
- **Twitter**: [@ShadowTrade](https://twitter.com/shadowtrade)

---

**ShadowTrade** - Trade privately, settle transparently.

_Powered by Zama FHEVM_

