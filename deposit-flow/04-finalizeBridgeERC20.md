# Function Review: finalizeBridgeERC20(...)

## Код функции

```solidity
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

    _emitERC20BridgeFinalized(_localToken, _remoteToken, _from, _to, _amount, _extraData);
}
```

## Что делает

Финализирует ERC20 bridge transfer на destination chain.

Для L1 -> L2 deposit обычно mint L2 token.

## Main Invariants

```text
1. Only the messenger can call finalizeBridgeERC20(...).
2. The original cross-chain sender must be the trusted counterpart bridge.
3. Minted amount must be backed by locked tokens.
```
