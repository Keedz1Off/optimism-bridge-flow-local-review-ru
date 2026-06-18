# Function Review: _initiateBridgeERC20(...)

## 1. Function Code

```solidity
/// @notice Sends ERC20 tokens to a receiver's address on the other chain.
/// @param _localToken  Address of the ERC20 on this chain.
/// @param _remoteToken Address of the corresponding token on the remote chain.
/// @param _to          Address of the receiver.
/// @param _amount      Amount of local tokens to deposit.
/// @param _minGasLimit Minimum amount of gas that the bridge can be relayed with.
/// @param _extraData   Extra data to be sent with the transaction. Note that the recipient will
///                     not be triggered with this data, but it will be emitted and can be used
///                     to identify the transaction.
function _initiateBridgeERC20(
    address _localToken,
    address _remoteToken,
    address _from,
    address _to,
    uint256 _amount,
    uint32 _minGasLimit,
    bytes memory _extraData
)
    internal
{
    require(msg.value == 0, "StandardBridge: cannot send value");

    if (_isOptimismMintableERC20(_localToken)) {
        require(
            _isCorrectTokenPair(_localToken, _remoteToken),
            "StandardBridge: wrong remote token for Optimism Mintable ERC20 local token"
        );

        IOptimismMintableERC20(_localToken).burn(_from, _amount);
    } else {
        IERC20(_localToken).safeTransferFrom(_from, address(this), _amount);
        deposits[_localToken][_remoteToken] = deposits[_localToken][_remoteToken] + _amount;
    }

    // Emit the correct events. By default this will be ERC20BridgeInitiated, but child
    // contracts may override this function in order to emit legacy events as well.
    _emitERC20BridgeInitiated(_localToken, _remoteToken, _from, _to, _amount, _extraData);

    messenger.sendMessage({
        _target: address(otherBridge),
        _message: abi.encodeWithSelector(
            this.finalizeBridgeERC20.selector,
            // Because this call will be executed on the remote chain, we reverse the order of
            // the remote and local token addresses relative to their order in the
            // finalizeBridgeERC20 function.
            _remoteToken,
            _localToken,
            _from,
            _to,
            _amount,
            _extraData
        ),
        _minGasLimit: _minGasLimit
    });
}
```

Note:

```text
Original source: `ethereum-optimism/optimism/packages/contracts-bedrock/src/universal/StandardBridge.sol`.
```

## 2. Что делает эта функция

`_initiateBridgeERC20(...)` содержит core L2 -> L1 ERC20 withdrawal logic when used on L2.

Простыми словами:

```text
The user burns tokens on L2.
The bridge creates a message.
The message later tells the L1 bridge to release tokens.
```

## 3. Important Parts Explained

### Burn on L2

```solidity
IOptimismMintableERC20(_localToken).burn(_from, _amount);
```

Это removes tokens from the user's L2 balance.

Security meaning:

```text
L1 release must be backed by a real L2 burn.
```

### Withdrawal Message

```solidity
_message: abi.encodeWithSelector(
    this.finalizeBridgeERC20.selector,
    _remoteToken,
    _localToken,
    _from,
    _to,
    _amount,
    _extraData
)
```

Это creates the message, который позже release tokens on L1.

Security meaning:

```text
The release message must match the real burn.
```

## 4. Invariants

### Main Invariant 1

```text
Tokens must be burned on L2 before the L1 release message is finalized.
```

### Main Invariant 2

```text
Burn amount must equal release amount.
```

### Main Invariant 3

```text
The L2 token must map to the correct L1 token.
```

## 5. Additional Invariants

### Additional Invariant 1

```text
The withdrawal recipient encoded in the message must be the intended recipient.
```

### Additional Invariant 2

```text
The withdrawal message must be created only after the burn step.
```

### Additional Invariant 3

```text
The message must be sent to the trusted counterpart bridge.
```

### Additional Invariant 4

```text
The withdrawal amount should be greater than zero.
```
