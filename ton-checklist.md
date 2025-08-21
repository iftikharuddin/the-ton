## Checklist of issues I have noticed during auditing TON protocols

- **Partial execution:** ETH reverts everything if one step fails, TON processes each message independently. In TON, tokens can vanish during transfers due to partial execution. Solution is to add bounce flag and allow recovery.