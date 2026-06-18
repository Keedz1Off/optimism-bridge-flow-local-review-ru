# Function Review: relayMessage(...)

## 1. Function Code

```solidity
/// @notice Relays a message that was sent by the other CrossDomainMessenger contract. Can only
///         be executed via cross-chain call from the other messenger OR if the message was
///         already received once and is currently being replayed.
/// @param _nonce       Nonce of the message being relayed.
/// @param _sender      Address of the user who sent the message.
/// @param _target      Address that the message is targeted at.
/// @param _value       ETH value to send with the message.
/// @param _minGasLimit Minimum amount of gas that the message can be executed with.
/// @param _message     Message to send to the target.
function relayMessage(
    uint256 _nonce,
    address _sender,
    address _target,
    uint256 _value,
    uint256 _minGasLimit,
    bytes calldata _message
)
    external
    payable
{
    // On L1 this function will check the Portal for its paused status.
    // On L2 this function should be a no-op, because paused will always return false.
    require(paused() == false, "CrossDomainMessenger: paused");

    (, uint16 version) = Encoding.decodeVersionedNonce(_nonce);
    require(version < 2, "CrossDomainMessenger: only version 0 or 1 messages are supported at this time");

    // If the message is version 0, then it's a migrated legacy withdrawal. We therefore need
    // to check that the legacy version of the message has not already been relayed.
    if (version == 0) {
        bytes32 oldHash = Hashing.hashCrossDomainMessageV0(_target, _sender, _message, _nonce);
        require(successfulMessages[oldHash] == false, "CrossDomainMessenger: legacy withdrawal already relayed");
    }

    // We use the v1 message hash as the unique identifier for the message because it commits
    // to the value and minimum gas limit of the message.
    bytes32 versionedHash =
        Hashing.hashCrossDomainMessageV1(_nonce, _sender, _target, _value, _minGasLimit, _message);

    if (_isOtherMessenger()) {
        // These properties should always hold when the message is first submitted (as
        // opposed to being replayed).
        assert(msg.value == _value);
        assert(!failedMessages[versionedHash]);
    } else {
        require(msg.value == 0, "CrossDomainMessenger: value must be zero unless message is from a system address");

        require(failedMessages[versionedHash], "CrossDomainMessenger: message cannot be replayed");
    }

    require(
        _isUnsafeTarget(_target) == false, "CrossDomainMessenger: cannot send message to blocked system address"
    );

    require(successfulMessages[versionedHash] == false, "CrossDomainMessenger: message has already been relayed");

    // If there is not enough gas left to perform the external call and finish the execution,
    // return early and assign the message to the failedMessages mapping.
    // We are asserting that we have enough gas to:
    // 1. Call the target contract (_minGasLimit + RELAY_CALL_OVERHEAD + RELAY_GAS_CHECK_BUFFER)
    //   1.a. The RELAY_CALL_OVERHEAD is included in `hasMinGas`.
    // 2. Finish the execution after the external call (RELAY_RESERVED_GAS).
    //
    // If `xDomainMsgSender` is not the default L2 sender, this function
    // is being re-entered. This marks the message as failed to allow it to be replayed.
    if (
        !SafeCall.hasMinGas(_minGasLimit, RELAY_RESERVED_GAS + RELAY_GAS_CHECK_BUFFER)
            || xDomainMsgSender != Constants.DEFAULT_L2_SENDER
    ) {
        failedMessages[versionedHash] = true;
        emit FailedRelayedMessage(versionedHash);

        // Revert in this case if the transaction was triggered by the estimation address. This
        // should only be possible during gas estimation or we have bigger problems. Reverting
        // here will make the behavior of gas estimation change such that the gas limit
        // computed will be the amount required to relay the message, even if that amount is
        // greater than the minimum gas limit specified by the user.
        if (tx.origin == Constants.ESTIMATION_ADDRESS) {
            revert("CrossDomainMessenger: failed to relay message");
        }

        return;
    }

    xDomainMsgSender = _sender;
    bool success = SafeCall.call(_target, gasleft() - RELAY_RESERVED_GAS, _value, _message);
    xDomainMsgSender = Constants.DEFAULT_L2_SENDER;

    if (success) {
        // This check is identical to one above, but it ensures that the same message cannot be relayed
        // twice, and adds a layer of protection against rentrancy.
        assert(successfulMessages[versionedHash] == false);
        successfulMessages[versionedHash] = true;
        emit RelayedMessage(versionedHash);
    } else {
        failedMessages[versionedHash] = true;
        emit FailedRelayedMessage(versionedHash);

        // Revert in this case if the transaction was triggered by the estimation address. This
        // should only be possible during gas estimation or we have bigger problems. Reverting
        // here will make the behavior of gas estimation change such that the gas limit
        // computed will be the amount required to relay the message, even if that amount is
        // greater than the minimum gas limit specified by the user.
        if (tx.origin == Constants.ESTIMATION_ADDRESS) {
            revert("CrossDomainMessenger: failed to relay message");
        }
    }
}
```

Note:

```text
Original source: `ethereum-optimism/optimism/packages/contracts-bedrock/src/universal/CrossDomainMessenger.sol`.
```

## 2. Что делает эта функция

`relayMessage(...)` validates and executes a cross-chain message.

Простыми словами:

```text
It checks that the message came from the trusted messenger,
checks that the message was not already executed,
then calls the target contract.
```

## 3. Important Parts Explained

### Trusted Messenger Check

```solidity
if (_isOtherMessenger()) {
    assert(msg.value == _value);
    assert(!failedMessages[versionedHash]);
} else {
    require(msg.value == 0, "CrossDomainMessenger: value must be zero unless message is from a system address");
    require(failedMessages[versionedHash], "CrossDomainMessenger: message cannot be replayed");
}
```

Первый relay должен прийти через other messenger. Если он не от other messenger, это должен быть replay previously failed message.

Security meaning:

```text
This is the auth boundary.
```

Если anyone can call this function directly, они могут execute fake bridge messages.

### Message Hash

```solidity
bytes32 versionedHash =
    Hashing.hashCrossDomainMessageV1(_nonce, _sender, _target, _value, _minGasLimit, _message);
```

Это создает unique message fingerprint для replay protection.

Он включает:

- nonce
- original sender
- destination target
- ETH value
- minimum gas limit
- message calldata

Security meaning:

```text
The hash identifies this exact message.
```

### Replay Protection

```solidity
require(successfulMessages[versionedHash] == false, "CrossDomainMessenger: message has already been relayed");
```

Это проверяет, что same message was not already executed.

### Mark Executed Before Call

```solidity
successfulMessages[versionedHash] = true;
emit RelayedMessage(versionedHash);
```

Message marked as successful after the target call succeeds.

Security meaning:

```text
The same message should not be executed twice.
```

### Execute Target

```solidity
bool success = SafeCall.call(_target, gasleft() - RELAY_RESERVED_GAS, _value, _message);
```

Это calls destination contract with the provided calldata.

Если execution fails, message stored in `failedMessages` и может быть replayed later.

## 4. Invariants

### Main Invariant 1

```text
Only the trusted messenger can relay messages.
```

### Main Invariant 2

```text
Each message must execute only once.
```

### Main Invariant 3

```text
Validation must happen before execution.
```

## 5. Additional Invariants

### Additional Invariant 1

```text
The message hash must uniquely identify the sender, target, and calldata.
```

### Additional Invariant 2

```text
The message must be marked as successfully relayed only after the target call succeeds.
```

### Additional Invariant 3

```text
The target must be the intended destination contract.
```

### Additional Invariant 4

```text
A failed target call must be tracked in `failedMessages` so it can be replayed.
```
