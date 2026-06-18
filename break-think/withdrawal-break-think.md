# Withdrawal Break Think

## _initiateBridgeERC20(...)

```text
INVARIANT
Tokens must be burned on L2 before the L1 release message is finalized.
Burn amount must equal release amount.
The L2 token must map to the correct L1 token.

CONSEQUENCES
This may lead to releasing tokens on L1 without burning on L2.
The bridge may release more tokens than were actually burned.
A user may receive the wrong token.
```

## burn(...)

```text
INVARIANT
Only the authorized bridge can burn bridge tokens.
Burned amount must be the amount encoded for withdrawal.
The user must have enough balance to burn the requested amount.

CONSEQUENCES
An attacker may change the _from parameter to steal the user's funds.
This may lead to withdrawing more tokens than were burned.
```

## finalizeBridgeERC20(...)

```text
INVARIANT
Only an authentic withdrawal message can release L1 tokens.
Released amount must equal burned amount.
Released token must be the correct L1 token for the burned L2 token.

CONSEQUENCES
An attacker may create a spoofed message that leads to a fake release.
The released amount may be greater than the burned amount.
A user may receive the wrong token instead of the intended token.
```
