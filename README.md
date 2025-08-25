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


#### Messages Between Contracts
Different contracts can communicate with each other only by sending messages

TON uses Actor Model - contracts are independent actors

One contract sends message, another receives it and change state or act on it

Contracts can't directly read each other's state - only communicate via messages e.g
````
// ❌ This doesn't work:
let counterValue = Counter.get_value();  // Can't call getters!

// ✅ Must send message instead:
send("query" to Counter);  // Send message, wait for reply
````

The Rule: Original sender pays for EVERYTHING!
````
User sends "Reach 5" with 1.0 TON gas
├── Message 1: BulkAdder processes (uses 0.01 TON)
├── Message 2: Counter processes (uses 0.01 TON) 
├── Message 3: BulkAdder processes (uses 0.01 TON)
└── ... 13 messages total (uses 0.13 TON)
Final: User paid 0.13 TON, gets 0.87 TON refunded
````

How Gas Forwarding Works in the example contract? ref https://tact-by-example.org/03-messages-between-contracts
````
send(SendParameters{
    to: msg.counter,
    value: 0,  // Don't send specific amount
    mode: SendRemainingValue,  // Forward ALL remaining gas!
    body: "query".asComment()
});
````
Translation: "Send this message and give it all my remaining gas"

#### Sending TON Coins
This example contract (ref https://tact-by-example.org/03-send-coins) allows to withdraw TON coins from its balance. Notice that only the deployer is permitted to do that, otherwise this money could be stolen.

The withdrawn funds are sent as value on an outgoing message to the sender. It's a good idea to set the bounce flag explicitly to true (although this also the default), so if the outgoing message fails for any reason, the money would return to the contract.

Contracts need to have a non-zero balance so they can pay storage costs occasionally, otherwise they may get deleted. This contract can make sure you always leave 0.01 TON which is enough to store 1 KB of state for 2.5 years.


#### The intricate math
`myBalance()` is the contract balance including the value for gas sent on the incoming message. myBalance() - context().value is the balance without the value for gas sent on the incoming message.

Send mode SendRemainingValue will add to the outgoing value any excess left from the incoming message after all gas costs are deducted from it.

Send mode SendRemainingBalance will ignore the outgoing value and send the entire balance of the contract. Note that this will not leave any balance for storage costs so the contract may be deleted.

in simple words when this mode is set -> SendRemainingBalance (Mode: Send Everything)
````
mode: SendRemainingBalance  
// Ignores 'value' field, sends ENTIRE contract balance
````
Use case: "Empty the piggy bank completely"

and if SendRemainingValue (Mode: Send Amount + Leftover Gas)
````
mode: SendRemainingValue
// Sends specified 'value' + any leftover gas from the transaction
````
Use case: "Send specific amount, plus give back any change"

#### Loops
Tact does not support traditional for loops, but its loop statements are equivalent and can easily implement the same things. Also note that Tact does not support break and continue statements in loops like some languages.

The repeat loop statement input number must fit within an int32, otherwise an exception will be thrown.

The condition of the while and until loop statements can be any boolean expression.

Smart contracts consume gas for execution. The amount of gas is proportional to the number of iterations. The last example iterates too many times and reverts due to an out of gas exception.

#### Functions
To make your code more readable and promote code reuse, you're encouraged to divide it into functions.

Functions in Tact start with the fun keyword. Functions can receive multiple input arguments and can optionally return a single output value. You can return a struct if you want to return multiple values.

Global static functions are defined outside the scope of contracts. You can call them from anywhere, but they can't access the contract or any of the contract state variables.

Contract methods are functions that are defined inside the scope of a contract. You can call them only from other contract methods like receivers and getters. They can access the contract's state variables.

#### Optionals
Optionals are variables or struct fields that can be null and don't necessarily hold a value. They are useful to reduce state size when the variable isn't necessarily used.

You can make any variable optional by adding ? after its type.

Optional variables that are not defined hold the null value. You cannot access them without checking for null first.

If you're certain an optional variable is not null, append to the end of its name !! to access its value. Trying to access the value without !! will result in a compilation error.

#### Maps
Maps are a dictionary type that can hold an arbitrary number of items, each under a different key.

The keys in maps can either be an Int type or an Address type.

You can check if a key is found in the map by calling the get() method. This will return null if the key is missing or the value if the key is found. Replace the value under a key by calling the set() method.

Integers in maps stored in state currently use the largest integer size (257-bit). Future versions of Tact will let you optimize the encoding size.

#### Limit the number of items
Maps are designed to hold a limited number of items. Only use a map if you know the upper bound of items that it may hold. It's also a good idea to write a test to add the maximum number of elements to the map and see how gas behaves under stress.

If the number of items is unbounded and can potentially grow to billions, you'll need to architect your contract differently. 

#### Arrays
You can implement simple arrays in Tact by using the map type.

To create an array, define a map with an Int type as the key. This key will represent the index in the array. Additionally, include a variable to keep track of the number of items in the array.

The example contract (https://tact-by-example.org/04-arrays) records the last five timestamps when the timer message was received. These timestamps are stored in a cyclical array, implemented as a map.

#### Current Time
Other blockchains: Use block numbers to track time

TON: Uses real clock time with now()

Why: TON has multiple chains running at different speeds, so block numbers don't work reliably.
````
// ❌ Expensive storage
timestamp: Int = now();

// ✅ Cheap storage (good until 2106)
timestamp: Int as uint32 = now();
````
#### TON's Architecture:
Masterchain: Controls everything, slower blocks

Workchains: Process transactions, faster blocks

Shards: Even smaller pieces, variable speed

Block speeds vary in TON → Block numbers are unreliable for timing

Time is universal → Same everywhere WHILE Multiple chains = multiple block numbers, so using block number is problem

#### Decimal Point
All numbers in Tact are integers. Floating point types are not used in smart contracts because they're unpredictable.

Arithmetics with dollars, for example, requires 2 decimal places. How can we represent the number 1.25 if we can only work with integers? The answer is to work with cents. So 1.25 becomes 125. We just remember that the two lowest digits are coming after the decimal point.

In the same way, working with TON coins has 9 decimal places instead of 2. So the amount 1.25 TON is actually the number 1250000000 - we call these nano-tons instead of cents.

#### Calculating interest

This example (https://tact-by-example.org/04-decimal-point) calculates the earned interest over a deposit of 500 TON coins. The yearly interest rate in the example is 3.25%.

Since we can't hold the number 3.25 we will use thousandth of a percent as unit (percent-mille). So 3.25% becomes 3250 (3.25 * 1000).

On withdraw, to calculate earned interest, we multiply the amount by the fraction of a year that passed (duration in seconds divided by total seconds in a year) and then by the interest rate divided by 100,000 (100% in percent-mille, meaning 100 * 1000).

Info: Notice that total is returned in nano-tons.

#### Decimals in TON
In TON smart contracts can't use decimals like 1.25 or 3.14 because they're unpredictable on different computers.

Solution: Use smaller units instead!

````
Real amount: $1.25
Computer stores: 125 (cents)
Rule: Divide by 100 to get real dollars
````
Translation: Instead of storing $1.25, store 125 cents

Now How TON Handles Coins?

Instead of storing 1.25 TON, store 1,250,000,000 nano-tons. Uses 9 decimals

````
Real amount: 1.25 TON
Computer stores: 1,250,000,000 (nano-tons)
Rule: Divide by 1,000,000,000 to get real TON
````

#### The TON Decimal System
TON's 9 Decimal Places:

````
1 TON = 1,000,000,000 nano-tons
0.1 TON = 100,000,000 nano-tons
0.01 TON = 10,000,000 nano-tons
0.001 TON = 1,000,000 nano-tons
````
Easy Conversion
````
let amount = ton("1.25");     // Easy way to write 1.25 TON
dump(amount);                 // Shows: 1250000000 (nano-tons)
````

Think of it: Like cents, but with 9 zeros instead of 2!

- Use nano-tons for TON amounts (×1,000,000,000)
- Use percent-mille for rates (×1,000)
- Integer math only - no floating point
- Divide back when showing to users



