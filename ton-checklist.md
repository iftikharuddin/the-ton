## Checklist of issues I have noticed during auditing TON protocols

- **Partial execution:** ETH reverts everything if one step fails, TON processes each message independently. In TON, tokens can vanish during transfers due to partial execution. Solution is to add bounce flag and allow recovery.
- TON Security Tip: Do ALL validation at entry point, not during message cascade.
  Bounced messages help but have limits - they need gas & don't chain. Plan for failures! 
- State can change between your message steps because other users' transactions execute in parallel.
e.g 
````
Step 1: ✅ User has 1000 tokens
Step 2: 💀 Parallel tx drains 900 tokens (another user or attacker withdraw first)
Step 3: ❌ Your code assumes 1000 still there
Always re-validate! 🔄 -> this is solution
````
- Use a Carry-Value Pattern: Don't ask "how much money?" - just take it and return what you don't need! 💰
  Query → outdated info by the time you use it ❌
  Carry-value → move actual tokens with message ✅
  No race conditions! 🔒
  
- TON Security (Return Value Instead of Rejecting): When receiving tokens, NEVER use throw_unless() - tokens will be lost forever! 💸
  ❌ Reject → tokens vanish
  ✅ Return to sender_address
  ❌ NEVER return to token contract address (vulnerability!)
  Always give back what you can't use! 🔄
  
- Think of gas as a budget that must be carefully allocated across ALL your messages, not just the first one.
- TON Gas Tip: Unlike ETH, you must manually budget gas across your entire message cascade! ⛽
  ❌ Give 1 TON to first message → others fail
  ✅ Split gas: 0.3 + 0.3 + 0.3 + 0.1 = 1.0 TON
  Add new messages? Recalculate everything! 🔄
  
- TON Gas Tip: Return excess gas or your contract becomes a black hole! 🕳️
  ✅ Auto-forward (mode 64) for simple flows
  ❌ Manual calculation when contract pays storage/fees
  Always reserve gas for storage fees - or your contract dies! 
  Keep some gas for yourself to pay storage fees
  
- TON Storage: Organize like a tidy room! Use Nested Storage
  ❌ Image 2 ( which was wrong approach): Dump all data every time (messy drawer)
  ✅ Image 1 ( correct and optimised approach ): Load only needed groups (organized drawers)
  Group related data, load selectively = cheaper gas & cleaner code! 