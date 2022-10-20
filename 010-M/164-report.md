WATCHPUG

medium

# Lack of sanity checks for new plugin address in `addPlugin()`

## Summary

Without sanity checks for new plugin address in `addPlugin()`, wrong address or duplicated address can be added.

## Vulnerability Detail

When adding a new plugin, there are no sanity checks for the new plugin's address.

However, adding a wrong address or duplicated address can cause severe damage to the users, and it may be irreversible:

When the new plugin is not a correct contract, `removePlugin()` will revert at `IPlugin(_plugin).withdraw(_amount);`.

## Impact

When a wrong address is added as a plugin, many essential features of the Vault contract will malfunction, including `deposit()` and `withdraw()`, as `totalSupply()` will revert at `IPlugin(plugins[i]).balance()`.

When an existing plugin is wrongfully added as a new plugin, the `totalSupply()` will double count the balance of that plugin, which makes the user who deposits receives fewer shares and the users who withdraw, receives more tokens.

## Code Snippet

https://github.com/sherlock-audit/2022-10-mycelium/blob/main/mylink-contracts/src/Vault.sol#L314-L329

## Tool used

Manual Review

## Recommendation

1. Add a check to ensure `_plugin.Vault == address(this)`, which will first ensure the plugin address is a valid contract with Vault interface, and it's set to the correct vault address.
2. Add a check to ensure `_plugin` is new (not an existing one).