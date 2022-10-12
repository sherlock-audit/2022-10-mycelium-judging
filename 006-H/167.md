WATCHPUG

medium

# When one of the plugins is broken or paused, `deposit()` or `withdraw()` of the whole Vault contract can malfunction

## Summary

One malfunctioning plugin can result in the whole Vault contract malfunctioning.

## Vulnerability Detail

A given plugin can temporally or even permanently becomes malfunctioning (cannot deposit/withdraw) for all sorts of reasons.

Eg, Aave V2 Lending Pool can be paused, which will prevent multiple core functions that the Aave v2 plugin depends on from working, including `lendingPool.deposit()` and `lendingPool.withdraw()`.

https://github.com/aave/protocol-v2/blob/master/contracts/protocol/lendingpool/LendingPool.sol#L54

```soldity
  modifier whenNotPaused() {
    _whenNotPaused();
    _;
  }
```

https://github.com/aave/protocol-v2/blob/master/contracts/protocol/lendingpool/LendingPool.sol#L142-L146

```solidity
  function withdraw(
    address asset,
    uint256 amount,
    address to
  ) external override whenNotPaused returns (uint256) {
```

That's because the deposit will always goes to the first plugin, and withdraw from the last plugin first.

## Impact

When Aave V2 Lending Pool is paused, users won't be able to deposit or withdraw from the vault.

Neither can the owner remove the plugin nor rebalanced it to other plugins to resume operation.

Because withdrawal from the plugin can not be done, and removing a plugin or rebalancing both rely on this.

## Code Snippet

https://github.com/sherlock-audit/2022-10-mycelium/blob/main/mylink-contracts/src/Vault.sol#L456-L473

https://github.com/sherlock-audit/2022-10-mycelium/blob/main/mylink-contracts/src/Vault.sol#L492-L519

## Tool used

Manual Review

## Recommendation

1. Consider introducing a new method to pause one plugin from the Vault contract level;

2. Aave V2's Lending Pool contract has a view function [`paused()`](https://github.com/aave/protocol-v2/blob/master/contracts/protocol/lendingpool/LendingPool.sol#L685), consider returning `0` for `availableForDeposit()` and ``availableForWithdrawal() when pool paused in AaveV2Plugin:

```solidity
function availableForDeposit() public view override returns (uint256) {
    if (lendingPool.paused()) return 0;
    return type(uint256).max - balance();
}
```

```solidity
function availableForWithdrawal() public view override returns (uint256) {
    if (lendingPool.paused()) return 0;
    return balance();
}
```