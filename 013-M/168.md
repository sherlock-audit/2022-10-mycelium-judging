WATCHPUG

medium

# `_withdrawFromPlugin()` will revert when `_withdrawalValues[i] == 0`

## Summary

## Vulnerability Detail

When `_withdrawalValues[i] == 0` in `rebalancePlugins()`, it means NOT to rebalance this plugin.

However, the current implementation still tries to withdraw 0 from the plugin.

This will revert in AaveV2Plugin as Aave V2's `validateWithdraw()` does not allow `0` withdrawals:

https://github.com/aave/protocol-v2/blob/554a2ed7ca4b3565e2ceaea0c454e5a70b3a2b41/contracts/protocol/libraries/logic/ValidationLogic.sol#L60-L70

```solidity
  function validateWithdraw(
    address reserveAddress,
    uint256 amount,
    uint256 userBalance,
    mapping(address => DataTypes.ReserveData) storage reservesData,
    DataTypes.UserConfigurationMap storage userConfig,
    mapping(uint256 => address) storage reserves,
    uint256 reservesCount,
    address oracle
  ) external view {
    require(amount != 0, Errors.VL_INVALID_AMOUNT);
```

`removePlugin()` will also always `_withdrawFromPlugin()` even if the plugin's balance is 0, as it will also tries to withdraw 0 in that case (balance is 0).

## Impact

For AaveV2Plugin (and any future plugins that dont allow withdraw 0):

1. In every rebalance call, it must at least withdraw 1 wei from the plugin for the rebalance to work.
2. The plugin can not be removed or rebalanced when there is no balance in it. 

If such a plugin can not deposit for some reason (paused by gov, AaveV2Plugin may face that), this will further cause the whole system unable to be rebalanced until the deposit resumes for that plugin.

## Code Snippet

https://github.com/sherlock-audit/2022-10-mycelium/blob/main/mylink-contracts/src/Vault.sol#L367-L373

## Tool used

Manual Review

## Recommendation

Only call `_withdrawFromPlugin()` when `IPlugin(pluginAddr).balance() > 0`:

```solidity
function removePlugin(uint256 _index) external onlyOwner {
    require(_index < pluginCount, "Index out of bounds");
    address pluginAddr = plugins[_index];
    if (IPlugin(pluginAddr).balance() > 0){
        _withdrawFromPlugin(pluginAddr, IPlugin(pluginAddr).balance());
    }
    uint256 pointer = _index;
    while (pointer < pluginCount - 1) {
        plugins[pointer] = plugins[pointer + 1];
        pointer++;
    }
    delete plugins[pluginCount - 1];
    pluginCount--;

    IERC20(LINK).approve(pluginAddr, 0);

    emit PluginRemoved(pluginAddr);
}
```

```solidity
function rebalancePlugins(uint256[] memory _withdrawalValues) external onlyOwner {
    require(_withdrawalValues.length == pluginCount, "Invalid withdrawal values");
    for (uint256 i = 0; i < pluginCount; i++) {
        if (_withdrawalValues[i] > 0)
            _withdrawFromPlugin(plugins[i], _withdrawalValues[i]);
    }
    _distributeToPlugins();
}
```