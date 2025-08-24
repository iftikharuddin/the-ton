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

### Jetton Tokens

Jetton is TON's token standard where each user gets their own personal wallet contract (unlike ERC-20 where everyone shares one contract).

TON Jettons: Distributed token architecture! 

- Unlike ERC-20 (1 contract), each user gets personal jetton vault
- Transfer = 4 contracts = 4 potential failure points
- Benefits: Scalability 
- Risks: Partial execution, gas manipulation, state desync 

### Transactions

Key transaction features include:

1. Asynchronous Nature: trxs in TON are not completed in a single call; they may require a series of calls to multiple different smart contracts through message passing. Due to different routing in the shard chains, TON cannot guarantee the order of message delivery between multiple smart contracts.

2. Fees: The asynchronous nature also brings the issue of unpredictable fee consumption. Therefore, when initiating a transaction, the wallet usually sends extra tokens as fees. If the called contract has a good fee handling mechanism, the remaining fees will be refunded to the user’s wallet. Users might observe their wallet tokens suddenly decreasing and then increasing again after a few minutes, which explains this behavior.

3. Bounce: Bounce is an error-handling mechanism for contracts. If the called contract does not exist or throws an error, and if the transaction is set to be bounceable, a bounced message will be returned to the calling contract. For example, if a user initiates a transfer and the call process fails, a bounce message is needed so that the user’s wallet contract can restore its balance. Almost all internal messages sent between smart contracts should be bounceable, which means their “bounce” bit should be set.

Never assume messages arrive in the order you sent them - design your contracts to work correctly regardless of message ordering!
