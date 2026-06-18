# _initiateBridgeERC20(...)

## Invariants 
1.Transfer must succeed before the bridge sends the message.

2.Message amount must equal the amount actually transferred.

3.The L1 token must match to the correct L2 token.

## Consequences

1. Это может привести к отправке message без locking tokens на L1.

2. Message amount может быть больше, чем amount actually transferred.

<img width="1258" height="430" alt="image" src="https://github.com/user-attachments/assets/a4e08003-fc77-4c08-ac14-8c41ec83d99a" />


3. Token на L1 не соответствует token на L2.

<img width="1024" height="517" alt="image" src="https://github.com/user-attachments/assets/6394f956-c7d3-4b4b-afba-f76ff5d89a1f" />


# relayMessage()

## Invariants

1. Only the trusted messenger can relay messages.

2. Each message must execute only once.

3. Validation must happen before execution.

## Consequences

1. Функцию сможет вызвать anyone. Attacker может изменить `_target` или `_sender` parameters.

2. Это может привести к replay или double execution.

3. Validation происходит после execution, что делает validation useless.
 



# finalizeBridgeERC20()

## Invariants
1. Only the messenger can call finalizeBridgeERC20(...).

2. The original cross-chain sender must be the trusted counterpart bridge.

## Consequences
1. Это может быть вызвано anyone, что может привести к ghost mint (mint without lock).

2. В худшем случае эту функцию может вызвать другой spoofed bridge instead of original L1 bridge.
