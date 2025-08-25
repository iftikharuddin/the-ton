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

#### Textual Messages

Most of the messages we saw in the previous example were defined with the message keyword. They are considered binary messages. This means that when somebody wants to send them, they serialize them into bits and bytes of binary data.

The disadvantage with binary messages is that they're not human readable.

#### Hardware wallets and blind signing

When working with dangerous contracts that handle a lot of money, users are encouraged to use hardware wallets like Ledger. Hardware wallets cannot decode binary messages to confirm to the user what they're actually signing.

Tact supports textual messages for this reason, since they're human readable and can easily be confirmed with users, eliminating phishing risks.

Textual messages are limited because they cannot contain arguments. Future versions of Tact will add this functionality.

#### Using the comment field

If you've ever made a transfer using a TON wallet, you probably noticed that you can add a comment (also known as a memo or a tag). This is how textual messages are sent.

Receivers for textual messages just define the string that they expect. Tact automatically does string matching and calls the matching receiver when a comment message arrives.

#### Structs
The order of fields does not matter. Unlike other languages, Tact does not have any padding between fields.

Structs allow you to combine multiple primitives together in a more semantic way. They're a great tool to make your code more readable.

Structs can define complex data types that contain multiple fields of different types. They can also be nested.

Structs can also include both default fields and optional fields. This can be quite useful when you have many fields but don't want to keep respecifying them.

Structs are also useful as return values from getters or other internal functions. They effectively allow a single getter to return multiple return values.

#### Structs vs. messages
Structs and messages are almost identical with the only difference that messages have a 32-bit header containing their unique numeric id. This allows messages to be used with receivers since the contract can tell different types of messages apart based on this id.


#### Message Sender
Every incoming message is sent from some contract that has an address.

You can query the address of the message sender by calling `sender()`. Alternatively, the address is also available through context().sender.

The sender during execution of the `init()` method of the contract is the address who deployed the contract.

#### Authenticating messages
The main way to authenticate an incoming message, particularly for priviliges actions, is to verify the sender. This field is secure and impossible to fake.

#### Throwing Errors
Processing an incoming message in a transaction isn't always successful. The contract may encounter some error and fail.

This can be due to an explicit decision of the contract author, usually by writing a require() on a condition that isn't met, or this may happen implicitly due to some computation error in run-time, like a math overflow.

When an error is thrown, the transaction reverts. This means that all persistent state changes that took place during this transaction, even those that happened before the error was thrown, are all reverted and return to their original values.

#### Notifying the sender about the error
How would the sender of the incoming message know that the message they had sent was rejected due to an error?

All communication is via messages, so naturally the sender should receive a message about the error. TON will actually return the original message back to the sender and mark it as bounced - just like a snail mail letter that couldn't be delivered.

#### Receiving TON Coins
Every contract has a TON coin balance. This balance is used to pay ongoing rent for storage and should not run out otherwise the contract may be deleted. You can store extra coins in the balance for any purpose.

Every incoming message normally carries some TON coin value sent by the sender. This value is used to pay gas for handling this message. Unused excess will stay in the contract balance. If the value doesn't cover the gas cost, the transaction will revert.

You can query the contract balance with myBalance() - note that the value is in nano-tons (like cents, just with 9 decimals). The balance already contains the incoming message value.

#### Refunding senders

If the transaction reverts, unused excess value will be sent back to sender on the bounced message.

You can also refund the excess if the transaction succeeds by sending it back using self.reply() in a response message. This is the best way to guarantee senders are only paying for the exact gas that their message consumed.




