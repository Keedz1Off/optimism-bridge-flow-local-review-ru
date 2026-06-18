# Function Review: burn(...) branch

## 1. Function Code

```solidity
if (_isOptimismMintableERC20(_localToken)) {
    require(
        _isCorrectTokenPair(_localToken, _remoteToken),
        "StandardBridge: wrong remote token for Optimism Mintable ERC20 local token"
    );

    IOptimismMintableERC20(_localToken).burn(_from, _amount);
}

interface IOptimismMintableERC20 is IERC165 {
    function remoteToken() external view returns (address);

    function bridge() external returns (address);

    function mint(address _to, uint256 _amount) external;

    function burn(address _from, uint256 _amount) external;
}
```

Note:

```text
Original source: `ethereum-optimism/optimism/packages/contracts-bedrock/src/universal/StandardBridge.sol` and `interfaces/universal/IOptimismMintableERC20.sol`.
```

## 2. Что делает эта функция

Burn branch removes OptimismMintableERC20 tokens from a user's balance.

В withdrawal flow это source-chain state transition.

Простыми словами:

```text
Before the user receives tokens on L1, their L2 tokens must be removed.
```

## 3. Important Parts Explained

### Bridge Authorization

```solidity
external onlyBridge
```

Only the bridge should be able to call this burn function.

Security meaning:

```text
Random users should not be able to burn other users' tokens.
```

### Burn State Change

```solidity
_burn(from, amount);
```

Это decreases:

- the user's token balance
- total token supply

Security meaning:

```text
This is the economic proof for the later L1 release.
```

## 4. Invariants

### Main Invariant 1

```text
Only the authorized bridge can burn bridge tokens.
```

### Main Invariant 2

```text
Burned amount must be the amount encoded for withdrawal.
```

### Main Invariant 3

```text
The user must have enough balance to burn the requested amount.
```

## 5. Additional Invariants

### Additional Invariant 1

```text
The burn must reduce the user's balance and total supply by the same amount.
```

### Additional Invariant 2

```text
The burn amount should be greater than zero.
```
