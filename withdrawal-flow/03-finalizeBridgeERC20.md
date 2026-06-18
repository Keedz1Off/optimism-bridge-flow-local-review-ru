# Function Review: finalizeBridgeERC20(...)

## Что делает

Для L2 -> L1 withdrawal эта функция release escrowed L1 tokens.

## Main Invariants

```text
1. Only an authentic withdrawal message can release L1 tokens.
2. Released amount must equal burned amount.
3. Released token must be the correct L1 token for the burned L2 token.
```
