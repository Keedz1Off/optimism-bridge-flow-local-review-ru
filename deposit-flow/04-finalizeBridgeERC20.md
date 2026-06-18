# Function Review: finalizeBridgeERC20(...)

## 1. Function Code

```solidity
/// @notice Finalizes an ERC20 bridge on this chain. Can only be triggered by the other
///         StandardBridge contract on the remote chain.
/// @param _localToken  Address of the ERC20 on this chain.
/// @param _remoteToken Address of the corresponding token on the remote chain.
/// @param _from        Address of the sender.
/// @param _to          Address of the receiver.
/// @param _amount      Amount of the ERC20 being bridged.
/// @param _extraData   Extra data to be sent with the transaction. Note that the recipient will
///                     not be triggered with this data, but it will be emitted and can be used
///                     to identify the transaction.
function finalizeBridgeERC20(
    address _localToken,
    address _remoteToken,
    address _from,
    address _to,
    uint256 _amount,
    bytes calldata _extraData
)
    public
    onlyOtherBridge
{
    require(paused() == false, "StandardBridge: paused");
    if (_isOptimismMintableERC20(_localToken)) {
        require(
            _isCorrectTokenPair(_localToken, _remoteToken),
            "StandardBridge: wrong remote token for Optimism Mintable ERC20 local token"
        );

        IOptimismMintableERC20(_localToken).mint(_to, _amount);
    } else {
        deposits[_localToken][_remoteToken] = deposits[_localToken][_remoteToken] - _amount;
        IERC20(_localToken).safeTransfer(_to, _amount);
    }

    // Emit the correct events. By default this will be ERC20BridgeFinalized, but child
    // contracts may override this function in order to emit legacy events as well.
    _emitERC20BridgeFinalized(_localToken, _remoteToken, _from, _to, _amount, _extraData);
}
```

Note:

```text
Original source: `ethereum-optimism/optimism/packages/contracts-bedrock/src/universal/StandardBridge.sol`.
```

## 2. Что делает эта функция

`finalizeBridgeERC20(...)` completes the deposit on the destination chain.

Простыми словами:

```text
The L2 bridge receives a valid cross-chain message and mints tokens to the user.
```

Эта функция critical, потому что creates destination-chain value.

## 3. Important Parts Explained

### Messenger Check

```solidity
modifier onlyOtherBridge() {
    require(
        msg.sender == address(messenger) && messenger.xDomainMessageSender() == address(otherBridge),
        "StandardBridge: function can only be called from the other bridge"
    );
    _;
}
```

Direct caller должен быть local messenger, а cross-chain sender должен быть counterpart bridge.

Security meaning:

```text
Users should not be able to call finalizeBridgeERC20(...) directly.
```

### Token Pair Check

```solidity
require(
    _isCorrectTokenPair(_localToken, _remoteToken),
    "StandardBridge: wrong remote token for Optimism Mintable ERC20 local token"
);
```

Это проверяет, что local mintable token maps to the correct remote token.

Security meaning:

```text
The token pair must be correct before minting.
```

Cross-chain sender check handled by `onlyOtherBridge`.

### Mint

```solidity
IOptimismMintableERC20(_localToken).mint(_to, _amount);
```

Это mints destination-chain tokens to the recipient.

Security meaning:

```text
Minting must be backed by a real lock / burn on the source chain.
```

## 4. Invariants

### Main Invariant 1

```text
Only an authentic message from the counterpart bridge can call finalizeBridgeERC20(...).
```

### Main Invariant 2

```text
The original cross-chain sender must be the trusted counterpart bridge.
```

### Main Invariant 3

```text
Mint amount must equal the real source-chain locked amount.
```

## 5. Additional Invariants

### Additional Invariant 1

```text
The minted token must be the correct local token for the remote token.
```

### Additional Invariant 2

```text
The recipient must be the recipient encoded in the authentic bridge message.
```

### Additional Invariant 3

```text
The function must not accept direct user calls.
```

### Additional Invariant 4

```text
The finalize calldata must not change token, recipient, or amount during execution.
```
