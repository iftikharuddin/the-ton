# the-ton

### 01 - Digging into the YouTube Playlist, Building Defi on Ton

- Building Defi on Ton | 01 | Wallet, Testnet Chain, Blockchain Interactions in Ton
https://youtu.be/U1mKD7wDQCo?list=PLSBBx0ivqFa7M1YvkCNyEQMoGkkbm4yFy
- Building Defi on Ton | 02 | First Smart Contract on Tact, Build, Deploy on Ton Testnet Chain
https://youtu.be/J7ZF3KWxX_8?list=PLSBBx0ivqFa7M1YvkCNyEQMoGkkbm4yFy

### 02 - https://tact-by-example.org/

#### Receivers and Messages

In TON, users interact with contracts by sending them messages. Different contracts can only communicate with each other by sending each other messages.

Since users actually use wallet contracts, messages from users are actually messages coming from just another contract.

Sending a message to a contract costs gas and is processed in the course of a transaction. The transaction executes when validators add the transaction to a new block. This can take a few seconds. Messages are also able to change the contract's persistent state.

#### Receivers

When designing your contract, make a list of every operation that your contract supports, then, define a message for each operation, and finally, implement a handler for each message containing the logic of what to do when it arrives.

Contract methods named receive() are the handlers that process each incoming message type. Tact will automatically route every incoming message to the correct receiver listening for it according to its type. A message is only handled by one receiver.

Messages are defined using the message keyword. They can carry input arguments. Notice that for integers, you must define the encoding size, just like in state variables. When somebody sends the message, they serialize it over the wire.

