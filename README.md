# Permissioned Pools Hook for Uniswap V4

A focused Uniswap V4 hook that creates KYC/AML-compliant liquidity pools using Kraken Verify attestations on Ink Protocol. Perfect for RWAs, institutions, and regulated DeFi.

## 🎯 Core Value Proposition

**beforeSwap verification** - Every swap checks Kraken Verify status. Large value prop for:
- **RWA Trading**: Tokenized securities, real estate, commodities
- **Institutional Liquidity**: Banks and funds providing compliant liquidity  
- **Regulated Markets**: CBDC pools, institutional stablecoins

## 🏗 How It Works

```
User Swap → Uniswap V4 → beforeSwap Hook → Query Kraken Verify → Allow/Reject
```

Simple gating: Only Kraken-verified users can swap in permissioned pools.

## 🚀 Quick Start

### Dependencies
```bash
forge install Uniswap/v4-core
forge install foundry-rs/forge-std
```

### Deploy
```bash
# Deploy to Ink Protocol
forge script script/Deploy.s.sol --rpc-url https://rpc-gel.inkonchain.com --broadcast

# Create permissioned pool
forge script script/CreatePool.s.sol --rpc-url https://rpc-gel.inkonchain.com --broadcast
```

## 📝 Core Implementation

### PermissionedPoolHook.sol
```solidity
contract PermissionedPoolHook is BaseHook {
    // Ink Protocol EAS address
    IEAS constant EAS = IEAS(0x4200000000000000000000000000000000000021);
    
    function beforeSwap(
        address sender,
        PoolKey calldata key,
        IPoolManager.SwapParams calldata,
        bytes calldata
    ) external override returns (bytes4, BeforeSwapDelta, uint24) {
        if (poolSettings[key.toId()].requiresVerification) {
            require(isKrakenVerified(sender), "Not verified");
        }
        return (BaseHook.beforeSwap.selector, BeforeSwapDeltaLibrary.ZERO_DELTA, 0);
    }
    
    function isKrakenVerified(address user) public view returns (bool) {
        // Check EAS for Kraken Verify attestation
        // Implementation queries Ink Protocol's EAS deployment
    }
}
```

## 💼 Use Cases

### 1. Tokenized Securities Pool
```solidity
// Only accredited investors can trade
PermissionSettings memory settings = PermissionSettings({
    requiresVerification: true,
    verificationLevel: ACCREDITED_INVESTOR
});
```

### 2. Institutional Stablecoin Pool  
```solidity
// Bank-to-bank CBDC trading
PermissionSettings memory settings = PermissionSettings({
    requiresVerification: true,
    verificationLevel: INSTITUTIONAL_KYC
});
```

### 3. RWA Marketplace
```solidity
// Real estate token trading
PermissionSettings memory settings = PermissionSettings({
    requiresVerification: true,
    verificationLevel: QUALIFIED_INVESTOR
});
```

## ⚙️ Configuration

### Environment Setup
```bash
# Ink Protocol RPC
RPC_URL=https://rpc-gel.inkonchain.com

# EAS Contract (Standard OP Stack address)
EAS_ADDRESS=0x4200000000000000000000000000000000000021

# Kraken Verify Schema ID
KRAKEN_VERIFY_SCHEMA=0x... # Get from Kraken docs
```

### Pool Creation
```solidity
// Initialize permissioned pool
bytes memory hookData = abi.encode(true); // requiresVerification = true

poolManager.initialize(
    PoolKey({
        currency0: USDC,
        currency1: tokenizedAsset,
        fee: 3000,
        tickSpacing: 60,
        hooks: IHooks(address(permissionedHook))
    }),
    SQRT_PRICE_1_1,
    hookData
);
```

## 🧪 Testing

```bash
# Run tests
forge test

# Gas report
forge test --gas-report

# Coverage
forge coverage
```

## 🔧 Project Structure

```
src/
├── PermissionedPoolHook.sol      # Main hook with beforeSwap logic
├── interfaces/IEAS.sol           # Minimal EAS interface
└── libraries/VerificationCache.sol # Gas optimization

script/
├── Deploy.s.sol                  # Hook deployment
└── CreatePool.s.sol             # Pool creation examples

test/
├── PermissionedPoolHook.t.sol    # Core tests
└── integration/SwapFlow.t.sol    # End-to-end tests
```

## 🔒 Security Features

- **Gas Optimized**: Caches verification status (1-hour default)
- **Admin Controls**: Pool creators can toggle permissions
- **Emergency Pause**: Circuit breaker for security incidents
- **Minimal Attack Surface**: Only reads from EAS, no complex logic

## 📚 Integration Guide

### Frontend Integration
```typescript
// Check if user needs verification
const isVerified = await checkKrakenVerification(userAddress);

if (!isVerified) {
    // Redirect to Kraken Verify
    window.location.href = "https://kraken.com/verify";
}
```

### Smart Contract Integration
```solidity
// Check before attempting swap
bool canSwap = permissionedHook.isKrakenVerified(msg.sender);
require(canSwap, "Complete Kraken verification first");
```

## 🌐 Resources

- [Ink Protocol Docs](https://docs.inkonchain.com)
- [Kraken Verify Integration](https://docs.inkonchain.com/build-on-ink/kraken-verify)
- [Uniswap V4 Hook Guide](https://docs.uniswap.org/contracts/v4/overview)

## 📞 Support

- [GitHub Issues](https://github.com/your-org/permissioned-pools-hook/issues)
- [Ink Protocol Discord](https://discord.gg/inkonchain)
- Email: builders@inkonchain.com

---

**Built for institutional-grade DeFi on Ink Protocol** 🚀