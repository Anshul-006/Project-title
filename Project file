// SPDX-License-Identifier: MIT
pragma solidity ^0.8.19;

import "@openzeppelin/contracts/token/ERC20/IERC20.sol";
import "@openzeppelin/contracts/access/Ownable.sol";

contract CrossChainBridge is Ownable {
    // Whitelisted tokens that can be bridged
    mapping(address => bool) public whitelistedTokens;
    // Track processed nonces for replay protection
    mapping(address => mapping(uint256 => bool)) public processedNonces;

    event TokenLocked(address indexed token, address indexed user, uint256 amount, uint256 nonce, string targetChain);
    event TokenReleased(address indexed token, address indexed user, uint256 amount, uint256 nonce);

    // Constructor: Pass initial owner to Ownable
    constructor(address initialOwner) Ownable(initialOwner) {}

    // Whitelist a token to be eligible for bridging
    function whitelistToken(address token, bool status) external onlyOwner {
        whitelistedTokens[token] = status;
    }

    // Lock tokens to initiate cross-chain transfer
    function lockToken(address token, uint256 amount, uint256 nonce, string calldata targetChain) external {
        require(whitelistedTokens[token], "Token not whitelisted");
        require(IERC20(token).allowance(msg.sender, address(this)) >= amount, "Insufficient allowance");
        require(!processedNonces[msg.sender][nonce], "Nonce already processed");

        IERC20(token).transferFrom(msg.sender, address(this), amount);
        processedNonces[msg.sender][nonce] = true;

        emit TokenLocked(token, msg.sender, amount, nonce, targetChain);
    }

    // Release tokens after transfer verification (called by bridge operator)
    function releaseToken(address token, address user, uint256 amount, uint256 nonce) external onlyOwner {
        require(whitelistedTokens[token], "Token not whitelisted");
        require(!processedNonces[user][nonce], "Nonce already processed");

        processedNonces[user][nonce] = true;
        IERC20(token).transfer(user, amount);

        emit TokenReleased(token, user, amount, nonce);
    }
}
