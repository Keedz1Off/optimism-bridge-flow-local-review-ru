# Function Review: sendMessage(...)

## Код функции

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

## Что делает

`sendMessage(...)` создает cross-chain message.

Bridge передает calldata, а messenger оборачивает его в `relayMessage(...)` для другой сети.

## Main Invariants

```text
1. The message must target the intended destination contract.
2. The message nonce must be unique.
3. The encoded relayMessage calldata must preserve sender, target, value, gas limit, and message.
```
