# 🕵️ Solana Hard Disperse

> Break wallet connections and fund wallets invisibly - bypass BubbleMaps and on-chain analytics

[![Privacy](https://img.shields.io/badge/Privacy-Maximum-black?style=for-the-badge)](https://obsidianbundler.com)
[![BubbleMaps](https://img.shields.io/badge/BubbleMaps-Invisible-green?style=for-the-badge)](https://obsidianbundler.com)
[![Untraceable](https://img.shields.io/badge/Chain-Broken-red?style=for-the-badge)](https://obsidianbundler.com)

## 🔗 The Problem: Connected Wallets

When you send SOL from wallet A to wallets B, C, and D, those wallets are permanently linked on the blockchain.

**Anyone can see:**
- All your wallets
- Your total holdings
- Your trading patterns
- When you're accumulating or selling

Tools like BubbleMaps, Arkham, and others visualize these connections.

## ❌ Normal Transfer (VISIBLE)

```
Main Wallet
    │
    ├──────────► Wallet 1 ◄──── CONNECTED!
    ├──────────► Wallet 2 ◄──── CONNECTED!
    └──────────► Wallet 3 ◄──── CONNECTED!

On BubbleMaps: All wallets clustered together
Verdict: OBVIOUSLY THE SAME PERSON
```

## ✅ Hard Disperse (INVISIBLE)

```
Main Wallet
    │
    ├──► Temp A ──► Wallet 1 (Temp A closed)
    ├──► Temp B ──► Wallet 2 (Temp B closed)
    └──► Temp C ──► Wallet 3 (Temp C closed)

On BubbleMaps: No visible connection
Verdict: APPEAR AS SEPARATE WALLETS
```

## How It Works

```typescript
import { HardDisperse } from '@obsidian/disperse';

const disperse = new HardDisperse({
  rpcUrl: process.env.RPC_URL
});

await disperse.execute({
  source: mainWallet,
  targets: [wallet1, wallet2, wallet3],
  amountPerWallet: 0.5, // SOL each
  
  // Options
  randomDelay: { min: 1000, max: 5000 }, // ms between transfers
  randomizeAmounts: true, // ±10% variation
  closeIntermediates: true // Recover rent from temp wallets
});
```

## The Algorithm

```
┌─────────────────────────────────────────────────────────┐
│                   HARD DISPERSE FLOW                     │
├─────────────────────────────────────────────────────────┤
│                                                          │
│  Step 1: Create intermediate wallet                     │
│  ┌──────────┐                                           │
│  │   Main   │──► Create Intermediate A                  │
│  │  Wallet  │                                           │
│  └──────────┘                                           │
│                                                          │
│  Step 2: Fund intermediate                              │
│  ┌──────────┐    ┌──────────────┐                      │
│  │   Main   │───►│Intermediate A│                      │
│  │  Wallet  │    │ (0.505 SOL)  │  ◄─ amount + rent    │
│  └──────────┘    └──────────────┘                      │
│                                                          │
│  Step 3: Random delay (1-5 seconds)                     │
│  ────────────────────────────────────                   │
│                                                          │
│  Step 4: Transfer to target                             │
│  ┌──────────────┐    ┌──────────┐                      │
│  │Intermediate A│───►│ Wallet 1 │                      │
│  │              │    │ (0.5 SOL)│                      │
│  └──────────────┘    └──────────┘                      │
│                                                          │
│  Step 5: Close intermediate (recover rent)              │
│  ┌──────────────┐                                       │
│  │Intermediate A│ ◄─ CLOSED (rent returned)            │
│  │  (deleted)   │                                       │
│  └──────────────┘                                       │
│                                                          │
│  Step 6: Repeat for each target wallet                  │
│                                                          │
│  RESULT: No on-chain link between Main and Wallet 1!    │
│                                                          │
└─────────────────────────────────────────────────────────┘
```

## Configuration

```typescript
interface HardDisperseConfig {
  // Required
  source: Keypair;           // Funding wallet
  targets: PublicKey[];      // Destination wallets
  amountPerWallet: number;   // SOL per wallet
  
  // Optional - Stealth features
  randomDelay?: {
    min: number;             // Min ms between transfers
    max: number;             // Max ms between transfers
  };
  randomizeAmounts?: boolean; // Vary amounts ±10%
  closeIntermediates?: boolean; // Close temp wallets
  
  // Optional - Advanced
  useMultipleHops?: number;   // 1-3 intermediate hops
  differentRpcs?: boolean;    // Use different RPCs per transfer
}
```

## Advanced: Multi-Hop Disperse

For maximum privacy, use multiple intermediate hops:

```typescript
// 3-hop disperse
await disperse.multiHop({
  source: mainWallet,
  target: targetWallet,
  amount: 1.0,
  hops: 3
});

// Flow: Main → Temp1 → Temp2 → Temp3 → Target
// All temps closed after use
// Nearly impossible to trace
```

## Comparison: Easy vs Hard Disperse

```typescript
// EASY DISPERSE (Fast but visible)
// Use when you don't need privacy
await disperse.easy({
  source: mainWallet,
  targets: wallets,
  amounts: [0.5, 0.5, 0.5]
});
// Direct transfers - fast but wallets are linked

// HARD DISPERSE (Slower but invisible)
// Use for privacy-critical operations
await disperse.hard({
  source: mainWallet,
  targets: wallets,
  amounts: [0.5, 0.5, 0.5]
});
// Routed through intermediates - slower but untraceable
```

## Use Cases

### Token Launch Privacy

```typescript
// Generate fresh wallets
const launchWallets = await generateWallets(20);

// Fund invisibly - these won't cluster on BubbleMaps
await disperse.hard({
  source: mainWallet,
  targets: launchWallets,
  amountPerWallet: 0.3,
  randomizeAmounts: true
});

// Launch token with distributed "holder" base
await bundleLauncher.launch({
  devWallet: launchWallets[0],
  sniperWallets: launchWallets.slice(1)
});
```

### Stealth Accumulation

```typescript
// Fund accumulation wallets invisibly
await disperse.hard({
  source: mainWallet,
  targets: accumulationWallets,
  amountPerWallet: 0.2
});

// Buy from each wallet separately
// Looks like 10 different buyers, not one whale
for (const wallet of accumulationWallets) {
  await buy(wallet, token, randomAmount(0.1, 0.2));
  await sleep(randomDelay(30, 120)); // seconds
}
```

### Private Selling

```typescript
// Transfer token to dispersed wallets
// Then sell from each separately
for (const wallet of dispersedWallets) {
  await sell(wallet, token, await getBalance(wallet, token));
  await sleep(randomDelay(60, 300));
}
// Looks like normal holder activity, not whale dump
```

## Security Notes

```
✅ DO:
• Use hard disperse for privacy-critical operations
• Close intermediates to recover rent
• Add random delays between transfers
• Vary amounts slightly

❌ DON'T:
• Use same intermediate wallet twice
• Fund all targets in rapid succession
• Use exact same amounts for each
• Skip delays in time-sensitive situations
```

## Production Solution

For hard disperse with UI and automation:

### 👉 [Obsidian Platform](https://obsidianbundler.com)

- ✅ One-click hard disperse
- ✅ Automatic intermediate management
- ✅ Random delays built-in
- ✅ Amount randomization
- ✅ Multi-hop support
- ✅ Free tier available

## Resources

- 💬 [Telegram](https://t.me/obsidianbundler)
- 🐦 [Twitter](https://x.com/obsidianbundler)
- 🌐 [Platform](https://obsidianbundler.com)

## Disclaimer

Educational code. Use responsibly and in compliance with applicable laws.

---

⭐ **Star this repo** for more privacy tools!

🕵️ **Go invisible:** [obsidianbundler.com](https://obsidianbundler.com)
