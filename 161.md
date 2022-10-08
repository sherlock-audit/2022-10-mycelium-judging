WATCHPUG

high

# Frontrun `deposit()` can cause the depositor to lose all the funds

## Summary

The attacker can frontrun the first depositor's `deposit()` transaction and transfer LINK tokens to the Vault contract directly and cause the depositor (and all the future depositors) to lose all the funds.

## Vulnerability Detail

In `onTokenTransfer()`, if there are some existing balance in the Vault contract, say 1 wei of LINK token, `supplyBeforeTransfer` will be `1`, since `totalShares == 0`, `newShares` will always be `0`.

The same applies for `deposit()`.

## Impact

The depositor who got frontrun by the attacker will lose all their funds. And all the future depositors.

## Code Snippet

https://github.com/sherlock-audit/2022-10-mycelium/blob/main/mylink-contracts/src/Vault.sol#L252-L276

https://github.com/sherlock-audit/2022-10-mycelium/blob/main/mylink-contracts/src/Vault.sol#L131-L142

https://github.com/sherlock-audit/2022-10-mycelium/blob/main/mylink-contracts/src/Vault.sol#L614-L620


## Tool used

Manual Review

## Recommendation

It should check if `totalShares == 0` to decide whether this is the first mint.