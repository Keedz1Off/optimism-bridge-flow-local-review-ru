# Withdrawal Break Think

## _initiateBridgeERC20(...)

```text
ИНВАРИАНТ
Токены должны быть сожжены на L2 до финализации release message на L1.
Burn amount должен равняться release amount.
L2 token должен соответствовать правильному L1 token.

ПОСЛЕДСТВИЯ
Это может привести к выдаче токенов на L1 без burn на L2.
Bridge может выдать больше токенов, чем было реально сожжено.
Пользователь может получить неправильный token.
```

## burn(...)

```text
ИНВАРИАНТ
Только authorized bridge может burn bridge tokens.
Burned amount должен быть amount, encoded для withdrawal.
У пользователя должен быть достаточный balance для burn requested amount.

ПОСЛЕДСТВИЯ
Attacker может изменить параметр _from и украсть funds пользователя.
Это может привести к withdrawal большего количества токенов, чем было burned.
```

## finalizeBridgeERC20(...)

```text
ИНВАРИАНТ
Только подлинное withdrawal message может выдать L1 tokens.
Released amount должен равняться burned amount.
Released token должен быть правильным L1 token для burned L2 token.

ПОСЛЕДСТВИЯ
Attacker может создать spoofed message, что приведет к fake release.
Released amount может быть больше, чем burned amount.
Пользователь может получить неправильный token вместо intended token.
```
