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