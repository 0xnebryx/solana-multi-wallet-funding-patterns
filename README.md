# solana-multi-wallet-funding-patterns

> Patterns for funding many Solana wallets without correlating them through a single graph hop.

Notes on hop-disperse, micro-fund, and rent-recovery patterns. Useful when you're operating a fleet of wallets and want the funding graph to look organic instead of a star topology with a single funder at the center.

This is operational-privacy material for legitimate use cases (treasury management, multi-sig hot wallet rotation, market-maker fleets). It is not advice on evading sanctions, OFAC compliance, or KYC obligations.

---

## Core ideas

### 1. Star topology is the obvious anti-pattern

```
       funder
      /   |   \
   w1    w2    w3
```

On-chain analytics tools (BubbleMaps, Arkham, Nansen) automatically cluster this. The 3 leaf wallets get tagged as "associated" within seconds of confirmation.

### 2. Hop disperse

Insert intermediate "hop" wallets that hold funds briefly, then forward to the final wallet:

```
   funder
     |
   hop_1   ←  random jitter delay (1–60s)
     |
   final_wallet
```

Multiplying hops or adding fan-out at each hop further weakens the correlation signal. Diminishing returns past 2 hops.

### 3. Random delay jitter

A funder that always batches and dispatches at the same second is a strong fingerprint. Add per-hop delay drawn from a distribution (e.g. uniform 800ms–8000ms) so the on-chain timing pattern isn't a giveaway.

### 4. Variable amounts

Distributing exact same amounts to N wallets creates a uniform signature. Add per-wallet jitter (±5–10%) so amounts don't look templated.

### 5. Micro-fund for gas closure

When a wallet has 0 SOL but holds a token bag you want to sweep, the wallet can't pay TX fees to close its own ATA. Pattern: transfer the minimum (~0.0008 SOL) from a designated micro-funder, then run the close → sweep sequence.

```ts
// pseudo-code
const FEE_MARGIN = 0.0008 * LAMPORTS_PER_SOL;
if (await connection.getBalance(orphan) < FEE_MARGIN) {
  await transferLamports(microFunder, orphan, FEE_MARGIN);
}
await closeAccount(orphan, ata);
await transfer(orphan.sol, sweepDestination);
```

### 6. Rent-recovery cycle

Every ATA holds ~0.002 SOL in rent-exempt minimum. Address Lookup Tables hold ~0.005 SOL each. On a fleet of 100 wallets that have all interacted with 5 tokens each, that's 1+ SOL in recoverable rent. The closure has a deactivation cooldown for ALTs (~150 slots) and an empty-balance precondition for ATAs.

---

## Footguns

- **Don't use this for sanctions evasion.** This is operational hygiene, not laundering. Legitimate users (treasuries, market makers, white-hat researchers) shouldn't have anything to hide and shouldn't structure for that goal.
- **Hop wallets that hold for too long** become identifiable patterns themselves. Aim for transit-only.
- **All hops on the same RPC endpoint** at the same exact slot creates a correlation via timing metadata your provider keeps. Mitigation: stagger with delays + use diverse RPC endpoints.

## Reading list

- [BubbleMaps documentation](https://docs.bubblemaps.io/)
- [Solana ATA rent mechanics](https://solana.com/docs/core/accounts#rent)
- [Solana Address Lookup Tables](https://solana.com/docs/advanced/lookup-tables)

## License

MIT
