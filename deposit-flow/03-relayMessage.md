# Function Review: relayMessage(...)

## Что делает

`relayMessage(...)` валидирует и исполняет cross-chain message на destination chain.

Функция проверяет message hash, replay status, target safety и потом вызывает target contract.

## Main Invariants

```text
1. Only the trusted messenger can relay messages.
2. Each message must execute only once.
3. Validation must happen before execution.
4. The target must be the intended destination contract.
```

## Важная мысль

Если replay protection сломан, одно сообщение может быть исполнено дважды.
