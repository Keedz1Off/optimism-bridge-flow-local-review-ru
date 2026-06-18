# Deposit Break Think

## _initiateBridgeERC20(...)

```text
ИНВАРИАНТ
Transfer должен успешно выполниться до того, как bridge отправит message.
Message amount должен равняться количеству токенов, которое реально было transferred.
L1 token должен соответствовать правильному L2 token.

ПОСЛЕДСТВИЯ
Это может привести к отправке message без lock токенов на L1.
Message amount может быть больше, чем реально transferred amount.
Пользователь может получить неправильный token.
```

## relayMessage(...)

```text
ИНВАРИАНТ
Только trusted messenger может relay messages.
Каждое message должно исполниться только один раз.
Validation должна происходить до execution.

ПОСЛЕДСТВИЯ
Функцию сможет вызвать кто угодно.
Это может привести к replay или double execution.
Validation будет происходить после execution, что делает validation бесполезной.
```

## finalizeBridgeERC20(...)

```text
ИНВАРИАНТ
Только messenger может вызвать finalizeBridgeERC20(...).
Original cross-chain sender должен быть trusted counterpart bridge.

ПОСЛЕДСТВИЯ
Это может привести к ghost mint.
Это может привести к spoofed message execution.
```
