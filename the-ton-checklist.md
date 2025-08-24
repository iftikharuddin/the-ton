Things to look for during auditing a protocol build on top of TON (works both for FunC and Tact lang)

## Message Flow & Partial Execution

| Check | Description | Risk Level | What to Verify |
|-------|-------------|------------|-----------------|
| **Message Flow Mapping** | Map all message chains and assume each can fail | 🔴 Critical | Document complete message flows, identify failure points |
| **Partial Execution Protection** | Verify bounce handlers prevent token loss | 🔴 Critical | Every token operation has bounce handling, `total_supply` consistency |
| **Race Condition Validation** | Re-validate state at each message step | 🟡 High | State checks repeated in multi-step operations |
| **Message Ordering Independence** | Contract works regardless of message delivery order | 🟡 High | Test with randomized message ordering |
| **Carry-Value Pattern** | Values carried with messages, not queried | 🟡 High | No balance queries, actual tokens transferred with messages |

## Authorization & Access Control

| Check | Description | Risk Level | What to Verify |
|-------|-------------|------------|-----------------|
| **Entry Point Authorization** | All entry points have proper access controls | 🔴 Critical | Every `receive()` function validates sender |
| **Address Calculation** | Correct address derivation for authorization | 🔴 Critical | `contractAddress(initOf ...)` or manual calculation accuracy |
| **Admin Functions Protection** | Critical functions properly restricted | 🔴 Critical | Multi-sig, timelock, or DAO governance for admin actions |
| **Centralization Risks** | Evaluate admin powers and decentralization | 🟡 High | Document admin capabilities, consider governance |

## Gas Management & Economics

| Check | Description | Risk Level | What to Verify |
|-------|-------------|------------|-----------------|
| **Gas Budget Planning** | Gas allocated across entire message cascade | 🔴 Critical | Total gas calculation includes all message steps |
| **Excess Gas Return** | Unused gas returned to sender | 🟡 High | Proper gas refund logic, avoid contract balance accumulation |
| **Storage Fee Reserves** | Contract reserves gas for storage costs | 🟡 High | Storage fee calculation, contract sustainability |
| **Loop Gas Limits** | Bounded iterations to prevent DoS | 🟡 High | Maximum iteration limits, gas cost per iteration |
| **Message Mode Correctness** | Send modes configured properly | 🟡 High | No conflicting flags, appropriate mode selection |

## Storage & Data Management

| Check | Description | Risk Level | What to Verify |
|-------|-------------|------------|-----------------|
| **Variable Ordering (FunC)** | Correct load/save data order | 🔴 Critical | `load_data()` and `save_data()` variable sequence match |
| **Storage Migration Safety** | Upgrade compatibility maintained | 🔴 Critical | Storage structure changes don't corrupt data |
| **Nested Storage Usage** | Efficient storage organization | 🟢 Medium | Only load needed data groups, avoid full storage parsing |
| **Data Growth Limits** | Prevent unlimited storage growth | 🟡 High | Cleanup mechanisms, mapping size controls |
| **Cell Limits Compliance** | Respect 1023 bits and 4 references | 🟡 High | Cell overflow/underflow protection |

## Randomness & Cryptography

| Check | Description | Risk Level | What to Verify |
|-------|-------------|------------|-----------------|
| **Secure Randomness** | Proper random number generation | 🟡 High | `randomize_lt()` in FunC, `randomInt()` in Tact |
| **Validator Manipulation** | Consider validator influence on randomness | 🟡 High | High-stakes scenarios use external oracles/commit-reveal |
| **Replay Protection** | Prevent transaction replay | 🔴 Critical | Sequence numbers (`seqno`) or nonces implemented |

## Contract Upgrades

| Check | Description | Risk Level | What to Verify |
|-------|-------------|------------|-----------------|
| **Upgrade Authorization** | Proper governance for upgrades | 🔴 Critical | Multi-sig, timelock, or DAO control |
| **Storage Migration** | Safe data migration during upgrades | 🔴 Critical | Test all storage state transitions |
| **Immutable Core Functions** | Critical functions cannot be changed | 🟡 High | Consider permanent upgrade disable mechanisms |

## Code Quality & Common Bugs

| Check | Description | Risk Level | What to Verify |
|-------|-------------|------------|-----------------|
| **Complete Parsing** | Use `end_parse()` for all data reading | 🟡 High | All `slice` parsing ends with `end_parse()` |
| **Mathematical Precision** | Correct operation order in calculations | 🟡 High | Multiplication before division to avoid precision loss |
| **Variable Shadowing** | No local variables overwrite storage | 🟡 High | Clear naming conventions, no shadowing in FunC |
| **Magic Numbers** | Constants used instead of magic numbers | 🟢 Medium | Named constants for clarity and maintenance |
| **Exit Code Usage** | Proper exit code ranges | 🟢 Medium | Custom codes > 127, avoid reserved codes 0-127 |

## TON-Specific Vulnerabilities

| Check | Description | Risk Level | What to Verify |
|-------|-------------|------------|-----------------|
| **External Message Security** | `recv_external` validates before `accept_message()` | 🔴 Critical | Validation uses only 10k free gas, no gas-sucking attacks |
| **Bounce Message Handling** | All bounced messages properly handled | 🔴 Critical | State recovery on bounced operations |
| **Address Format Validation** | Proper address type handling | 🟡 High | Bounceable vs non-bounceable validation |
| **Fee Scavenging Protection** | Prevent malicious gas consumption | 🟡 High | Gas estimation limits, suspicious transaction detection |

## External Dependencies

| Check | Description | Risk Level | What to Verify |
|-------|-------------|------------|-----------------|
| **Oracle Reliability** | External data source security | 🟡 High | Oracle manipulation resistance, multiple sources |
| **Cross-Contract Calls** | Safe interaction with external contracts | 🟡 High | Proper authorization, bounce handling for external calls |
| **Third-Party Code** | Avoid executing untrusted external code | 🔴 Critical | No dynamic code execution, `CATCH` limitations |

## Documentation & Standards

| Check | Description | Risk Level | What to Verify |
|-------|-------------|------------|-----------------|
| **Code Documentation** | Clear documentation of functionality | 🟢 Medium | Function purposes, complex logic explained |
| **Standards Compliance** | Adherence to TON standards (TEP-74, etc.) | 🟡 High | Standard interfaces implemented correctly |
| **Independent Audit** | Professional security review conducted | 🔴 Critical | Third-party audit from reputable firm |

## Language-Specific Checks

### FunC Specific

| Check | Description | Risk Level | What to Verify |
|-------|-------------|------------|-----------------|
| **Function Modifiers** | `impure` modifier on state-changing functions | 🟡 High | All state modifications marked `impure` |
| **Manual Storage Management** | Correct `c4` register handling | 🔴 Critical | Complete storage overwrite safety |
| **TVM Assembly Usage** | Minimize and carefully review assembly code | 🟡 High | Assembly operations audited by experts |

### Tact Specific

| Check | Description | Risk Level | What to Verify |
|-------|-------------|------------|-----------------|
| **Optional Type Usage** | Proper null handling or remove optional | 🟢 Medium | Optional variables actually use null functionality |
| **Double Initialization** | Initialize variables in `init()` only | 🟡 High | No declaration-time initialization |
| **Trait Variable Modification** | Respect trait variable update rules | 🟡 High | No direct modification of trait variables |
| **Native Function Usage** | Prefer high-level over native functions | 🟡 High | Use `send()` over `nativeSendMessage()` |
