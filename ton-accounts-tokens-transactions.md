## Ton accounts, tokens and transactions

### TON Accounts

TON Accounts ARE Smart Contracts. Unlike Ethereum where address comes directly from public key. In TON -> Address comes from smart contract code + your public key

- TON Address Security:
  Unlike ETH, TON addresses are smart contract addresses that need deployment
  
  Same account = multiple formats (bounceable/non-bounceable)
  
  Address confusion attacks possible! Always validate format + workchain
  
  Complexity = New attack surface! 
  
- TON accounts have 4 different address formats (mainnet/testnet × bounceable/non-bounceable) with the same middle account_id but different prefixes, creating potential confusion and security risks.

### Wallet Contracts

TON Wallet Security: Every transaction needs 3 checks! 
1️ Seqno match (replay protection)
2️ Subwallet ID match (isolation)
3️ Valid signature (authentication)

Same private key = multiple addresses via different subwallets

Complex security = more attack vectors! 