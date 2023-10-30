1. `<x> += <y>` costs more gas than `<x> = <x> + <y>` for state variables:
  SingleAdminAccessControl.sol => result += 1;
  SingleAdminAccessControl.sol => result += 128;
  SingleAdminAccessControl.sol => result += 64;
  SingleAdminAccessControl.sol => result += 32;
  SingleAdminAccessControl.sol => result += 16;
  SingleAdminAccessControl.sol => result += 8;
  SingleAdminAccessControl.sol => result += 4;
  SingleAdminAccessControl.sol => result += 2;
  SingleAdminAccessControl.sol => result += 1;
  SingleAdminAccessControl.sol => result += 64;
  SingleAdminAccessControl.sol => result += 32;
  SingleAdminAccessControl.sol => result += 16;
  SingleAdminAccessControl.sol => result += 8;
  SingleAdminAccessControl.sol => result += 4;
  SingleAdminAccessControl.sol => result += 2;
  SingleAdminAccessControl.sol => result += 1;
  SingleAdminAccessControl.sol => result += 16;
  SingleAdminAccessControl.sol => result += 8;
  SingleAdminAccessControl.sol => result += 4;
  SingleAdminAccessControl.sol => result += 2;
  SingleAdminAccessControl.sol => result += 1;
  
  2. Use multiple `require()` and `if` statements instead of `&&`:
  SingleAdminAccessControl.sol => if (rounding == Rounding.Up && mulmod(x, y, denominator) > 0) {
  StakedUSDeV2.sol => if (rounding == Rounding.Up && mulmod(x, y, denominator) > 0) {
  StakedUSDeV2.sol => if (address(this) == _cachedThis && block.chainid == _cachedChainId) {
  StakedUSDeV2.sol => if (success && encodedDecimals.length >= 32) {
  StakedUSDeV2.sol => if (hasRole(FULL_RESTRICTED_STAKER_ROLE, from) && 
  !hasRole(FULL_RESTRICTED_STAKER_ROLE, to)) {
  StakedUSDeV2.sol => if (_totalSupply > 0 && _totalSupply < MIN_SHARES) revert MinSharesViolation();
  StakedUSDeV2.sol => if (hasRole(FULL_RESTRICTED_STAKER_ROLE, from) && to != address(0)) {
  
  3. Block values as a proxy for time:
  StakedUSDe.sol => transferInRewards(uint256)
  StakedUSDe.sol => getUnvestedAmount()
  StakedUSDe.sol => _checkMinShares() 
  StakedUSDeV2.sol => unstake(address)
  StakedUSDeV2.sol =>  block.timestamp >= userCooldown.cooldownEnd 
  StakedUSDeV2.sol => cooldownAssets(uint256,address)
  StakedUSDeV2.sol =>  assets > maxWithdraw(owner)
  StakedUSDeV2.sol => cooldownShares(uint256,address) 
  
  4. Use custom errors instead of require:
  SingleAdminAccessControl.sol => require(denominator > prod1, "Math: mulDiv overflow");
  SingleAdminAccessControl.sol => require(value == 0, "Strings: hex length insufficient");
  SingleAdminAccessControl.sol => require(hasRole(MY_ROLE, msg.sender));
  SingleAdminAccessControl.sol => require(account == _msgSender(), "AccessControl: can only renounce 
  roles for self");
  USDeSilo.sol => require(address(this).balance >= amount, "Address: insufficient balance");
  USDeSilo.sol => require(success, "Address: unable to send value, recipient may have reverted");
  USDeSilo.sol => require(address(this).balance >= value, "Address: insufficient balance for call");
  USDeSilo.sol => require(isContract(target), "Address: call to non-contract");
  USDeSilo.sol => require(oldAllowance >= value, "SafeERC20: decreased allowance below zero");
  USDeSilo.sol => require(nonceAfter == nonceBefore + 1, "SafeERC20: permit did not succeed");
  USDeSilo.sol => require(returndata.length == 0 || abi.decode(returndata, (bool)), "SafeERC20: ERC20 
  operation did not succeed");
  StakedUSDeV2.sol => require(hasRole(MY_ROLE, msg.sender));
  StakedUSDeV2.sol => require(value > 0, "Counter: decrement overflow");
  StakedUSDeV2.sol => require(value == 0, "Strings: hex length insufficient");
  StakedUSDeV2.sol => require(address(this).balance >= amount, "Address: insufficient balance");
  StakedUSDeV2.sol => require(success, "Address: unable to send value, recipient may have reverted");
  StakedUSDeV2.sol => require(address(this).balance >= value, "Address: insufficient balance for call");
  StakedUSDeV2.sol => require(isContract(target), "Address: call to non-contract");
  StakedUSDeV2.sol => require(oldAllowance >= value, "SafeERC20: decreased allowance below zero");
  StakedUSDeV2.sol => require(nonceAfter == nonceBefore + 1, "SafeERC20: permit did not succeed");
  StakedUSDeV2.sol => require(returndata.length == 0 || abi.decode(returndata, (bool)), "SafeERC20: 
  ERC20 operation did not succeed");
  StakedUSDeV2.sol => require(denominator > prod1, "Math: mulDiv overflow");
  StakedUSDeV2.sol => require(currentAllowance >= subtractedValue, "ERC20: decreased allowance below 
  zero");
  StakedUSDeV2.sol => require(from != address(0), "ERC20: transfer from the zero address");
  StakedUSDeV2.sol => require(to != address(0), "ERC20: transfer to the zero address");
  StakedUSDeV2.sol => require(fromBalance >= amount, "ERC20: transfer amount exceeds balance");
  StakedUSDeV2.sol => require(account != address(0), "ERC20: mint to the zero address");
  StakedUSDeV2.sol => require(account != address(0), "ERC20: burn from the zero address");
  StakedUSDeV2.sol => require(accountBalance >= amount, "ERC20: burn amount exceeds balance");
  StakedUSDeV2.sol => require(owner != address(0), "ERC20: approve from the zero address");
  StakedUSDeV2.sol => require(spender != address(0), "ERC20: approve to the zero address");
  StakedUSDeV2.sol => require(currentAllowance >= amount, "ERC20: insufficient allowance");
  StakedUSDeV2.sol => require(block.timestamp <= deadline, "ERC20Permit: expired deadline");
  StakedUSDeV2.sol => require(signer == owner, "ERC20Permit: invalid signature");
  StakedUSDeV2.sol => require(account == _msgSender(), "AccessControl: can only renounce roles for 
  self");
  StakedUSDeV2.sol => require(assets <= maxDeposit(receiver), "ERC4626: deposit more than max");
  StakedUSDeV2.sol => require(shares <= maxMint(receiver), "ERC4626: mint more than max");
  StakedUSDeV2.sol => require(assets <= maxWithdraw(owner), "ERC4626: withdraw more than max");
  StakedUSDeV2.sol => require(shares <= maxRedeem(owner), "ERC4626: redeem more than max");
  StakedUSDeV2.sol => require(_status != _ENTERED, "ReentrancyGuard: reentrant call");
  
  5. Using storage instead of memory for structs/arrays saves gas:
  SingleAdminAccessControl.sol => string memory buffer = new string(length);
  SingleAdminAccessControl.sol => bytes memory buffer = new bytes(2 * length + 2);
  USDeSilo.sol => bytes memory approvalCall = abi.encodeWithSelector(token.approve.selector, spender, 
  value);
  USDeSilo.sol => bytes memory returndata = address(token).functionCall(data, "SafeERC20: low-level call 
  failed");

  6. Use calldata instead of memory for function parameters:
  SingleAdminAccessControl.sol => function equal(string memory a, string memory b) internal pure returns 
  (bool) {
  USDeSilo.sol => function functionCall(address target, bytes memory data) internal returns (bytes 
  memory) {
  USDeSilo.sol => function functionStaticCall(address target, bytes memory data) internal view returns 
  (bytes memory) {
  USDeSilo.sol => function functionDelegateCall(address target, bytes memory data) internal returns 
  (bytes memory) {
  USDeSilo.sol => function _revert(bytes memory returndata, string memory errorMessage) private pure {
  USDeSilo.sol => function _callOptionalReturn(IERC20 token, bytes memory data) private {
  USDeSilo.sol => function _callOptionalReturnBool(IERC20 token, bytes memory data) private returns 
  (bool) {

  7. Multiplication by two should use bit shifting:
  SingleAdminAccessControl.sol => // variables such that product = prod1 * 2^256 + prod0.

  8. Division by two should use bit shifting:
  SingleAdminAccessControl.sol => // (a + b) / 2 can overflow.
  SingleAdminAccessControl.sol => return (a & b) + (a ^ b) / 2;
  SingleAdminAccessControl.sol => * @dev Original credit to Remco Bloemen under MIT license (https://xn- 
  -2-umb.com/21/muldiv)
  SingleAdminAccessControl.sol => // → `2**(k/2) <= sqrt(a) < 2**((k+1)/2) <= 2**(k/2 + 1)`
  SingleAdminAccessControl.sol => // Consequently, `2**(log2(a) / 2)` is a good first approximation of 
  `sqrt(a)` with at least 1 correct bit.
  
  9. Private and internal variables must start with an underscore:
  SingleAdminAccessControl.sol mapping(bytes32 => RoleData) private _roles;
  SingleAdminAccessControl.sol => address private _currentDefaultAdmin;
  SingleAdminAccessControl.sol => address private _pendingDefaultAdmin;
  
  10. Constants in comparisons should appear on the left side:
  SingleAdminAccessControl.sol => * @notice Calculates floor(x * y / denominator) with full precision. 
  Throws if result overflows a uint256 or denominator == 0
  SingleAdminAccessControl.sol => if (prod1 == 0) {
  SingleAdminAccessControl.sol => // Solidity will revert if denominator == 0, unlike the div opcode on 
  its own.
  SingleAdminAccessControl.sol => if (rounding == Rounding.Up && mulmod(x, y, denominator) > 0) {
  SingleAdminAccessControl.sol => if (a == 0) {
  SingleAdminAccessControl.sol => if (value >> 128 > 0) {
  SingleAdminAccessControl.sol => if (value >> 64 > 0) {
  SingleAdminAccessControl.sol => if (value >> 32 > 0) {
  SingleAdminAccessControl.sol => if (value >> 16 > 0) {
  SingleAdminAccessControl.sol => if (value >> 8 > 0) {
  SingleAdminAccessControl.sol => if (value >> 4 > 0) {
  SingleAdminAccessControl.sol => if (value >> 2 > 0) {
  SingleAdminAccessControl.sol => if (value >> 1 > 0) {
  SingleAdminAccessControl.sol => if (value >= 10 ** 64) {
  SingleAdminAccessControl.sol => if (value >= 10 ** 32) {
  SingleAdminAccessControl.sol => if (value >= 10 ** 16) {
  SingleAdminAccessControl.sol => if (value >= 10 ** 8) {
  SingleAdminAccessControl.sol => if (value >= 10 ** 4) {
  SingleAdminAccessControl.sol => if (value >= 10 ** 2) {
  SingleAdminAccessControl.sol => if (value >= 10 ** 1) {
  SingleAdminAccessControl.sol => if (value >> 128 > 0) {
  SingleAdminAccessControl.sol => if (value >> 64 > 0) {
  SingleAdminAccessControl.sol => if (value >> 32 > 0) {
  SingleAdminAccessControl.sol => if (value >> 16 > 0) {
  SingleAdminAccessControl.sol => if (value >> 8 > 0) {
  SingleAdminAccessControl.sol => if (value == 0) break;
  SingleAdminAccessControl.sol => require(value == 0, "Strings: hex length insufficient");
  
  11. Unsafe casting may overflow:
  SingleAdminAccessControl.sol => return x + (int256(uint256(x) >> 255) & (a ^ b));
  SingleAdminAccessControl.sol => return uint256(n >= 0 ? n : -n);
  SingleAdminAccessControl.sol => return toHexString(uint256(uint160(addr)), _ADDRESS_LENGTH);
  EthenaMinting.sol => uint8 v = uint8((uint256(vs) >> 255) + 27);
  EthenaMinting.sol => return _add(set._inner, bytes32(uint256(uint160(value))));
  EthenaMinting.sol => return _remove(set._inner, bytes32(uint256(uint160(value))));
  EthenaMinting.sol => return _contains(set._inner, bytes32(uint256(uint160(value))));
  EthenaMinting.sol => return address(uint160(uint256(_at(set._inner, index))));
  EthenaMinting.sol => return uint256(_at(set._inner, index));
  EthenaMinting.sol => return x + (int256(uint256(x) >> 255) & (a ^ b));
  EthenaMinting.sol => return uint256(n >= 0 ? n : -n);
  EthenaMinting.sol => return toHexString(uint256(uint160(addr)), _ADDRESS_LENGTH);
  EthenaMinting.sol => uint256 invalidatorBit = 1 << uint8(nonce);
  
  12. Use scientific notation (e.g. `1e18`) rather than exponentiation (e.g. `10**18`):
  SingleAdminAccessControl.sol => // `msb(a) <= a < 2*msb(a)`. This value can be written `msb(a)=2**k` 
  with `k=log2(a)`.
  SingleAdminAccessControl.sol => // This can be rewritten `2**log2(a) <= a < 2**(log2(a) + 1)`
  SingleAdminAccessControl.sol => // → `sqrt(2**k) <= sqrt(a) < sqrt(2**(k+1))`
  SingleAdminAccessControl.sol => if (value >= 10 ** 64) {
  SingleAdminAccessControl.sol => value /= 10 ** 64;
  SingleAdminAccessControl.sol => if (value >= 10 ** 32) {
  SingleAdminAccessControl.sol => value /= 10 ** 32;
  SingleAdminAccessControl.sol => if (value >= 10 ** 16) {
  SingleAdminAccessControl.sol => value /= 10 ** 16;
  SingleAdminAccessControl.sol => if (value >= 10 ** 8) {
  SingleAdminAccessControl.sol => value /= 10 ** 8;
  SingleAdminAccessControl.sol => if (value >= 10 ** 4) {
  SingleAdminAccessControl.sol => value /= 10 ** 4;
  SingleAdminAccessControl.sol => if (value >= 10 ** 2) {
  SingleAdminAccessControl.sol => value /= 10 ** 2;
  SingleAdminAccessControl.sol => if (value >= 10 ** 1) {
  SingleAdminAccessControl.sol => return result + (rounding == Rounding.Up && 10 ** result < value ? 1 : 
  0);

  

### Time spent:
8 hours