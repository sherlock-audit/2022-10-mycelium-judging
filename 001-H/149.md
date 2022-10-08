WATCHPUG

high

# Attacker can manipulate the pricePerShare to profit from future users' deposits

## Summary

By manipulating and inflating the pricePerShare to a super high value, the attacker can cause all future depositors to lose a significant portion of their deposits to the attacker due to precision loss.

## Vulnerability Detail

A malicious early user can `deposit()` with `1 wei` of `LINK` token as the first depositor of the Vault, and get `(1 * STARTING_SHARES_PER_LINK) wei` of shares.

Then the attacker can send `STARTING_SHARES_PER_LINK - 1` of `LINK` tokens and inflate the price per share from `1 / STARTING_SHARES_PER_LINK` to 1.0000 .

Then the attacker call `withdraw()` to withdraw `STARTING_SHARES_PER_LINK - 1` shares, and send `1e22` of `LINK` token and inflate the price per share from 1.000 to 1.000e22.

As a result, the future user who deposits `9999e18` will only receive `0` (from `9999e18 * 1 / 10000e18`) of shares token.

They will immediately lose all of their deposits.

## Impact

Users may suffer a significant portion or even 100% of the funds they deposited to the Vault.

## Code Snippet

https://github.com/sherlock-audit/2022-10-mycelium/blob/main/mylink-contracts/src/Vault.sol#L131-142

```solidity
    function deposit(uint256 _amount) external {
        require(_amount > 0, "Amount must be greater than 0");
        require(_amount <= availableForDeposit(), "Amount exceeds available capacity");

        uint256 newShares = convertToShares(_amount);
        _mintShares(msg.sender, newShares);

        IERC20(LINK).transferFrom(msg.sender, address(this), _amount);
        _distributeToPlugins();

        emit Deposit(msg.sender, _amount);
    }
```

https://github.com/sherlock-audit/2022-10-mycelium/blob/main/mylink-contracts/src/Vault.sol#L614-L620

```solidity
    function convertToShares(uint256 _tokens) public view returns (uint256) {
        uint256 tokenSupply = totalSupply(); // saves one SLOAD
        if (tokenSupply == 0) {
            return _tokens * STARTING_SHARES_PER_LINK;
        }
        return _tokens.mulDivDown(totalShares, tokenSupply);
    }
```


## Tool used

Manual Review

## Recommendation

Consider requiring a minimal amount of share tokens to be minted for the first minter, and send part of the initial mints as a permanent reserve to the DAO/treasury/deployer so that the pricePerShare can be more resistant to manipulation.