## TON Blockchain Architecture - Security Critical Details

### **Message-Driven Execution Model**

TON uses an **Actor Model** - every contract is an isolated actor that only communicates via messages. This creates unique attack vectors:

```tact
// CRITICAL: Messages are processed asynchronously and independently
receive(msg: Transfer) {
    // This function execution is ATOMIC
    // But multiple messages can be in flight simultaneously
    self.balance = self.balance - msg.amount;  // RACE CONDITION RISK
}
```

**Key Security Implications:**
1. **No Global State Locks** - Multiple messages can modify state concurrently
2. **Message Ordering Not Guaranteed** - Later messages can arrive first
3. **Partial Failures** - One message can fail while others succeed

### **Address Generation Algorithm - Attack Surface**

```typescript
// TON address = hash(stateInit)
// stateInit = { code: Cell, data: Cell }
const address = contractAddress(workchain, { code, data });
```

**Security Vulnerabilities:**
1. **Code Collision**: Same code + data = same address
2. **Address Prediction**: Attackers can predict future contract addresses
3. **Address Squatting**: Deploy at predicted address before legitimate deployment

```tact
// VULNERABLE: Predictable initialization
contract VulnerableContract {
    id: Int;
    
    init(id: Int) {
        self.id = id;  // If id is predictable, address is predictable
    }
}

// SECURE: Unpredictable initialization
contract SecureContract {
    id: Int;
    
    init(id: Int, salt: Int) {
        self.id = id;
        // salt should be random/unpredictable
    }
}
```

## Precise Tact Language Vulnerabilities

### **1. Integer Arithmetic - Exact Behavior**

Tact uses **modular arithmetic** (wrapping on overflow):

```tact
// CRITICAL: These operations wrap silently
contract IntegerVulns {
    counter: Int as uint256;
    
    receive(msg: Add) {
        // VULNERABLE: Can wrap around
        self.counter = self.counter + msg.amount;
        // If counter = 2^256-1 and amount = 1, result = 0
    }
    
    receive(msg: SafeAdd) {
        // SECURE: Explicit overflow check
        let newValue: Int = self.counter + msg.amount;
        require(newValue >= self.counter, "Overflow");
        self.counter = newValue;
    }
}
```

**Attack Vector**: Send `amount = (2^256 - current_counter)` to reset counter to 0.

### **2. Gas Economics - Precise Cost Model**

```tact
// Gas costs in TON:
// - Storage: ~1 gas per bit per second
// - Computation: varies by opcode
// - Message sending: base cost + per-bit cost

receive(msg: ProcessArray) {
    // VULNERABLE: Unbounded gas consumption
    repeat(msg.iterations) {
        // Each iteration costs gas
        // Can exceed block gas limit
    }
}

receive(msg: SafeProcess) {
    // SECURE: Bounded operations
    require(msg.iterations <= 1000, "Too many iterations");
    let processed: Int = 0;
    repeat(msg.iterations) {
        processed = processed + 1;
        // Process in batches if needed
    }
}
```

### **3. Message Bounce Mechanics - Critical Details**

```tact
// Bounce behavior:
// 1. Message fails -> automatically bounced back
// 2. Bounced message has different structure
// 3. Can create bounce loops

message Transfer {
    queryId: Int as uint64;
    amount: Int as coins;
}

message TransferBounced {
    queryId: Int as uint64;  // Same queryId
    // Original message data available
}

receive(msg: Transfer) {
    // Send to external contract
    send(SendParameters{
        to: msg.destination,
        value: msg.amount,
        bounce: true  // CRITICAL: Enable bouncing
    });
}

receive(bounced: TransferBounced) {
    // VULNERABLE: No validation
    // Could bounce back and forth infinitely
    send(SendParameters{
        to: someAddress,
        value: ton("1"),
        bounce: true  // Creates bounce loop risk
    });
}
```

**Attack Pattern**: Create contracts that always fail and bounce messages back, causing infinite loops.

## Workchain Security Model - Precise Details

```tact
// Workchain -1 (Masterchain):
// - Validators only
// - Higher fees
// - Special privileges

// Workchain 0 (Basechain):
// - Regular contracts
// - Lower fees
// - Most DeFi lives here

receive(msg: CrossChain) {
    // VULNERABLE: No workchain validation
    let dest: Address = msg.destination;
    // dest could be in masterchain
    
    // SECURE: Validate workchain
    require(dest.workchain == 0, "Invalid workchain");
}
```

## Sequence Number (Seqno) - Exact Implementation

```tact
// Wallet contract seqno mechanism:
contract Wallet {
    seqno: Int as uint32;
    
    receive(msg: ExternalMessage) {
        // CRITICAL: Must verify signature AND seqno
        let expectedSeqno: Int = self.seqno;
        require(msg.seqno == expectedSeqno, "Invalid seqno");
        
        // Process message
        // ...
        
        // CRITICAL: Must increment AFTER processing
        self.seqno = self.seqno + 1;
    }
}

// VULNERABILITY: Seqno overflow
// After 2^32 transactions, seqno wraps to 0
// Replay attacks become possible
```

**Attack Vector**: 
1. Wait for seqno to wrap around (very long-term attack)
2. Replay old signed transactions with seqno = 0, 1, 2, etc.

## TON Cell Structure - Low-Level Security

```tact
// Cells have precise structure:
// - Max 1023 bits of data
// - Max 4 references to other cells
// - Depth limit (usually 512)

// VULNERABLE: Cell construction without limits
fun buildLargeCell(data: Slice): Cell {
    let builder: Builder = beginCell();
    // RISK: No size validation
    while(!data.empty()) {
        builder = builder.storeSlice(data.loadBits(1023));
    }
    return builder.endCell();
}

// SECURE: Validate cell constraints
fun buildSafeCell(data: Slice): Cell {
    let builder: Builder = beginCell();
    let bitsStored: Int = 0;
    
    while(!data.empty() && bitsStored < 1023) {
        let chunk: Slice = data.loadBits(min(64, 1023 - bitsStored));
        builder = builder.storeSlice(chunk);
        bitsStored = bitsStored + chunk.bits();
    }
    return builder.endCell();
}
```

## Precise Attack Vectors for TON DeFi

### **1. Reentrancy via Message Ordering**

```tact
contract VulnerableBank {
    balances: map<Address, Int>;
    
    receive(msg: Withdraw) {
        let balance: Int = self.balances.get(sender()) ?: 0;
        require(balance >= msg.amount, "Insufficient balance");
        
        // VULNERABLE: External call before state update
        send(SendParameters{
            to: sender(),
            value: msg.amount,
            bounce: false
        });
        
        // PROBLEM: If receiver sends another Withdraw message,
        // it will be processed before this line executes
        self.balances.set(sender(), balance - msg.amount);
    }
}

// ATTACK:
// 1. Call withdraw(100) with balance = 100
// 2. In receive TON callback, immediately call withdraw(100) again
// 3. Second call sees balance = 100 (unchanged)
// 4. Both withdrawals succeed, draining 200 TON
```

### **2. Front-Running via Message Timing**

```tact
contract VulnerableAMM {
    price: Int;
    
    receive(msg: UpdatePrice) {
        // VULNERABLE: Price update is visible before execution
        self.price = msg.newPrice;
    }
    
    receive(msg: Swap) {
        // Uses current price
        let output: Int = (msg.input * self.price) / 1000000;
        // ... execute swap
    }
}

// ATTACK:
// 1. Monitor mempool for UpdatePrice messages
// 2. Send Swap message with higher gas before UpdatePrice executes
// 3. Swap at old price, profit from price difference
```

### **3. Address Collision for Upgrades**

```tact
// VULNERABLE: Predictable upgrade addresses
contract UpgradeableProxy {
    implementation: Address;
    
    receive(msg: Upgrade) {
        // Calculate new implementation address
        let newCode: Cell = msg.code;
        let newData: Cell = beginCell().endCell();
        let newAddress: Address = contractAddress(0, {code: newCode, data: newData});
        
        // PROBLEM: newAddress is predictable
        // Attacker can deploy malicious contract at that address first
        self.implementation = newAddress;
    }
}
```

## Precise Testing Methodology

### **1. Message Race Condition Testing**

```typescript
// Test concurrent message processing
async function testRaceCondition() {
    const contract = /* deployed contract */;
    
    // Send multiple messages simultaneously
    const promises = Array(10).fill(0).map(() => 
        contract.send({
            value: toNano("1"),
            body: Transfer.pack({ amount: 100 })
        })
    );
    
    // Check if final state is consistent
    await Promise.all(promises);
    
    const finalBalance = await contract.getBalance();
    // Should be initialBalance - (10 * 100)
    // If not, race condition exists
}
```

### **2. Gas Limit Testing**

```typescript
async function testGasLimits() {
    // Find gas limit boundary
    let iterations = 1;
    while (iterations < 10000) {
        try {
            await contract.send({
                body: ProcessArray.pack({ iterations })
            });
            iterations *= 2;
        } catch (error) {
            // Found gas limit
            console.log(`Gas limit hit at ${iterations} iterations`);
            break;
        }
    }
}
```

### **3. Bounce Loop Detection**

```typescript
async function testBounceLoop() {
    // Deploy two contracts that bounce to each other
    const contractA = await deploy(BouncerContract, { target: contractB.address });
    const contractB = await deploy(BouncerContract, { target: contractA.address });
    
    // Trigger bounce loop
    await contractA.send({
        value: toNano("1"),
        body: StartBounce.pack({})
    });
    
    // Monitor for excessive transactions
    // Should detect infinite bouncing
}
```

