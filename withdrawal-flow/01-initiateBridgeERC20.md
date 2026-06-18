# Function Review: _initiateBridgeERC20(...)

## Что делает

Для withdrawal на L2 эта функция burn токены и создает L2 -> L1 message.

## Main Invariants

```text
1. Tokens must be burned on L2 before the L1 release message is finalized.
2. Burn amount must equal release amount.
3. The L2 token must map to the correct L1 token.
```

## Важная мысль

```text
L2 burned amount = L1 released amount
```
