# _initiateBridgeERC20(...)

## Invariants

1. Tokens must be burned on L2 before the L1 release message is finalized.

2. Burn amount must equal release amount.

3. The L2 token must map to the correct L1 token.

## Consequences

1. Если tokens are not burned on L2, это может привести к releasing tokens on L1 without burning on L2.

2. Bridge может release more tokens than were actually burned.

3. Token on L1 does not match the token on L2.


# burn()

## Invariant


1. Only the authorized bridge can burn bridge tokens.

2. Burned amount must be the amount encoded for withdrawal.

3. The user must have enough balance to burn the requested amount.

## Consequences
1. Attacker может вызвать function и изменить `_from` parameter, чтобы steal user's funds.
 <img width="970" height="523" alt="image" src="https://github.com/user-attachments/assets/41ee12ca-57fc-48e1-8398-b041773f2fee" />

2. Это может привести к withdrawing more tokens than were burned.

<img width="840" height="232" alt="image" src="https://github.com/user-attachments/assets/e66fe9aa-02ea-4454-9f7e-76d9f8ae5cef" />



# finalizeBridgeERC20(...)

## Invariants

1. Only an authentic withdrawal message can release L1 tokens.

2. Released amount must equal burned amount.

3. Released token must be the correct L1 token for the burned L2 token.

## Consequences

1. Attacker может создать spoofed message, который приведет к fake release.

2. Bridge может release more tokens than were actually burned.
   <img width="922" height="261" alt="image" src="https://github.com/user-attachments/assets/414285db-10ee-4760-9bd3-97773a55e011" />
3. User может receive the wrong token.
   <img width="783" height="339" alt="image" src="https://github.com/user-attachments/assets/f93a08b9-ba18-4670-b247-b803ad6795b6" />
