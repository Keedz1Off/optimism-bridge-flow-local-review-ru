# Deposit Break Think

## _initiateBridgeERC20(...)

```text
INVARIANT
Transfer must succeed before the bridge sends the message.
Message amount must equal the amount actually transferred.
The L1 token must match the correct L2 token.

CONSEQUENCES
This may lead to sending a message without locking tokens on L1.
The message amount may be greater than the amount actually transferred.
A user may receive the wrong token.
```

## relayMessage(...)

```text
INVARIANT
Only the trusted messenger can relay messages.
Each message must execute only once.
Validation must happen before execution.

CONSEQUENCES
The function can be called by anyone.
It may lead to replay or double execution.
Validation happens after execution, which makes validation useless.
```

## finalizeBridgeERC20(...)

```text
INVARIANT
Only the messenger can call finalizeBridgeERC20(...).
The original cross-chain sender must be the trusted counterpart bridge.

CONSEQUENCES
This may lead to ghost mint.
This may lead to spoofed message execution.
```
