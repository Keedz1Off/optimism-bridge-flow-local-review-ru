# Function Review: sendMessage(...)

## 1. Function Code

```solidity
function sendMessage(address _target, bytes calldata _message, uint32 _minGasLimit) external payable {
    _sendMessage({
        _to: address(otherMessenger),
        _gasLimit: baseGas(_message, _minGasLimit),
        _value: msg.value,
        _data: abi.encodeWithSelector(
            this.relayMessage.selector, messageNonce(), msg.sender, _target, msg.value, _minGasLimit, _message
        )
    });

    emit SentMessage(_target, msg.sender, _message, messageNonce(), _minGasLimit);
    emit SentMessageExtension1(msg.sender, msg.value);

    unchecked {
        ++msgNonce;
    }
}
```

Note:

```text
Original source: ethereum-optimism/optimism/packages/contracts-bedrock/src/universal/CrossDomainMessenger.sol
```

## 2. Что делает эта функция

`sendMessage(...)` creates the cross-chain message.

В bridge flow `StandardBridge._initiateBridgeERC20(...)` вызывает:

```text
messenger.sendMessage(...)
```

Потом messenger wraps target calldata into a message, который позже вызовет `relayMessage(...)` на другой chain.

Simple flow:

```text
Bridge calldata
-> sendMessage(...)
-> encoded relayMessage(...)
-> message goes to the other messenger
```

## 3. Important Parts Explained

### Target and Message

```solidity
function sendMessage(address _target, bytes calldata _message, uint32 _minGasLimit) external payable
```

`_target` - contract, который должен быть вызван на other chain.

Для ERC20 bridge deposits этот target обычно:

```text
otherBridge
```

`_message` - calldata, которая должна executed on the target.

Для ERC20 bridge deposits эта calldata обычно содержит:

```text
finalizeBridgeERC20(...)
```

### Wrap Into relayMessage(...)

```solidity
_data: abi.encodeWithSelector(
    this.relayMessage.selector,
    messageNonce(),
    msg.sender,
    _target,
    msg.value,
    _minGasLimit,
    _message
)
```

Это builds calldata for the remote messenger.

Meaning:

```text
The remote messenger will receive relayMessage(...)
relayMessage(...) will call the target contract
```

### Gas Calculation

```solidity
_gasLimit: baseGas(_message, _minGasLimit)
```

Это добавляет messenger overhead к user's minimum gas limit.

Simple meaning:

```text
The message needs enough gas to be relayed and executed.
```

### Nonce Update

```solidity
unchecked {
    ++msgNonce;
}
```

Это increments the message nonce after sending.

Simple meaning:

```text
Each message should have a unique nonce.
```
