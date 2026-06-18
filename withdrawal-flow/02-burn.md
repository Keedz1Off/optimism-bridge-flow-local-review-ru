# Function Review: burn(...) branch

## Код

```solidity
IOptimismMintableERC20(_localToken).burn(_from, _amount);
```

## Что делает

Burn branch удаляет OptimismMintableERC20 tokens с баланса пользователя перед withdrawal.

## Main Invariants

```text
1. Only the authorized bridge can burn bridge tokens.
2. Burned amount must be the amount encoded for withdrawal.
3. The user must have enough balance to burn the requested amount.
```
