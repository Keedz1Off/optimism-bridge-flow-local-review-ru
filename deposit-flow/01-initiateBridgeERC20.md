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

`_initiateBridgeERC20(...)` содержит core L1 -> L2 ERC20 deposit logic.

Простыми словами:

```text
The user sends tokens to the L1 bridge.
The bridge creates a message.
The message later tells the L2 bridge to mint / credit tokens.
```

Эта функция соединяет:

- user token transfer
- L1 lock / escrow
- calldata creation
- cross-chain message sending
- later L2 minting

## 3. Important Parts Explained

### Token Transfer

```solidity
IERC20(_localToken).safeTransferFrom(_from, address(this), _amount);
deposits[_localToken][_remoteToken] = deposits[_localToken][_remoteToken] + _amount;
```

Это transfers canonical tokens into the bridge и увеличивает bridge deposit accounting.

Meaning:

```text
_from -> bridge contract
```

Security meaning:

```text
This is the source-chain accounting step.
```

Bridge не должен отправлять mint message, если L1 token transfer реально не произошел.

### Message Encoding

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

Это builds calldata, которая будет executed на destination chain.

Важные значения внутри message:

- `_remoteToken`
- `_localToken`
- `_from`
- `_to`
- `_amount`
- `_extraData`

Security meaning:

```text
The message must represent what really happened on L1.
```

Если encoded `amount` неправильный, L2 может mint неправильный amount.

### Send Message

```solidity
messenger.sendMessage({
    _target: address(otherBridge),
    _message: abi.encodeWithSelector(
        this.finalizeBridgeERC20.selector,
        _remoteToken,
        _localToken,
        _from,
        _to,
        _amount,
        _extraData
    ),
    _minGasLimit: _minGasLimit
});
```

Это sends encoded message to the counterpart bridge.

Security meaning:

```text
The target must be the trusted bridge on the other chain.
```

## 4. Invariants

### Main Invariant 1

```text
Transfer must succeed before the bridge sends the message.
```

### Main Invariant 2

```text
Message amount must equal the amount actually transferred.
```

### Main Invariant 3

```text
The L1 token must map to the correct L2 token.
```

## 5. Additional Invariants

### Additional Invariant 1

```text
The recipient encoded in the message must be the intended recipient.
```

### Additional Invariant 2

```text
The message must be sent to the trusted counterpart bridge.
```

### Additional Invariant 3

```text
The encoded calldata must match the expected finalizeBridgeERC20(...) format.
```

### Additional Invariant 4

```text
The deposit amount should be greater than zero.
```
