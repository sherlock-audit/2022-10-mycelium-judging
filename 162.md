WATCHPUG

medium

# `AaveV2Plugin#availableForWithdrawal()` should consider the available liquidity in the aToken

## Summary

aToken's available liquidity should be considered in `AaveV2Plugin#availableForWithdrawal()`.

## Vulnerability Detail

As a lending protocol, Aave v2 can not always ensure all the aToken balance is redeemable 100% at any time, as part or even the majority of the tokens can be lent out.

Therefore, `AaveV2Plugin` should not always return all the aToken balance as the `availableForWithdrawal()` amount.

## Impact

Users cannot withdraw `availableForWithdrawal()` .

## Code Snippet

https://github.com/sherlock-audit/2022-10-mycelium/blob/main/mylink-contracts/src/plugins/AaveV2Plugin.sol#L76-L84

## Tool used

Manual Review

## Recommendation

Change to:

```solidity
function availableForWithdrawal() public view override returns (uint256) {
    uint256 availableLiquidity = LINK.balanceOf(address(aLINK));
    uint256 aTokenBalance = balance();
    return availableLiquidity < aTokenBalance ? availableLiquidity: aTokenBalance;
}
```