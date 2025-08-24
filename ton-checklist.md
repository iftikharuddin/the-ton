## Checklist of issues I have noticed during auditing TON protocols

refs: 

https://coinmarketcap.com/community/articles/63f5e0b394b07d2baed4aa49/

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
  
- TON Tip: Always use end_parse() when reading data! 
  ❌ Silent bugs from incomplete parsing
  ✅ Immediate error if data left unread
  Catches: forgotten fields, format changes, malicious data
  1 line of code = hours of debugging saved! 
  
- Professional TON Auditor: "First step = map ALL message chains, assume each can fail" 
Most common bugs: missing auth checks, no bounce handling, partial execution risks
TON's async model = fundamentally different security challenges vs ETH
Message flow analysis is critical

- It is necessary to pay attention to the processing of external messages (from the Internet) by the function `recv_external` (in smart contracts in the FunC language): whether the function is applied accept_message()only after all the necessary checks. This is necessary to prevent gas-sucking attacks, since after the call, accept_message()the contract pays for all further operations. External messages have no context (for example, sender, value), 10,000 gas units are given as a credit for processing, which is enough to check the signature and accept the message. Of course, everything depends on the contract design, but if possible, it makes sense to write a contract without the ability to accept external messages. The function recv_externalis one of the entry points that needs to be checked several times.
  
- Never assume messages will arrive in the order you sent them! 
- Your contract should work correctly regardless of message delivery order (add proper code)
- Always test with randomized message ordering to catch race conditions
- TON Message Ordering: UNPREDICTABLE delivery! 
  Contract A sends: msg1→B, msg2→C
  Arrival order: Could be C then B, or B then C! 
  Security risk: State races, price manipulation, auth bypasses
  Solution: Carry-value pattern, sequence numbers, timeouts! 🛡️